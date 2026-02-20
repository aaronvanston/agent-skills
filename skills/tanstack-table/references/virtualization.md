# Virtualization

Virtualization renders only visible rows/columns in the viewport, enabling large datasets without pagination. Uses **@tanstack/react-virtual** alongside TanStack Table.

## Install

```bash
npm install @tanstack/react-virtual
```

## Row Virtualization

```tsx
import { useReactTable, getCoreRowModel, getSortedRowModel, flexRender } from '@tanstack/react-table'
import { useVirtualizer } from '@tanstack/react-virtual'

function VirtualTable({ data, columns }: Props) {
  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
    getSortedRowModel: getSortedRowModel(),
  })

  const { rows } = table.getRowModel()

  const parentRef = React.useRef<HTMLDivElement>(null)

  const virtualizer = useVirtualizer({
    count: rows.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 35, // estimated row height in px
    overscan: 10,           // render extra rows outside viewport
  })

  return (
    <div ref={parentRef} style={{ height: '600px', overflow: 'auto' }}>
      <table style={{ width: '100%' }}>
        <thead>
          {table.getHeaderGroups().map(headerGroup => (
            <tr key={headerGroup.id}>
              {headerGroup.headers.map(header => (
                <th key={header.id}>
                  {flexRender(header.column.columnDef.header, header.getContext())}
                </th>
              ))}
            </tr>
          ))}
        </thead>
        <tbody
          style={{
            height: `${virtualizer.getTotalSize()}px`,
            position: 'relative',
          }}
        >
          {virtualizer.getVirtualItems().map(virtualRow => {
            const row = rows[virtualRow.index]
            return (
              <tr
                key={row.id}
                data-index={virtualRow.index}
                ref={node => virtualizer.measureElement(node)} // dynamic measurement
                style={{
                  position: 'absolute',
                  top: 0,
                  transform: `translateY(${virtualRow.start}px)`,
                  width: '100%',
                }}
              >
                {row.getVisibleCells().map(cell => (
                  <td key={cell.id}>
                    {flexRender(cell.column.columnDef.cell, cell.getContext())}
                  </td>
                ))}
              </tr>
            )
          })}
        </tbody>
      </table>
    </div>
  )
}
```

## Column Virtualization

For tables with many columns, virtualize horizontally:

```tsx
const visibleColumns = table.getVisibleLeafColumns()

const columnVirtualizer = useVirtualizer({
  horizontal: true,
  count: visibleColumns.length,
  getScrollElement: () => parentRef.current,
  estimateSize: index => visibleColumns[index].getSize(),
  overscan: 3,
})

// Render only virtual columns in header:
<tr>
  {/* Left spacer */}
  <th style={{ width: columnVirtualizer.getVirtualItems()[0]?.start ?? 0 }} />
  {columnVirtualizer.getVirtualItems().map(virtualColumn => {
    const header = headerGroup.headers[virtualColumn.index]
    return (
      <th key={header.id} style={{ width: header.getSize() }}>
        {flexRender(header.column.columnDef.header, header.getContext())}
      </th>
    )
  })}
  {/* Right spacer */}
  <th style={{ width: columnVirtualizer.getTotalSize() - (columnVirtualizer.getVirtualItems().at(-1)?.end ?? 0) }} />
</tr>

// Same pattern for cells in each row
```

## Both Row + Column Virtualization

Combine both virtualizers — use row virtualizer for `<tr>` and column virtualizer for `<td>` within each virtual row. This enables rendering massive grids (100k rows × 100+ columns):

```tsx
const rowVirtualizer = useVirtualizer({
  count: rows.length,
  getScrollElement: () => parentRef.current,
  estimateSize: () => 35,
  overscan: 10,
})

const columnVirtualizer = useVirtualizer({
  horizontal: true,
  count: visibleColumns.length,
  getScrollElement: () => parentRef.current,
  estimateSize: index => visibleColumns[index].getSize(),
  overscan: 3,
})

<tbody style={{ height: `${rowVirtualizer.getTotalSize()}px`, position: 'relative' }}>
  {rowVirtualizer.getVirtualItems().map(virtualRow => {
    const row = rows[virtualRow.index]
    return (
      <tr
        key={row.id}
        style={{
          position: 'absolute',
          transform: `translateY(${virtualRow.start}px)`,
          display: 'flex',
          width: columnVirtualizer.getTotalSize(),
        }}
      >
        {columnVirtualizer.getVirtualItems().map(virtualColumn => {
          const cell = row.getVisibleCells()[virtualColumn.index]
          return (
            <td
              key={cell.id}
              style={{
                position: 'absolute',
                left: virtualColumn.start,
                width: virtualColumn.size,
              }}
            >
              {flexRender(cell.column.columnDef.cell, cell.getContext())}
            </td>
          )
        })}
      </tr>
    )
  })}
</tbody>
```

## Key Virtualizer Options

```tsx
const virtualizer = useVirtualizer({
  count: number,                    // total items
  getScrollElement: () => element,  // scrollable container ref
  estimateSize: (index) => number,  // estimated item size in px
  overscan: number,                 // extra items to render (default: 1)
  horizontal: boolean,              // horizontal virtualization
  measureElement: (el) => number,   // dynamic measurement (optional)
  paddingStart: number,             // padding before first item
  paddingEnd: number,               // padding after last item
  scrollPaddingStart: number,       // scroll padding start
  scrollPaddingEnd: number,         // scroll padding end
  initialOffset: number,            // initial scroll offset
  scrollToFn: (offset, options, instance) => void, // custom scroll behavior
  observeElementRect: fn,           // custom element rect observer
  observeElementOffset: fn,         // custom element offset observer
  lanes: number,                    // number of lanes (for masonry layouts)
})
```

## Virtualizer APIs

```tsx
virtualizer.getVirtualItems()   // VirtualItem[] to render
virtualizer.getTotalSize()      // total scroll height/width

// Scrolling
virtualizer.scrollToIndex(index, { align: 'start' | 'center' | 'end' | 'auto' })
virtualizer.scrollToOffset(px, { align })
virtualizer.scrollBy(delta)

// Measurement
virtualizer.measureElement(el)  // measure element dynamically (use as ref callback)
virtualizer.measure()           // force re-measure all items
virtualizer.resizeItem(index, size) // manually set item size

// State
virtualizer.isScrolling         // boolean — currently scrolling?
virtualizer.scrollOffset        // current scroll position
virtualizer.scrollDirection     // 'forward' | 'backward' | null
virtualizer.range               // { startIndex, endIndex } of visible items
```

## VirtualItem Shape

```tsx
interface VirtualItem {
  index: number   // index in the source list
  key: string     // unique key
  start: number   // start offset in px
  end: number     // end offset in px
  size: number    // measured or estimated size in px
  lane: number    // lane index (for multi-lane layouts)
}
```

## Dynamic Row Heights

For rows with variable content, use `measureElement` ref callback:

```tsx
<tr
  data-index={virtualRow.index}
  ref={node => virtualizer.measureElement(node)}
  style={{
    position: 'absolute',
    transform: `translateY(${virtualRow.start}px)`,
  }}
>
```

This measures actual DOM height and updates virtual positions. Set `estimateSize` close to average row height to minimize layout jumps.

## Infinite Scrolling

Combine virtualization with data fetching for infinite scroll:

```tsx
import { useInfiniteQuery } from '@tanstack/react-query'

const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery({
  queryKey: ['items'],
  queryFn: ({ pageParam = 0 }) => fetchItems(pageParam),
  getNextPageParam: (lastPage) => lastPage.nextCursor,
})

const allRows = data?.pages.flatMap(page => page.items) ?? []

const table = useReactTable({
  data: allRows,
  columns,
  getCoreRowModel: getCoreRowModel(),
})

const { rows } = table.getRowModel()

const virtualizer = useVirtualizer({
  count: hasNextPage ? rows.length + 1 : rows.length,
  getScrollElement: () => parentRef.current,
  estimateSize: () => 35,
  overscan: 5,
})

// Fetch more when scrolling near bottom:
useEffect(() => {
  const lastItem = virtualizer.getVirtualItems().at(-1)
  if (!lastItem) return
  if (lastItem.index >= rows.length - 1 && hasNextPage && !isFetchingNextPage) {
    fetchNextPage()
  }
}, [virtualizer.getVirtualItems()])
```

## Sticky Headers

Position thead outside the virtual scroll area or use CSS:

```css
thead {
  position: sticky;
  top: 0;
  z-index: 1;
  background: white;
}
```

Or render the header outside the scrollable div entirely.

## Tips & Gotchas

- **No pagination needed** — virtualization replaces pagination for "all data loaded" scenarios
- **Set `overscan`** to 5-20 for smoother scrolling (trades memory for UX)
- **Use `estimateSize`** close to actual row height to reduce layout shift
- **For dynamic heights**, use `measureElement` with a ref callback on each row
- **Combine with sorting/filtering** — all TanStack Table features work; you just virtualize the final `getRowModel().rows`
- **Performance**: handles 100k+ rows easily with virtualization
- **Dev mode is slower** — React StrictMode double-renders affect virtualization perf; test in production builds
- **Table layout**: use `table-layout: fixed` or flexbox for consistent column widths with absolute positioning
- **Accessibility**: screen readers may struggle with virtualized content since off-screen rows don't exist in DOM
- **Window virtualizer**: use `useWindowVirtualizer` instead of `useVirtualizer` when the window itself is the scroll container (no container ref needed)
