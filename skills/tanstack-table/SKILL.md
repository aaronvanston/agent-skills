---
name: tanstack-table
description: Build tables and datagrids with TanStack Table v8. Covers column definitions, row models, sorting, filtering, pagination, grouping, expanding, row selection, column sizing/ordering/pinning/visibility, virtualization, row pinning, faceting, fuzzy filtering, custom features, state management, and rendering. React adapter primary, core API is framework-agnostic. Use when asked to "build a data table", "add sorting to a table", "paginate results", "filter table data", "create a datagrid", "add row selection", "virtualize a large table", or any task involving TanStack Table.
---

# TanStack Table

Headless UI library for building tables and datagrids. Framework-agnostic core, React adapter (`@tanstack/react-table`) primary.

## Workflow

Determine which feature applies and load the relevant reference:

1. **Columns** - define accessors, headers, groups, sizing, ordering, pinning, visibility: [references/columns.md](references/columns.md)
2. **Data & Row Models** - data shape, row model pipeline, Row/Cell types: [references/data.md](references/data.md)
3. **Sorting** - sort state, multi-sort, custom sort functions, manual sorting: [references/sorting.md](references/sorting.md)
4. **Filtering** - column filters, global filters, faceting (column + global): [references/filtering.md](references/filtering.md)
5. **Fuzzy Filtering** - `@tanstack/match-sorter-utils` integration: [references/fuzzy-filtering.md](references/fuzzy-filtering.md)
6. **Pagination** - client-side and server-side pagination: [references/pagination.md](references/pagination.md)
7. **Row Selection** - checkbox patterns, single/multi select, sub-row selection: [references/selection.md](references/selection.md)
8. **Grouping** - grouping state, aggregation, rendering grouped cells: [references/grouping.md](references/grouping.md)
9. **Expanding** - sub-rows, detail panels, expand/collapse APIs: [references/expanding.md](references/expanding.md)
10. **Row Pinning** - pin rows to top/bottom, keep pinned rows option: [references/row-pinning.md](references/row-pinning.md)
11. **Virtualization** - @tanstack/react-virtual integration, row/column virtualization: [references/virtualization.md](references/virtualization.md)
12. **State** - controlled vs uncontrolled, onChange handlers, auto-reset: [references/state.md](references/state.md)
13. **Rendering** - flexRender, table structure, types, framework adapters: [references/rendering.md](references/rendering.md)
14. **Advanced** - editable cells, DnD reordering, div-based tables: [references/advanced.md](references/advanced.md)
15. **Custom Features** - extending table with `_features` and TypeScript declaration merging: [references/custom-features.md](references/custom-features.md)

## Principles

- **`getCoreRowModel()` is the only required row model** - everything else is opt-in
- **Memoize column definitions** with `useMemo` or define outside the component
- **Keep data references stable** - avoid inline `data={fetch()}`, use state/memo
- **Row models are composable** - pipeline: Core → Filtered → Grouped → Sorted → Expanded → Paginated
- **Only import row models you need** - each is tree-shakeable
- **`flexRender`** renders cells and headers from column defs
- **For server-side operations** - set `manualSorting`/`manualFiltering`/`manualPagination` and manage state yourself
- **`createColumnHelper<T>()`** for maximum type safety
