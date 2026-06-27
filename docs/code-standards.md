# Code Standards

## TSX / JSX

### Extract logic out of JSX

Keep JSX declarative and free of inline logic. Before the `return` statement, extract:

- **Sub-components** for reusable or non-trivial render blocks (e.g. `ResizeHandle`, `SortBadge`, `HeaderCell`)
- **Named functions** for event handlers and derived values (e.g. `handleRowClick`, `cellStyle`)
- **Style objects** for non-trivial inline styles

**Don't:**
```tsx
// Nested conditionals and inline logic inside JSX
{headers.map((header) => (
  <TableCell style={{ position: 'relative', width: header.getSize() }}>
    {header.column.getCanSort() ? (
      <TableSortLabel onClick={(e) => onSort(header.id, e.shiftKey)}>
        {header.label}
        {isSorted && sortCriteria.length > 1 && (
          <span>{sortIndex + 1}</span>
        )}
      </TableSortLabel>
    ) : header.label}
    {header.column.getCanResize() && (
      <div onMouseDown={header.getResizeHandler()} style={{ position: 'absolute', right: 0, ... }} />
    )}
  </TableCell>
))}
```

**Do:**
```tsx
const ResizeHandle = ({ header }) => <div onMouseDown={header.getResizeHandler()} style={resizeStyle} />;
const SortBadge = ({ index }) => <Typography>{index + 1}</Typography>;
const SortableLabel = ({ header, showBadge }) => (
  <TableSortLabel ...>
    {flexRender(...)}
    {showBadge && <SortBadge index={header.column.getSortIndex()} />}
  </TableSortLabel>
);
const HeaderContent = ({ header, sortedCount }) => { /* early return pattern */ };
const HeaderCell = ({ header, sortedCount }) => (
  <TableCell style={cellStyle(header.getSize())}>
    <HeaderContent header={header} sortedCount={sortedCount} />
    {header.column.getCanResize() && <ResizeHandle header={header} />}
  </TableCell>
);

// JSX stays flat and readable
{headerGroup.headers.map((header) => (
  <HeaderCell key={header.id} header={header} sortedCount={sortedCount} />
))}
```
