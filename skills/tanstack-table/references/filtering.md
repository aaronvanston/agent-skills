# Filtering

Two types: **Column Filtering** (per-column) and **Global Filtering** (across all columns).

## Column Filtering Setup

```tsx
import {
  getCoreRowModel,
  getFilteredRowModel,
  useReactTable,
  ColumnFiltersState,
} from '@tanstack/react-table'

const [columnFilters, setColumnFilters] = useState<ColumnFiltersState>([])

const table = useReactTable({
  data,
  columns,
  state: { columnFilters },
  onColumnFiltersChange: setColumnFilters,
  getCoreRowModel: getCoreRowModel(),
  getFilteredRowModel: getFilteredRowModel(), // required for client-side filtering
})
```

## Column Filter State

```tsx
interface ColumnFilter {
  id: string       // column id
  value: unknown   // filter value (type depends on your filter fn)
}
type ColumnFiltersState = ColumnFilter[]
```

## Global Filtering Setup

```tsx
const [globalFilter, setGlobalFilter] = useState('')

const table = useReactTable({
  data,
  columns,
  state: { globalFilter },
  onGlobalFilterChange: setGlobalFilter,
  getCoreRowModel: getCoreRowModel(),
  getFilteredRowModel: getFilteredRowModel(),
  globalFilterFn: 'includesString', // or custom function
})

// Search input:
<input
  value={globalFilter ?? ''}
  onChange={e => setGlobalFilter(e.target.value)}
  placeholder="Search all columns..."
/>
```

## Server-Side (Manual) Filtering

```tsx
const table = useReactTable({
  data, // already filtered by server
  columns,
  state: { columnFilters },
  onColumnFiltersChange: setColumnFilters,
  getCoreRowModel: getCoreRowModel(),
  manualFiltering: true, // skip client-side filtering
})
```

## Built-in Filter Functions

| Name | Description |
|------|-------------|
| `includesString` | Case-insensitive string inclusion |
| `includesStringSensitive` | Case-sensitive string inclusion |
| `equalsString` | Case-insensitive string equality |
| `equalsStringSensitive` | Case-sensitive string equality |
| `arrIncludes` | Item in array |
| `arrIncludesAll` | All items in array |
| `arrIncludesSome` | Some items in array |
| `equals` | `Object.is` / `===` equality |
| `weakEquals` | `==` equality |
| `inNumberRange` | Number range `[min, max]` |

## Per-Column Filter Function

```tsx
columnHelper.accessor('name', {
  filterFn: 'includesString',  // built-in
})

columnHelper.accessor('age', {
  filterFn: 'inNumberRange',
})

columnHelper.accessor('status', {
  filterFn: (row, columnId, filterValue) => {
    // return true to include row, false to exclude
    return row.getValue<string>(columnId) === filterValue
  },
})
```

## Custom Filter Functions

```tsx
// Register custom filter functions:
const table = useReactTable({
  filterFns: {
    myFilter: (row, columnId, filterValue, addMeta) => {
      return row.getValue<string>(columnId).toLowerCase().startsWith(filterValue.toLowerCase())
    },
  },
})

// FilterFn signature:
type FilterFn<TData> = (row: Row<TData>, columnId: string, filterValue: any, addMeta: (meta: any) => void) => boolean
```

## Filtering Options

```tsx
// Table-level:
const table = useReactTable({
  enableFilters: true,              // default (global kill switch)
  enableColumnFilters: true,        // default
  enableGlobalFilter: true,         // default
  filterFromLeafRows: false,        // filter from leaf rows up (for nested)
  maxLeafRowFilterDepth: undefined, // limit filter depth for nested rows
})

// Column-level:
columnHelper.accessor('name', {
  enableColumnFilter: true,  // default
  enableGlobalFilter: true,  // include in global filter (default: true)
  filterFn: 'includesString',
})
```

## Column Filter APIs

```tsx
// Set filter value:
column.setFilterValue('search term')
column.setFilterValue([min, max])     // for range filters
column.setFilterValue(prev => newVal) // updater function

// Get filter value:
column.getFilterValue()  // current filter value
column.getIsFiltered()   // boolean
column.getCanFilter()    // boolean

// Table-level:
table.setColumnFilters(updater)
table.resetColumnFilters()
table.setGlobalFilter(value)
table.resetGlobalFilter()
```

## Column Faceting (for Filter UIs)

Column faceting generates per-column value lists for building filter dropdowns and range sliders.

```tsx
import {
  getFacetedRowModel,
  getFacetedUniqueValues,
  getFacetedMinMaxValues,
} from '@tanstack/react-table'

const table = useReactTable({
  // ...
  getFacetedRowModel: getFacetedRowModel(),       // base faceted model (required)
  getFacetedUniqueValues: getFacetedUniqueValues(), // unique values per column
  getFacetedMinMaxValues: getFacetedMinMaxValues(), // min/max per column
})

// Column APIs:
const uniqueValues = column.getFacetedUniqueValues() // Map<any, number> (value → count)
const [min, max] = column.getFacetedMinMaxValues() ?? [0, 0]
const rowCount = column.getFacetedRowModel().rows.length
```

### Filter Dropdown Example

```tsx
function FilterDropdown({ column }: { column: Column<any> }) {
  const uniqueValues = column.getFacetedUniqueValues()
  const sortedValues = Array.from(uniqueValues.keys()).sort()

  return (
    <select onChange={e => column.setFilterValue(e.target.value || undefined)}>
      <option value="">All</option>
      {sortedValues.map(value => (
        <option key={value} value={value}>
          {value} ({uniqueValues.get(value)})
        </option>
      ))}
    </select>
  )
}
```

### Range Filter Example

```tsx
function RangeFilter({ column }: { column: Column<any> }) {
  const [min, max] = column.getFacetedMinMaxValues() ?? [0, 100]
  return (
    <div>
      <input type="number" min={min} max={max}
        onChange={e => column.setFilterValue((old: [number, number]) => [+e.target.value, old?.[1]])} />
      <input type="number" min={min} max={max}
        onChange={e => column.setFilterValue((old: [number, number]) => [old?.[0], +e.target.value])} />
    </div>
  )
}
```

### Server-Side Faceting

Override faceted methods with server data instead of using client-side row models. Or skip TanStack faceting APIs entirely and pass server data directly to filter components.

## Filter Input Pattern

```tsx
function ColumnFilter({ column }: { column: Column<any> }) {
  const filterValue = column.getFilterValue()

  return (
    <input
      value={(filterValue ?? '') as string}
      onChange={e => column.setFilterValue(e.target.value)}
      placeholder={`Filter ${column.id}...`}
    />
  )
}
```

## Fuzzy Filtering (match-sorter-utils)

See [fuzzy-filtering](fuzzy-filtering.md) for full setup with `@tanstack/match-sorter-utils`.

## Global Faceting

Global faceting generates value lists from ALL columns (vs column faceting which is per-column).
Useful for global search autocomplete suggestions.

```tsx
import {
  getFacetedRowModel,
  getFacetedUniqueValues,
  getFacetedMinMaxValues,
} from '@tanstack/react-table'

const table = useReactTable({
  // ...
  getFacetedRowModel: getFacetedRowModel(),       // required base for faceting
  getFacetedUniqueValues: getFacetedUniqueValues(), // depends on getFacetedRowModel
  getFacetedMinMaxValues: getFacetedMinMaxValues(), // depends on getFacetedRowModel
})

// Global unique values across all columns:
const globalUniqueValues = table.getGlobalFacetedUniqueValues() // Map<any, number>

// Global min/max:
const [min, max] = table.getGlobalFacetedMinMaxValues() ?? [0, 1]

// Autocomplete suggestions:
const suggestions = Array.from(table.getGlobalFacetedUniqueValues().keys())
  .sort()
  .slice(0, 5000)
```

### Server-Side Global Faceting

```tsx
// Override faceting methods with server data:
table.getGlobalFacetedUniqueValues = () => serverData.uniqueValues
table.getGlobalFacetedMinMaxValues = () => serverData.minMaxValues
```

### Column vs Global Faceting

| | Column Faceting | Global Faceting |
|---|---|---|
| Scope | Per-column values | All columns combined |
| Access | `column.getFacetedUniqueValues()` | `table.getGlobalFacetedUniqueValues()` |
| Use case | Column filter dropdowns | Global search autocomplete |
| Row model | Same `getFacetedRowModel` | Same `getFacetedRowModel` |

## Leaf Row Filtering (for nested/expanded rows)

```tsx
const table = useReactTable({
  // ...
  filterFromLeafRows: true,         // filter from leaf rows up
  maxLeafRowFilterDepth: 0,         // 0 = only filter root rows, preserve all sub-rows
  // maxLeafRowFilterDepth: 1,      // filter root + 1 level deep
})
```
