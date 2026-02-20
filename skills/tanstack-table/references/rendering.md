# Rendering

## Minimal Table Structure

```tsx
import {
  useReactTable,
  getCoreRowModel,
  flexRender,
  ColumnDef,
} from '@tanstack/react-table'

function DataTable<TData>({ data, columns }: { data: TData[]; columns: ColumnDef<TData>[] }) {
  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
  })

  return (
    <table>
      <thead>
        {table.getHeaderGroups().map(headerGroup => (
          <tr key={headerGroup.id}>
            {headerGroup.headers.map(header => (
              <th key={header.id} colSpan={header.colSpan}>
                {header.isPlaceholder
                  ? null
                  : flexRender(header.column.columnDef.header, header.getContext())}
              </th>
            ))}
          </tr>
        ))}
      </thead>
      <tbody>
        {table.getRowModel().rows.map(row => (
          <tr key={row.id}>
            {row.getVisibleCells().map(cell => (
              <td key={cell.id}>
                {flexRender(cell.column.columnDef.cell, cell.getContext())}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
      <tfoot>
        {table.getFooterGroups().map(footerGroup => (
          <tr key={footerGroup.id}>
            {footerGroup.headers.map(header => (
              <th key={header.id} colSpan={header.colSpan}>
                {header.isPlaceholder
                  ? null
                  : flexRender(header.column.columnDef.footer, header.getContext())}
              </th>
            ))}
          </tr>
        ))}
      </tfoot>
    </table>
  )
}
```

## flexRender

The standard utility for rendering column def templates:

```tsx
import { flexRender } from '@tanstack/react-table'

// Renders whatever the column def provides (string, JSX, component)
flexRender(header.column.columnDef.header, header.getContext())
flexRender(cell.column.columnDef.cell, cell.getContext())
flexRender(header.column.columnDef.footer, header.getContext())
```

`flexRender` handles:
- Strings → rendered as-is
- Functions → called with context props
- Components → rendered with context as props
- `undefined` → renders nothing

## Header Groups

For nested/grouped column headers:

```tsx
// Single-level headers:
table.getHeaderGroups()  // HeaderGroup[]

// Each header group contains:
headerGroup.id          // string
headerGroup.headers     // Header[]

// Each header:
header.id               // string
header.colSpan          // number (for grouped headers)
header.isPlaceholder    // boolean (spacer in grouped layouts)
header.column           // Column instance
header.getContext()      // context for flexRender
```

## With Column Pinning

```tsx
<table>
  <thead>
    {table.getHeaderGroups().map(headerGroup => (
      <tr key={headerGroup.id}>
        {/* Left pinned */}
        {headerGroup.headers
          .filter(h => h.column.getIsPinned() === 'left')
          .map(renderHeader)}
        {/* Center (unpinned) */}
        {headerGroup.headers
          .filter(h => !h.column.getIsPinned())
          .map(renderHeader)}
        {/* Right pinned */}
        {headerGroup.headers
          .filter(h => h.column.getIsPinned() === 'right')
          .map(renderHeader)}
      </tr>
    ))}
  </thead>
</table>

// OR use the dedicated APIs:
table.getLeftHeaderGroups()
table.getCenterHeaderGroups()
table.getRightHeaderGroups()
// Same for rows:
row.getLeftVisibleCells()
row.getCenterVisibleCells()
row.getRightVisibleCells()
```

## Table Meta (Passing Context)

```tsx
// Extend the TableMeta type globally:
declare module '@tanstack/react-table' {
  interface TableMeta<TData extends RowData> {
    updateData: (rowIndex: number, columnId: string, value: unknown) => void
  }
}

const table = useReactTable({
  // ...
  meta: {
    updateData: (rowIndex, columnId, value) => {
      setData(old => old.map((row, index) =>
        index === rowIndex ? { ...row, [columnId]: value } : row
      ))
    },
  },
})

// Access in cell renderer:
columnHelper.accessor('name', {
  cell: ({ getValue, row, column, table }) => {
    const meta = table.options.meta
    return (
      <input
        value={getValue()}
        onChange={e => meta?.updateData(row.index, column.id, e.target.value)}
      />
    )
  },
})
```

## Row Click Handler

```tsx
<tr
  key={row.id}
  onClick={() => handleRowClick(row.original)}
  style={{ cursor: 'pointer' }}
>
```

## Empty State

```tsx
<tbody>
  {table.getRowModel().rows.length === 0 ? (
    <tr>
      <td colSpan={columns.length} style={{ textAlign: 'center' }}>
        No results found
      </td>
    </tr>
  ) : (
    table.getRowModel().rows.map(row => /* ... */)
  )}
</tbody>
```

## Key Types

```tsx
import type {
  ColumnDef,            // Column definition
  Table,                // Table instance
  Row,                  // Row instance
  Cell,                 // Cell instance
  Column,               // Column instance
  Header,               // Header instance
  HeaderGroup,          // Header group
  SortingState,         // Sorting state
  ColumnFiltersState,   // Column filter state
  PaginationState,      // Pagination state
  RowSelectionState,    // Row selection state
  VisibilityState,      // Column visibility state
  ExpandedState,        // Expanded state
  GroupingState,        // Grouping state
  ColumnOrderState,     // Column order state
  ColumnPinningState,   // Column pinning state
  ColumnSizingState,    // Column sizing state
  CellContext,          // Context passed to cell renderers
  HeaderContext,        // Context passed to header renderers
  FilterFn,             // Custom filter function type
  SortingFn,            // Custom sorting function type
  AggregationFn,        // Custom aggregation function type
} from '@tanstack/react-table'
```

See [advanced](advanced.md) for editable cells, sub-components, row pinning, DnD reordering, and custom features.

## Framework Adapters

| Framework | Hook/Function | Package |
|-----------|--------------|---------|
| React | `useReactTable` | `@tanstack/react-table` |
| Vue | `useVueTable` | `@tanstack/vue-table` |
| Solid | `createSolidTable` | `@tanstack/solid-table` |
| Svelte | `createSvelteTable` | `@tanstack/svelte-table` |
| Qwik | `useQwikTable` | `@tanstack/qwik-table` |

All use the same core API — only the table creation function and `flexRender` differ.
