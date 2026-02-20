# Fuzzy Filtering

Approximate matching using `@tanstack/match-sorter-utils` (fork of `match-sorter` optimized for row-by-row filtering). Mostly used with global filtering, but can also apply to individual columns.

## Install

```bash
npm install @tanstack/match-sorter-utils
```

## RankingInfo Type

```tsx
import { RankingInfo } from '@tanstack/match-sorter-utils'

// RankingInfo shape returned by rankItem:
interface RankingInfo {
  rankedValue: string      // the value that was ranked
  rank: number             // numeric rank (higher = better match)
  passed: boolean          // whether the item passed the ranking threshold
  accessorIndex: number    // index of the accessor that matched
  accessorThreshold: number // threshold that was used
  keyThreshold: number     // threshold for the matched key
}
```

## Ranking Thresholds

`rankItem` uses these ranking thresholds (from best to worst match):

| Ranking | Value | Description |
|---------|-------|-------------|
| CASE_SENSITIVE_EQUAL | 7 | Exact case-sensitive match |
| EQUAL | 6 | Exact case-insensitive match |
| STARTS_WITH | 5 | Value starts with search term |
| WORD_STARTS_WITH | 4 | A word in the value starts with search term |
| CONTAINS | 3 | Value contains the search term |
| ACRONYM | 2 | Search matches as an acronym |
| MATCHES | 1 | Letters match in order (fuzzy) |
| NO_MATCH | 0 | No match found |

## Define Fuzzy Filter & Sort Functions

```tsx
import { rankItem, compareItems, RankingInfo } from '@tanstack/match-sorter-utils'
import {
  FilterFn,
  SortingFn,
  sortingFns,
  ColumnFiltersState,
} from '@tanstack/react-table'

// Extend types for fuzzy filter
declare module '@tanstack/react-table' {
  interface FilterFns {
    fuzzy: FilterFn<unknown>
  }
  interface FilterMeta {
    itemRank: RankingInfo
  }
}

// Fuzzy filter — ranks items and stores rank metadata
const fuzzyFilter: FilterFn<any> = (row, columnId, value, addMeta) => {
  const itemRank = rankItem(row.getValue(columnId), value)
  addMeta({ itemRank })
  return itemRank.passed
}

// Fuzzy sort — sort by rank, fallback to alphanumeric
const fuzzySort: SortingFn<any> = (rowA, rowB, columnId) => {
  let dir = 0
  if (rowA.columnFiltersMeta[columnId]) {
    dir = compareItems(
      rowA.columnFiltersMeta[columnId]?.itemRank!,
      rowB.columnFiltersMeta[columnId]?.itemRank!,
    )
  }
  return dir === 0 ? sortingFns.alphanumeric(rowA, rowB, columnId) : dir
}
```

## Use with Global Filtering (most common)

```tsx
const [columnFilters, setColumnFilters] = useState<ColumnFiltersState>([])
const [globalFilter, setGlobalFilter] = useState('')

const table = useReactTable({
  data,
  columns,
  filterFns: { fuzzy: fuzzyFilter },
  state: { columnFilters, globalFilter },
  onColumnFiltersChange: setColumnFilters,
  onGlobalFilterChange: setGlobalFilter,
  globalFilterFn: 'fuzzy', // apply fuzzy to global filter
  getCoreRowModel: getCoreRowModel(),
  getFilteredRowModel: getFilteredRowModel(),
  getSortedRowModel: getSortedRowModel(),
})
```

## Use with Column Filtering

```tsx
columnHelper.accessor(row => `${row.firstName} ${row.lastName}`, {
  id: 'fullName',
  header: 'Full Name',
  filterFn: 'fuzzy',      // use fuzzy filter on this column
  sortingFn: fuzzySort,    // sort by rank when filtered
})
```

## Mixing Filter Types Per Column

Different columns can use different filter functions — fuzzy on some, exact on others:

```tsx
const columns: ColumnDef<Person, any>[] = [
  {
    accessorKey: 'id',
    filterFn: 'equalsString',              // exact match
  },
  {
    accessorKey: 'firstName',
    filterFn: 'includesStringSensitive',    // case-sensitive substring
  },
  {
    accessorFn: row => row.lastName,
    id: 'lastName',
    filterFn: 'includesString',            // case-insensitive substring
  },
  {
    accessorFn: row => `${row.firstName} ${row.lastName}`,
    id: 'fullName',
    filterFn: 'fuzzy',                     // fuzzy matching
    sortingFn: fuzzySort,                  // sort by rank
  },
]
```

## Auto-Apply Fuzzy Sort When Filtering

When a fuzzy-filtered column is active, auto-sort by rank for best UX:

```tsx
useEffect(() => {
  if (table.getState().columnFilters[0]?.id === 'fullName') {
    if (table.getState().sorting[0]?.id !== 'fullName') {
      table.setSorting([{ id: 'fullName', desc: false }])
    }
  }
}, [table.getState().columnFilters[0]?.id])
```

## Debounced Search Input

Fuzzy filtering on large datasets benefits from debouncing:

```tsx
function DebouncedInput({
  value: initialValue,
  onChange,
  debounce = 300,
  ...props
}: {
  value: string
  onChange: (value: string) => void
  debounce?: number
} & Omit<React.InputHTMLAttributes<HTMLInputElement>, 'onChange'>) {
  const [value, setValue] = useState(initialValue)

  useEffect(() => {
    setValue(initialValue)
  }, [initialValue])

  useEffect(() => {
    const timeout = setTimeout(() => onChange(value), debounce)
    return () => clearTimeout(timeout)
  }, [value])

  return (
    <input
      {...props}
      value={value}
      onChange={e => setValue(e.target.value)}
    />
  )
}

// Usage:
<DebouncedInput
  value={globalFilter ?? ''}
  onChange={value => setGlobalFilter(String(value))}
  placeholder="Search all columns..."
/>
```

## rankItem Options

You can customize ranking behavior:

```tsx
import { rankItem, rankings } from '@tanstack/match-sorter-utils'

const itemRank = rankItem(row.getValue(columnId), value, {
  threshold: rankings.MATCHES, // default — most permissive fuzzy
  // threshold: rankings.CONTAINS,  // stricter — must contain the term
  // threshold: rankings.WORD_STARTS_WITH, // even stricter
})
```

## Complete Full Example

```tsx
import {
  useReactTable,
  getCoreRowModel,
  getFilteredRowModel,
  getSortedRowModel,
  getPaginationRowModel,
  flexRender,
  ColumnDef,
  ColumnFiltersState,
} from '@tanstack/react-table'
import { rankItem, compareItems, RankingInfo } from '@tanstack/match-sorter-utils'

declare module '@tanstack/react-table' {
  interface FilterFns { fuzzy: FilterFn<unknown> }
  interface FilterMeta { itemRank: RankingInfo }
}

const fuzzyFilter: FilterFn<any> = (row, columnId, value, addMeta) => {
  const itemRank = rankItem(row.getValue(columnId), value)
  addMeta({ itemRank })
  return itemRank.passed
}

const fuzzySort: SortingFn<any> = (rowA, rowB, columnId) => {
  let dir = 0
  if (rowA.columnFiltersMeta[columnId]) {
    dir = compareItems(
      rowA.columnFiltersMeta[columnId]?.itemRank!,
      rowB.columnFiltersMeta[columnId]?.itemRank!,
    )
  }
  return dir === 0 ? sortingFns.alphanumeric(rowA, rowB, columnId) : dir
}

function FuzzyTable({ data }: { data: Person[] }) {
  const [columnFilters, setColumnFilters] = useState<ColumnFiltersState>([])
  const [globalFilter, setGlobalFilter] = useState('')

  const columns = useMemo<ColumnDef<Person, any>[]>(() => [
    { accessorKey: 'firstName', filterFn: 'includesString' },
    { accessorKey: 'lastName', filterFn: 'includesString' },
    {
      accessorFn: row => `${row.firstName} ${row.lastName}`,
      id: 'fullName',
      header: 'Full Name',
      filterFn: 'fuzzy',
      sortingFn: fuzzySort,
    },
    { accessorKey: 'age' },
  ], [])

  const table = useReactTable({
    data,
    columns,
    filterFns: { fuzzy: fuzzyFilter },
    state: { columnFilters, globalFilter },
    onColumnFiltersChange: setColumnFilters,
    onGlobalFilterChange: setGlobalFilter,
    globalFilterFn: 'fuzzy',
    getCoreRowModel: getCoreRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
  })

  return (
    <div>
      <DebouncedInput
        value={globalFilter ?? ''}
        onChange={value => setGlobalFilter(String(value))}
        placeholder="Search all columns..."
      />
      <table>
        <thead>
          {table.getHeaderGroups().map(hg => (
            <tr key={hg.id}>
              {hg.headers.map(h => (
                <th key={h.id}>
                  {flexRender(h.column.columnDef.header, h.getContext())}
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
      </table>
    </div>
  )
}
```

## Edge Cases & Gotchas

- **`match-sorter-utils` is required** — it's a separate package, not bundled with TanStack Table
- **Global filter applies to ALL columns** by default. Use `enableGlobalFilter: false` on columns to exclude them
- **Fuzzy sort only works when column has ranking info** — the `columnFiltersMeta` is only populated when that column's filter runs. For global filter, metadata is per-column
- **Performance**: `rankItem` is fast but on 50k+ rows with many columns, debounce the input (300ms+)
- **`sortingFn` vs `sortFn`**: use `sortingFn` (not `sortFn`) in column definitions — `sortFn` is incorrect
- **Multiple fuzzy columns**: each column independently ranks; global filter ranks across all columns
- **Empty string filter**: passing `''` to global filter effectively clears it (all rows pass)
