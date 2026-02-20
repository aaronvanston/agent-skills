# Pagination

## Client-Side Pagination

```tsx
import {
  getCoreRowModel,
  getPaginationRowModel,
  useReactTable,
  PaginationState,
} from '@tanstack/react-table'

const [pagination, setPagination] = useState<PaginationState>({
  pageIndex: 0,
  pageSize: 10,
})

const table = useReactTable({
  data, // ALL rows — table handles slicing
  columns,
  state: { pagination },
  onPaginationChange: setPagination,
  getCoreRowModel: getCoreRowModel(),
  getPaginationRowModel: getPaginationRowModel(),
})
```

## Uncontrolled (Initial State Only)

```tsx
const table = useReactTable({
  data,
  columns,
  getCoreRowModel: getCoreRowModel(),
  getPaginationRowModel: getPaginationRowModel(),
  initialState: {
    pagination: { pageIndex: 0, pageSize: 25 },
  },
})
```

## Server-Side (Manual) Pagination

```tsx
const [pagination, setPagination] = useState<PaginationState>({
  pageIndex: 0,
  pageSize: 10,
})

const table = useReactTable({
  data, // only current page data from server
  columns,
  state: { pagination },
  onPaginationChange: setPagination,
  getCoreRowModel: getCoreRowModel(),
  manualPagination: true,
  rowCount: totalRowCount,    // tell table total rows for page calculations
  // OR: pageCount: totalPages, // if you know page count directly
  // If unknown: pageCount: -1  (getCanNextPage always returns true)
})

// Use pagination state to fetch:
useEffect(() => {
  fetchData({ page: pagination.pageIndex, size: pagination.pageSize })
}, [pagination])
```

## Pagination State

```tsx
interface PaginationState {
  pageIndex: number  // zero-based
  pageSize: number
}

type PaginationTableState = {
  pagination: PaginationState
}

type PaginationInitialTableState = {
  pagination?: Partial<PaginationState>  // both fields optional in initialState
}
```

## Table Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `manualPagination` | `boolean` | `false` | Skip client-side pagination, handle externally |
| `pageCount` | `number` | — | Total pages (for manual pagination). `-1` if unknown |
| `rowCount` | `number` | — | Total rows (for manual pagination). `pageCount` calculated from this |
| `autoResetPageIndex` | `boolean` | `true` (`false` if `manualPagination`) | Reset to page 0 when data/filters/sorting change |
| `onPaginationChange` | `OnChangeFn<PaginationState>` | — | Controlled state handler |
| `getPaginationRowModel` | `(table) => () => RowModel<TData>` | — | Row model function for client-side pagination |

## Table APIs

| Method | Signature | Description |
|--------|-----------|-------------|
| `setPageIndex` | `(updater: Updater<number>) => void` | Go to specific page |
| `resetPageIndex` | `(defaultState?: boolean) => void` | Reset page index. `true` = reset to 0 |
| `setPageSize` | `(updater: Updater<number>) => void` | Change page size |
| `resetPageSize` | `(defaultState?: boolean) => void` | Reset page size. `true` = reset to 10 |
| `setPagination` | `(updater: Updater<PaginationState>) => void` | Set full pagination state |
| `resetPagination` | `(defaultState?: boolean) => void` | Reset to initialState. `true` = `{pageIndex:0, pageSize:10}` |
| `previousPage` | `() => void` | Go to previous page (if possible) |
| `nextPage` | `() => void` | Go to next page (if possible) |
| `firstPage` | `() => void` | Go to first page (page 0) |
| `lastPage` | `() => void` | Go to last page |
| `getCanPreviousPage` | `() => boolean` | Can go back? |
| `getCanNextPage` | `() => boolean` | Can go forward? |
| `getPageCount` | `() => number` | Total number of pages |
| `getPageOptions` | `() => number[]` | Array of page indices `[0, 1, 2, ...]` |
| `getPrePaginationRowModel` | `() => RowModel<TData>` | All rows before pagination |
| `getPaginationRowModel` | `() => RowModel<TData>` | Current page rows only |

## Pagination UI Pattern

```tsx
<div className="pagination">
  <button onClick={() => table.firstPage()} disabled={!table.getCanPreviousPage()}>
    {'<<'}
  </button>
  <button onClick={() => table.previousPage()} disabled={!table.getCanPreviousPage()}>
    {'<'}
  </button>
  <span>
    Page {table.getState().pagination.pageIndex + 1} of {table.getPageCount()}
  </span>
  <button onClick={() => table.nextPage()} disabled={!table.getCanNextPage()}>
    {'>'}
  </button>
  <button onClick={() => table.lastPage()} disabled={!table.getCanNextPage()}>
    {'>>'}
  </button>
  <select
    value={table.getState().pagination.pageSize}
    onChange={e => table.setPageSize(Number(e.target.value))}
  >
    {[10, 20, 50, 100].map(size => (
      <option key={size} value={size}>Show {size}</option>
    ))}
  </select>
</div>
```

## Go-To-Page Input

```tsx
<span>
  Go to page:
  <input
    type="number"
    min={1}
    max={table.getPageCount()}
    defaultValue={table.getState().pagination.pageIndex + 1}
    onChange={e => {
      const page = e.target.value ? Number(e.target.value) - 1 : 0
      table.setPageIndex(page)
    }}
  />
</span>
```

## Page Number Buttons

```tsx
function PageButtons({ table }: { table: Table<any> }) {
  const pageCount = table.getPageCount()
  const currentPage = table.getState().pagination.pageIndex

  return (
    <div>
      {table.getPageOptions().map(pageIndex => (
        <button
          key={pageIndex}
          onClick={() => table.setPageIndex(pageIndex)}
          className={pageIndex === currentPage ? 'active' : ''}
        >
          {pageIndex + 1}
        </button>
      ))}
    </div>
  )
}
```

## Row Count Display

```tsx
<div>
  Showing {table.getRowModel().rows.length} of{' '}
  {table.getPrePaginationRowModel().rows.length} rows
  {' '}({table.getPageCount()} pages)
</div>
```

## Programmatic Navigation

```tsx
// Jump to page 5:
table.setPageIndex(4) // zero-based

// Conditional jump:
table.setPageIndex(old => Math.min(old + 5, table.getPageCount() - 1))

// Change page size and reset to first page:
table.setPageSize(50) // autoResetPageIndex will handle resetting to page 0
```

## Pagination with Other Features

### With Sorting/Filtering

By default, changing sort/filter resets to page 0 (`autoResetPageIndex: true`). Set to `false` to preserve the current page:

```tsx
const table = useReactTable({
  // ...
  autoResetPageIndex: false, // keep current page when sorting/filtering changes
})
```

### With Expanding

```tsx
const table = useReactTable({
  // ...
  paginateExpandedRows: true,  // default — expanded rows count toward page size
  // paginateExpandedRows: false, // expanded rows don't affect pagination
})
```

### With Row Selection

- `getSelectedRowModel()` only returns selected rows from **current page data**
- The `rowSelection` state stores all selections across pages
- Use `getFilteredSelectedRowModel()` for accurate counts

## Performance Note

Client-side pagination handles 100k+ rows well with few columns. Consider virtualization as an alternative to pagination for infinite-scroll UIs. Be consistent: don't mix client-side pagination with server-side filtering/sorting — it will only paginate the loaded subset.

## Edge Cases & Gotchas

- `pageCount: -1` means "unknown number of pages" — `getCanNextPage()` always returns `true`
- `rowCount` and `pageCount` are alternatives — provide one or the other, not both
- Default page size is 10 if not specified in `initialState` or `state`
- `setPageIndex` accepts an `Updater<number>` — can be a function: `table.setPageIndex(old => old + 1)`
- With `manualPagination`, `autoResetPageIndex` defaults to `false`
- `getPageOptions()` returns `[]` when `pageCount` is -1 (unknown)
- Changing `pageSize` may put you on a page that no longer exists — the table handles this by clamping to the last valid page
