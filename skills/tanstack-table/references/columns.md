# Column Definitions

## Column Def Types

There are three categories of column definitions:

- **Accessor Columns** — have a data model, can be sorted/filtered/grouped
- **Display Columns** — no data model, for action buttons, checkboxes, expanders
- **Grouping Columns** — group other columns together, define shared headers/footers

## Column Helper (Recommended)

```tsx
import { createColumnHelper } from '@tanstack/react-table'

type Person = {
  id: string
  firstName: string
  lastName: string
  age: number
  status: 'active' | 'inactive'
}

const columnHelper = createColumnHelper<Person>()

const columns = [
  // Display column (no accessor)
  columnHelper.display({
    id: 'actions',
    cell: ({ row }) => <button onClick={() => edit(row.original)}>Edit</button>,
  }),

  // Accessor column with object key
  columnHelper.accessor('firstName', {
    header: 'First Name',
    cell: info => info.getValue(),
  }),

  // Accessor column with function (requires id)
  columnHelper.accessor(row => `${row.firstName} ${row.lastName}`, {
    id: 'fullName',
    header: 'Full Name',
  }),

  // Group column
  columnHelper.group({
    header: 'Info',
    columns: [
      columnHelper.accessor('age', { header: 'Age' }),
      columnHelper.accessor('status', { header: 'Status' }),
    ],
  }),
]
```

## Plain Object Column Defs

```tsx
import { ColumnDef } from '@tanstack/react-table'

const columns: ColumnDef<Person>[] = [
  { accessorKey: 'firstName', header: 'First Name' },
  { accessorFn: row => row.lastName, id: 'lastName', header: 'Last Name' },
  { id: 'actions', cell: ({ row }) => <button>Edit</button> },
]
```

## Column ID Rules

- `accessorKey` → used as ID (periods replaced with underscores: `address.city` → `address_city`)
- `accessorFn` → must provide `id` or a string `header`
- Display columns → must provide `id`
- Always provide explicit `id` when in doubt

## Cell / Header / Footer Rendering

```tsx
columnHelper.accessor('age', {
  header: () => <span>Age</span>,           // JSX header
  header: 'Age',                             // string header
  header: ({ column, header, table }) => {}, // full context
  cell: info => info.getValue(),             // cell value
  cell: ({ row, getValue, column, table, renderValue }) => (
    <span>{row.original.id}: {getValue()}</span>
  ),
  footer: ({ column }) => column.id,         // footer
  aggregatedCell: ({ getValue }) => getValue(), // for grouped rows
})
```

### Cell Context Properties

```tsx
interface CellContext<TData, TValue> {
  table: Table<TData>
  column: Column<TData, TValue>
  row: Row<TData>
  cell: Cell<TData, TValue>
  getValue: () => TValue               // accessor value
  renderValue: () => TValue | null     // getValue() with null fallback
}
```

### Header Context Properties

```tsx
interface HeaderContext<TData, TValue> {
  table: Table<TData>
  column: Column<TData, TValue>
  header: Header<TData, TValue>
}
```

## Column Meta

Attach arbitrary typed metadata to columns:

```tsx
declare module '@tanstack/react-table' {
  interface ColumnMeta<TData extends RowData, TValue> {
    tooltip?: string
    align?: 'left' | 'center' | 'right'
    className?: string
    filterVariant?: 'text' | 'select' | 'range'
  }
}

columnHelper.accessor('revenue', {
  header: 'Revenue',
  meta: {
    tooltip: 'Total revenue in USD',
    align: 'right',
    filterVariant: 'range',
  },
})

// Access in cell/header:
cell.column.columnDef.meta?.align
```

## Default Column Options

```tsx
const table = useReactTable({
  columns,
  data,
  defaultColumn: {
    size: 150,
    minSize: 50,
    maxSize: 500,
    cell: ({ getValue }) => getValue(), // default cell renderer
    enableSorting: true,
    enableColumnFilter: true,
    enableGlobalFilter: true,
    enableGrouping: false,
    enableHiding: true,
    enableResizing: true,
    enablePinning: true,
  },
  getCoreRowModel: getCoreRowModel(),
})
```

## Core Column API

Every column instance has these core properties/methods:

| Property/Method | Type | Description |
|--------|------|-------------|
| `id` | `string` | Unique identifier |
| `depth` | `number` | Depth in column group hierarchy (0 = root) |
| `accessorFn` | `AccessorFn<TData> \| undefined` | Resolved accessor function |
| `columnDef` | `ColumnDef<TData>` | Original column def |
| `columns` | `Column<TData>[]` | Child columns (if group column). Empty array if leaf |
| `parent` | `Column<TData> \| undefined` | Parent column (if nested in group) |
| `getFlatColumns` | `() => Column<TData>[]` | Flattened array of this + all descendant columns |
| `getLeafColumns` | `() => Column<TData>[]` | All leaf-node columns (no children) |

## Column Sizing

```tsx
import { getCoreRowModel, useReactTable } from '@tanstack/react-table'

const table = useReactTable({
  columns,
  data,
  getCoreRowModel: getCoreRowModel(),
  columnResizeMode: 'onChange', // 'onChange' | 'onEnd'
  columnResizeDirection: 'ltr', // 'ltr' | 'rtl'
})

// In headers:
<th style={{ width: header.getSize() }}>
  {flexRender(header.column.columnDef.header, header.getContext())}
  <div
    onMouseDown={header.getResizeHandler()}
    onTouchStart={header.getResizeHandler()}
    className={header.column.getIsResizing() ? 'isResizing' : ''}
    style={{
      position: 'absolute',
      right: 0,
      top: 0,
      height: '100%',
      width: '5px',
      cursor: 'col-resize',
    }}
  />
</th>
```

### Column Size Properties

```tsx
// Per column def:
columnHelper.accessor('name', {
  size: 200,    // default size in px
  minSize: 50,  // minimum resize size
  maxSize: 500, // maximum resize size
})

// Column API:
column.getSize()        // current size
column.getStart()       // start offset
column.getIsResizing()  // currently being resized?
column.resetSize()      // reset to default
```

Column size state: `Record<string, number>` for `columnSizing` and `ColumnSizingInfoState` for resize drag state.

## Column Ordering

```tsx
const [columnOrder, setColumnOrder] = useState<string[]>([])

const table = useReactTable({
  columns,
  data,
  state: { columnOrder },
  onColumnOrderChange: setColumnOrder,
  getCoreRowModel: getCoreRowModel(),
})

// APIs:
table.resetColumnOrder()
table.setColumnOrder(['col1', 'col2', 'col3'])
table.setColumnOrder(old => [...old].reverse()) // reverse order
```

## Column Pinning

```tsx
const [columnPinning, setColumnPinning] = useState<ColumnPinningState>({
  left: ['id'],
  right: ['actions'],
})

const table = useReactTable({
  state: { columnPinning },
  onColumnPinningChange: setColumnPinning,
  getCoreRowModel: getCoreRowModel(),
})

// Column APIs:
column.pin('left')           // pin left
column.pin('right')          // pin right
column.pin(false)            // unpin
column.getIsPinned()         // 'left' | 'right' | false
column.getPinnedIndex()      // index within pinned group
column.getCanPin()           // boolean

// Table APIs for split rendering:
table.getLeftHeaderGroups()    // header groups for left-pinned columns
table.getCenterHeaderGroups()  // header groups for unpinned columns
table.getRightHeaderGroups()   // header groups for right-pinned columns
table.getLeftVisibleLeafColumns()
table.getCenterVisibleLeafColumns()
table.getRightVisibleLeafColumns()

// Row-level (for split cell rendering):
row.getLeftVisibleCells()
row.getCenterVisibleCells()
row.getRightVisibleCells()
```

## Column Visibility

```tsx
const [columnVisibility, setColumnVisibility] = useState<VisibilityState>({
  someColumn: false, // hidden by default
})

const table = useReactTable({
  state: { columnVisibility },
  onColumnVisibilityChange: setColumnVisibility,
  getCoreRowModel: getCoreRowModel(),
})

// Toggle UI:
{table.getAllColumns()
  .filter(col => col.getCanHide())
  .map(column => (
    <label key={column.id}>
      <input
        type="checkbox"
        checked={column.getIsVisible()}
        onChange={column.getToggleVisibilityHandler()}
      />
      {column.id}
    </label>
  ))}

// Column APIs:
column.getIsVisible()                // boolean
column.getCanHide()                  // boolean (respects enableHiding)
column.toggleVisibility(value?)      // toggle or set
column.getToggleVisibilityHandler()  // checkbox onChange handler

// Table APIs:
table.getVisibleLeafColumns()        // only visible leaf columns
table.getVisibleFlatColumns()        // visible columns flattened
table.toggleAllColumnsVisible(value?)
table.getIsAllColumnsVisible()
table.getIsSomeColumnsVisible()

// Disable hiding per column:
{ accessorKey: 'id', enableHiding: false }
```

## IMPORTANT: Visibility-Aware APIs

Always use visibility-aware APIs for rendering:

```tsx
// ✅ Correct:
table.getVisibleLeafColumns()  // not getAllLeafColumns()
row.getVisibleCells()          // not getAllCells()

// Header group APIs already account for visibility
table.getHeaderGroups()        // only visible headers
```

## Edge Cases & Gotchas

- `accessorKey` with dots (e.g., `'address.city'`) does deep property access; the column ID replaces dots with underscores
- Group columns can't be sorted/filtered — only leaf columns with accessors can
- `columnHelper.group()` columns don't need `id` if they have a `header` string
- `defaultColumn` applies to ALL columns — use per-column overrides for exceptions
- `enableHiding: false` prevents both programmatic and UI-driven visibility changes
- Column order state `[]` (empty array) means "use original column order" — it's not "no columns"
- Resizing: `columnResizeMode: 'onChange'` updates live during drag; `'onEnd'` only on mouse up (better performance)
- Pinned columns render in separate header groups — you need to render left/center/right separately for proper pinning UI
