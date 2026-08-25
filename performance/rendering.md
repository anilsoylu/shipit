# Performance: Rendering & render count

Part of `../performance.md` (loop, budgets, severity legend, and triage rules
live there). Covers React on web (Next.js), React Native (Expo), and SwiftUI.

The theme: work repeated per frame or per keystroke. A render is cheap; a
thousand renders is jank. Budget: <= 2 renders per component per user
interaction, and no frame over 16.7ms.

## Measure first

```bash
# React, web — render counts and why, live on the page
npx react-scan@latest http://localhost:3000

# React Native — Perf Monitor (JS FPS and UI FPS), then the Hermes sampling
# profiler through React DevTools. Release build only.
```

SwiftUI: put `let _ = Self._printChanges()` at the top of a suspect `body`; it
logs which property triggered each re-evaluation. For frame-level data use
Instruments with the **SwiftUI** and **Hangs** templates.

Dev-mode warning: React StrictMode double-invokes renders in development. Halve
what you see, or measure a production build. See the triage rules in the router.

## React (web and native)

- **[BLOCKER] Virtualize any list that grows with the data** — mapping over an
  array renders every row, mounts every subtree, and holds every DOM node or
  native view. Fine at 20 rows, fatal at 2000.
  ```tsx
  // wrong
  {items.map((i) => <Row key={i.id} item={i} />)}      // 5000 mounted rows
  // right — web
  const virtualizer = useVirtualizer({ count: items.length, getScrollElement, estimateSize })
  // right — native
  <FlashList data={items} renderItem={renderRow} keyExtractor={(i) => i.id} />
  ```
  Verify: scroll a 5000-item list — mounted row count stays roughly constant (react-scan / element inspector), JS FPS stays >= 55.
- **[BLOCKER] Never build the context value inline** — a fresh object identity on
  every provider render re-renders every consumer in the subtree, no matter how
  deep or how unrelated.
  ```tsx
  // wrong
  <UserCtx.Provider value={{ user, setUser }}>        // new object each render
  // right
  const value = useMemo(() => ({ user, setUser }), [user])
  <UserCtx.Provider value={value}>
  ```
  Verify: react-scan — changing an unrelated piece of parent state no longer highlights the consumers. Better still, split one context per concern so a consumer only subscribes to what it reads.
- **[BLOCKER] Keep the list item's props stable** — an inline `renderItem` arrow,
  an inline style object, or an index-based key defeats every memoization below
  it and forces a full re-render on each parent update.
  ```tsx
  // wrong
  <FlatList renderItem={({ item }) => <Row item={item} onPress={() => open(item)} />} />
  // right
  const renderRow = useCallback(({ item }) => <Row item={item} onPress={open} />, [open])
  const Row = memo(function Row({ item, onPress }) { /* ... */ })
  ```
  Verify: react-scan renders-per-interaction for `Row` drops to 1 when one unrelated row's data changes.
- **[BLOCKER] Animate off the JS thread** — driving an animation with `setState`
  per frame re-renders the tree 60 times a second and stalls the moment the JS
  thread does anything else. Reanimated worklets run on the UI thread.
  ```tsx
  // wrong
  useEffect(() => { const id = setInterval(() => setX((x) => x + 1), 16); ... })
  // right
  const style = useAnimatedStyle(() => ({ transform: [{ translateX: x.value }] }))
  ```
  Verify: Perf Monitor during the animation — UI FPS stays 60 while JS FPS is unaffected; the animation keeps running when you block the JS thread.
- **[HARDEN] Push state down, not up** — state held in a root layout re-renders
  the whole app on every change. Move it to the smallest component that needs
  it, or into a store with selector subscriptions (Zustand, Jotai) so only the
  readers re-render.
  Verify: react-scan — typing in one input does not highlight the rest of the page.
- **[HARDEN] Memoize because a profile said so** — `useMemo`, `useCallback`, and
  `memo` each cost a comparison and a retained reference. On React 19 with the
  React Compiler enabled, most of these are inserted for you and hand-written
  ones become noise. Add them to fix a measured re-render, remove them when the
  measurement says they do nothing.
  Verify: remove the memo, re-run react-scan. No change in render count means it was dead weight.

## SwiftUI

- **[BLOCKER] Observe per property, not per object** — with
  `ObservableObject`/`@Published`, every view observing the object re-evaluates
  when *any* property changes. The `@Observable` macro tracks only the
  properties a given `body` actually reads.
  ```swift
  // wrong
  final class Store: ObservableObject { @Published var items: [Item] = []; @Published var query = "" }
  // right
  @Observable final class Store { var items: [Item] = []; var query = "" }
  ```
  Verify: `Self._printChanges()` on a view that reads only `items` — typing in the search field no longer logs a change for it.
- **[BLOCKER] Lazy containers for anything data-sized** — `VStack`/`ForEach`
  inside a `ScrollView` builds every row up front. `LazyVStack` and `List` build
  only what is on screen.
  ```swift
  // wrong
  ScrollView { VStack { ForEach(items) { RowView(item: $0) } } }
  // right
  ScrollView { LazyVStack { ForEach(items) { RowView(item: $0) } } }
  ```
  Verify: Instruments SwiftUI template — view body count stays flat as the data grows; scrolling 5000 rows holds 60fps.
- **[HARDEN] Nothing expensive inside `body`** — `body` is re-evaluated far more
  often than you expect. Date formatting, sorting, filtering, and image decoding
  belong in the model or a cached property, not in the view builder.
  ```swift
  // wrong
  Text(items.sorted { $0.date > $1.date }.first?.title ?? "")
  // right — sorted once in the @Observable model
  Text(store.latestTitle ?? "")
  ```
  Verify: Instruments Time Profiler — the sort no longer appears under the view's body; the Hangs instrument reports no main-thread stall over 250ms.
- **[HARDEN] Give `ForEach` a stable identity** — identity by index or by a value
  that changes re-creates rows instead of updating them, losing state and
  animation continuity.
  Verify: rows keep their scroll position and local state across a data refresh.

## Do not

- Report a render count from a development build. StrictMode and the dev
  runtime inflate it.
- Wrap everything in `memo` after seeing one slow component. Fix the identity
  churn that caused the re-render; memo hides the cause.
- Assume a re-render is the problem. A component rendering 10 times cheaply is
  fine; one rendering twice with a sort in it is not. Profile time, not count,
  once count is bounded.
