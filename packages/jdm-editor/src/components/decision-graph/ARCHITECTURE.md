# Decision Graph Architecture Documentation

## Table of Contents
1. [Overview](#overview)
2. [Architecture Diagram](#architecture-diagram)
3. [Core Components](#core-components)
4. [Data Flow](#data-flow)
5. [State Management](#state-management)
6. [Node System](#node-system)
7. [Graph Rendering](#graph-rendering)
8. [Simulation System](#simulation-system)
9. [Key Design Patterns](#key-design-patterns)

---

## Overview

The Decision Graph is a visual flow-based editor for creating and managing decision logic. It's built on top of React Flow and provides a rich interface for creating, editing, and simulating decision workflows.

### Key Technologies
- **React Flow**: Core graph rendering and interaction library
- **Zustand**: State management
- **Immer**: Immutable state updates
- **TypeScript**: Type safety throughout
- **Ant Design**: UI components
- **Monaco Editor**: Code editing capabilities

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                          DecisionGraph (dg.tsx)                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │              ReactFlowProvider (React Flow Context)            │  │
│  │  ┌─────────────────────────────────────────────────────────┐  │  │
│  │  │     DecisionGraphProvider (Global State Management)      │  │  │
│  │  │                                                           │  │  │
│  │  │  ┌───────────────────────────────────────────────────┐  │  │  │
│  │  │  │         DecisionGraphWrapper (Main Layout)        │  │  │  │
│  │  │  │                                                     │  │  │  │
│  │  │  │  ┌────────────────┐  ┌──────────────────────────┐ │  │  │  │
│  │  │  │  │ GraphSideToolbar│  │      Graph (Canvas)      │ │  │  │  │
│  │  │  │  │  (Undo/Redo)   │  │   - ReactFlow Component  │ │  │  │  │
│  │  │  │  └────────────────┘  │   - Node Rendering       │ │  │  │  │
│  │  │  │                      │   - Edge Rendering       │ │  │  │  │
│  │  │  │  ┌────────────────┐  │   - Drag & Drop          │ │  │  │  │
│  │  │  │  │   GraphTabs    │  │   - Node Interactions    │ │  │  │  │
│  │  │  │  │ (Tab Navigation)│ └──────────────────────────┘ │  │  │  │
│  │  │  │  └────────────────┘                               │  │  │  │
│  │  │  │                      ┌──────────────────────────┐ │  │  │  │
│  │  │  │  ┌────────────────┐  │   GraphNodes (List View) │ │  │  │  │
│  │  │  │  │  TabContents   │  │  (Alternative to Canvas) │ │  │  │  │
│  │  │  │  │ (Node Editors) │  └──────────────────────────┘ │  │  │  │
│  │  │  │  └────────────────┘                               │  │  │  │
│  │  │  │                      ┌──────────────────────────┐ │  │  │  │
│  │  │  │  ┌────────────────┐  │      GraphPanel          │ │  │  │  │
│  │  │  │  │ GraphComponents│  │   (Simulator/Panels)     │ │  │  │  │
│  │  │  │  │(Drag Palette)  │  └──────────────────────────┘ │  │  │  │
│  │  │  │  └────────────────┘                               │  │  │  │
│  │  │  └───────────────────────────────────────────────────┘  │  │  │
│  │  │                                                           │  │  │
│  │  │  ┌───────────────────────────────────────────────────┐  │  │  │
│  │  │  │       DecisionGraphInferTypes (Type Inference)    │  │  │  │
│  │  │  └───────────────────────────────────────────────────┘  │  │  │
│  │  │                                                           │  │  │
│  │  │  ┌───────────────────────────────────────────────────┐  │  │  │
│  │  │  │        DecisionGraphEmpty (Empty State UI)        │  │  │  │
│  │  │  └───────────────────────────────────────────────────┘  │  │  │
│  │  └─────────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Core Components

### 1. DecisionGraph (dg.tsx)
**Location**: `packages/jdm-editor/src/components/decision-graph/dg.tsx`

**Purpose**: Root component that sets up the entire decision graph environment.

**Responsibilities**:
- Initializes React Flow provider
- Wraps everything in DecisionGraphProvider for state management
- Provides the main container structure
- Manages refs for imperative API

**Props**:
```typescript
type DecisionGraphProps = {
  manager?: DragDropManager;
  reactFlowProOptions?: ProOptions;
  tabBarExtraContent?: React.ReactNode;
  // ... + DecisionGraphContextProps + DecisionGraphEmptyType
}
```

---

### 2. DecisionGraphProvider (dg-store.context.tsx)
**Location**: `packages/jdm-editor/src/components/decision-graph/context/dg-store.context.tsx`

**Purpose**: Central state management using Zustand stores.

**Architecture**: Uses three separate Zustand stores for optimal performance:

#### Store 1: State Store
Manages the core application state:
- `decisionGraph`: Current graph data (nodes and edges)
- `openTabs`: Array of open node IDs
- `activeTab`: Currently selected tab
- `components`: Available node specifications
- `customNodes`: Custom node types
- `panels`: Side panel configurations
- `simulate`: Simulation results
- `compactMode`: UI display mode
- `nodeTypes`: Type information for nodes

#### Store 2: Listener Store
Manages callbacks and event handlers:
- `onChange`: Called when graph changes
- `onPanelsChange`: Called when panel state changes
- `onReactFlowInit`: Called when React Flow initializes
- `onCodeExtension`: Monaco editor extensions
- `onFunctionReady`: Monaco setup callback

#### Store 3: Reference Store
Manages mutable references:
- `nodesState`: Reference to React Flow nodes state
- `edgesState`: Reference to React Flow edges state
- `reactFlowInstance`: Reference to React Flow instance
- `graphClipboard`: Reference to clipboard manager

**Key Actions**:
```typescript
- setDecisionGraph(): Update the entire graph
- addNodes() / removeNodes(): Node management
- addEdges() / removeEdges(): Edge management
- updateNode(): Update individual node
- duplicateNodes() / copyNodes() / pasteNodes(): Clipboard operations
- openTab() / closeTab(): Tab management
- goToNode(): Navigate to a specific node
- setCompactMode() / toggleCompactMode(): UI mode management
```

---

### 3. DecisionGraphWrapper (dg-wrapper.tsx)
**Location**: `packages/jdm-editor/src/components/decision-graph/dg-wrapper.tsx`

**Purpose**: Main layout orchestrator that combines all major UI elements.

**Structure**:
```
GraphSideToolbar (Left toolbar)
│
├── Graph (Main canvas)
│   └── GraphTabs (Tab navigation)
│
├── GraphNodes (List view alternative)
│
├── TabContents (Individual node editors)
│   ├── Decision Table Editor
│   ├── Function Editor
│   ├── Expression Editor
│   ├── Input Editor
│   └── Output Editor
│
└── GraphPanel (Right panel for simulator/custom panels)
```

---

### 4. Graph (graph/graph.tsx)
**Location**: `packages/jdm-editor/src/components/decision-graph/graph/graph.tsx`

**Purpose**: Core React Flow canvas component that handles all graph interactions.

**Key Features**:

#### Node Management
- **addNodeInner()**: Creates and validates new nodes
  - Validates node types
  - Handles custom nodes
  - Enforces single input node rule
  - Calls `onNodeAdd` hooks
  - Validates against schema

#### Edge Management
- **isValidConnection()**: Validates edge connections
  - Prevents self-references
  - Checks for cycles using DFS
  - Prevents duplicate edges
  - Respects disabled state

#### Drag and Drop
- **onDrop()**: Handles component drops from palette
  - Calculates correct position
  - Handles JSON node data
  - Creates nodes at drop location

#### Keyboard Shortcuts
- `Cmd/Ctrl + C`: Copy selected nodes
- `Cmd/Ctrl + D`: Duplicate selected nodes
- `Cmd/Ctrl + V`: Paste nodes
- `Backspace`: Delete selected nodes/edges (with confirmation)

#### Graph Controls
- Zoom controls
- Compact mode toggle
- Background grid
- Pan and zoom

---

### 5. Node System

#### Node Specification Architecture
**Location**: `packages/jdm-editor/src/components/decision-graph/nodes/specifications/`

Every node type implements the `NodeSpecification` interface:

```typescript
type NodeSpecification<T = any> = {
  type: string;              // Unique node type identifier
  icon?: React.ReactNode;    // Icon displayed in UI
  color?: string;            // Node color theme
  displayName: string;       // Human-readable name
  group?: string;            // Category for grouping
  documentationUrl?: string; // Help documentation link
  shortDescription?: string; // Tooltip description

  // Node creation
  generateNode: (params: GenerateNodeParams) => Omit<DecisionNode<T>, 'position' | 'id' | 'type'>;

  // Rendering
  renderNode: React.FC<MinimalNodeProps>;
  renderTab?: React.FC<{ id: string }>;
  renderSettings?: React.FC<{ id: string }>;

  // Type inference
  inferTypes?: {
    needsUpdate: (content: T, prevContent: T) => boolean;
    determineOutputType: (state: InferTypeData<T>) => VariableType;
  };

  // Diff support
  getDiffContent?: (current: T, previous: T) => T;

  // Lifecycle
  onNodeAdd?: (node: DecisionNode<T>) => Promise<DecisionNode<T>>;
};
```

#### Built-in Node Types (NodeKind)
1. **Input Node** (`inputNode`): Entry point for the graph
2. **Output Node** (`outputNode`): Exit point and result definition
3. **Decision Table** (`decisionTableNode`): Rule-based decision logic
4. **Function** (`functionNode`): Custom code execution
5. **Expression** (`expressionNode`): Simple expression evaluation
6. **Switch** (`switchNode`): Conditional branching

#### Custom Nodes
Custom nodes use the `CustomNodeSpecification` interface and can be created using the `createJdmNode()` helper:

```typescript
const myCustomNode = createJdmNode({
  kind: 'myNode',
  displayName: 'My Custom Node',
  icon: <CustomIcon />,
  color: '#FF5733',
  generateNode: ({ index }) => ({
    name: `MyNode${index}`,
    config: { /* initial config */ }
  }),
  inputs: [
    { name: 'enabled', control: 'bool', label: 'Enable feature' },
    { name: 'template', control: 'text', label: 'Template' }
  ]
});
```

---

### 6. GraphNode Component (nodes/graph-node.tsx)
**Location**: `packages/jdm-editor/src/components/decision-graph/nodes/graph-node.tsx`

**Purpose**: Wrapper component for all node types, providing consistent UI and behavior.

**Features**:
- **Handles**: Left and right connection points
- **Context Menu**: Right-click actions (copy, duplicate, delete, documentation)
- **Status Indicators**: Success/error states from simulation
- **Diff Indicators**: Visual diff status (added/modified/removed/moved)
- **Settings Panel**: Expandable settings for node configuration
- **Compact Mode**: Reduced UI for better overview
- **Node Selection**: Click handling with multi-select support (Cmd/Ctrl)

---

## Data Flow

### 1. Graph Data Structure

```typescript
type DecisionGraphType = {
  nodes: DecisionNode[];
  edges: DecisionEdge[];
};

type DecisionNode<T = any> = {
  id: string;                    // Unique identifier
  name: string;                  // Display name
  description?: string;          // Optional description
  type?: string;                 // Node type (NodeKind)
  content?: T;                   // Type-specific content
  position: Position;            // X/Y coordinates
  [privateSymbol]?: {            // Internal data (not persisted)
    dimensions?: { height?: number; width?: number };
    selected?: boolean;
  };
  _diff?: DiffMetadata;          // Diff information for comparisons
};

type DecisionEdge = {
  id: string;                    // Unique identifier
  name?: string;                 // Optional label
  sourceId: string;              // Source node ID
  targetId: string;              // Target node ID
  sourceHandle?: string;         // Source connection point
  targetHandle?: string;         // Target connection point
  type?: string;                 // Edge type
  _diff?: { status: DiffStatus }; // Diff status
};
```

### 2. Data Transformation Flow

```
User Action
    ↓
Actions (dg-store.context.tsx)
    ↓
Immer Producer (immutable update)
    ↓
Update Zustand Stores
    ↓
    ├→ Update DecisionGraph (internal format)
    ├→ Update React Flow State (via mapToGraphNode/Edge)
    └→ Trigger onChange listener
        ↓
    Parent component receives update
```

### 3. Mapping Functions (dg-util.ts)

**Purpose**: Convert between internal format and React Flow format.

#### DecisionNode → React Flow Node
```typescript
mapToGraphNode(node: DecisionNode): Node {
  return {
    id: node.id,
    type: node.type,
    position: node.position,
    height: node[privateSymbol]?.dimensions?.height,
    width: node[privateSymbol]?.dimensions?.width,
    selected: node[privateSymbol]?.selected,
    data: {
      name: node.name,
      kind: node?.content?.kind,
    },
  };
}
```

#### DecisionEdge → React Flow Edge
```typescript
mapToGraphEdge(edge: DecisionEdge): Edge {
  return {
    id: edge.id,
    source: edge.sourceId,
    type: edge?.type || 'edge',
    target: edge.targetId,
    label: edge.name,
    sourceHandle: edge.sourceHandle,
    targetHandle: edge.targetHandle,
    markerEnd: { type: MarkerType.ArrowClosed },
  };
}
```

---

## State Management

### State Update Pattern

All state updates follow this pattern:

1. **Action Called**: User triggers an action (e.g., `updateNode()`)
2. **Produce Draft**: Use Immer to create a draft of the state
3. **Modify Draft**: Make changes to the draft
4. **Update Store**: Apply changes to Zustand store
5. **Sync React Flow**: Update React Flow's internal state
6. **Notify Listeners**: Trigger `onChange` callback

### Example: Node Update Flow

```typescript
updateNode: (id, updater) => {
  const { decisionGraph } = stateStore.getState();
  const { nodesState } = referenceStore.getState();
  const [nodes, setNodes] = nodesState.current;

  // 1. Update internal state using Immer
  const newDecisionGraph = produce(decisionGraph, (draft) => {
    const node = draft.nodes.find((node) => node?.id === id);
    if (!node) return;
    updater(node); // Apply user changes
  });

  // 2. Map to React Flow format
  const changedNode = newDecisionGraph.nodes.find((n) => n.id === id);
  const graphChangedNode = mapToGraphNode(changedNode);

  // 3. Update React Flow state if changed
  const existingGraphNode = nodes.find((n) => n.id === id);
  if (!equal(graphChangedNode, existingGraphNode)) {
    setNodes((nodes) => nodes.map((n) => (n.id === id ? graphChangedNode : n)));
  }

  // 4. Update store and notify
  stateStore.setState({ decisionGraph: newDecisionGraph });
  listenerStore.getState().onChange?.(newDecisionGraph);
}
```

### Performance Optimizations

1. **Separate Stores**: State, listeners, and references in separate stores to minimize re-renders
2. **Equality Checking**: Uses `fast-deep-equal` to prevent unnecessary updates
3. **React.memo**: All node components are memoized
4. **Selective Updates**: Only changed nodes are re-rendered
5. **Private Symbol**: Non-serializable data stored separately

---

## Graph Rendering

### React Flow Integration

#### Node Types Registration
```typescript
const nodeTypes = {
  [NodeKind.Input]: InputNodeComponent,
  [NodeKind.Output]: OutputNodeComponent,
  [NodeKind.DecisionTable]: DecisionTableNodeComponent,
  [NodeKind.Function]: FunctionNodeComponent,
  [NodeKind.Expression]: ExpressionNodeComponent,
  [NodeKind.Switch]: SwitchNodeComponent,
  customNode: CustomNodeRenderer,
};
```

#### Edge Types
```typescript
const edgeTypes = {
  edge: CustomEdge, // Bezier curve with delete button on hover
};
```

### Custom Edge Component (custom-edge.tsx)

**Features**:
- Bezier path rendering
- Delete button on hover
- Diff status coloring (added = green, removed = red)
- Disabled state support

---

## Simulation System

### Architecture

The simulation system allows users to test their decision graphs with input data and see the execution trace.

**Location**: `packages/jdm-editor/src/components/decision-graph/simulator/`

### Components

#### 1. GraphSimulator (dg-simulator.tsx)
Main simulator UI with three panels:
- **Request Panel** (left): Input data editor
- **Nodes Panel** (middle): List of executed nodes with performance metrics
- **Response Panel** (right): Output/input/trace data viewer

#### 2. Simulation Flow
```
User Input
    ↓
SimulatorRequestPanel
    ↓
onRun() callback
    ↓
External execution (zen-engine)
    ↓
Simulation Result
    ↓
Update state.simulate
    ↓
GraphSimulator displays results
    ↓
    ├→ Node trace list (execution order)
    ├→ Performance metrics per node
    ├→ Success/error indicators
    └→ Detailed input/output/trace data
```

### Simulation Data Structure

```typescript
type Simulation = {
  result?: {
    result: any;           // Final output
    trace: Record<string, SimulationTrace>; // Per-node execution data
    performance: string;   // Total execution time
  };
  error?: {
    message: string;
    data?: {
      nodeId: string;      // Node where error occurred
    };
  };
};

type SimulationTrace = {
  name: string;            // Node name
  order: number;           // Execution order
  input: any;              // Node input data
  output: any;             // Node output data
  trace?: any;             // Detailed trace info
  performance: string;     // Node execution time
};
```

### Trace Visualization

- **Success Nodes**: Green checkmark icon
- **Error Nodes**: Red X icon
- **Node Performance**: Displayed next to each node
- **Navigation**: Double-click node in trace list to navigate to it on canvas
- **Filtering**: Search box to filter nodes by name

---

## Key Design Patterns

### 1. Provider Pattern
- `DecisionGraphProvider` wraps the entire component tree
- Provides access to state, actions, and listeners via context
- Custom hooks for accessing specific parts: `useDecisionGraphState()`, `useDecisionGraphActions()`

### 2. Specification Pattern
- Node types defined as specifications
- Common interface for all node types
- Easy to extend with new node types
- Separation of concerns (data, rendering, behavior)

### 3. Compound Component Pattern
- `DecisionGraph` composes multiple sub-components
- Each component has a specific responsibility
- Components communicate via shared context

### 4. Command Pattern
- Actions in the store act as commands
- Encapsulate operations on the graph
- Provide imperative API for external control

### 5. Observer Pattern
- `onChange` listener for graph changes
- `onPanelsChange` for panel state
- Enables integration with external state management

### 6. Adapter Pattern
- Mapping functions between internal and React Flow formats
- `mapToGraphNode()` / `mapToDecisionEdge()`
- Allows independent evolution of data structures

### 7. Factory Pattern
- `generateNode()` in specifications
- Creates nodes with default values
- Handles node initialization logic

### 8. Ref Forwarding Pattern
- `DecisionGraph` forwards ref to provide imperative API
- Allows parent components to control the graph programmatically

---

## Performance Considerations

### 1. Re-render Optimization
- All node components are memoized with custom comparison
- Only re-render when `id`, `selected`, or `data` changes
- Zustand stores prevent unnecessary context re-renders

### 2. Large Graphs
- React Flow handles virtualization
- Only visible nodes are rendered
- Smooth performance with 100+ nodes

### 3. State Updates
- Immer for efficient immutable updates
- Batch updates where possible
- Reference equality checks prevent cascading updates

### 4. Memory Management
- Private symbol data not serialized
- Refs used for mutable data
- Cleanup on unmount

---

## Extension Points

### 1. Custom Node Types
Implement `CustomNodeSpecification` or use `createJdmNode()` helper.

### 2. Custom Panels
Add to `panels` prop in state store.

### 3. Custom Actions
Add to node context menu via `actions` prop on `GraphNode`.

### 4. Type Inference
Implement `inferTypes` in node specification for automatic type propagation.

### 5. Diff Support
Implement `getDiffContent()` for version comparison support.

### 6. Lifecycle Hooks
- `onNodeAdd()`: Called when node is added
- `onChange()`: Called on any graph change
- `onReactFlowInit()`: Called when React Flow initializes

---

## Common Patterns and Best Practices

### 1. Accessing Graph State
```typescript
// In a component inside DecisionGraphProvider
const myData = useDecisionGraphState((state) => ({
  nodes: state.decisionGraph.nodes,
  activeTab: state.activeTab,
}));
```

### 2. Modifying Graph
```typescript
const actions = useDecisionGraphActions();

// Add a node
actions.addNodes([newNode]);

// Update a node
actions.updateNode(nodeId, (draft) => {
  draft.name = 'New Name';
  draft.content.someField = 'value';
  return draft;
});
```

### 3. Creating a Custom Node
```typescript
const myNode: CustomNodeSpecification<MyConfig, 'myNode'> = {
  kind: 'myNode',
  displayName: 'My Node',
  color: '#FF5733',
  icon: <IconComponent />,
  generateNode: ({ index }) => ({
    name: `MyNode${index}`,
    config: { /* defaults */ },
  }),
  renderNode: ({ id, specification, data, selected }) => (
    <GraphNode
      id={id}
      specification={specification}
      name={data.name}
      isSelected={selected}
    >
      {/* Custom UI */}
    </GraphNode>
  ),
};
```

### 4. Tab Rendering
```typescript
renderTab: ({ id }) => {
  const node = useDecisionGraphState(
    (state) => state.decisionGraph.nodes.find((n) => n.id === id)
  );

  return (
    <div>
      {/* Custom editor for this node type */}
    </div>
  );
}
```

---

## Troubleshooting Common Issues

### Issue: Node not updating
**Cause**: Direct mutation of state
**Solution**: Always use Immer or return a new object

### Issue: Edges not connecting
**Cause**: `isValidConnection()` returning false
**Solution**: Check for cycles, duplicates, and disabled state

### Issue: Performance degradation
**Cause**: Too many re-renders
**Solution**: Use selectors in `useDecisionGraphState()`, ensure proper memoization

### Issue: Tab not opening
**Cause**: Node type doesn't implement `renderTab()`
**Solution**: Add `renderTab` to node specification

---

## File Structure Reference

```
decision-graph/
├── dg.tsx                          # Root component
├── dg-types.ts                     # Type definitions
├── dg-util.ts                      # Utility functions (mapping)
├── dg-wrapper.tsx                  # Main layout orchestrator
├── dg-panel.tsx                    # Side panel container
├── dg-empty.tsx                    # Empty state component
├── dg-infer.tsx                    # Type inference system
├── custom-edge.tsx                 # Custom edge component
├── index.ts                        # Public exports
├── context/
│   └── dg-store.context.tsx        # State management (Zustand)
├── graph/
│   ├── graph.tsx                   # Main canvas component
│   ├── graph-tabs.tsx              # Tab navigation
│   ├── graph-nodes.tsx             # List view alternative
│   ├── graph-components.tsx        # Component palette
│   ├── graph-side-toolbar.tsx      # Left toolbar
│   ├── tab-*.tsx                   # Individual tab editors
│   └── json-to-json-schema-dialog.tsx
├── nodes/
│   ├── graph-node.tsx              # Node wrapper component
│   ├── decision-node.tsx           # Base node UI
│   ├── graph-card.tsx              # Node card layout
│   ├── custom-node/
│   │   ├── index.tsx               # Custom node implementation
│   │   └── controls.ts             # Custom node controls
│   └── specifications/
│       ├── specifications.tsx      # Node registry
│       ├── specification-types.ts  # Type definitions
│       ├── colors.ts               # Color constants
│       ├── input.specification.tsx
│       ├── output.specification.tsx
│       ├── decision-table.specification.tsx
│       ├── function.specification.tsx
│       ├── expression.specification.tsx
│       └── switch.specification.tsx
├── simulator/
│   ├── dg-simulator.tsx            # Main simulator UI
│   ├── simulator-editor.tsx        # Code editor for I/O
│   ├── simulator-request-panel.tsx # Input panel
│   └── simulation.types.ts         # Type definitions
├── hooks/
│   └── use-graph-clipboard.ts      # Clipboard operations
└── diff/
    ├── comparison.ts               # Diff algorithm
    └── utility.ts                  # Diff utilities
```