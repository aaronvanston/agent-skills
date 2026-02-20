# Grouping

## Setup

```tsx
import {
  getCoreRowModel,
  getGroupedRowModel,
  getExpandedRowModel,
  useReactTable,
  GroupingState,
  ExpandedState,
} from '@tanstack/react-table'

const [grouping, setGrouping] = useState<GroupingState>([])
const [expanded, setExpanded] = useState<ExpandedState>({})

const table = useReactTable({
  data,
  columns,
  state: { grouping, expanded },
  onGroupingChange: setGrouping,
  onExpandedChange: setExpanded,
  getCoreRowModel: getCoreRowModel(),
  getGroupedRowModel: getGroupedRowModel(),
  getExpandedRowModel: getExpandedRowModel(), // needed to expand/collapse groups
})
```

## Grouping State

```tsx
type GroupingState = string[] // array of column IDs to group by

type GroupingTableState = {
  grouping: GroupingState
}

// Order matters: ['status', 'department'] groups by status first, then department within
table.setGrouping(['status', 'department'])
table.resetGrouping()
```

## Table Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `enableGrouping` | `boolean` | `true` | Enable/disable grouping for all columns |
| `manualGrouping` | `boolean` | `false` | Skip client-side grouping, data already grouped by server |
| `groupedColumnMode` | `false \| 'reorder' \| 'remove'` | `'reorder'` | How to handle grouped columns in display |
| `onGroupingChange` | `OnChangeFn<GroupingState>` | — | Controlled state handler |
| `getGroupedRowModel` | `(table) => () => RowModel<TData>` | — | Row model function for client-side grouping |
| `aggregationFns` | `Record<string, AggregationFn>` | — | Custom aggregation functions registry |

### groupedColumnMode Values

| Value | Behavior |
|-------|----------|
| `'reorder'` | Move grouped columns to start of column list (default) |
| `'remove'` | Remove grouped columns from display entirely |
| `false` | Don't move or remove grouped columns |

## Column Def Options

| Option | Type | Description |
|--------|------|-------------|
| `enableGrouping` | `boolean` | Enable/disable grouping for this column |
| `aggregationFn` | `AggregationFn \| BuiltInAggregationFn \| string` | Aggregation function for this column |
| `aggregatedCell` | `Renderable` | Cell renderer for aggregated (group header) rows |
| `getGroupingValue` | `(row: TData) => any` | Custom value for grouping (default: accessor value) |

## Aggregation Functions

### Built-in Aggregation Functions

| Name | Description |
|------|-------------|
| `sum` | Sum of values |
| `count` | Row count |
| `min` | Minimum value |
| `max` | Maximum value |
| `extent` | `[min, max]` tuple |
| `mean` | Average |
| `median` | Median value |
| `unique` | Array of unique values |
| `uniqueCount` | Count of unique values |

### Aggregation Function Signature

```tsx
type AggregationFn<TData> = (
  getLeafRows: () => Row<TData>[],  // all leaf (non-grouped) rows in this group
  getChildRows: () => Row<TData>[]  // immediate child rows of this group
) => any
```

### Custom Aggregation

```tsx
// Register with type safety:
declare module '@tanstack/table-core' {
  interface AggregationFns {
    weightedAvg: AggregationFn<unknown>
  }
}

const table = useReactTable({
  // ...
  aggregationFns: {
    weightedAvg: (getLeafRows, getChildRows) => {
      const leafRows = getLeafRows()
      const sum = leafRows.reduce((acc, row) => acc + row.getValue<number>('score'), 0)
      return sum / leafRows.length
    },
  },
})

columnHelper.accessor('score', {
  aggregationFn: 'weightedAvg',
})
```

### Per-Column Aggregation

```tsx
columnHelper.accessor('amount', {
  aggregationFn: 'sum',
  cell: ({ getValue }) => `$${getValue()}`,
  aggregatedCell: ({ getValue }) => `Total: $${getValue()}`,
})

columnHelper.accessor('status', {
  aggregationFn: 'uniqueCount',
  aggregatedCell: ({ getValue }) => `${getValue()} statuses`,
})
```

## Column APIs

| Method | Signature | Description |
|--------|-----------|-------------|
| `getCanGroup` | `() => boolean` | Can this column be grouped? |
| `getIsGrouped` | `() => boolean` | Is this column currently grouped? |
| `getGroupedIndex` | `() => number` | Index in grouping state |
| `toggleGrouping` | `() => void` | Toggle this column's grouping |
| `getToggleGroupingHandler` | `() => () => void` | Event handler for toggle |
| `getAutoAggregationFn` | `() => AggregationFn<TData> \| undefined` | Auto-inferred aggregation fn |
| `getAggregationFn` | `() => AggregationFn<TData> \| undefined` | Resolved aggregation fn |

## Row APIs

| Property/Method | Type | Description |
|--------|------|-------------|
| `groupingColumnId` | `string \| undefined` | Column ID this row is grouped by |
| `groupingValue` | `any` | Shared value for the grouping column |
| `getIsGrouped` | `() => boolean` | Is this a group row? |
| `getGroupingValue` | `(columnId: string) => unknown` | Grouping value for any column |

## Cell APIs

| Method | Signature | Description |
|--------|-----------|-------------|
| `getIsGrouped` | `() => boolean` | Is this the grouped cell (shows group value)? |
| `getIsAggregated` | `() => boolean` | Is this an aggregated cell? |
| `getIsPlaceholder` | `() => boolean` | Is this a placeholder (empty in grouped row)? |

## Table APIs

| Method | Signature | Description |
|--------|-----------|-------------|
| `setGrouping` | `(updater: Updater<GroupingState>) => void` | Set grouping state |
| `resetGrouping` | `(defaultState?: boolean) => void` | Reset to initial. `true` = `[]` |
| `getPreGroupedRowModel` | `() => RowModel<TData>` | Rows before grouping |
| `getGroupedRowModel` | `() => RowModel<TData>` | Rows after grouping |

## Rendering Grouped Rows

```tsx
{table.getRowModel().rows.map(row => (
  <tr key={row.id}>
    {row.getVisibleCells().map(cell => (
      <td key={cell.id}>
        {cell.getIsGrouped() ? (
          // Grouped cell — render expander + value
          <button onClick={row.getToggleExpandedHandler()}>
            {row.getIsExpanded() ? '▼' : '▶'}
            {' '}
            {flexRender(cell.column.columnDef.cell, cell.getContext())}
            {' '}({row.subRows.length})
          </button>
        ) : cell.getIsAggregated() ? (
          // Aggregated cell — render aggregated value
          flexRender(
            cell.column.columnDef.aggregatedCell ?? cell.column.columnDef.cell,
            cell.getContext()
          )
        ) : cell.getIsPlaceholder() ? null : (
          // Regular cell
          flexRender(cell.column.columnDef.cell, cell.getContext())
        )}
      </td>
    ))}
  </tr>
))}
```

## Grouping Toggle UI

```tsx
{table.getAllLeafColumns().map(column => (
  <button
    key={column.id}
    onClick={column.getToggleGroupingHandler()}
    disabled={!column.getCanGroup()}
    style={{ fontWeight: column.getIsGrouped() ? 'bold' : 'normal' }}
  >
    {column.getIsGrouped() ? `🔲 ${column.id} (${column.getGroupedIndex()})` : `☐ ${column.id}`}
  </button>
))}
```

## Manual Server-Side Grouping

```tsx
const table = useReactTable({
  // ...
  manualGrouping: true, // data is already grouped by server
  // getGroupedRowModel not needed
})
```

## Expanding APIs (for grouped rows)

Grouping creates parent/child row relationships that use the expanding feature:

```tsx
row.getIsExpanded()             // boolean
row.getCanExpand()              // has sub-rows
row.toggleExpanded()
row.getToggleExpandedHandler()  // event handler

table.setExpanded(updater)
table.resetExpanded()
table.toggleAllRowsExpanded()
table.getIsAllRowsExpanded()    // boolean
table.getExpandedDepth()        // max expanded depth
```

## Multi-Level Grouping

```tsx
// Group by status, then by department within each status
table.setGrouping(['status', 'department'])

// The resulting row model is a tree:
// ▶ Active (status group)
//   ▶ Engineering (department group)
//     - Row 1
//     - Row 2
//   ▶ Design (department group)
//     - Row 3
// ▶ Inactive (status group)
//   ...
```

## Edge Cases & Gotchas

- **Must include `getExpandedRowModel`** — without it, grouped rows can't be expanded to show children
- Grouped column values come from `accessorKey`/`accessorFn` by default — use `getGroupingValue` to customize
- `aggregatedCell` is optional — falls back to `cell` if not provided
- `cell.getIsPlaceholder()` returns true for non-grouped, non-aggregated cells in grouped rows — render `null` for clean display
- Grouping interacts with sorting: sorted groups maintain sort order within each group
- With pagination: expanded groups count toward page size (controlled by `paginateExpandedRows`)
- `groupedColumnMode: 'reorder'` physically moves columns — this can surprise users if column order matters
