# Row Selection

## Setup

```tsx
import {
  getCoreRowModel,
  useReactTable,
  RowSelectionState,
} from '@tanstack/react-table'

const [rowSelection, setRowSelection] = useState<RowSelectionState>({})

const table = useReactTable({
  data,
  columns,
  state: { rowSelection },
  onRowSelectionChange: setRowSelection,
  getCoreRowModel: getCoreRowModel(),
  getRowId: row => row.id, // use meaningful IDs, not array indices
})
```

## Selection State

```tsx
// RowSelectionState is a map of row ID → boolean
type RowSelectionState = Record<string, boolean>

// Example: { 'user-123': true, 'user-456': true }

type RowSelectionTableState = {
  rowSelection: RowSelectionState
}
```

## Table Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `enableRowSelection` | `boolean \| ((row: Row<TData>) => boolean)` | `true` | Enable/disable selection globally or per-row |
| `enableMultiRowSelection` | `boolean \| ((row: Row<TData>) => boolean)` | `true` | Enable/disable multi-select. `false` = single select (radio mode) |
| `enableSubRowSelection` | `boolean \| ((row: Row<TData>) => boolean)` | `true` | Auto-select sub-rows when parent selected. Use with expanding/grouping |
| `onRowSelectionChange` | `OnChangeFn<RowSelectionState>` | — | Controlled state handler. Overrides internal state |

## Checkbox Column Pattern

```tsx
const columns = [
  {
    id: 'select',
    header: ({ table }) => (
      <input
        type="checkbox"
        checked={table.getIsAllRowsSelected()}
        ref={el => {
          if (el) el.indeterminate = table.getIsSomeRowsSelected()
        }}
        onChange={table.getToggleAllRowsSelectedHandler()}
      />
    ),
    cell: ({ row }) => (
      <input
        type="checkbox"
        checked={row.getIsSelected()}
        disabled={!row.getCanSelect()}
        onChange={row.getToggleSelectedHandler()}
      />
    ),
  },
  // ...other columns
]
```

## Radio Button (Single Select) Pattern

```tsx
const table = useReactTable({
  // ...
  enableMultiRowSelection: false, // single select mode
})

const columns = [
  {
    id: 'select',
    header: 'Select',
    cell: ({ row }) => (
      <input
        type="radio"
        name="row-selection"
        checked={row.getIsSelected()}
        disabled={!row.getCanSelect()}
        onChange={row.getToggleSelectedHandler()}
      />
    ),
  },
]
```

## Select All Variants

```tsx
// Select ALL rows (across all pages):
table.getToggleAllRowsSelectedHandler()   // event handler for onChange
table.getIsAllRowsSelected()              // boolean — all rows selected
table.getIsSomeRowsSelected()             // boolean — some (not all) rows selected
table.toggleAllRowsSelected(value?)       // programmatic toggle

// Select only CURRENT PAGE rows:
table.getToggleAllPageRowsSelectedHandler()  // event handler
table.getIsAllPageRowsSelected()             // boolean
table.getIsSomePageRowsSelected()            // boolean
table.toggleAllPageRowsSelected(value?)      // programmatic toggle
```

## Table APIs

| Method | Signature | Description |
|--------|-----------|-------------|
| `setRowSelection` | `(updater: Updater<RowSelectionState>) => void` | Set or update selection state |
| `resetRowSelection` | `(defaultState?: boolean) => void` | Reset to initial. Pass `true` for empty `{}` |
| `toggleAllRowsSelected` | `(value?: boolean) => void` | Select/deselect all rows |
| `toggleAllPageRowsSelected` | `(value?: boolean) => void` | Select/deselect current page rows |
| `getIsAllRowsSelected` | `() => boolean` | All rows selected? |
| `getIsAllPageRowsSelected` | `() => boolean` | All current page rows selected? |
| `getIsSomeRowsSelected` | `() => boolean` | Any rows selected? |
| `getIsSomePageRowsSelected` | `() => boolean` | Any current page rows selected? |
| `getToggleAllRowsSelectedHandler` | `() => (event: unknown) => void` | Event handler for select-all checkbox |
| `getToggleAllPageRowsSelectedHandler` | `() => (event: unknown) => void` | Event handler for select-all-page checkbox |
| `getPreSelectedRowModel` | `() => RowModel<TData>` | Row model before selection processing |
| `getSelectedRowModel` | `() => RowModel<TData>` | Selected rows (from loaded data only) |
| `getFilteredSelectedRowModel` | `() => RowModel<TData>` | Selected rows after filtering |
| `getGroupedSelectedRowModel` | `() => RowModel<TData>` | Selected rows after grouping |

## Row APIs

| Method | Signature | Description |
|--------|-----------|-------------|
| `getIsSelected` | `() => boolean` | Is this row selected? |
| `getIsSomeSelected` | `() => boolean` | Are some sub-rows selected? |
| `getIsAllSubRowsSelected` | `() => boolean` | Are all sub-rows selected? |
| `getCanSelect` | `() => boolean` | Can this row be selected? |
| `getCanMultiSelect` | `() => boolean` | Can this row multi-select? |
| `getCanSelectSubRows` | `() => boolean` | Can sub-rows auto-select with parent? |
| `toggleSelected` | `(value?: boolean) => void` | Toggle or set selection |
| `getToggleSelectedHandler` | `() => (event: unknown) => void` | Event handler for checkbox onChange |

## Conditional Selection

```tsx
const table = useReactTable({
  // ...
  enableRowSelection: row => row.original.age >= 18,   // only adults
  enableMultiRowSelection: row => !row.original.isLocked, // no multi-select for locked rows
  enableSubRowSelection: false,                         // don't cascade to children
})
```

## Accessing Selected Rows

```tsx
// Get selected row data:
const selectedRows = table.getSelectedRowModel().rows
const selectedData = selectedRows.map(row => row.original)

// Get selection state for API calls:
const selectedIds = Object.keys(rowSelection).filter(id => rowSelection[id])

// Count:
const selectedCount = Object.keys(rowSelection).length
```

## Programmatic Selection

```tsx
// Select specific rows:
table.setRowSelection({ 'row-1': true, 'row-5': true })

// Add to existing selection:
table.setRowSelection(old => ({
  ...old,
  'row-10': true,
}))

// Clear all:
table.resetRowSelection()

// Select all:
table.toggleAllRowsSelected(true)
```

## Selection with Sub-Rows

```tsx
const table = useReactTable({
  // ...
  enableSubRowSelection: true, // default: true
  getSubRows: row => row.children,
  getExpandedRowModel: getExpandedRowModel(),
})

// When parent is selected:
// - All children auto-selected (if enableSubRowSelection is true)
// - Parent shows indeterminate when some children selected
// - row.getIsSomeSelected() returns true for partial child selection
// - row.getIsAllSubRowsSelected() returns true when all children selected
```

## Bulk Actions Pattern

```tsx
function BulkActions() {
  const selectedRows = table.getFilteredSelectedRowModel().rows

  if (selectedRows.length === 0) return null

  return (
    <div>
      <span>{selectedRows.length} row(s) selected</span>
      <button onClick={() => handleDelete(selectedRows.map(r => r.original.id))}>
        Delete Selected
      </button>
      <button onClick={() => handleExport(selectedRows.map(r => r.original))}>
        Export Selected
      </button>
      <button onClick={() => table.resetRowSelection()}>
        Clear Selection
      </button>
    </div>
  )
}
```

## Important Notes & Gotchas

- With `manualPagination`, `getSelectedRowModel()` only returns selected rows from the **current page data** passed to the table. The `rowSelection` state itself correctly stores all selections across pages.
- Use `getRowId` to map to database IDs — default index-based IDs break when data changes (sorting, filtering, pagination).
- Single selection mode (`enableMultiRowSelection: false`): selecting a new row auto-deselects the previous one.
- `getToggleSelectedHandler()` checks `event.target.checked` — works with checkbox/radio inputs, not buttons. For buttons use `row.toggleSelected()`.
- Selection state is a flat map — even with nested sub-rows, all selected row IDs appear at the top level of the state object.
- `getFilteredSelectedRowModel()` is useful for bulk actions — it excludes rows that have been filtered out but are still in `rowSelection` state.
- When using with grouping, grouped rows can also be selected — use `enableSubRowSelection` to control cascade behavior.
