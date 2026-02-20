# Row Pinning

Pin rows to the top or bottom of the table, keeping them visible regardless of sorting, filtering, or pagination.

## Setup

```tsx
import { getCoreRowModel, useReactTable, RowPinningState } from '@tanstack/react-table'

const [rowPinning, setRowPinning] = useState<RowPinningState>({
  top: [],
  bottom: [],
})

const table = useReactTable({
  data,
  columns,
  state: { rowPinning },
  onRowPinningChange: setRowPinning,
  enableRowPinning: true, // or (row) => row.original.canPin
  keepPinnedRows: true,   // default: true — pinned rows stay visible even if filtered/paginated out
  getCoreRowModel: getCoreRowModel(),
  getRowId: (row) => row.id, // recommended: use stable IDs for pinning
})
```

## State Shape

```tsx
type RowPinningPosition = false | 'top' | 'bottom'

type RowPinningState = {
  top?: string[]    // array of row IDs pinned to top
  bottom?: string[] // array of row IDs pinned to bottom
}

type RowPinningRowState = {
  rowPinning: RowPinningState
}
```

## Can-Pin Resolution

A row can be pinned when:
- `options.enableRowPinning` resolves to `true` (globally or per-row via function)
- `options.enablePinning` is not set to `false`

## Table Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `enableRowPinning` | `boolean \| ((row: Row<TData>) => boolean)` | — | Enable/disable row pinning globally or per-row |
| `keepPinnedRows` | `boolean` | `true` | When `true`, pinned rows always visible even if filtered/paginated out. When `false`, pinned rows hidden if filtered/paginated out |
| `onRowPinningChange` | `OnChangeFn<RowPinningState>` | — | State change handler for controlled state. Overrides internal state management |

## Table APIs

| Method | Signature | Description |
|--------|-----------|-------------|
| `setRowPinning` | `(updater: Updater<RowPinningState>) => void` | Set or update row pinning state |
| `resetRowPinning` | `(defaultState?: boolean) => void` | Reset to `initialState.rowPinning`. Pass `true` to reset to `{}` |
| `getIsSomeRowsPinned` | `(position?: RowPinningPosition) => boolean` | Any rows pinned? Optionally check only 'top' or 'bottom' |
| `getTopRows` | `() => Row<TData>[]` | All top-pinned rows |
| `getBottomRows` | `() => Row<TData>[]` | All bottom-pinned rows |
| `getCenterRows` | `() => Row<TData>[]` | All non-pinned rows |

## Row APIs

| Method | Signature | Description |
|--------|-----------|-------------|
| `pin` | `(position: RowPinningPosition) => void` | Pin to 'top', 'bottom', or unpin with `false` |
| `getCanPin` | `() => boolean` | Whether this row can be pinned |
| `getIsPinned` | `() => RowPinningPosition` | Returns 'top', 'bottom', or `false` |
| `getPinnedIndex` | `() => number` | Numeric index within the pinned group |

## Rendering Pattern

```tsx
<tbody>
  {/* Top pinned rows */}
  {table.getTopRows().map(row => (
    <tr key={row.id} className="pinned-top">
      {row.getVisibleCells().map(cell => (
        <td key={cell.id}>{flexRender(cell.column.columnDef.cell, cell.getContext())}</td>
      ))}
    </tr>
  ))}

  {/* Center (unpinned) rows */}
  {table.getCenterRows().map(row => (
    <tr key={row.id}>
      {row.getVisibleCells().map(cell => (
        <td key={cell.id}>{flexRender(cell.column.columnDef.cell, cell.getContext())}</td>
      ))}
    </tr>
  ))}

  {/* Bottom pinned rows */}
  {table.getBottomRows().map(row => (
    <tr key={row.id} className="pinned-bottom">
      {row.getVisibleCells().map(cell => (
        <td key={cell.id}>{flexRender(cell.column.columnDef.cell, cell.getContext())}</td>
      ))}
    </tr>
  ))}
</tbody>
```

## Pin Toggle Button

```tsx
const columns = [
  columnHelper.display({
    id: 'pin',
    header: 'Pin',
    cell: ({ row }) => {
      const isPinned = row.getIsPinned()
      return (
        <div>
          <button
            onClick={() => row.pin(isPinned === 'top' ? false : 'top')}
            disabled={!row.getCanPin()}
          >
            {isPinned === 'top' ? '📌 Unpin' : '⬆️ Pin Top'}
          </button>
          <button
            onClick={() => row.pin(isPinned === 'bottom' ? false : 'bottom')}
            disabled={!row.getCanPin()}
          >
            {isPinned === 'bottom' ? '📌 Unpin' : '⬇️ Pin Bottom'}
          </button>
        </div>
      )
    },
  }),
  // ...other columns
]
```

## Conditional Per-Row Pinning

```tsx
const table = useReactTable({
  // ...
  enableRowPinning: (row) => {
    // Only allow pinning for rows with a certain status
    return row.original.status !== 'archived'
  },
})
```

## Programmatic Pinning

```tsx
// Pin specific rows by ID
table.setRowPinning({
  top: ['row-1', 'row-5'],
  bottom: ['row-99'],
})

// Add a row to top pinned without removing existing
table.setRowPinning(old => ({
  ...old,
  top: [...(old.top ?? []), 'row-10'],
}))

// Unpin all
table.resetRowPinning()
// Or force empty state:
table.resetRowPinning(true)
```

## Sticky Positioning with CSS

```css
/* Sticky top-pinned rows */
.pinned-top {
  position: sticky;
  top: 0;
  z-index: 1;
  background: white;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

/* Sticky bottom-pinned rows */
.pinned-bottom {
  position: sticky;
  bottom: 0;
  z-index: 1;
  background: white;
  box-shadow: 0 -2px 4px rgba(0, 0, 0, 0.1);
}
```

## Row Pinning with Pagination

When `keepPinnedRows: true` (default):
- Pinned rows appear on **every page**, even if their original position is on another page
- `getCenterRows()` returns only the unpinned rows for the current page

When `keepPinnedRows: false`:
- Pinned rows only appear when their data is on the current page
- They disappear when paginated away

## Row Pinning with Sorting/Filtering

- Pinned rows maintain their pinned position regardless of sort order
- With `keepPinnedRows: true`, pinned rows stay visible even if filtered out
- The `getTopRows()` / `getBottomRows()` arrays maintain the order rows were pinned

## Edge Cases & Gotchas

- **Always use `getRowId`** — default index-based IDs break when data changes, causing wrong rows to be pinned
- **`keepPinnedRows: true`** means pinned rows appear even when they don't match active filters — this can confuse users
- Pinned row IDs that don't exist in the data are silently ignored
- Row pinning state persists across data changes — stale IDs won't cause errors
- Pinning works with sub-rows: pinning a parent row pins the parent (sub-rows follow if expanded)
- `getPinnedIndex()` returns the index within the pinned group, not the overall table index
