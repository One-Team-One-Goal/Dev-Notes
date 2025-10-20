# Quick Reference Guide - Digital Circuit Simulator

A cheat sheet for developers working on the simulator.

---

## 🎯 Key Files

```
simulator/
├── CircuitSimulator.tsx          ← Main container, toolbar state, undo/redo
├── hooks/
│   └── useCircuitSimulator.ts    ← State management hook (MOST IMPORTANT)
├── engine/
│   └── simulator.ts              ← Simulation logic engine
├── components/
│   ├── CircuitCanvas.tsx         ← Main canvas, user interactions
│   ├── ComponentRenderer.tsx     ← Visual rendering of components
│   ├── ConnectionRenderer.tsx    ← Visual rendering of wires
│   ├── ComponentPalette.tsx      ← Component selection sidebar
│   └── PropertiesPanel.tsx       ← Selected component properties
├── utils/
│   ├── componentFactory.ts       ← Create component instances
│   ├── circuitGenerator.ts       ← Generate circuits from expressions
│   └── expressionParser.ts       ← Parse Boolean expressions
└── types/
    └── index.ts                  ← TypeScript type definitions
```

---

## 🔧 Common Tasks

### Add a New Component Type

1. **Add type to union:**
```typescript
// types/index.ts
export type ComponentType = 
  | 'AND' | 'OR' | 'NOT'
  | 'YOUR_NEW_COMPONENT';  // ← Add here
```

2. **Add definition:**
```typescript
// utils/componentFactory.ts
YOUR_NEW_COMPONENT: {
  type: 'YOUR_NEW_COMPONENT',
  name: 'My Component',
  category: 'gates',  // or 'flipflops', 'inputs', 'outputs', 'circuits'
  icon: '⚡',
  inputs: 2,
  outputs: 1,
  defaultSize: { width: 80, height: 50 },
  description: 'What this component does'
}
```

3. **Add logic:**
```typescript
// engine/simulator.ts
private updateComponent(component: Component): void {
  switch (component.type) {
    case 'YOUR_NEW_COMPONENT':
      this.updateYourNewComponent(component);
      break;
    // ... other cases
  }
}

private updateYourNewComponent(component: Component): void {
  // Implement logic
  const input1 = component.inputs[0].value;
  const input2 = component.inputs[1].value;
  component.outputs[0].value = /* your logic here */;
}
```

4. **Add rendering:**
```typescript
// components/ComponentRenderer.tsx
const renderGateShape = () => {
  switch (component.type) {
    case 'YOUR_NEW_COMPONENT':
      return (
        <svg width={width} height={height}>
          {/* Your SVG drawing */}
        </svg>
      );
    // ... other cases
  }
}
```

---

## 📊 State Structure

```typescript
CircuitState {
  components: [
    {
      id: "comp_123",
      type: "AND",
      position: { x: 100, y: 100 },
      size: { width: 80, height: 50 },
      inputs: [
        { id: "comp_123_input_0", value: false, connected: true },
        { id: "comp_123_input_1", value: true, connected: true }
      ],
      outputs: [
        { id: "comp_123_output_0", value: false, connected: true }
      ]
    }
  ],
  connections: [
    {
      id: "conn_456",
      from: { componentId: "comp_123", connectionPointId: "comp_123_output_0" },
      to: { componentId: "comp_789", connectionPointId: "comp_789_input_0" },
      path: [{ x: 180, y: 125 }, { x: 300, y: 125 }],
      value: false
    }
  ],
  selectedComponent: "comp_123",
  selectedConnection: null,
  isSimulating: true,
  simulationSpeed: 100,
  gridSize: 20,
  snapToGrid: true
}
```

---

## 🎨 Component Rendering Order

```
Canvas
  ↓
Connections (wires)
  ↓
Components
  ↓
  ├── Component Shape (SVG)
  ├── Connection Points
  │   ├── Hover ring
  │   ├── Main circle
  │   ├── Inner glow (if active)
  │   └── Value indicator
  └── Label (if present)
```

---

## 🔄 Simulation Flow

```
1. User changes input (clicks SWITCH)
   ↓
2. updateComponent() updates SWITCH output
   ↓
3. propagateSignals() runs
   ↓
4. Connection copies output to connected input
   ↓
5. updateComponent() updates receiving component
   ↓
6. Repeat steps 4-5 for all connections
   ↓
7. React re-renders with new values
   ↓
8. Wait simulationSpeed ms (default 100ms)
   ↓
9. Loop back to step 3
```

---

## 🎮 Tool Modes

```typescript
toolbarState = {
  selectedTool: 'select' | 'pan' | 'wire' | 'component',
  selectedComponentType: ComponentType | null
}

// Mode behaviors:
'select'    → Click to select, drag to move
'pan'       → Drag canvas to navigate
'wire'      → Click connection points to create wires
'component' → Click canvas to place component
```

---

## 🔌 Connection Rules

✅ **VALID:**
- Output → Input
- One output → Multiple inputs (fan-out)
- Component A → Component B

❌ **INVALID:**
- Input → Output
- Multiple sources → One input (will auto-replace)
- Component → Same component (self-loop)
- Input → Input
- Output → Output

---

## 📦 Hook API

```typescript
const {
  circuitState,          // Current state
  
  // Component operations
  addComponent,          // (type, position) → Component
  removeComponent,       // (componentId) → void
  updateComponent,       // (componentId, updates) → void
  moveComponent,         // (componentId, position) → void
  
  // Connection operations
  addConnection,         // (fromId, fromPointId, toId, toPointId) → Connection
  removeConnection,      // (connectionId) → void
  updateConnection,      // (connectionId, updates) → void
  
  // Selection
  selectComponent,       // (componentId) → void
  selectConnection,      // (connectionId) → void
  
  // Simulation
  startSimulation,       // () → void
  stopSimulation,        // () → void
  resetSimulation,       // () → void
  setSimulationSpeed,    // (speed) → void
  
  // Utility
  clearAll,              // () → void
  toggleSnapToGrid,      // () → void
  setGridSize,           // (size) → void
  setCircuitState,       // (state) → void (for undo/redo)
  simulator              // CircuitSimulator instance
} = useCircuitSimulator();
```

---

## 🐛 Common Bugs & Solutions

### Bug: Infinite re-render
**Cause:** useEffect depends on object/array that changes every render
```typescript
// ❌ WRONG
useEffect(() => {
  doSomething(circuitState.components);
}, [circuitState.components]); // New array every render!

// ✅ CORRECT
useEffect(() => {
  doSomething(circuitState.components);
}, [circuitState.components.length]); // Length only changes when needed

// Or use useMemo
const componentIds = useMemo(() => 
  circuitState.components.map(c => c.id),
  [circuitState.components]
);
```

### Bug: State not updating
**Cause:** Mutating state instead of creating new object
```typescript
// ❌ WRONG
component.label = "New Label";
setCircuitState({ ...prev, components });

// ✅ CORRECT
setCircuitState(prev => ({
  ...prev,
  components: prev.components.map(c =>
    c.id === componentId ? { ...c, label: "New Label" } : c
  )
}));
```

### Bug: Connection not creating
**Cause:** Usually from/to validation issue
```typescript
// Check these:
1. Is fromComponent an output? ✓
2. Is toComponent an input? ✓
3. Are they different components? ✓
4. Does the connection already exist? ✗
```

---

## 🎯 Performance Tips

1. **Use React.memo for expensive components:**
```typescript
export const ComponentRenderer = React.memo(({ component, ... }) => {
  // ...
}, (prev, next) => {
  // Custom comparison
  return prev.component.id === next.component.id &&
         prev.isSelected === next.isSelected;
});
```

2. **Use useCallback for event handlers:**
```typescript
const handleClick = useCallback((id: string) => {
  // Handler logic
}, [/* minimal dependencies */]);
```

3. **Limit simulation events:**
```typescript
// Already implemented in simulator.ts
if (this.simulationEvents.length > 1000) {
  this.simulationEvents = this.simulationEvents.slice(-500);
}
```

---

## 🧪 Testing Checklist

When making changes, test:
- [ ] Add component → works
- [ ] Delete component → removes connections too
- [ ] Create connection → validates correctly
- [ ] Move component → wires follow
- [ ] Undo/Redo → works without loops
- [ ] Zoom/Pan → renders correctly
- [ ] Simulation → signals propagate
- [ ] Interactive components → respond to clicks
- [ ] Long-running → no memory leak

---

## 💡 Pro Tips

1. **Use console.log with component ID:**
```typescript
console.log('Component updated:', component.id, component);
```

2. **Inspect state in React DevTools:**
   - Install React DevTools extension
   - Find `CircuitSimulator` component
   - View `circuitHook.circuitState`

3. **Debug simulation:**
```typescript
// In simulator.ts, add logs
private propagateSignals(): void {
  console.log('=== Propagation Start ===');
  // ... logic
  console.log('=== Propagation End ===');
}
```

4. **Visualize connection paths:**
```typescript
// In ConnectionRenderer.tsx
console.log('Connection path:', connection.path);
```

5. **Check connection point positions:**
```typescript
// In ComponentRenderer.tsx
console.log('Connection point:', point.id, point.position, point.value);
```

---

## 📞 Need Help?

1. **Read the full docs:** `Digital-Circuit-Simulator-Architecture.md`
2. **Check bug fixes:** `Simulator-Bug-Fixes-Summary.md`
3. **Ask the team:** Post in dev channel
4. **Debug step-by-step:** Use browser DevTools breakpoints

---

**Remember:** When in doubt, follow the data flow in the architecture diagram! 🚀
