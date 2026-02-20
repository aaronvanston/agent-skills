# Sorting

## Setup

```tsx
import {
  getCoreRowModel,
  getSortedRowModel,
  useReactTable,
  SortingState,
} from '@tanstack/react-table'

const [sorting, setSorting] = useState<SortingState>([])

const table = useReactTable({
  data,
  columns,
  state: { sorting },
  onSortingChange: setSorting,
  getCoreRowModel: getCoreRowModel(),
  getSortedRowModel: getSortedRowModel(), // required for client-side sorting
})
```

## Sorting State

```tsx
type SortDirection = 'asc' | 'desc'

type ColumnSort = { id: string; desc: boolean }
type SortingState = ColumnSort[]

type SortingTableState = {
  sorting: SortingState
}

// Multi-sort example:
const sorting: SortingState = [
  { id: 'status', desc: false },
  { id: 'name', desc: true },
]
```

## Initial Sorting (Uncontrolled)

```tsx
const table = useReactTable({
  // ...
  initialState: {
    sorting: [{ id: 'name', desc: true }],
  },
})
```

## Server-Side (Manual) Sorting

```tsx
const table = useReactTable({
  data, // already sorted by server
  columns,
  state: { sorting },
  onSortingChange: setSorting,
  getCoreRowModel: getCoreRowModel(),
  manualSorting: true, // skip client-side sort
  // getSortedRowModel not needed
})
```

## Built-in Sort Functions

| Name | Description |
|------|-------------|
| `alphanumeric` | Mixed alphanumeric, case-insensitive. Slower but natural sort order. |
| `alphanumericCaseSensitive` | Same but case-sensitive |
| `text` | String comparison, case-insensitive. Faster than alphanumeric. |
| `textCaseSensitive` | Same but case-sensitive |
| `datetime` | For Date objects |
| `basic` | Simple `a > b ? 1 : a < b ? -1 : 0` comparison |

## Sorting Function Signature

```tsx
type SortingFn<TData> = (
  rowA: Row<TData>,
  rowB: Row<TData>,
  columnId: string
) => number // -1 (a < b), 0 (equal), 1 (a > b) — always ascending order

// The desc direction is handled automatically by the table — your function
// should ALWAYS return in ascending order.
```

## Per-Column Sort Function

```tsx
columnHelper.accessor('name', {
  sortingFn: 'alphanumeric',  // built-in by name
})

columnHelper.accessor('birthday', {
  sortingFn: 'datetime',      // for Date values
})

columnHelper.accessor('score', {
  sortingFn: (rowA, rowB, columnId) => {
    // custom: return -1, 0, or 1 (ascending order only — desc handled automatically)
    return rowA.original.score - rowB.original.score
  },
})
```

## Global Custom Sort Functions

```tsx
// Register with declaration merging for type safety:
declare module '@tanstack/table-core' {
  interface SortingFns {
    mySort: SortingFn<unknown>
  }
}

const table = useReactTable({
  // ...
  sortingFns: {
    mySort: (rowA, rowB, columnId) => {
      return rowA.getValue(columnId) > rowB.getValue(columnId) ? 1 : -1
    },
  },
})

// Then reference by name:
columnHelper.accessor('field', { sortingFn: 'mySort' })
```

## Table Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `enableSorting` | `boolean` | `true` | Enable/disable sorting globally |
| `manualSorting` | `boolean` | `false` | Skip client-side sort, handle externally |
| `enableMultiSort` | `boolean` | `true` | Allow multi-column sort |
| `enableSortingRemoval` | `boolean` | `true` | Allow removing sort (3-state toggle: asc → desc → none) |
| `enableMultiRemove` | `boolean` | `true` | Allow removing individual multi-sort columns |
| `maxMultiSortColCount` | `number` | `Infinity` | Limit number of multi-sort columns |
| `isMultiSortEvent` | `(e: unknown) => boolean` | `e.shiftKey` | How to trigger multi-sort (default: shift+click) |
| `sortDescFirst` | `boolean` | `false` | First sort direction globally (true = desc first) |
| `sortingFns` | `Record<string, SortingFn>` | — | Custom sorting functions registry |
| `onSortingChange` | `OnChangeFn<SortingState>` | — | Controlled state handler |

## Column Def Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `sortingFn` | `SortingFn \| BuiltInSortingFn \| string` | auto | Sort function for this column |
| `enableSorting` | `boolean` | `true` | Enable/disable sorting for this column |
| `enableMultiSort` | `boolean` | `true` | Enable/disable multi-sort for this column |
| `sortDescFirst` | `boolean` | `false` | First sort direction for this column |
| `sortUndefined` | `'first' \| 'last' \| false \| -1 \| 1` | `1` | Where to place undefined values |
| `invertSorting` | `boolean` | `false` | Flip sort direction (for inverted scales like rankings) |

### sortUndefined Values

| Value | Description |
|-------|-------------|
| `'first'` | Undefined values at beginning (v8.16.0+) |
| `'last'` | Undefined values at end (v8.16.0+) |
| `false` | Undefined values treated as tied, fall through to next sort |
| `-1` | Higher priority ascending (undefined at start when ascending) |
| `1` | Lower priority descending (undefined at end when ascending) — **default** |

## Column APIs

| Method | Signature | Description |
|--------|-----------|-------------|
| `getIsSorted` | `() => false \| SortDirection` | Current sort direction or false |
| `getCanSort` | `() => boolean` | Can this column be sorted? |
| `getCanMultiSort` | `() => boolean` | Can this column multi-sort? |
| `toggleSorting` | `(desc?: boolean, isMulti?: boolean) => void` | Toggle sort. Optional force direction + multi |
| `clearSorting` | `() => void` | Remove this column from sorting state |
| `getSortIndex` | `() => number` | Position in multi-sort (-1 if not sorted) |
| `getNextSortingOrder` | `() => false \| SortDirection` | Next sort direction on toggle |
| `getFirstSortDir` | `() => SortDirection` | Initial sort direction for this column |
| `getAutoSortingFn` | `() => SortingFn<TData>` | Auto-inferred sort function |
| `getAutoSortDir` | `() => SortDirection` | Auto-inferred sort direction |
| `getSortingFn` | `() => SortingFn<TData>` | Resolved sort function |
| `getToggleSortingHandler` | `() => ((event: unknown) => void) \| undefined` | Click handler for header |

## Table APIs

| Method | Signature | Description |
|--------|-----------|-------------|
| `setSorting` | `(updater: Updater<SortingState>) => void` | Set sorting state |
| `resetSorting` | `(defaultState?: boolean) => void` | Reset sorting. `true` = empty `[]` |
| `getPreSortedRowModel` | `() => RowModel<TData>` | Rows before sorting |
| `getSortedRowModel` | `() => RowModel<TData>` | Rows after sorting |

## Header Rendering with Sort Indicators

```tsx
<th
  onClick={header.column.getToggleSortingHandler()}
  style={{ cursor: header.column.getCanSort() ? 'pointer' : 'default' }}
>
  {flexRender(header.column.columnDef.header, header.getContext())}
  {{
    asc: ' 🔼',
    desc: ' 🔽',
  }[header.column.getIsSorted() as string] ?? null}
  {/* Multi-sort index indicator */}
  {header.column.getSortIndex() > 0 && (
    <span style={{ fontSize: '0.7em' }}>{header.column.getSortIndex() + 1}</span>
  )}
</th>
```

## Programmatic Sorting

```tsx
// Set single sort:
table.setSorting([{ id: 'name', desc: false }])

// Multi-sort:
table.setSorting([
  { id: 'status', desc: false },
  { id: 'name', desc: true },
])

// Add to existing sort:
table.setSorting(old => [...old, { id: 'age', desc: true }])

// Clear all sorting:
table.resetSorting()
```

## Sorting Toggle Cycle

Default 3-state toggle per column click:
1. **None** → Ascending
2. **Ascending** → Descending
3. **Descending** → None (removed)

With `enableSortingRemoval: false`: 2-state toggle (asc ↔ desc, never removed)

With `sortDescFirst: true`: desc → asc → none

## Edge Cases & Gotchas

- Sort functions must always return values in **ascending** order — the table handles desc inversion
- `getSortedRowModel()` must be provided for client-side sorting, even with controlled state
- `manualSorting: true` means `getSortedRowModel` is not needed and not used
- Default multi-sort is shift+click — users may not discover this. Consider adding UI hints
- `sortUndefined: 'first'` and `'last'` are available since v8.16.0 — older versions only support `-1`, `1`, `false`
- `invertSorting` is useful for golf-like scores or rankings where lower = better
- Sorting interacts with pagination: `autoResetPageIndex` (default true) resets to page 0 when sort changes
