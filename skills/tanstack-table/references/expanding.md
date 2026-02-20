# Expanding

Expand rows to show sub-rows (hierarchical data) or custom detail panels. Works independently of grouping.

## Two Use Cases

1. **Sub-rows** — child rows that share parent column structure (hierarchical data)
2. **Custom UI** — detail panels, sub-tables, arbitrary content below a row

## Setup — Sub-rows

```tsx
import {
  getCoreRowModel,
  getExpandedRowModel,
  useReactTable,
  ExpandedState,
} from '@tanstack/react-table'

type Person = {
  id: number
  name: string
  age: number
  children?: Person[]
}

const [expanded, setExpanded] = useState<ExpandedState>({})

const table = useReactTable({
  data,
  columns,
  state: { expanded },
  onExpandedChange: setExpanded,
  getSubRows: (row) => row.children, // tells table where to find child rows
  getCoreRowModel: getCoreRowModel(),
  getExpandedRowModel: getExpandedRowModel(),
})
```

## Setup — Custom Detail Panels

```tsx
const table = useReactTable({
  data,
  columns,
  state: { expanded },
  onExpandedChange: setExpanded,
  getRowCanExpand: (row) => true, // override — all rows expandable
  getCoreRowModel: getCoreRowModel(),
  getExpandedRowModel: getExpandedRowModel(),
})
```

## Expanded State

```tsx
type ExpandedState = true | Record<string, boolean>
// true = ALL rows expanded
// { 'row-1': true, 'row-3': true } = specific rows expanded
// {} = none expanded

type ExpandedTableState = {
  expanded: ExpandedState
}
```

## Table Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `enableExpanding` | `boolean` | `true` | Enable/disable expanding for all rows |
| `manualExpanding` | `boolean` | `false` | Skip `getExpandedRowModel`, handle expansion yourself |
| `autoResetExpanded` | `boolean` | `false` | Auto-reset expanded state when data changes |
| `paginateExpandedRows` | `boolean` | `true` | If `true`, expanded rows count toward page size |
| `getExpandedRowModel` | `(table) => () => RowModel<TData>` | — | Row model function for client-side expanding |
| `getSubRows` | `(row: TData) => TData[] \| undefined` | — | Extract child rows from data |
| `getRowCanExpand` | `(row: Row<TData>) => boolean` | checks subRows | Override which rows can expand |
| `getIsRowExpanded` | `(row: Row<TData>) => boolean` | — | Override expanded check (custom source of truth) |
| `onExpandedChange` | `OnChangeFn<ExpandedState>` | — | Controlled state handler |

## Table APIs

| Method | Signature | Description |
|--------|-----------|-------------|
| `setExpanded` | `(updater: Updater<ExpandedState>) => void` | Set expanded state |
| `resetExpanded` | `(defaultState?: boolean) => void` | Reset to initial. `true` = `{}` |
| `toggleAllRowsExpanded` | `(expanded?: boolean) => void` | Toggle all. Pass `true`/`false` to force |
| `getIsAllRowsExpanded` | `() => boolean` | All rows expanded? |
| `getIsSomeRowsExpanded` | `() => boolean` | Some rows expanded? |
| `getCanSomeRowsExpand` | `() => boolean` | Any rows expandable? |
| `getToggleAllRowsExpandedHandler` | `() => (event: unknown) => void` | Checkbox handler for expand-all |
| `getExpandedDepth` | `() => number` | Max depth of expanded rows |
| `getExpandedRowModel` | `() => RowModel<TData>` | Rows after expansion applied |
| `getPreExpandedRowModel` | `() => RowModel<TData>` | Rows before expansion |

## Row APIs

| Method/Property | Signature | Description |
|--------|-----------|-------------|
| `toggleExpanded` | `(expanded?: boolean) => void` | Toggle or force expand/collapse |
| `getIsExpanded` | `() => boolean` | Is this row expanded? |
| `getCanExpand` | `() => boolean` | Has subRows or `getRowCanExpand` allows it |
| `getIsAllParentsExpanded` | `() => boolean` | All ancestors expanded? |
| `getToggleExpandedHandler` | `() => () => void` | Click handler for expand button |
| `subRows` | `Row<TData>[]` | Child rows |
| `depth` | `number` | Nesting depth (0 = root) |
| `parentId` | `string \| undefined` | Parent row ID |
| `getParentRow` | `() => Row<TData> \| undefined` | Parent row instance |

## Rendering — Sub-rows

```tsx
const columns = [
  columnHelper.display({
    id: 'expander',
    header: () => null,
    cell: ({ row }) =>
      row.getCanExpand() ? (
        <button
          onClick={row.getToggleExpandedHandler()}
          style={{ cursor: 'pointer' }}
        >
          {row.getIsExpanded() ? '▼' : '▶'}
        </button>
      ) : (
        <span style={{ paddingLeft: '1rem' }}>•</span>
      ),
  }),
  columnHelper.accessor('name', {
    cell: ({ row, getValue }) => (
      <span style={{ paddingLeft: `${row.depth * 2}rem` }}>
        {getValue()}
      </span>
    ),
  }),
  // ...more columns
]

// Render normally — getRowModel() includes expanded sub-rows
<tbody>
  {table.getRowModel().rows.map(row => (
    <tr key={row.id}>
      {row.getVisibleCells().map(cell => (
        <td key={cell.id}>{flexRender(cell.column.columnDef.cell, cell.getContext())}</td>
      ))}
    </tr>
  ))}
</tbody>
```

## Rendering — Detail Panels

```tsx
<tbody>
  {table.getRowModel().rows.map(row => (
    <React.Fragment key={row.id}>
      <tr>
        {row.getVisibleCells().map(cell => (
          <td key={cell.id}>{flexRender(cell.column.columnDef.cell, cell.getContext())}</td>
        ))}
      </tr>
      {row.getIsExpanded() && (
        <tr>
          <td colSpan={row.getAllCells().length}>
            <DetailPanel data={row.original} />
          </td>
        </tr>
      )}
    </React.Fragment>
  ))}
</tbody>
```

## Expand All Toggle

```tsx
<button onClick={table.getToggleAllRowsExpandedHandler()}>
  {table.getIsAllRowsExpanded() ? 'Collapse All' : 'Expand All'}
</button>

// Or with a checkbox:
<input
  type="checkbox"
  checked={table.getIsAllRowsExpanded()}
  onChange={table.getToggleAllRowsExpandedHandler()}
/>
```

## Programmatic Expansion

```tsx
// Expand specific rows:
table.setExpanded({ 'row-1': true, 'row-5': true })

// Expand all:
table.setExpanded(true)

// Collapse all:
table.resetExpanded()

// Toggle a specific row:
table.setExpanded(old => ({
  ...(old === true ? {} : old),
  'row-10': !(old === true || (old as Record<string, boolean>)['row-10']),
}))
```

## Filtering Expanded Rows

```tsx
const table = useReactTable({
  // ...
  getSubRows: (row) => row.subRows,
  getCoreRowModel: getCoreRowModel(),
  getFilteredRowModel: getFilteredRowModel(),
  getExpandedRowModel: getExpandedRowModel(),
  filterFromLeafRows: true,       // filter from children up (parent kept if child matches)
  maxLeafRowFilterDepth: undefined, // how deep to filter (undefined = all levels)
})
```

### Filter Direction

| Option | Behavior |
|--------|----------|
| `filterFromLeafRows: false` (default) | Filter parent rows first; if parent filtered out, children hidden too |
| `filterFromLeafRows: true` | Filter leaf rows first; parent kept if any child matches |

## Pagination with Expanding

```tsx
const table = useReactTable({
  // ...
  paginateExpandedRows: true,  // default — expanded rows count toward page size
  // paginateExpandedRows: false, // expanded rows don't affect pagination (all show on parent's page)
})
```

| `paginateExpandedRows` | Behavior |
|------------------------|----------|
| `true` (default) | Expanded sub-rows count toward `pageSize`. Expanding many rows can push items to next page. |
| `false` | Sub-rows don't count. Page always shows same number of top-level rows. More total rows rendered. |

## Lazy Sub-Row Loading

```tsx
function LazyExpandableTable() {
  const [expanded, setExpanded] = useState<ExpandedState>({})

  // Fetch children only when expanded
  const table = useReactTable({
    data,
    columns,
    state: { expanded },
    onExpandedChange: setExpanded,
    getRowCanExpand: (row) => row.original.hasChildren, // server tells us if expandable
    getSubRows: (row) => row.children, // may be undefined until loaded
    getCoreRowModel: getCoreRowModel(),
    getExpandedRowModel: getExpandedRowModel(),
  })

  // Load children when row expands
  useEffect(() => {
    const expandedIds = Object.keys(expanded === true ? {} : expanded)
    expandedIds.forEach(id => {
      const row = table.getRow(id)
      if (row && !row.original.children) {
        fetchChildren(row.original.id).then(children => {
          // Update data with children loaded
          updateDataWithChildren(row.original.id, children)
        })
      }
    })
  }, [expanded])
}
```

## Edge Cases & Gotchas

- `getExpandedRowModel` is required for expanding to work client-side — without it, expanded state changes but rows don't appear
- `ExpandedState = true` means ALL rows expanded — not just root rows, all depths
- `getRowCanExpand` defaults to checking `row.subRows.length > 0`. Override for detail panels where no sub-rows exist
- `getIsRowExpanded` overrides the normal expanded check — useful for external expansion state
- Sub-rows share column definitions with parent rows — they render in the same columns
- `row.depth` starts at 0 for root rows. Use it for indentation: `paddingLeft: ${row.depth * 1.5}rem`
- `row.getIsAllParentsExpanded()` is useful for animations — a row should only be visible when all parents are expanded
- With `manualExpanding: true`, you handle the row model yourself — `getExpandedRowModel` is not called
