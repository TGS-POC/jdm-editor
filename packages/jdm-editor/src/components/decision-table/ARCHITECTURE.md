# Decision Table Architecture Documentation

## Table of Contents
1. [Overview](#overview)
2. [Architecture Diagram](#architecture-diagram)
3. [Core Components](#core-components)
4. [Data Flow](#data-flow)
5. [State Management](#state-management)
6. [Table Rendering](#table-rendering)
7. [Cell System](#cell-system)
8. [Drag and Drop](#drag-and-drop)
9. [Excel Integration](#excel-integration)
10. [Debugging and Simulation](#debugging-and-simulation)
11. [Key Design Patterns](#key-design-patterns)

---

## Overview

The Decision Table component is a sophisticated spreadsheet-like editor for creating and managing decision logic in a tabular format. It provides an Excel-like experience with advanced features like drag-and-drop row reordering, virtualized rendering, column resizing, and Excel import/export.

### Key Technologies
- **React Table (TanStack Table)**: Core table management and rendering
- **TanStack Virtual**: Row virtualization for performance
- **React DnD**: Drag-and-drop functionality
- **Zustand**: State management
- **Immer**: Immutable state updates
- **ExcelJS**: Excel file import/export
- **TypeScript**: Type safety throughout

### Key Features
- **Excel-like Interface**: Familiar spreadsheet experience
- **Virtualized Rendering**: Handles thousands of rows efficiently
- **Column Management**: Add, remove, reorder, and resize columns
- **Row Management**: Add, remove, and reorder rows via drag-and-drop
- **Expression Editing**: Built-in code editor for complex expressions
- **Type Inference**: Automatic type checking with variable types
- **Debug Mode**: Real-time rule evaluation visualization
- **Excel Integration**: Import/export to Excel format
- **Diff Support**: Visual diff indicators for version comparison
- **Permission System**: Granular edit permissions

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                     DecisionTable (dt.tsx)                          │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    DndProvider (React DnD)                     │  │
│  │  ┌─────────────────────────────────────────────────────────┐  │  │
│  │  │       DecisionTableProvider (State Management)          │  │  │
│  │  │  ┌───────────────────────────────────────────────────┐  │  │  │
│  │  │  │  DecisionTableDialogProvider (Dialog State)      │  │  │  │
│  │  │  │                                                   │  │  │  │
│  │  │  │  ┌────────────────────────────────────────────┐  │  │  │  │
│  │  │  │  │  DecisionTableCommandBar                   │  │  │  │  │
│  │  │  │  │  - Export/Import Excel                     │  │  │  │  │
│  │  │  │  │  - Add Row Above/Below                     │  │  │  │  │
│  │  │  │  │  - Delete Row                              │  │  │  │  │
│  │  │  │  │  - Simulation Index Selector               │  │  │  │  │
│  │  │  │  └────────────────────────────────────────────┘  │  │  │  │
│  │  │  │                                                   │  │  │  │
│  │  │  │  ┌────────────────────────────────────────────┐  │  │  │  │
│  │  │  │  │  Table (table/table.tsx)                   │  │  │  │  │
│  │  │  │  │                                            │  │  │  │  │
│  │  │  │  │  ┌──────────────────────────────────────┐ │  │  │  │  │
│  │  │  │  │  │  TableHeadRow                        │ │  │  │  │  │
│  │  │  │  │  │  ├─ TableHeadCellInput               │ │  │  │  │  │
│  │  │  │  │  │  │  └─ TableHeadCellInputField (×N)  │ │  │  │  │  │
│  │  │  │  │  │  └─ TableHeadCellOutput              │ │  │  │  │  │
│  │  │  │  │  │     └─ TableHeadCellOutputField (×N) │ │  │  │  │  │
│  │  │  │  │  └──────────────────────────────────────┘ │  │  │  │  │
│  │  │  │  │                                            │  │  │  │  │
│  │  │  │  │  ┌──────────────────────────────────────┐ │  │  │  │  │
│  │  │  │  │  │  TableBody (Virtualized)             │ │  │  │  │  │
│  │  │  │  │  │  - TableRow (×N)                     │ │  │  │  │  │
│  │  │  │  │  │    └─ TableDefaultCell (×M)          │ │  │  │  │  │
│  │  │  │  │  │       ├─ DiffCodeEditor              │ │  │  │  │  │
│  │  │  │  │  │       ├─ DiffAutosizeTextArea        │ │  │  │  │  │
│  │  │  │  │  │       └─ TableInputCellStatus        │ │  │  │  │  │
│  │  │  │  │  └──────────────────────────────────────┘ │  │  │  │  │
│  │  │  │  │                                            │  │  │  │  │
│  │  │  │  │  ┌──────────────────────────────────────┐ │  │  │  │  │
│  │  │  │  │  │  TableContextMenu                    │ │  │  │  │  │
│  │  │  │  │  │  - Add Row Above/Below               │ │  │  │  │  │
│  │  │  │  │  │  - Remove Row                        │ │  │  │  │  │
│  │  │  │  │  └──────────────────────────────────────┘ │  │  │  │  │
│  │  │  │  └────────────────────────────────────────────┘  │  │  │  │
│  │  │  │                                                   │  │  │  │
│  │  │  │  ┌────────────────────────────────────────────┐  │  │  │  │
│  │  │  │  │  DecisionTableDialogs                      │  │  │  │  │
│  │  │  │  │  - FieldAdd: Add columns                   │  │  │  │  │
│  │  │  │  │  - FieldUpdate: Edit columns               │  │  │  │  │
│  │  │  │  │  - FieldsReorder: Reorder columns          │  │  │  │  │
│  │  │  │  └────────────────────────────────────────────┘  │  │  │  │
│  │  │  │                                                   │  │  │  │
│  │  │  │  ┌────────────────────────────────────────────┐  │  │  │  │
│  │  │  │  │  DecisionTableEmpty                        │  │  │  │  │
│  │  │  │  │  (Empty state UI)                          │  │  │  │  │
│  │  │  │  └────────────────────────────────────────────┘  │  │  │  │
│  │  │  └───────────────────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Core Components

### 1. DecisionTable (dt.tsx)
**Location**: `packages/jdm-editor/src/components/decision-table/dt.tsx`

**Purpose**: Root component that sets up the entire decision table environment.

**Responsibilities**:
- Initializes DnD provider for drag-and-drop
- Sets up state management providers
- Manages dialog context
- Provides container structure

**Props**:
```typescript
type DecisionTableProps = {
  id?: string;                      // Table identifier for persistence
  tableHeight: string | number;     // Max height of table
  mountDialogsOnBody?: boolean;     // Mount dialogs on body vs container
  manager?: DragDropManager;        // Optional external DnD manager
  // + DecisionTableContextProps
  // + DecisionTableEmptyType
};
```

**Key Features**:
- DnD context with custom root element
- Theme integration (light/dark mode)
- Dialog container management

---

### 2. DecisionTableProvider (context/dt-store.context.tsx)
**Location**: `packages/jdm-editor/src/components/decision-table/context/dt-store.context.tsx`

**Purpose**: Central state management using Zustand stores.

**Architecture**: Uses two separate Zustand stores for optimal performance:

#### Store 1: State Store
Manages the core application state:
- `decisionTable`: Current table data (inputs, outputs, rules)
- `cursor`: Currently selected cell position
- `disabled`: Edit disabled state
- `permission`: Permission level ('edit:full' | 'edit:rules' | 'edit:values')
- `colWidth` / `minColWidth`: Column sizing
- `inputVariableType`: Type information for expressions
- `derivedVariableTypes`: Computed type information
- `debug`: Simulation/debug state

#### Store 2: Listener Store
Manages callbacks:
- `onChange`: Called when table changes
- `cellRenderer`: Custom cell renderer function
- `onColumnResize`: Column resize callback

**Key Actions**:
```typescript
- setDecisionTable(): Update entire table
- setCursor(): Set selected cell
- commitData(): Update cell value
- swapRows(): Reorder rows via drag-and-drop
- addRowAbove() / addRowBelow(): Add rows
- removeRow(): Delete row
- addColumn(): Add input/output column
- updateColumn(): Update column properties
- removeColumn(): Delete column
- reorderColumns(): Reorder columns
- updateHitPolicy(): Change hit policy (first/collect)
```

---

### 3. Table (table/table.tsx)
**Location**: `packages/jdm-editor/src/components/decision-table/table/table.tsx`

**Purpose**: Core table component using TanStack Table with virtualization.

**Key Features**:

#### Column Definition
```typescript
columns: [
  {
    id: 'inputs',
    header: TableHeadCellInput,
    columns: inputs.map(input => ({
      id: input.id,
      header: TableHeadCellInputField,
      cell: TableDefaultCell,
    })),
  },
  {
    id: 'outputs',
    header: TableHeadCellOutput,
    columns: outputs.map(output => ({
      id: output.id,
      header: TableHeadCellOutputField,
      cell: TableDefaultCell,
    })),
  },
  {
    id: '_description',
    header: 'Description',
    cell: TableDefaultCell,
  },
]
```

#### Virtual Scrolling
- Uses `@tanstack/react-virtual` for row virtualization
- Estimates 38px per row
- 5 rows overscan for smooth scrolling
- Dynamic measurement with ResizeObserver

#### Column Resizing
- Persisted to localStorage (per table ID)
- `columnResizeMode: 'onChange'` for instant feedback
- Minimum column width enforced

#### Keyboard Shortcuts
- `Cmd/Ctrl + ↑`: Add row above current
- `Cmd/Ctrl + ↓`: Add row below current
- `Cmd/Ctrl + Backspace`: Delete current row

---

### 4. TableRow (table/table-row.tsx)
**Location**: `packages/jdm-editor/src/components/decision-table/table/table-row.tsx`

**Purpose**: Individual row component with drag-and-drop support.

**Key Features**:

#### Drag and Drop
```typescript
// Drop target
const [{ isDropping, direction }, dropRef] = useDrop({
  accept: 'row',
  collect: (monitor) => ({
    isDropping: monitor.isOver(),
    direction: monitor.getDifferenceFromInitialOffset()?.y > 0 ? 'down' : 'up',
  }),
  drop: (draggedRow) => tableActions.swapRows(draggedRow.index, row.index),
});

// Drag source
const [{ isDragging }, dragRef, previewRef] = useDrag({
  item: () => row,
  type: 'row',
});
```

#### Visual States
- **Selected**: Current cursor row (highlighted)
- **Active**: Current simulation trace row (debug mode)
- **Disabled**: Non-editable state
- **Dragging**: 50% opacity while dragging
- **Dropping**: Visual indicator (up/down) while hovering
- **Diff**: Visual diff states (added/modified/removed)

#### Dynamic Sizing
- ResizeObserver for dynamic height measurement
- Updates virtualizer on size change

---

### 5. TableDefaultCell (table/table-default-cell.tsx)
**Location**: `packages/jdm-editor/src/components/decision-table/table/table-default-cell.tsx`

**Purpose**: Default cell renderer with expression editing and type inference.

**Architecture**:

#### Cell Types
1. **Description Cell**: Simple textarea (no field mapping)
2. **Input Cell**: Expression editor with unary mode
3. **Output Cell**: Standard expression editor

#### Expression Evaluation
```typescript
enum LocalVariableKind {
  Root,     // Uses inputVariableType directly
  Derived,  // Uses calculated type from field path
}
```

#### Type Inference Flow
```
Input Field → Calculate Type → Store in derivedVariableTypes → Use in Cell Editor
```

#### Debug Mode Features
- **TableInputCellStatus**: Visual indicator showing if rule matches
  - Green dot: Rule matches (hit)
  - Red dot: Rule doesn't match (no-hit)
  - Hidden: Field not in reference map (skip)

#### Cell Editing
- Auto-focus on cell click
- Inline editing with DiffCodeEditor or DiffAutosizeTextArea
- Commit on blur
- Live diff display

---

### 6. TableHeadCell Components (table/table-head-cell.tsx)
**Location**: `packages/jdm-editor/src/components/decision-table/table/table-head-cell.tsx`

**Purpose**: Header cells for inputs, outputs, and individual fields.

#### TableHeadCellInput / TableHeadCellOutput
- Section headers ("Inputs" / "Outputs")
- Add column button
- Reorder columns button (when > 1 column)
- Permission-aware (hide buttons based on permission)

#### TableHeadCellInputField / TableHeadCellOutputField
- Editable column name (TextEdit component)
- Field selector/editor (InputFieldEdit / OutputFieldEdit)
- Diff indicators
- Debug data display (in debug mode)

---

### 7. DecisionTableCommandBar (dt-command-bar.tsx)
**Location**: `packages/jdm-editor/src/components/decision-table/dt-command-bar.tsx`

**Purpose**: Top toolbar with table-level actions.

**Features**:
- **Export Excel**: Download table as .xlsx file
- **Import Excel**: Upload and parse .xlsx file
- **Row Actions** (when cell selected):
  - Add Row Above
  - Add Row Below
  - Delete Row
  - Deselect
- **Debug Controls** (in debug mode):
  - Simulation index selector (for loop execution mode)

---

### 8. DecisionTableDialogs (dialog/dt-dialogs.tsx)
**Location**: `packages/jdm-editor/src/components/decision-table/dialog/`

**Purpose**: Modal dialogs for table management.

#### FieldAdd (field-add-dialog.tsx)
- Add new input or output column
- Cascader for schema selection
- Form fields:
  - Label (required)
  - Field/Selector (required for outputs)
  - Default Value (optional)

#### FieldUpdate (field-update-dialog.tsx)
- Edit existing column
- Same fields as FieldAdd
- Cannot change column ID

#### FieldsReorder (fields-reorder-dialog.tsx)
- Drag-and-drop reordering of columns
- Separate for inputs and outputs
- Live preview

---

## Data Flow

### 1. Data Structure

```typescript
type DecisionTableType = {
  hitPolicy: HitPolicy | string;    // 'first' | 'collect'
  passThorough?: boolean;           // Pass input through to output
  inputField?: string;              // Loop execution input field
  outputPath?: string;              // Loop execution output path
  executionMode?: 'single' | 'loop'; // Execution mode
  inputs: TableSchemaItem[];        // Input columns
  outputs: TableSchemaItem[];       // Output columns
  rules: Record<string, string>[];  // Row data
  _diff?: DiffMetadata;             // Diff metadata
};

type TableSchemaItem = {
  id: string;                       // Unique identifier
  name: string;                     // Display name
  field?: string;                   // Field path or selector
  defaultValue?: string;            // Default value for new rows
  _diff?: DiffMetadata;             // Diff metadata
};

type TableCursor = {
  x: string;                        // Column ID
  y: number;                        // Row index
};
```

### 2. Data Transformation Flow

```
User Edit
    ↓
Cell onChange
    ↓
commitData(value, cursor)
    ↓
Immer Producer (immutable update)
    ↓
decisionTable.rules[y][x] = value
    ↓
Update Zustand State
    ↓
Trigger onChange listener
    ↓
Parent component receives update
    ↓
React Table re-renders affected cells
```

### 3. Column Operations Flow

```
Add/Update/Remove Column
    ↓
Action (addColumn/updateColumn/removeColumn)
    ↓
Update inputs/outputs array
    ↓
cleanupTableRules()
    ↓
Ensure all rules have keys for all columns
    ↓
Update State
    ↓
React Table rebuilds column definitions
    ↓
Re-render table
```

### 4. Row Operations Flow

```
Add/Remove/Swap Row
    ↓
Action (addRowAbove/addRowBelow/removeRow/swapRows)
    ↓
Modify rules array
    ↓
Update State
    ↓
Update cursor (if affected)
    ↓
Virtualizer recalculates
    ↓
Re-render visible rows
```

---

## State Management

### State Update Pattern

All state updates follow this pattern:

1. **Action Called**: User triggers an action
2. **Produce Draft**: Use Immer to create a draft
3. **Modify Draft**: Make changes to the draft
4. **Update Store**: Apply changes to Zustand store
5. **Notify Listeners**: Trigger `onChange` callback

### Example: Cell Update Flow

```typescript
commitData: (value: string, cursor: TableCursor) => {
  const { decisionTable } = stateStore.getState();

  // 1. Create draft with Immer
  const updatedDecisionTable = produce(decisionTable, (draft) => {
    const { x, y } = cursor;
    draft.rules[y][x] = value;
    return draft;
  });

  // 2. Update store
  stateStore.setState({ decisionTable: updatedDecisionTable });

  // 3. Notify listener
  listenerStore.getState().onChange?.(updatedDecisionTable);
}
```

### Example: Row Swap Flow

```typescript
swapRows: (source: number, target: number) => {
  const { decisionTable } = stateStore.getState();

  const updatedDecisionTable = produce(decisionTable, (draft) => {
    // Remove source row
    const input = draft.rules[source];
    draft.rules.splice(source, 1);

    // Insert at target position
    draft.rules.splice(target, 0, input);
    return draft;
  });

  // Clear cursor after reorder
  stateStore.setState({
    decisionTable: updatedDecisionTable,
    cursor: null,
  });

  listenerStore.getState().onChange?.(updatedDecisionTable);
}
```

### Data Cleanup Functions

#### cleanupTableRule()
Ensures a single rule has all required fields:
```typescript
const cleanupTableRule = (
  decisionTable: DecisionTableType,
  rule: Record<string, string>,
  defaultId?: string,
): Record<string, string> => {
  const schemaItems = [...decisionTable.inputs, ...decisionTable.outputs];
  const newRule: Record<string, string> = {
    _id: rule._id || crypto.randomUUID(),
    _description: rule._description,
  };

  schemaItems.forEach((schemaItem) => {
    if (defaultId && newRule._id === defaultId) {
      // Use default value for new row
      newRule[schemaItem.id] = rule?.[schemaItem.id] || schemaItem?.defaultValue || '';
    } else {
      newRule[schemaItem.id] = rule?.[schemaItem.id] || '';
    }
  });

  return newRule;
};
```

#### cleanupTableRules()
Applies cleanup to all rules:
```typescript
const cleanupTableRules = (
  decisionTable: DecisionTableType,
  defaultId?: string
): Record<string, string>[] => {
  const rules = decisionTable?.rules || [];
  return rules.map((rule) => cleanupTableRule(decisionTable, rule, defaultId));
};
```

Called after:
- Adding a column
- Removing a column
- Reordering columns
- Updating column properties

---

## Table Rendering

### TanStack Table Integration

#### Table Initialization
```typescript
const table = useReactTable({
  data: rules,                      // Row data
  columnResizeMode: 'onChange',     // Instant resize feedback
  getRowId: (row) => row._id,       // Use _id as row key
  columns,                          // Column definitions
  getCoreRowModel: getCoreRowModel(), // Core table logic
  defaultColumn: {
    cell: (context) => <TableDefaultCell context={context} />,
  },
  meta: {
    getCell: cellRenderer,          // Custom cell renderer
  },
  state: { columnSizing },          // Controlled column sizing
  onColumnSizingChange: setColumnSizing, // Update sizing state
});
```

#### Header Rendering
Two header rows:
1. **Group Row**: "Inputs" and "Outputs" section headers
2. **Column Row**: Individual input/output column headers

```typescript
<thead>
  {table.getHeaderGroups()[0] && (
    <TableHeadRow headerGroup={table.getHeaderGroups()[0]} />
  )}
</thead>
<thead>
  {table.getHeaderGroups()[1] && (
    <TableHeadRow headerGroup={table.getHeaderGroups()[1]} />
  )}
</thead>
```

#### Body Rendering (Virtualized)
```typescript
const { rows } = table.getRowModel();
const virtualizer = useVirtualizer({
  getScrollElement: () => tableContainerRef.current,
  estimateSize: () => 38,           // Estimated row height
  count: rows.length,
  overscan: 5,                      // Render 5 extra rows
});

const virtualItems = virtualizer.getVirtualItems();
```

**Padding Calculation**:
```typescript
const paddingTop = virtualItems[0]?.start || 0;
const paddingBottom = totalSize - virtualItems[virtualItems.length - 1]?.end || 0;
```

This creates placeholder rows to maintain scroll position.

---

## Cell System

### Cell Lifecycle

```
1. Cell Rendered
    ↓
2. useDecisionTableState (get cell value)
    ↓
3. Local state initialized (inner)
    ↓
4. User edits
    ↓
5. Local state updated (setInner)
    ↓
6. onChange triggered
    ↓
7. commitData(value, cursor)
    ↓
8. Store updated
    ↓
9. Cell value changes
    ↓
10. useLayoutEffect syncs local state
```

### Cell Focus Management

```typescript
useEffect(() => {
  const parentContainer = textareaRef.current.closest('div.cell-wrapper');
  const eventListener = (e: Event) => {
    if (e.target === textareaRef.current) return;

    // Focus textarea when clicking anywhere in cell
    textareaRef.current.focus();
    textareaRef.current.selectionStart = textareaRef.current.value.length;
  };

  parentContainer.style.cursor = 'text';
  parentContainer.addEventListener('click', eventListener);

  return () => {
    parentContainer.removeEventListener('click', eventListener);
  };
}, []);
```

### Custom Cell Renderer

```typescript
<DecisionTable
  cellRenderer={(props: TableCellProps) => {
    if (props.column?.id === 'special-column') {
      return <MyCustomCell {...props} />;
    }
    return null; // Fall back to default
  }}
/>
```

### Type Inference in Cells

#### Root Variable Type
Used for standard expressions:
```typescript
inputVariableType.evaluateExpression(expression)
```

#### Derived Variable Type
Used for unary expressions with field reference:
```typescript
// Calculate type from field path
const calculatedType = inputVariableType.calculateType(column.field);
const resultingType = inputVariableType.clone();
resultingType.set('$', calculatedType);

// Evaluate unary expression
resultingType.evaluateUnaryExpression(expression)
```

---

## Drag and Drop

### Row Reordering

#### DnD Item Type
```typescript
type: 'row'
```

#### Drag Source
```typescript
const [{ isDragging }, dragRef, previewRef] = useDrag({
  item: () => row,
  type: 'row',
  collect: (monitor) => ({
    isDragging: monitor.isDragging(),
  }),
});
```

#### Drop Target
```typescript
const [{ isDropping, direction }, dropRef] = useDrop({
  accept: 'row',
  collect: (monitor) => ({
    isDropping: monitor.isOver({ shallow: true }),
    direction: (monitor.getDifferenceFromInitialOffset()?.y || 0) > 0 ? 'down' : 'up',
  }),
  drop: (draggedRow: Row) => tableActions.swapRows(draggedRow.index, row.index),
});
```

#### Visual Feedback
```typescript
className={clsx(
  'table-row',
  isDropping && direction === 'down' && 'dropping-down',
  isDropping && direction === 'up' && 'dropping-up',
)}
style={{
  opacity: isDragging ? 0.5 : 1,
}}
```

### Column Reordering

Handled by separate dialog (FieldsReorderDialog) with its own DnD context.

---

## Excel Integration

### Export to Excel (excel.ts)

**Process**:
1. Create workbook with ExcelJS
2. Add worksheet for table
3. Create header structure:
   - Row 1: Table ID (merged cell)
   - Row 2: "Inputs" and "Outputs" sections (merged cells)
   - Row 3: Column headers
4. Add data rows
5. Apply styling (colors, borders, alignment)
6. Download as .xlsx file

**Excel Structure**:
```
Row 1: [Table ID - merged across all columns]
Row 2: [Inputs - merged] [Outputs - merged] [Description] [Rule ID]
Row 3: [Col1] [Col2] ... [ColN] [OutCol1] ... [OutColM] [Description] [Rule ID]
Row 4+: Data rows
```

### Import from Excel

**Process**:
1. Read .xlsx file with FileReader
2. Parse with ExcelJS
3. Extract metadata from header rows
4. Map columns back to inputs/outputs
5. Parse data rows
6. Create DecisionTableType object
7. Validate and update state

**Error Handling**:
- Invalid file format
- Missing required columns
- Malformed data
- Schema mismatch

---

## Debugging and Simulation

### Debug Mode Architecture

When connected to a simulation, the table shows:
1. **Active Rule**: Highlight rule that matched
2. **Cell Status**: Visual indicators for input cells (hit/no-hit)
3. **Reference Data**: Display actual values from simulation
4. **Simulation Index**: For loop mode, select which iteration to view

### Cell Status Calculation

```typescript
const status = useMemo<'hit' | 'no-hit' | 'skip' | null>(() => {
  if (!inputData || !expressionContext) return null;

  const { type, expression, $ } = expressionContext;

  if (type === 'skip') return 'skip'; // Field not in reference map

  if (!expression) return 'hit'; // Empty expression always matches

  try {
    let isOk: boolean;
    if (type === 'unary') {
      const newInputData = inputData.cloneWith('$', $);
      isOk = newInputData.evaluateUnaryExpression(expression);
    } else {
      isOk = inputData.evaluateExpression(expression) === true;
    }

    return isOk ? 'hit' : 'no-hit';
  } catch {
    return 'no-hit';
  }
}, [inputData, expressionContext]);
```

### Debug State Structure

```typescript
debug?: {
  snapshot: DecisionTableType;              // Table state at execution time
  trace: SimulationTrace<SimulationTraceDataTable>; // Execution trace
  inputData?: GetNodeDataResult;            // Input data
};

// For loop mode
traceData: Array<{
  rule: { _id: string };                    // Matched rule
  reference_map: Record<string, unknown>;   // Field values
}>;

// For single mode
traceData: {
  rule: { _id: string };
  reference_map: Record<string, unknown>;
};
```

---

## Key Design Patterns

### 1. Provider Pattern
- `DecisionTableProvider` wraps entire component tree
- `DecisionTableDialogProvider` for dialog state
- Context hooks for accessing state/actions

### 2. Compound Component Pattern
- Table composed of multiple sub-components
- Each component has specific responsibility
- Components communicate via shared context

### 3. Virtual Scrolling Pattern
- Only render visible rows
- Dynamic measurement
- Padding for scroll position

### 4. Cell Renderer Pattern
- Customizable cell rendering
- Default renderer with fallback
- Meta object for custom rendering logic

### 5. Cleanup Pattern
- `cleanupTableRule()` / `cleanupTableRules()`
- Ensures data consistency
- Called after structural changes

### 6. Cursor Pattern
- Track selected cell
- Enable context-aware actions
- Clear on structural changes

### 7. Permission Pattern
- Granular permissions ('edit:full', 'edit:rules', 'edit:values')
- Conditional UI rendering
- Action gating

### 8. Diff Pattern
- `_diff` metadata on nodes/edges/cells
- Visual indicators (added/modified/removed)
- Previous value tracking

---

## Performance Considerations

### 1. Virtualization
- Only render visible rows (~10-15 at a time)
- Handles thousands of rows efficiently
- Dynamic measurement for varying row heights

### 2. React Table Performance
- Memoized column definitions
- Equality checks for rule updates (by _id)
- Controlled column sizing state

### 3. State Updates
- Immer for efficient immutable updates
- Zustand prevents unnecessary re-renders
- Equality functions in selectors

### 4. Cell Rendering
- Memoized cells
- Local state for editing
- Debounced commits (via blur)

### 5. Drag and Drop
- Shallow monitoring
- Opacity instead of removal while dragging
- Efficient swap operations

---

## Extension Points

### 1. Custom Cell Renderer
Implement `cellRenderer` prop:
```typescript
<DecisionTable
  cellRenderer={(props) => {
    if (props.column?.id === 'my-column') {
      return <MyCustomCell {...props} />;
    }
    return null;
  }}
/>
```

### 2. Custom Permissions
Control what users can edit:
```typescript
<DecisionTable
  permission="edit:rules" // or "edit:full" or "edit:values"
/>
```

### 3. Column Sizing Persistence
Automatically saved to localStorage with table ID.

### 4. Debug Integration
Connect simulation results:
```typescript
<DecisionTable
  debug={{
    snapshot: tableBeforeExecution,
    trace: executionTrace,
    inputData: inputDataVariables,
  }}
/>
```

---

## Common Patterns and Best Practices

### 1. Accessing Table State
```typescript
const { inputs, outputs, rules } = useDecisionTableState((state) => ({
  inputs: state.decisionTable.inputs,
  outputs: state.decisionTable.outputs,
  rules: state.decisionTable.rules,
}));
```

### 2. Modifying Table
```typescript
const actions = useDecisionTableActions();

// Add column
actions.addColumn('inputs', {
  id: crypto.randomUUID(),
  name: 'New Input',
  field: 'input.field',
});

// Add row
actions.addRowBelow(); // Adds at end
actions.addRowBelow(5); // Adds after row 5
```

### 3. Handling Table Changes
```typescript
<DecisionTable
  value={table}
  onChange={(newTable) => {
    console.log('Table updated:', newTable);
    saveTable(newTable);
  }}
/>
```

---

## File Structure Reference

```
decision-table/
├── dt.tsx                          # Root component
├── dt-command-bar.tsx              # Top toolbar
├── dt-empty.tsx                    # Empty state
├── dt.stories.tsx                  # Storybook stories
├── excel.ts                        # Excel import/export
├── util.ts                         # Utility functions
├── index.ts                        # Public exports
├── context/
│   ├── dt-store.context.tsx        # State management (Zustand)
│   └── dt-dialog.context.tsx       # Dialog state management
├── table/
│   ├── table.tsx                   # Main table component
│   ├── table-head-row.tsx          # Header row wrapper
│   ├── table-head-cell.tsx         # Header cells
│   ├── table-row.tsx               # Body row with DnD
│   ├── table-default-cell.tsx      # Default cell renderer
│   └── table-context-menu.tsx      # Right-click menu
├── dialog/
│   ├── dt-dialogs.tsx              # Dialog container
│   ├── field-add-dialog.tsx        # Add column dialog
│   ├── field-update-dialog.tsx     # Edit column dialog
│   └── fields-reorder-dialog.tsx   # Reorder columns dialog
└── components/
    ├── input-field-edit.tsx        # Input field editor
    └── output-field-edit.tsx       # Output field editor
```

---

## Troubleshooting

### Common Issues

**Q: Table not rendering**
- Ensure `tableHeight` prop is provided
- Check that DnD ref is initialized

**Q: Columns not resizing**
- Verify table `id` prop is provided for persistence
- Check that columns have `minSize` and `size` defined

**Q: Rows not reordering via drag-and-drop**
- Ensure `disabled` is false
- Check DnD provider is wrapping table

**Q: Cell values not updating**
- Verify `onChange` callback is provided
- Check that cursor is set correctly

**Q: Excel import failing**
- Ensure file format matches expected structure
- Check that all required columns exist
- Verify data types are compatible

**Q: Type inference not working**
- Ensure `inputVariableType` is provided
- Check that WASM is available
- Verify field paths are correct

---

## Migration Guide

### From Version 1.x to 2.x

#### Changed: Store Architecture
```typescript
// Old (v1.x)
const store = useDecisionTableStore();
store.state.rules;

// New (v2.x)
const rules = useDecisionTableState((state) => state.decisionTable.rules);
```

#### Changed: Column Type
```typescript
// Old (v1.x)
type: 'input' | 'output'

// New (v2.x)
columnType: 'inputs' | 'outputs'
```

#### Changed: Action Names
```typescript
// Old (v1.x)
actions.addInput(input);
actions.addOutput(output);

// New (v2.x)
actions.addColumn('inputs', input);
actions.addColumn('outputs', output);
```

---

## Additional Resources

- [TanStack Table Documentation](https://tanstack.com/table/latest)
- [TanStack Virtual Documentation](https://tanstack.com/virtual/latest)
- [React DnD Documentation](https://react-dnd.github.io/react-dnd)
- [ExcelJS Documentation](https://github.com/exceljs/exceljs)
- [Zustand Documentation](https://github.com/pmndrs/zustand)

---