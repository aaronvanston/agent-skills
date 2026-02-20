# Custom Features

Extend TanStack Table with custom features using the `_features` table option (v8.14.0+). Custom features integrate exactly like built-in features — same lifecycle, same type safety, same patterns.

## TableFeature Interface

```tsx
import { TableFeature, RowData } from '@tanstack/react-table'

interface TableFeature<TData extends RowData = any> {
  createCell?: (
    cell: Cell<TData, unknown>,
    column: Column<TData>,
    row: Row<TData>,
    table: Table<TData>
  ) => void
  createColumn?: (column: Column<TData, unknown>, table: Table<TData>) => void
  createHeader?: (header: Header<TData, unknown>, table: Table<TData>) => void
  createRow?: (row: Row<TData>, table: Table<TData>) => void
  createTable?: (table: Table<TData>) => void
  getDefaultColumnDef?: () => Partial<ColumnDef<TData, unknown>>
  getDefaultOptions?: (
    table: Table<TData>
  ) => Partial<TableOptionsResolved<TData>>
  getInitialState?: (initialState?: InitialTableState) => Partial<TableState>
}
```

## How Built-in Features Work

All built-in features (sorting, filtering, pagination, etc.) follow the same `TableFeature` pattern. Each feature:

1. Defines initial state via `getInitialState`
2. Sets default table/column options via `getDefaultOptions` / `getDefaultColumnDef`
3. Adds API methods to instances via `createTable`, `createColumn`, `createRow`, `createCell`, `createHeader`

Custom features use the exact same mechanism — the `_features` option just registers additional feature objects alongside the built-in ones.

## Step-by-Step: Custom "Density" Feature

### 1. Define Types

```tsx
import { OnChangeFn, Updater, RowData } from '@tanstack/react-table'

type DensityState = 'sm' | 'md' | 'lg'

interface DensityTableState {
  density: DensityState
}

interface DensityOptions {
  enableDensity?: boolean
  onDensityChange?: OnChangeFn<DensityState>
}

interface DensityInstance {
  setDensity: (updater: Updater<DensityState>) => void
  toggleDensity: (value?: DensityState) => void
}
```

### 2. Declaration Merging

```tsx
declare module '@tanstack/react-table' {
  // Merge into table state
  interface TableState extends DensityTableState {}

  // Merge into table options
  interface TableOptionsResolved<TData extends RowData> extends DensityOptions {}

  // Merge into table instance
  interface Table<TData extends RowData> extends DensityInstance {}

  // Also available for per-instance APIs:
  // interface Row<TData extends RowData> { myRowMethod: () => void }
  // interface Column<TData extends RowData, TValue> { myColMethod: () => void }
  // interface Cell<TData extends RowData, TValue> { myCellMethod: () => void }
  // interface Header<TData extends RowData, TValue> { myHeaderMethod: () => void }
}
```

**Important:** `ColumnDef` cannot be extended via declaration merging (it's a complex type, not an interface). Use `ColumnDef.meta` for custom column-level config instead.

### 3. Create the Feature Object

```tsx
import { TableFeature, makeStateUpdater, Table } from '@tanstack/react-table'

const DensityFeature: TableFeature<any> = {
  // Set default state
  getInitialState: (state): Partial<TableState> => ({
    density: 'md',
    ...state, // preserve any user-provided initial state
  }),

  // Set default options
  getDefaultOptions: <TData extends RowData>(
    table: Table<TData>
  ): Partial<TableOptionsResolved<TData>> => ({
    enableDensity: true,
    onDensityChange: makeStateUpdater('density', table),
  }),

  // Add APIs to table instance
  createTable: <TData extends RowData>(table: Table<TData>): void => {
    table.setDensity = (updater) => {
      const safeUpdater: Updater<DensityState> = (old) => {
        const newState = typeof updater === 'function' ? updater(old) : updater
        return newState
      }
      table.options.onDensityChange?.(safeUpdater)
    }

    table.toggleDensity = (value) => {
      table.setDensity((old) => {
        if (value) return value
        return old === 'lg' ? 'md' : old === 'md' ? 'sm' : 'lg'
      })
    }
  },
}
```

### 4. Use the Feature

```tsx
const [density, setDensity] = useState<DensityState>('md')

const table = useReactTable({
  _features: [DensityFeature], // register custom features
  columns,
  data,
  state: { density },
  onDensityChange: setDensity,
  getCoreRowModel: getCoreRowModel(),
})

// Now type-safe:
table.toggleDensity()
table.setDensity('sm')
table.getState().density // 'sm' | 'md' | 'lg'
```

## Feature Lifecycle

Each `create*` method runs during table/row/column/cell instantiation and **mutates** the instance to add new APIs:

| Method | Runs on | Timing | Use for |
|--------|---------|--------|---------|
| `getInitialState` | Table init | Once | Set default state values |
| `getDefaultOptions` | Table init | Once | Set default table options |
| `getDefaultColumnDef` | Table init | Once | Set default column def options |
| `createTable` | Table init | Once | Add table-level APIs (`table.myMethod()`) |
| `createColumn` | Each column | Per column | Add column-level APIs (`column.myMethod()`) |
| `createRow` | Each row | Per row (can be frequent) | Add row-level APIs (`row.myMethod()`) |
| `createCell` | Each cell | Per cell (most frequent) | Add cell-level APIs (`cell.myMethod()`) |
| `createHeader` | Each header | Per header | Add header-level APIs (`header.myMethod()`) |

## makeStateUpdater Helper

`makeStateUpdater` creates a standard `on*Change` handler that updates internal table state:

```tsx
import { makeStateUpdater } from '@tanstack/react-table'

// This:
onDensityChange: makeStateUpdater('density', table)

// Is equivalent to:
onDensityChange: (updater) => {
  table.options.onStateChange?.((old) => ({
    ...old,
    density: typeof updater === 'function' ? updater(old.density) : updater,
  }))
}
```

## Row-Level Feature Example

Add a "bookmark" feature to individual rows:

```tsx
interface BookmarkRowState {
  isBookmarked: boolean
}

interface BookmarkRowInstance {
  toggleBookmark: () => void
  getIsBookmarked: () => boolean
}

declare module '@tanstack/react-table' {
  interface Row<TData extends RowData> extends BookmarkRowInstance {}
}

const BookmarkFeature: TableFeature<any> = {
  createRow: (row, table) => {
    row.toggleBookmark = () => {
      // Access table state/options from within row APIs
      const bookmarks = (table.getState() as any).bookmarks ?? {}
      table.options.onBookmarksChange?.({
        ...bookmarks,
        [row.id]: !bookmarks[row.id],
      })
    }
    row.getIsBookmarked = () => {
      return (table.getState() as any).bookmarks?.[row.id] ?? false
    }
  },
}
```

## Column-Level Feature Example

Add column-level description tooltips:

```tsx
const TooltipFeature: TableFeature<any> = {
  createColumn: (column, table) => {
    ;(column as any).getTooltip = () => {
      return column.columnDef.meta?.tooltip ?? null
    }
  },
}

// Usage in column def:
columnHelper.accessor('revenue', {
  header: 'Revenue',
  meta: { tooltip: 'Total revenue in USD for the current fiscal year' },
})
```

## Multiple Custom Features

```tsx
const table = useReactTable({
  _features: [DensityFeature, BookmarkFeature, TooltipFeature],
  columns,
  data,
  // ...
})
```

Features are processed in order. Later features can access APIs added by earlier features.

## Feature Depending on Built-in Features

Custom features can access all built-in table state and APIs:

```tsx
const SelectionSummaryFeature: TableFeature<any> = {
  createTable: (table) => {
    ;(table as any).getSelectionSummary = () => {
      const selected = table.getSelectedRowModel().rows
      return {
        count: selected.length,
        total: table.getRowModel().rows.length,
        percentage: (selected.length / table.getRowModel().rows.length) * 100,
      }
    }
  },
}
```

## Packaging Custom Features

Custom features can be published as standalone packages:

```tsx
// my-table-density/index.ts
export { DensityFeature } from './feature'
export type { DensityState, DensityOptions, DensityInstance } from './types'

// Consumer:
import { DensityFeature } from 'my-table-density'

const table = useReactTable({
  _features: [DensityFeature],
  // ...
})
```

## Tips & Gotchas

- Use `makeStateUpdater(stateKey, table)` for standard `on*Change` handlers — don't reinvent the wheel
- Declaration merging on `ColumnDef` is **not possible** (it's a complex type, not an interface) — use `ColumnDef.meta` instead
- `create*` methods **mutate** the instance — they don't return anything
- `createRow` and `createCell` run frequently (every re-render for visible rows/cells) — keep them lightweight
- Multiple features via `_features: [Feature1, Feature2]` — order matters if features depend on each other
- Custom features have access to the full table instance including all built-in APIs
- Use TypeScript declaration merging for full type safety — without it, you'll need type assertions
- The `_features` option is additive — built-in features are always included
- In a future v9 release, all features may become opt-in to reduce bundle size
