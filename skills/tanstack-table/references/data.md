# Data & Row Models

## Data Requirements

```tsx
const table = useReactTable({
  data, // TData[] - must be a stable reference!
  columns,
  getCoreRowModel: getCoreRowModel(), // REQUIRED
})
```

**Critical:** `data` must be a stable reference. Don't create inline arrays or fetch results directly:

```tsx
// ❌ BAD - new array every render, causes infinite reprocessing
<Table data={fetchResult?.items ?? []} />

// ✅ GOOD - stable reference
const data = React.useMemo(() => fetchResult?.items ?? [], [fetchResult])
// or
const [data, setData] = useState<Person[]>([])
```

## Row Model Pipeline

Row models execute in this order when features are enabled:

```
getCoreRowModel
  → getFilteredRowModel
    → getGroupedRowModel
      → getSortedRowModel
        → getExpandedRowModel
          → getPaginationRowModel
            → getRowModel (final)
```

If a feature is disabled or uses `manual*` option, that step is skipped (uses `getPre*RowModel` instead).

## Available Row Models

| Row Model | Import | Purpose |
|---|---|---|
| `getCoreRowModel` | `@tanstack/react-table` | **Required.** 1:1 mapping of source data |
| `getFilteredRowModel` | `@tanstack/react-table` | Client-side column + global filtering |
| `getGroupedRowModel` | `@tanstack/react-table` | Groups rows by column values, creates sub-rows |
| `getSortedRowModel` | `@tanstack/react-table` | Client-side sorting |
| `getExpandedRowModel` | `@tanstack/react-table` | Expands/collapses grouped or nested sub-rows |
| `getPaginationRowModel` | `@tanstack/react-table` | Client-side pagination |
| `getFacetedRowModel` | `@tanstack/react-table` | Faceted data for filter UIs |
| `getFacetedUniqueValues` | `@tanstack/react-table` | Unique values per column (for dropdowns) |
| `getFacetedMinMaxValues` | `@tanstack/react-table` | Min/max per column (for range sliders) |

## Usage Pattern

Only import what you need — each is tree-shakeable:

```tsx
import {
  getCoreRowModel,
  getSortedRowModel,
  getFilteredRowModel,
  getPaginationRowModel,
  useReactTable,
} from '@tanstack/react-table'

const table = useReactTable({
  data,
  columns,
  getCoreRowModel: getCoreRowModel(),
  getSortedRowModel: getSortedRowModel(),
  getFilteredRowModel: getFilteredRowModel(),
  getPaginationRowModel: getPaginationRowModel(),
})
```

## Accessing Row Models

```tsx
// For rendering — use this one (applies all enabled transformations)
table.getRowModel().rows

// Intermediate row models (for debugging or advanced use):
table.getCoreRowModel()           // raw data rows
table.getFilteredRowModel()       // after filtering
table.getPreFilteredRowModel()    // before filtering
table.getSortedRowModel()         // after sorting
table.getPreSortedRowModel()      // before sorting
table.getGroupedRowModel()        // after grouping
table.getExpandedRowModel()       // after expanding
table.getPaginationRowModel()     // after pagination
table.getPrePaginationRowModel()  // before pagination (all rows)
table.getSelectedRowModel()       // selected rows only
```

## Row Model Data Structure

Each row model provides three formats:

```tsx
const rowModel = table.getRowModel()
rowModel.rows       // Row<TData>[] - array of rows
rowModel.flatRows   // Row<TData>[] - all sub-rows flattened to top level
rowModel.rowsById   // Record<string, Row<TData>> - keyed by row id
```

## Row ID

```tsx
const table = useReactTable({
  data,
  columns,
  getRowId: (row) => row.id, // use database ID instead of array index
  getCoreRowModel: getCoreRowModel(),
})
```

Default row ID is the array index. Nested rows join with `.` (e.g., `0.1.2`).

## Sub-Rows (Nested Data)

```tsx
type Person = {
  name: string
  children?: Person[]
}

const table = useReactTable({
  data,
  columns,
  getSubRows: (row) => row.children, // access nested data
  getCoreRowModel: getCoreRowModel(),
  getExpandedRowModel: getExpandedRowModel(),
})
```

## Key Row Properties

```tsx
interface Row<TData> {
  id: string                    // unique row identifier
  index: number                 // row index within its parent
  original: TData               // original data object
  depth: number                 // nesting depth (0 = root)
  subRows: Row<TData>[]         // child rows
  parentId?: string             // parent row id
  getValue<TValue>(columnId: string): TValue  // get cell value
  getVisibleCells(): Cell<TData>[]
  getIsExpanded(): boolean
  getIsSelected(): boolean
  getCanSelect(): boolean
  toggleExpanded(): void
  toggleSelected(): void
}
```

## Expanding Sub-Rows

```tsx
import { getExpandedRowModel } from '@tanstack/react-table'

type Person = { name: string; age: number; children?: Person[] }

const [expanded, setExpanded] = useState<ExpandedState>({})
// ExpandedState = Record<string, boolean> | true (true = all expanded)

const table = useReactTable({
  data,
  columns,
  state: { expanded },
  onExpandedChange: setExpanded,
  getSubRows: row => row.children,       // how to find sub-rows
  getRowCanExpand: row => !!row.subRows.length, // default: checks getSubRows
  getCoreRowModel: getCoreRowModel(),
  getExpandedRowModel: getExpandedRowModel(),
})
```

## Table API Methods (Core)

```tsx
// Column access:
table.getAllColumns()           // Column[] — all columns including parents
table.getAllFlatColumns()       // Column[] — all columns flattened
table.getAllLeafColumns()       // Column[] — only leaf columns (no groups)
table.getColumn(id)            // Column | undefined

// Row access:
table.getRow(rowId)            // Row — get specific row by ID
table.getRowModel()            // RowModel — final processed rows

// Header/Footer groups:
table.getHeaderGroups()        // HeaderGroup[] — for thead
table.getFooterGroups()        // HeaderGroup[] — for tfoot
table.getLeftHeaderGroups()    // HeaderGroup[] — left pinned
table.getCenterHeaderGroups()  // HeaderGroup[] — center unpinned
table.getRightHeaderGroups()   // HeaderGroup[] — right pinned

// Flat headers (includes parent headers):
table.getFlatHeaders()         // Header[]
table.getLeftFlatHeaders()     // Header[]
table.getCenterFlatHeaders()   // Header[]
table.getRightFlatHeaders()    // Header[]

// Size:
table.getTotalSize()           // total width of all columns
table.getLeftTotalSize()       // width of left pinned columns
table.getCenterTotalSize()     // width of center columns
table.getRightTotalSize()      // width of right pinned columns

// Visibility:
table.getIsAllColumnsVisible()   // boolean
table.getIsSomeColumnsVisible()  // boolean
table.getVisibleLeafColumns()    // Column[] — only visible leaf columns
table.getVisibleFlatColumns()    // Column[] — only visible flat columns
table.getToggleAllColumnsVisibilityHandler() // checkbox event handler

// State:
table.getState()               // full table state
table.setState(updater)        // update full state
table.reset()                  // reset all state to initialState
```

## Key Cell Properties

```tsx
interface Cell<TData> {
  id: string                    // `${rowId}_${columnId}`
  getValue<TValue>(): TValue    // cell value from accessor
  row: Row<TData>
  column: Column<TData>
  getContext(): CellContext      // pass to flexRender
  getIsGrouped(): boolean
  getIsPlaceholder(): boolean
  getIsAggregated(): boolean
}
```
