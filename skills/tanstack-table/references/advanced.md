# Advanced Patterns

## Editable Cells

```tsx
// 1. Extend TableMeta type
declare module '@tanstack/react-table' {
  interface TableMeta<TData extends RowData> {
    updateData: (rowIndex: number, columnId: string, value: unknown) => void
  }
}

// 2. Default editable cell component
const defaultColumn: Partial<ColumnDef<Person>> = {
  cell: ({ getValue, row: { index }, column: { id }, table }) => {
    const initialValue = getValue()
    const [value, setValue] = React.useState(initialValue)

    const onBlur = () => {
      table.options.meta?.updateData(index, id, value)
    }

    React.useEffect(() => { setValue(initialValue) }, [initialValue])

    return (
      <input
        value={value as string}
        onChange={e => setValue(e.target.value)}
        onBlur={onBlur}
      />
    )
  },
}

// 3. Table setup
const table = useReactTable({
  data,
  columns,
  defaultColumn, // applies editable cell to all columns
  getCoreRowModel: getCoreRowModel(),
  meta: {
    updateData: (rowIndex, columnId, value) => {
      setData(old => old.map((row, index) =>
        index === rowIndex ? { ...old[rowIndex]!, [columnId]: value } : row
      ))
    },
  },
})
```

## Sub-Components (Expandable Row Detail)

```tsx
// Column with expander:
const columns = [
  {
    id: 'expander',
    header: () => null,
    cell: ({ row }) => row.getCanExpand() ? (
      <button onClick={row.getToggleExpandedHandler()}>
        {row.getIsExpanded() ? '▼' : '▶'}
      </button>
    ) : null,
  },
  // ...data columns
]

// Render with sub-component:
{table.getRowModel().rows.map(row => (
  <React.Fragment key={row.id}>
    <tr>
      {row.getVisibleCells().map(cell => (
        <td key={cell.id}>{flexRender(cell.column.columnDef.cell, cell.getContext())}</td>
      ))}
    </tr>
    {row.getIsExpanded() && (
      <tr>
        <td colSpan={row.getVisibleCells().length}>
          <SubComponent row={row} />
        </td>
      </tr>
    )}
  </React.Fragment>
))}

// Table setup:
const table = useReactTable({
  data, columns,
  getRowCanExpand: () => true, // or row => row.original.hasDetail
  getCoreRowModel: getCoreRowModel(),
  getExpandedRowModel: getExpandedRowModel(),
})
```

## Row Pinning

```tsx
import { RowPinningState } from '@tanstack/react-table'

const [rowPinning, setRowPinning] = useState<RowPinningState>({ top: [], bottom: [] })

const table = useReactTable({
  data, columns,
  state: { rowPinning },
  onRowPinningChange: setRowPinning,
  keepPinnedRows: true, // keep pinned rows visible even when filtered/paginated out
  getCoreRowModel: getCoreRowModel(),
})

// Pin/unpin APIs:
row.pin('top')              // pin to top
row.pin('bottom')           // pin to bottom
row.pin(false)              // unpin
row.getIsPinned()           // 'top' | 'bottom' | false

// Render pinned rows separately:
table.getTopRows()          // Row[] pinned to top
table.getBottomRows()       // Row[] pinned to bottom
table.getCenterRows()       // Row[] not pinned
```

## DnD Column Reordering (@dnd-kit)

```bash
npm install @dnd-kit/core @dnd-kit/sortable @dnd-kit/modifiers @dnd-kit/utilities
```

```tsx
import { DndContext, closestCenter, DragEndEvent, useSensor, useSensors, MouseSensor, TouchSensor, KeyboardSensor } from '@dnd-kit/core'
import { SortableContext, horizontalListSortingStrategy, useSortable, arrayMove } from '@dnd-kit/sortable'
import { CSS } from '@dnd-kit/utilities'
import { restrictToHorizontalAxis } from '@dnd-kit/modifiers'

// Draggable header cell:
function DraggableHeader({ header }: { header: Header<Person, unknown> }) {
  const { attributes, listeners, setNodeRef, transform } = useSortable({ id: header.column.id })
  const style: CSSProperties = { transform: CSS.Translate.toString(transform) }
  return (
    <th ref={setNodeRef} style={style} {...attributes} {...listeners} colSpan={header.colSpan}>
      {flexRender(header.column.columnDef.header, header.getContext())}
    </th>
  )
}

// State + handler:
const [columnOrder, setColumnOrder] = useState<string[]>(() =>
  table.getAllLeafColumns().map(c => c.id)
)

const sensors = useSensors(useSensor(MouseSensor), useSensor(TouchSensor), useSensor(KeyboardSensor))

function handleDragEnd(event: DragEndEvent) {
  const { active, over } = event
  if (active && over && active.id !== over.id) {
    setColumnOrder(old => {
      const oldIndex = old.indexOf(active.id as string)
      const newIndex = old.indexOf(over.id as string)
      return arrayMove(old, oldIndex, newIndex)
    })
  }
}

// Render:
<DndContext sensors={sensors} collisionDetection={closestCenter} modifiers={[restrictToHorizontalAxis]} onDragEnd={handleDragEnd}>
  <table>
    <thead>
      {table.getHeaderGroups().map(headerGroup => (
        <SortableContext key={headerGroup.id} items={columnOrder} strategy={horizontalListSortingStrategy}>
          <tr>
            {headerGroup.headers.map(header => (
              <DraggableHeader key={header.id} header={header} />
            ))}
          </tr>
        </SortableContext>
      ))}
    </thead>
    {/* tbody with matching DraggableCell components */}
  </table>
</DndContext>
```

## Custom Features (_features)

Extend TanStack Table with your own state, options, and instance methods.

```tsx
import { TableFeature, Table, OnChangeFn, Updater } from '@tanstack/react-table'

// 1. Define types
type DensityState = 'sm' | 'md' | 'lg'
interface DensityTableState { density: DensityState }
interface DensityOptions { enableDensity?: boolean; onDensityChange?: OnChangeFn<DensityState> }
interface DensityInstance {
  setDensity: (updater: Updater<DensityState>) => void
  toggleDensity: (value?: DensityState) => void
}

// 2. Create feature
const DensityFeature: TableFeature<any> = {
  getInitialState: (state): DensityTableState => ({ density: 'md', ...state }),
  getDefaultOptions: <TData>(table: Table<TData>): DensityOptions => ({
    enableDensity: true,
  }),
  createTable: <TData>(table: Table<TData>): void => {
    table.setDensity = updater => table.options.onDensityChange?.(updater)
    table.toggleDensity = value => {
      table.setDensity(old =>
        value ?? (old === 'lg' ? 'md' : old === 'md' ? 'sm' : 'lg')
      )
    }
  },
}

// 3. Use
const [density, setDensity] = useState<DensityState>('md')

const table = useReactTable({
  _features: [DensityFeature],
  columns, data,
  state: { density },
  onDensityChange: setDensity,
  getCoreRowModel: getCoreRowModel(),
})

// Access in rendering:
const { density } = table.getState()
const padding = density === 'sm' ? '4px' : density === 'md' ? '8px' : '16px'
```

### TableFeature Interface

```tsx
interface TableFeature<TData> {
  getInitialState?: (state: Partial<TableState>) => Partial<TableState>
  getDefaultOptions?: (table: Table<TData>) => Partial<TableOptions<TData>>
  createTable?: (table: Table<TData>) => void
  createColumn?: (column: Column<TData>, table: Table<TData>) => void
  createRow?: (row: Row<TData>, table: Table<TData>) => void
  createCell?: (cell: Cell<TData>, column: Column<TData>, row: Row<TData>, table: Table<TData>) => void
  createHeader?: (header: Header<TData>, table: Table<TData>) => void
}
```

## Div-Based Table (CSS Grid/Flexbox)

Use `<div>` elements instead of `<table>` for more flexible layouts:

```tsx
<div style={{ width: table.getTotalSize() }}>
  <div className="thead">
    {table.getHeaderGroups().map(headerGroup => (
      <div key={headerGroup.id} className="tr" style={{ display: 'flex' }}>
        {headerGroup.headers.map(header => (
          <div key={header.id} className="th" style={{ width: header.getSize() }}>
            {flexRender(header.column.columnDef.header, header.getContext())}
          </div>
        ))}
      </div>
    ))}
  </div>
  <div className="tbody">
    {table.getRowModel().rows.map(row => (
      <div key={row.id} className="tr" style={{ display: 'flex' }}>
        {row.getVisibleCells().map(cell => (
          <div key={cell.id} className="td" style={{ width: cell.column.getSize() }}>
            {flexRender(cell.column.columnDef.cell, cell.getContext())}
          </div>
        ))}
      </div>
    ))}
  </div>
</div>
```

Useful for: sticky columns with CSS `position: sticky`, absolute positioning for virtualization, or CSS Grid layouts.
