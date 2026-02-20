# Table State Management

## State Model

TanStack Table manages state internally but allows you to control any/all of it externally. Every feature's state follows the same pattern: `state.*` for controlled, `initialState.*` for defaults, `on*Change` for updates.

### Uncontrolled (Default)

Table manages its own state. Use `initialState` to set defaults:

```tsx
const table = useReactTable({
  data,
  columns,
  getCoreRowModel: getCoreRowModel(),
  initialState: {
    sorting: [{ id: 'name', desc: false }],
    pagination: { pageIndex: 0, pageSize: 20 },
    columnVisibility: { internalId: false },
  },
})

// Read state anytime:
table.getState().sorting
table.getState().pagination
```

### Controlled

Manage state in your own React state for external access:

```tsx
const [sorting, setSorting] = useState<SortingState>([])
const [columnFilters, setColumnFilters] = useState<ColumnFiltersState>([])
const [pagination, setPagination] = useState<PaginationState>({ pageIndex: 0, pageSize: 10 })
const [rowSelection, setRowSelection] = useState<RowSelectionState>({})

const table = useReactTable({
  data,
  columns,
  state: {
    sorting,
    columnFilters,
    pagination,
    rowSelection,
  },
  onSortingChange: setSorting,
  onColumnFiltersChange: setColumnFilters,
  onPaginationChange: setPagination,
  onRowSelectionChange: setRowSelection,
  getCoreRowModel: getCoreRowModel(),
  getSortedRowModel: getSortedRowModel(),
  getFilteredRowModel: getFilteredRowModel(),
  getPaginationRowModel: getPaginationRowModel(),
})
```

### Partial Control

You can control some state and leave the rest to the table:

```tsx
const [sorting, setSorting] = useState<SortingState>([])

const table = useReactTable({
  data,
  columns,
  state: { sorting },              // only control sorting
  onSortingChange: setSorting,
  initialState: {                   // set defaults for uncontrolled state
    pagination: { pageIndex: 0, pageSize: 50 },
  },
  getCoreRowModel: getCoreRowModel(),
  getSortedRowModel: getSortedRowModel(),
})
```

### Fully Controlled (Hoisted State)

For complete external state management:

```tsx
const [tableState, setTableState] = useState(() => ({
  ...table.initialState, // start with all defaults
}))

const table = useReactTable({
  data,
  columns,
  state: tableState,
  onStateChange: setTableState,
  getCoreRowModel: getCoreRowModel(),
})
```

## All State Slices

| State | Type | onChange | Notes |
|-------|------|---------|-------|
| `sorting` | `SortingState` (`ColumnSort[]`) | `onSortingChange` | |
| `columnFilters` | `ColumnFiltersState` (`ColumnFilter[]`) | `onColumnFiltersChange` | |
| `globalFilter` | `any` (usually `string`) | `onGlobalFilterChange` | |
| `pagination` | `PaginationState` | `onPaginationChange` | `{ pageIndex, pageSize }` |
| `rowSelection` | `RowSelectionState` (`Record<string, boolean>`) | `onRowSelectionChange` | |
| `expanded` | `ExpandedState` (`Record<string, boolean> \| true`) | `onExpandedChange` | `true` = all expanded |
| `grouping` | `GroupingState` (`string[]`) | `onGroupingChange` | Column IDs |
| `columnVisibility` | `VisibilityState` (`Record<string, boolean>`) | `onColumnVisibilityChange` | |
| `columnOrder` | `ColumnOrderState` (`string[]`) | `onColumnOrderChange` | |
| `columnPinning` | `ColumnPinningState` | `onColumnPinningChange` | `{ left: [], right: [] }` |
| `rowPinning` | `RowPinningState` | `onRowPinningChange` | `{ top: [], bottom: [] }` |
| `columnSizing` | `ColumnSizingState` (`Record<string, number>`) | `onColumnSizingChange` | |
| `columnSizingInfo` | `ColumnSizingInfoState` | `onColumnSizingInfoChange` | Internal resize drag state |

## Updater Functions

All `on*Change` callbacks receive an `Updater<T>` — either a new value or a function:

```tsx
type Updater<T> = T | ((old: T) => T)

// The onChange handler receives an updater, not a raw value:
onSortingChange: (updater) => {
  // updater could be a function or a direct value
  const newValue = typeof updater === 'function'
    ? updater(sorting)
    : updater
  setSorting(newValue)
}

// React's setState already handles this pattern, so this works:
onSortingChange: setSorting // ← simplest form
```

## Full State Change Listener

```tsx
const table = useReactTable({
  // ...
  onStateChange: (updater) => {
    // called for ANY state change
    // useful for persisting full table state
    const newState = typeof updater === 'function'
      ? updater(tableState)
      : updater
    setTableState(newState)
    saveToLocalStorage(newState) // persist
  },
})
```

## Resetting State

```tsx
table.reset()                  // reset ALL state to initialState
table.resetSorting()           // reset individual slices
table.resetColumnFilters()
table.resetGlobalFilter()
table.resetPagination()
table.resetRowSelection()
table.resetExpanded()
table.resetGrouping()
table.resetColumnVisibility()
table.resetColumnOrder()
table.resetColumnPinning()
table.resetColumnSizing()
table.resetRowPinning()

// Pass `true` to reset to default (empty) state instead of initialState:
table.resetSorting(true)       // resets to [] instead of initialState.sorting
```

## Auto-Reset Options

When data or other state changes, some features auto-reset:

```tsx
const table = useReactTable({
  autoResetAll: false,           // disable ALL auto-resets
  autoResetPageIndex: true,      // reset page when data/filters/sort change (default: true)
  autoResetExpanded: false,      // don't reset expanded when data changes
  // Each feature has its own autoReset option
})
```

| Option | Default | Triggers |
|--------|---------|----------|
| `autoResetPageIndex` | `true` (`false` if `manualPagination`) | Data, filters, sorting, grouping change |
| `autoResetExpanded` | `false` | Data changes |
| `autoResetAll` | — | Overrides all individual auto-reset settings |

## Persisting State (URL, localStorage, etc.)

### URL Search Params

```tsx
const [searchParams, setSearchParams] = useSearchParams()

const sorting = JSON.parse(searchParams.get('sort') ?? '[]')
const setSorting = (updater: Updater<SortingState>) => {
  const newSorting = typeof updater === 'function' ? updater(sorting) : updater
  setSearchParams(prev => {
    prev.set('sort', JSON.stringify(newSorting))
    return prev
  })
}

const table = useReactTable({
  state: { sorting },
  onSortingChange: setSorting,
  // ...
})
```

### localStorage

```tsx
const STORAGE_KEY = 'my-table-state'

function usePersistedTableState() {
  const [state, setState] = useState(() => {
    const saved = localStorage.getItem(STORAGE_KEY)
    return saved ? JSON.parse(saved) : {}
  })

  const onStateChange = useCallback((updater: Updater<TableState>) => {
    setState((old: TableState) => {
      const newState = typeof updater === 'function' ? updater(old) : updater
      localStorage.setItem(STORAGE_KEY, JSON.stringify(newState))
      return newState
    })
  }, [])

  return { state, onStateChange }
}

// Usage:
const { state, onStateChange } = usePersistedTableState()
const table = useReactTable({
  state,
  onStateChange,
  // ...
})
```

### TanStack Router Search Params

```tsx
import { useNavigate, useSearch } from '@tanstack/react-router'

// In route definition:
export const Route = createRoute({
  validateSearch: (search) => ({
    sort: (search.sort as SortingState) ?? [],
    page: (search.page as number) ?? 0,
  }),
})

// In component:
const { sort, page } = useSearch({ from: Route.id })
const navigate = useNavigate()

const table = useReactTable({
  state: {
    sorting: sort,
    pagination: { pageIndex: page, pageSize: 20 },
  },
  onSortingChange: updater => {
    const newSort = typeof updater === 'function' ? updater(sort) : updater
    navigate({ search: prev => ({ ...prev, sort: newSort }) })
  },
  // ...
})
```

## Important: `state` vs `initialState`

- **Never use both** for the same slice — `state` overrides `initialState`
- `initialState` is read once at creation; changing it later has no effect
- `state` must always be the latest value (it's reactive)
- If you provide `state.sorting` you MUST also provide `onSortingChange` or sorting won't work

## Edge Cases & Gotchas

- `onStateChange` fires for **every** state change — including high-frequency ones like column resizing drag. Be careful with expensive operations in this handler
- When using `state` without the corresponding `on*Change`, the table will try to update state internally but your stale `state` value will override it — the UI appears frozen
- `initialState` and `state` can coexist for different slices: `initialState.pagination` + `state.sorting` is fine
- Auto-reset options default to `true` for pagination (resets to page 0) but `false` for most other features
- With `manualPagination: true`, `autoResetPageIndex` defaults to `false`
- State changes are batched within a single render cycle — multiple `set*` calls in the same handler result in one re-render
