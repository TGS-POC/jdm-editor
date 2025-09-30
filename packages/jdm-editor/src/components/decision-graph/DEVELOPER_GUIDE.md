# Decision Graph Developer Guide

## Table of Contents
1. [Getting Started](#getting-started)
2. [Basic Usage](#basic-usage)
3. [Creating Custom Nodes](#creating-custom-nodes)
4. [Working with the Graph](#working-with-the-graph)
5. [State Management](#state-management)
6. [Tab Editors](#tab-editors)
7. [Type Inference](#type-inference)
8. [Simulation Integration](#simulation-integration)
9. [Styling and Theming](#styling-and-theming)
10. [Advanced Topics](#advanced-topics)
11. [Code Examples](#code-examples)
12. [API Reference](#api-reference)

---

## Getting Started

### Installation

The Decision Graph is part of the `jdm-editor` package. Ensure you have all dependencies installed:

```bash
npm install
# or
pnpm install
```

### Basic Integration

```tsx
import { DecisionGraph } from '@/components/decision-graph';
import type { DecisionGraphType } from '@/components/decision-graph/dg-types';

function MyApp() {
  const [graph, setGraph] = useState<DecisionGraphType>({
    nodes: [],
    edges: [],
  });

  return (
    <DecisionGraph
      value={graph}
      onChange={setGraph}
    />
  );
}
```

---

## Basic Usage

### Controlled Component

The Decision Graph is a controlled component, meaning you manage the state:

```tsx
function ControlledExample() {
  const [graph, setGraph] = useState<DecisionGraphType>({
    nodes: [
      {
        id: '1',
        type: 'inputNode',
        name: 'Input',
        position: { x: 100, y: 100 },
        content: { schema: {} },
      },
      {
        id: '2',
        type: 'outputNode',
        name: 'Output',
        position: { x: 400, y: 100 },
        content: { schema: {} },
      },
    ],
    edges: [
      {
        id: 'e1-2',
        sourceId: '1',
        targetId: '2',
      },
    ],
  });

  const handleChange = (newGraph: DecisionGraphType) => {
    console.log('Graph changed:', newGraph);
    setGraph(newGraph);
  };

  return (
    <DecisionGraph
      value={graph}
      onChange={handleChange}
    />
  );
}
```

### Imperative API

Access the graph programmatically using refs:

```tsx
function ImperativeExample() {
  const graphRef = useRef<DecisionGraphRef>(null);

  const addNode = () => {
    graphRef.current?.addNodes([{
      id: crypto.randomUUID(),
      type: 'decisionTableNode',
      name: 'New Decision Table',
      position: { x: 200, y: 200 },
      content: {
        rules: [],
        inputs: [],
        outputs: [],
      },
    }]);
  };

  const goToNode = (nodeId: string) => {
    graphRef.current?.goToNode(nodeId);
  };

  return (
    <>
      <button onClick={addNode}>Add Node</button>
      <DecisionGraph ref={graphRef} />
    </>
  );
}
```

---

## Creating Custom Nodes

### Using createJdmNode Helper

The easiest way to create a custom node:

```tsx
import { createJdmNode } from '@/components/decision-graph/nodes/custom-node';

const emailNode = createJdmNode({
  kind: 'emailNode',
  displayName: 'Email Sender',
  color: '#4CAF50',
  icon: <MailOutlined />,
  shortDescription: 'Send an email notification',
  group: 'Notifications',

  inputs: [
    { name: 'to', control: 'text', label: 'To Address' },
    { name: 'subject', control: 'text', label: 'Subject' },
    { name: 'body', control: 'text', label: 'Email Body' },
    { name: 'urgent', control: 'bool', label: 'Mark as Urgent' },
  ],

  generateNode: ({ index }) => ({
    name: `Email${index}`,
    config: {
      to: '',
      subject: '',
      body: '',
      urgent: false,
    },
  }),
});

// Use in DecisionGraph
<DecisionGraph
  customNodes={[emailNode]}
  // ... other props
/>
```

### Advanced Custom Node

For full control, implement the `CustomNodeSpecification` interface:

```tsx
import type { CustomNodeSpecification } from '@/components/decision-graph/nodes/custom-node';
import { GraphNode } from '@/components/decision-graph/nodes/graph-node';

type MyNodeConfig = {
  apiUrl: string;
  method: 'GET' | 'POST';
  headers: Record<string, string>;
};

const apiNode: CustomNodeSpecification<MyNodeConfig, 'apiNode'> = {
  kind: 'apiNode',
  displayName: 'API Call',
  color: '#2196F3',
  icon: <ApiOutlined />,
  documentationUrl: 'https://docs.example.com/api-node',

  generateNode: ({ index }) => ({
    name: `API${index}`,
    config: {
      apiUrl: '',
      method: 'GET',
      headers: {},
    },
  }),

  renderNode: ({ id, specification, data, selected }) => {
    const { updateNode } = useDecisionGraphActions();
    const node = useDecisionGraphState(
      (state) => state.decisionGraph.nodes.find((n) => n.id === id)
    );

    const config = node?.content?.config as MyNodeConfig;

    return (
      <GraphNode
        id={id}
        specification={specification}
        name={data.name}
        isSelected={selected}
        handleLeft
        handleRight
      >
        <div style={{ padding: 8 }}>
          <div>Method: {config?.method}</div>
          <div>URL: {config?.apiUrl || 'Not set'}</div>
        </div>
      </GraphNode>
    );
  },

  renderTab: ({ id }) => {
    const { updateNode } = useDecisionGraphActions();
    const node = useDecisionGraphState(
      (state) => state.decisionGraph.nodes.find((n) => n.id === id)
    );

    const config = node?.content?.config as MyNodeConfig;

    return (
      <div style={{ padding: 16 }}>
        <Form
          initialValues={config}
          onValuesChange={(_, values) => {
            updateNode(id, (draft) => {
              draft.content.config = values;
              return draft;
            });
          }}
        >
          <Form.Item name="apiUrl" label="API URL">
            <Input />
          </Form.Item>
          <Form.Item name="method" label="HTTP Method">
            <Select>
              <Select.Option value="GET">GET</Select.Option>
              <Select.Option value="POST">POST</Select.Option>
            </Select>
          </Form.Item>
        </Form>
      </div>
    );
  },

  onNodeAdd: async (node) => {
    // Validate or transform the node before it's added
    if (!node.content?.config?.apiUrl) {
      throw new Error('API URL is required');
    }
    return node;
  },
};
```

### Custom Node with Type Inference

Add automatic type propagation:

```tsx
const transformNode: CustomNodeSpecification<TransformConfig, 'transformNode'> = {
  // ... other properties

  inferTypes: {
    needsUpdate: (state, prevState) => {
      // Return true if output type needs recalculation
      return state.content.transformType !== prevState.content.transformType;
    },

    determineOutputType: ({ input, content }) => {
      // Return the output type based on input type and node config
      if (content.transformType === 'toArray') {
        return {
          type: 'array',
          items: input,
        };
      }
      return input; // Pass through
    },
  },
};
```

---

## Working with the Graph

### Accessing State in Components

```tsx
import { useDecisionGraphState, useDecisionGraphActions } from '@/components/decision-graph/context/dg-store.context';

function MyComponent() {
  // Select only the data you need
  const { nodes, activeTab } = useDecisionGraphState((state) => ({
    nodes: state.decisionGraph.nodes,
    activeTab: state.activeTab,
  }));

  const actions = useDecisionGraphActions();

  return (
    <div>
      <button onClick={() => actions.openTab('graph')}>
        Open Graph
      </button>
      <div>Total nodes: {nodes.length}</div>
    </div>
  );
}
```

### Common Operations

#### Adding Nodes

```tsx
const actions = useDecisionGraphActions();

// Add a single node
actions.addNodes([{
  id: crypto.randomUUID(),
  type: 'expressionNode',
  name: 'Calculate Total',
  position: { x: 300, y: 300 },
  content: {
    key: 'total',
    value: 'price * quantity',
  },
}]);

// Add multiple nodes at once
actions.addNodes([node1, node2, node3]);
```

#### Updating Nodes

```tsx
// Update node properties
actions.updateNode(nodeId, (draft) => {
  draft.name = 'New Name';
  draft.description = 'Updated description';
  return draft;
});

// Update node content
actions.updateNode(nodeId, (draft) => {
  draft.content.someField = 'new value';
  draft.content.nested.value = 123;
  return draft;
});
```

#### Removing Nodes

```tsx
// Remove single node
actions.removeNodes(['node-id']);

// Remove multiple nodes
actions.removeNodes(['id1', 'id2', 'id3']);
```

#### Working with Edges

```tsx
// Add an edge
actions.addEdges([{
  id: crypto.randomUUID(),
  sourceId: 'node1',
  targetId: 'node2',
  sourceHandle: 'output',
  targetHandle: 'input',
}]);

// Remove edges
actions.removeEdges(['edge-id']);

// Remove edge by handle
actions.removeEdgeByHandleId('handle-id');
```

#### Clipboard Operations

```tsx
// Copy nodes
const selectedNodeIds = ['id1', 'id2'];
actions.copyNodes(selectedNodeIds);

// Paste nodes
actions.pasteNodes();

// Duplicate nodes
actions.duplicateNodes(selectedNodeIds);
```

#### Navigation

```tsx
// Open a tab for a node
actions.openTab(nodeId);

// Close a tab
actions.closeTab(nodeId);

// Navigate to node on canvas
actions.goToNode(nodeId);

// Close all tabs
actions.closeTab('any-id', 'close-all');

// Close tabs to the right
actions.closeTab(nodeId, 'close-right');
```

---

## State Management

### Understanding the Store Architecture

The Decision Graph uses three Zustand stores:

1. **State Store**: Core application state
2. **Listener Store**: Callbacks and event handlers
3. **Reference Store**: Mutable references

### Custom Hooks

#### useDecisionGraphState

```tsx
// Basic usage
const nodes = useDecisionGraphState((state) => state.decisionGraph.nodes);

// Multiple values
const { nodes, edges, activeTab } = useDecisionGraphState((state) => ({
  nodes: state.decisionGraph.nodes,
  edges: state.decisionGraph.edges,
  activeTab: state.activeTab,
}));

// Computed values
const nodeCount = useDecisionGraphState(
  (state) => state.decisionGraph.nodes.length
);

// With custom equality function
const node = useDecisionGraphState(
  (state) => state.decisionGraph.nodes.find((n) => n.id === nodeId),
  (a, b) => a?.id === b?.id && a?.name === b?.name
);
```

#### useDecisionGraphActions

```tsx
const actions = useDecisionGraphActions();

// All actions are available
actions.addNodes([...]);
actions.updateNode(...);
actions.removeNodes([...]);
// etc.
```

#### useDecisionGraphListeners

```tsx
const { onChange } = useDecisionGraphListeners((state) => ({
  onChange: state.onChange,
}));

// Usually used internally, but can be accessed if needed
```

#### useDecisionGraphRaw

```tsx
// Access all stores and actions
const { stateStore, listenerStore, referenceStore, actions } = useDecisionGraphRaw();

// Useful for advanced scenarios
const reactFlowInstance = referenceStore((state) => state.reactFlowInstance);
```

### Special Hooks

#### useNodeDiff

```tsx
import { useNodeDiff } from '@/components/decision-graph/context/dg-store.context';

function MyNodeComponent({ id }: { id: string }) {
  const { diff, contentDiff } = useNodeDiff(id);

  return (
    <div>
      {diff?.status === 'added' && <Badge>New</Badge>}
      {diff?.status === 'modified' && <Badge>Modified</Badge>}
    </div>
  );
}
```

#### useEdgeDiff

```tsx
import { useEdgeDiff } from '@/components/decision-graph/context/dg-store.context';

function MyEdgeComponent({ id }: { id: string }) {
  const { diff } = useEdgeDiff(id);

  return (
    <div style={{
      color: diff?.status === 'added' ? 'green' : 'inherit'
    }} />
  );
}
```

---

## Tab Editors

### Creating a Tab Editor

Tab editors are full-screen editors for specific node types. They render when a node is double-clicked or opened from the tabs.

```tsx
const myNodeSpec: NodeSpecification<MyContent> = {
  // ... other properties

  renderTab: ({ id, manager }) => {
    const { updateNode } = useDecisionGraphActions();
    const node = useDecisionGraphState(
      (state) => state.decisionGraph.nodes.find((n) => n.id === id)
    );

    if (!node) return null;

    const content = node.content as MyContent;

    return (
      <div className="my-tab-editor">
        {/* Your custom editor UI */}
        <MyEditor
          value={content}
          onChange={(newContent) => {
            updateNode(id, (draft) => {
              draft.content = newContent;
              return draft;
            });
          }}
        />
      </div>
    );
  },
};
```

### Example: Decision Table Tab

```tsx
renderTab: ({ id }) => {
  const { updateNode } = useDecisionGraphActions();
  const node = useDecisionGraphState(
    (state) => state.decisionGraph.nodes.find((n) => n.id === id)
  );

  const content = node?.content as DecisionTableContent;

  return (
    <DecisionTableEditor
      inputs={content.inputs}
      outputs={content.outputs}
      rules={content.rules}
      onInputsChange={(inputs) => {
        updateNode(id, (draft) => {
          draft.content.inputs = inputs;
          return draft;
        });
      }}
      onOutputsChange={(outputs) => {
        updateNode(id, (draft) => {
          draft.content.outputs = outputs;
          return draft;
        });
      }}
      onRulesChange={(rules) => {
        updateNode(id, (draft) => {
          draft.content.rules = rules;
          return draft;
        });
      }}
    />
  );
},
```

---

## Type Inference

Type inference allows nodes to automatically determine their output types based on input types and configuration.

### Implementing Type Inference

```tsx
const myNode: NodeSpecification<MyContent> = {
  // ... other properties

  inferTypes: {
    // Determine if type inference needs to run
    needsUpdate: (content, prevContent) => {
      return (
        content.operation !== prevContent.operation ||
        content.field !== prevContent.field
      );
    },

    // Calculate the output type
    determineOutputType: ({ input, content }) => {
      // input: VariableType from zen-engine-wasm
      // content: Your node's content

      if (content.operation === 'filter') {
        // If filtering an array, output is also an array
        if (input.type === 'array') {
          return input; // Same type
        }
        return { type: 'null' }; // Error case
      }

      if (content.operation === 'extract') {
        // Extract a field from an object
        if (input.type === 'object' && input.properties) {
          const fieldType = input.properties[content.field];
          return fieldType || { type: 'any' };
        }
      }

      // Default: pass through
      return input;
    },
  },
};
```

### Type Inference System

The type inference system runs automatically in the background:

1. **Trigger**: When node content changes (determined by `needsUpdate`)
2. **Propagation**: Types flow from Input node → through graph → to Output node
3. **Calculation**: Each node's `determineOutputType` is called
4. **Storage**: Types stored in `state.nodeTypes`
5. **Usage**: Types available for validation, UI hints, etc.

### Accessing Inferred Types

```tsx
const { nodeTypes } = useDecisionGraphState((state) => ({
  nodeTypes: state.nodeTypes,
}));

// Get input type for a node
const inputType = nodeTypes[nodeId]?.[NodeTypeKind.Input];

// Get output type for a node
const outputType = nodeTypes[nodeId]?.[NodeTypeKind.Output];

// Get inferred input type (from incoming edges)
const inferredInput = nodeTypes[nodeId]?.[NodeTypeKind.InferredInput];

// Get inferred output type (from node logic)
const inferredOutput = nodeTypes[nodeId]?.[NodeTypeKind.InferredOutput];
```

---

## Simulation Integration

### Setting Up Simulation

```tsx
import { DecisionGraph } from '@/components/decision-graph';

function SimulatorExample() {
  const [graph, setGraph] = useState<DecisionGraphType>({
    nodes: [],
    edges: [],
  });

  const [simulationResult, setSimulationResult] = useState<Simulation>();

  const handleSimulate = async (request: any) => {
    try {
      // Execute the graph with your decision engine
      const result = await executeGraph(graph, request);

      // Update simulation state
      setSimulationResult({
        result: {
          result: result.output,
          trace: result.trace,
          performance: result.performance,
        },
      });
    } catch (error) {
      setSimulationResult({
        error: {
          message: error.message,
          data: { nodeId: error.nodeId },
        },
      });
    }
  };

  return (
    <DecisionGraph
      value={graph}
      onChange={setGraph}
      simulate={simulationResult}
      panels={[
        {
          id: 'simulator',
          icon: <PlayCircleOutlined />,
          title: 'Simulator',
          renderPanel: () => (
            <GraphSimulator
              onRun={handleSimulate}
              onClear={() => setSimulationResult(undefined)}
            />
          ),
        },
      ]}
    />
  );
}
```

### Simulation Data Structure

```typescript
type Simulation = {
  result?: {
    result: any;                              // Final output
    trace: Record<string, {                   // Per-node trace
      name: string;
      order: number;
      input: any;
      output: any;
      trace?: any;
      performance: string;
    }>;
    performance: string;                      // Total time
  };
  error?: {
    message: string;
    data?: {
      nodeId: string;                         // Node where error occurred
    };
  };
};
```

### Custom Simulator Panel

```tsx
const MySimulator: React.FC = () => {
  const { simulate } = useDecisionGraphState((state) => ({
    simulate: state.simulate,
  }));

  return (
    <div>
      {simulate?.error ? (
        <Alert type="error" message={simulate.error.message} />
      ) : simulate?.result ? (
        <div>
          <h3>Result</h3>
          <pre>{JSON.stringify(simulate.result.result, null, 2)}</pre>

          <h3>Performance</h3>
          <div>{simulate.result.performance}</div>

          <h3>Trace</h3>
          {Object.entries(simulate.result.trace).map(([id, trace]) => (
            <div key={id}>
              {trace.name}: {trace.performance}
            </div>
          ))}
        </div>
      ) : (
        <div>No simulation results</div>
      )}
    </div>
  );
};
```

---

## Styling and Theming

### Custom Node Colors

```tsx
import { NodeColor } from '@/components/decision-graph/nodes/specifications/colors';

const myNode = createJdmNode({
  kind: 'myNode',
  color: NodeColor.Green, // Use predefined colors
  // or
  color: '#FF5733',       // Use custom hex color
  // ...
});
```

### Available Colors

```typescript
export enum NodeColor {
  Blue = '#1677FF',
  Purple = '#9333EA',
  Green = '#52C41A',
  Orange = '#FA8C16',
  Red = '#FF4D4F',
  Cyan = '#13C2C2',
  Pink = '#EB2F96',
  Yellow = '#FAAD14',
}
```

### Custom Styles

Add custom CSS classes to your nodes:

```tsx
<GraphNode
  id={id}
  specification={specification}
  className="my-custom-node"
  // ...
/>
```

```css
.my-custom-node {
  border: 2px solid #ff0000;
  border-radius: 8px;
}

.my-custom-node .grl-dn__header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}
```

### Compact Mode

The graph supports a compact mode that reduces node sizes:

```tsx
const actions = useDecisionGraphActions();

// Toggle compact mode
actions.toggleCompactMode();

// Set compact mode explicitly
actions.setCompactMode(true);

// Check current mode
const { compactMode } = useDecisionGraphState((state) => ({
  compactMode: state.compactMode,
}));
```

---

## Advanced Topics

### Custom Panels

Add custom side panels to the decision graph:

```tsx
<DecisionGraph
  panels={[
    {
      id: 'my-panel',
      icon: <SettingOutlined />,
      title: 'My Custom Panel',
      renderPanel: () => <MyCustomPanel />,
    },
    {
      id: 'another-panel',
      icon: <DatabaseOutlined />,
      title: 'Data Browser',
      renderPanel: () => <DataBrowser />,
    },
  ]}
  activePanel="my-panel"  // Optional: set default active panel
  onPanelsChange={(panelId) => {
    console.log('Active panel:', panelId);
  }}
/>
```

### View Config (Permissions)

Control what users can see and edit:

```tsx
<DecisionGraph
  viewConfig={{
    enabled: true,
    description: 'Limited view mode',
    permissions: {
      'node-id-1': 'edit:values',  // Can edit values only
      'node-id-2': 'edit:rules',   // Can edit rules only
      'node-id-3': 'edit:full',    // Full edit access
      'node-id-4': null,           // No access (hidden)
    },
  }}
  viewConfigCta="Upgrade to edit all nodes"
  onViewConfigCta={() => {
    // Handle CTA click (e.g., show upgrade modal)
  }}
/>
```

### Diff Support

Compare two versions of a graph:

```tsx
import { compareGraphs } from '@/components/decision-graph/diff/comparison';

const oldGraph = { nodes: [...], edges: [...] };
const newGraph = { nodes: [...], edges: [...] };

const graphWithDiff = compareGraphs(oldGraph, newGraph);

// Now render with diff indicators
<DecisionGraph
  value={graphWithDiff}
  // Nodes and edges will show diff status (added/removed/modified/moved)
/>
```

### Custom Clipboard Behavior

The clipboard system can be customized via the `graphClipboard` reference:

```tsx
const MyComponent = () => {
  const { graphClipboard } = useDecisionGraphReferences((state) => ({
    graphClipboard: state.graphClipboard,
  }));

  const copyWithMetadata = () => {
    const nodes = [...]; // Get nodes to copy
    graphClipboard.current?.copyNodes(nodes, {
      metadata: { source: 'my-app', timestamp: Date.now() },
    });
  };
};
```

### Code Editor Extensions

Add custom Monaco Editor extensions:

```tsx
<DecisionGraph
  onCodeExtension={(monaco) => {
    // Add custom languages, themes, etc.
    monaco.languages.register({ id: 'my-lang' });
    monaco.languages.setMonarchTokensProvider('my-lang', {
      // ... tokenizer rules
    });
  }}
  onFunctionReady={(monaco) => {
    // Called when Monaco is ready in function node
    monaco.editor.defineTheme('my-theme', {
      // ... theme definition
    });
  }}
/>
```

### Custom Node Handles

Customize connection points on nodes:

```tsx
<GraphNode
  id={id}
  specification={specification}
  handleLeft={{
    id: 'custom-input',
    type: 'target',
    position: Position.Left,
    style: { background: '#ff0000' },
  }}
  handleRight={{
    id: 'custom-output-1',
    type: 'source',
    position: Position.Right,
    style: { top: '30%' },
  }}
/>

// Or use multiple handles
<GraphNode id={id} specification={specification}>
  <Handle id="input-1" type="target" position={Position.Left} style={{ top: '25%' }} />
  <Handle id="input-2" type="target" position={Position.Left} style={{ top: '75%' }} />

  <div>Node content</div>

  <Handle id="output" type="source" position={Position.Right} />
</GraphNode>
```

---

## Code Examples

### Complete Custom Node Example

```tsx
import { createJdmNode } from '@/components/decision-graph/nodes/custom-node';
import { useDecisionGraphActions, useDecisionGraphState } from '@/components/decision-graph/context/dg-store.context';
import { GraphNode } from '@/components/decision-graph/nodes/graph-node';

// Define the config type
type HttpRequestConfig = {
  url: string;
  method: 'GET' | 'POST' | 'PUT' | 'DELETE';
  headers: Record<string, string>;
  body?: string;
  timeout: number;
};

// Create the node specification
export const httpRequestNode = createJdmNode<'httpRequest', string, [
  { name: 'url'; control: 'text' },
  { name: 'method'; control: 'text' },
  { name: 'timeout'; control: 'text' }
]>({
  kind: 'httpRequest',
  displayName: 'HTTP Request',
  icon: <ApiOutlined />,
  color: '#1890ff',
  shortDescription: 'Make an HTTP request to an external API',
  group: 'Integration',
  documentationUrl: 'https://docs.example.com/http-request',

  handleLeft: true,
  handleRight: true,

  inputs: [
    { name: 'url', control: 'text', label: 'URL' },
    { name: 'method', control: 'text', label: 'Method' },
    { name: 'timeout', control: 'text', label: 'Timeout (ms)' },
  ],

  generateNode: ({ index }) => ({
    name: `HttpRequest${index}`,
    description: 'HTTP request node',
    config: {
      url: '',
      method: 'GET' as const,
      headers: {},
      timeout: 5000,
    },
  }),

  // Custom node renderer
  renderNode: ({ id, specification, data, selected }) => {
    const [expanded, setExpanded] = useState(false);
    const { updateNode } = useDecisionGraphActions();
    const node = useDecisionGraphState(
      (state) => state.decisionGraph.nodes.find((n) => n.id === id)
    );

    const config = node?.content?.config as HttpRequestConfig;

    return (
      <GraphNode
        id={id}
        specification={specification}
        name={data.name}
        isSelected={selected}
        handleLeft
        handleRight
        actions={[
          <Button
            key="expand"
            type="text"
            icon={expanded ? <UpOutlined /> : <DownOutlined />}
            onClick={() => setExpanded(!expanded)}
          />,
        ]}
      >
        {expanded && (
          <div style={{ padding: 8, fontSize: 12 }}>
            <div><strong>Method:</strong> {config?.method}</div>
            <div><strong>URL:</strong> {config?.url || 'Not configured'}</div>
            <div><strong>Timeout:</strong> {config?.timeout}ms</div>
          </div>
        )}
      </GraphNode>
    );
  },

  // Custom tab editor
  renderTab: ({ id }) => {
    const { updateNode } = useDecisionGraphActions();
    const node = useDecisionGraphState(
      (state) => state.decisionGraph.nodes.find((n) => n.id === id)
    );

    const config = node?.content?.config as HttpRequestConfig;

    return (
      <div style={{ padding: 24 }}>
        <Typography.Title level={4}>HTTP Request Configuration</Typography.Title>

        <Form
          layout="vertical"
          initialValues={config}
          onValuesChange={(_, values) => {
            updateNode(id, (draft) => {
              draft.content.config = values;
              return draft;
            });
          }}
        >
          <Form.Item name="url" label="URL" rules={[{ required: true }]}>
            <Input placeholder="https://api.example.com/endpoint" />
          </Form.Item>

          <Form.Item name="method" label="HTTP Method">
            <Radio.Group>
              <Radio value="GET">GET</Radio>
              <Radio value="POST">POST</Radio>
              <Radio value="PUT">PUT</Radio>
              <Radio value="DELETE">DELETE</Radio>
            </Radio.Group>
          </Form.Item>

          <Form.Item name="timeout" label="Timeout (ms)">
            <InputNumber min={0} max={60000} />
          </Form.Item>

          <Form.Item label="Headers">
            <KeyValueEditor
              value={config?.headers || {}}
              onChange={(headers) => {
                updateNode(id, (draft) => {
                  draft.content.config.headers = headers;
                  return draft;
                });
              }}
            />
          </Form.Item>

          <Form.Item name="body" label="Request Body">
            <CodeEditor type="json" />
          </Form.Item>
        </Form>
      </div>
    );
  },

  // Validation before adding
  onNodeAdd: async (node) => {
    const config = node.content?.config as HttpRequestConfig;

    if (!config.url) {
      Modal.confirm({
        title: 'URL not configured',
        content: 'Do you want to add this node without configuring the URL?',
        onOk: () => Promise.resolve(node),
        onCancel: () => Promise.reject(),
      });
    }

    return node;
  },

  // Type inference
  inferTypes: {
    needsUpdate: (state, prevState) => {
      return state.content.method !== prevState.content.method;
    },

    determineOutputType: ({ input, content }) => {
      // HTTP request always returns an object
      return {
        type: 'object',
        properties: {
          status: { type: 'number' },
          data: { type: 'any' },
          headers: { type: 'object' },
        },
      };
    },
  },
});
```

### Integration Example

```tsx
import { DecisionGraph, type DecisionGraphRef } from '@/components/decision-graph';
import { httpRequestNode } from './nodes/http-request-node';

function MyApplication() {
  const graphRef = useRef<DecisionGraphRef>(null);
  const [graph, setGraph] = useState<DecisionGraphType>({
    nodes: [],
    edges: [],
  });
  const [simulation, setSimulation] = useState<Simulation>();

  const handleSave = async () => {
    try {
      await api.saveGraph(graph);
      message.success('Graph saved successfully');
    } catch (error) {
      message.error('Failed to save graph');
    }
  };

  const handleSimulate = async (request: any) => {
    setSimulation(undefined); // Clear previous results

    try {
      const result = await api.executeGraph(graph, request);
      setSimulation({
        result: {
          result: result.output,
          trace: result.trace,
          performance: result.executionTime,
        },
      });
    } catch (error) {
      setSimulation({
        error: {
          message: error.message,
          data: error.nodeId ? { nodeId: error.nodeId } : undefined,
        },
      });
    }
  };

  return (
    <div style={{ height: '100vh', display: 'flex', flexDirection: 'column' }}>
      <div style={{ padding: 16, borderBottom: '1px solid #d9d9d9' }}>
        <Space>
          <Button icon={<SaveOutlined />} onClick={handleSave}>
            Save
          </Button>
          <Button
            icon={<PlusOutlined />}
            onClick={() => {
              graphRef.current?.addNodes([{
                id: crypto.randomUUID(),
                type: 'decisionTableNode',
                name: 'New Decision',
                position: { x: 200, y: 200 },
                content: { rules: [], inputs: [], outputs: [] },
              }]);
            }}
          >
            Add Node
          </Button>
        </Space>
      </div>

      <div style={{ flex: 1 }}>
        <DecisionGraph
          ref={graphRef}
          value={graph}
          onChange={setGraph}
          simulate={simulation}
          customNodes={[httpRequestNode]}
          panels={[
            {
              id: 'simulator',
              icon: <PlayCircleOutlined />,
              title: 'Simulator',
              renderPanel: () => (
                <GraphSimulator
                  onRun={handleSimulate}
                  onClear={() => setSimulation(undefined)}
                  loading={false}
                />
              ),
            },
          ]}
        />
      </div>
    </div>
  );
}
```

---

## API Reference

### DecisionGraph Props

| Prop | Type | Description |
|------|------|-------------|
| `value` | `DecisionGraphType` | Current graph state (nodes and edges) |
| `onChange` | `(graph: DecisionGraphType) => void` | Callback when graph changes |
| `id` | `string` | Optional graph identifier |
| `name` | `string` | Graph name (default: "graph.json") |
| `disabled` | `boolean` | Disable editing |
| `components` | `NodeSpecification[]` | Additional node types |
| `customNodes` | `CustomNodeSpecification[]` | Custom node types |
| `panels` | `PanelType[]` | Side panel definitions |
| `activePanel` | `string` | Active panel ID |
| `onPanelsChange` | `(panelId?: string) => void` | Panel change callback |
| `simulate` | `Simulation` | Simulation results |
| `compactMode` | `boolean` | Enable compact mode |
| `viewConfig` | `ViewConfig` | Permission configuration |
| `viewConfigCta` | `string` | CTA text for limited permissions |
| `onViewConfigCta` | `() => void` | CTA click handler |
| `reactFlowProOptions` | `ProOptions` | React Flow Pro options |
| `tabBarExtraContent` | `React.ReactNode` | Extra content in tab bar |
| `onReactFlowInit` | `(instance: ReactFlowInstance) => void` | React Flow init callback |
| `onCodeExtension` | `(monaco: Monaco) => void` | Monaco extension callback |
| `onFunctionReady` | `(monaco: Monaco) => void` | Function editor ready callback |

### DecisionGraphRef Methods

| Method | Signature | Description |
|--------|-----------|-------------|
| `setDecisionGraph` | `(graph: Partial<DecisionGraphType>, options?) => void` | Set graph data |
| `addNodes` | `(nodes: DecisionNode[]) => void` | Add nodes |
| `removeNodes` | `(ids: string[]) => void` | Remove nodes |
| `updateNode` | `(id: string, updater: (draft) => draft) => void` | Update node |
| `duplicateNodes` | `(ids: string[]) => void` | Duplicate nodes |
| `copyNodes` | `(ids: string[]) => void` | Copy to clipboard |
| `pasteNodes` | `() => void` | Paste from clipboard |
| `addEdges` | `(edges: DecisionEdge[]) => void` | Add edges |
| `removeEdges` | `(ids: string[]) => void` | Remove edges |
| `removeEdgeByHandleId` | `(handleId: string) => void` | Remove edge by handle |
| `openTab` | `(id: string) => void` | Open tab |
| `closeTab` | `(id: string, action?: string) => void` | Close tab |
| `goToNode` | `(id: string) => void` | Navigate to node |
| `setActivePanel` | `(panel?: string) => void` | Set active panel |
| `setCompactMode` | `(mode: boolean) => void` | Set compact mode |
| `toggleCompactMode` | `() => void` | Toggle compact mode |
| `triggerNodeSelect` | `(id: string, mode: 'toggle' \| 'only') => void` | Select node programmatically |

### Type Definitions

```typescript
type DecisionGraphType = {
  nodes: DecisionNode[];
  edges: DecisionEdge[];
};

type DecisionNode<T = any> = {
  id: string;
  name: string;
  description?: string;
  type?: string;
  content?: T;
  position: Position;
};

type DecisionEdge = {
  id: string;
  name?: string;
  sourceId: string;
  targetId: string;
  sourceHandle?: string;
  targetHandle?: string;
  type?: string;
};

type Position = {
  x: number;
  y: number;
};
```

---

## Best Practices

### 1. State Management
- Always use selectors in `useDecisionGraphState` to prevent unnecessary re-renders
- Use Immer draft updates in `updateNode` - never mutate directly
- Keep selectors simple and memoize expensive computations

### 2. Performance
- Memoize custom node renderers with proper comparison functions
- Use `React.memo` for custom components inside nodes
- Avoid reading large state objects if you only need small parts

### 3. Custom Nodes
- Keep node UI lightweight - heavy editors go in tabs
- Always implement `generateNode` with sensible defaults
- Validate data in `onNodeAdd` to prevent invalid nodes
- Use `shortDescription` for helpful tooltips

### 4. Type Safety
- Define TypeScript types for your node configs
- Use type parameters in `CustomNodeSpecification<Config, Kind>`
- Leverage type inference for automatic type checking

### 5. User Experience
- Provide clear error messages
- Use confirmations for destructive actions
- Show loading states during async operations
- Add documentation URLs to your custom nodes

### 6. Error Handling
- Wrap async operations in try/catch
- Show user-friendly error messages
- Don't let errors break the entire graph
- Log errors for debugging

---

## Troubleshooting

### Common Issues

**Q: My node isn't updating when I change state**
- Ensure you're using `updateNode` with an Immer draft
- Check that your selector in `useDecisionGraphState` is correct
- Verify you're not mutating state directly

**Q: Custom node not rendering**
- Check that the node `kind` matches exactly
- Ensure `customNodes` prop is passed to `DecisionGraph`
- Verify `generateNode` returns valid data structure

**Q: Edges won't connect**
- Check `isValidConnection` logic
- Ensure handles have correct `type` ('source' or 'target')
- Verify no circular dependencies are being created

**Q: Tab not opening**
- Implement `renderTab` in your node specification
- Check that node type is correctly registered
- Verify tab is in `openTabs` state

**Q: Simulation not showing results**
- Check that `simulate` prop is passed correctly
- Ensure trace data includes node IDs that exist in graph
- Verify result structure matches expected format

---

## Migration Guide

### From Version 1.x to 2.x

#### Changed: Store Architecture
```tsx
// Old (v1.x)
const store = useDecisionGraphStore();
store.state.nodes;

// New (v2.x)
const nodes = useDecisionGraphState((state) => state.decisionGraph.nodes);
```

#### Changed: Node Creation
```tsx
// Old (v1.x)
const node = {
  id: '1',
  type: 'custom',
  data: { /* config */ },
};

// New (v2.x)
const node = {
  id: '1',
  type: 'customNode',
  content: {
    kind: 'custom',
    config: { /* config */ },
  },
};
```

#### Changed: Custom Node API
```tsx
// Old (v1.x)
const node = {
  type: 'myNode',
  render: (props) => <div />,
};

// New (v2.x)
const node = createJdmNode({
  kind: 'myNode',
  renderNode: (props) => <GraphNode {...props} />,
});
```

---

## Additional Resources

- [React Flow Documentation](https://reactflow.dev/)
- [Zustand Documentation](https://github.com/pmndrs/zustand)
- [Immer Documentation](https://immerjs.github.io/immer/)
- [Zen Engine WASM Documentation](https://github.com/gorules/zen)

---

## Support

For issues, questions, or contributions:
- Open an issue on GitHub
- Check existing documentation
- Review code examples in the repository
- Consult the architecture documentation

---

Happy coding! 🚀