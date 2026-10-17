# Canvas Hosting, Controls & Chrome

The `<Wireflow />` component renders the full interactive React Flow canvas with automatic layout, search, minimap, annotations panel, and export capabilities.

---

## 1. Hosting the Canvas

Import `Wireflow` and the mandatory stylesheet from `@microx-technologies/microwireflow`:

```tsx
// App.tsx
import * as React from "react";
import { Wireflow } from "@microx-technologies/microwireflow";
import "@microx-technologies/microwireflow/styles.css";
import { authFlow } from "./flows/authFlow";

export function App() {
  return (
    <div style={{ width: "100vw", height: "100vh" }}>
      <Wireflow
        flow={authFlow}
        showMinimap
        showSearch
        showControls
        nodeWidth={360}
      />
    </div>
  );
}
```

---

## 2. `<Wireflow />` Props & Configuration

```ts
export type WireflowProps = {
  flow: Flow;                                       // Output of defineFlow()
  className?: string;                               // Optional canvas wrapper class
  style?: React.CSSProperties;                      // Optional canvas wrapper style
  showMinimap?: boolean;                            // Display bottom-right minimap (default: true)
  showSearch?: boolean;                             // Display top search bar (default: true)
  showControls?: boolean;                           // Display zoom/fit controls (default: true)
  edgeRouting?: "smoothstep" | "step" | "bezier" | "straight"; // Default edge routing
  avoidCollisions?: boolean;                        // Smart edge routing around nodes
  lineHops?: boolean;                               // Draw line hops over intersecting edges
  arrow?: boolean | "closed" | "open" | "none";     // Default arrow style
  triggerDots?: boolean;                            // Display dot markers on active handles
  nodeWidth?: number;                               // Base width for screen node containers
  maxFitZoom?: number;                              // Maximum zoom level on fit-view
  onPositionsExport?: (positions: FlowPositions, snippet: string) => void;
  onNodeSelect?: (nodeId: string | null) => void;
};
```

---

## 3. Chrome Features

### Search Bar (`showSearch`)
- Pressing `Ctrl+K` or clicking the search field filters screen nodes by `id`, `title`, and `notes`.
- Selecting a search result smoothly pans and zooms the viewport to center that screen.

### Annotations Panel
- Clicking any node card opens the right-hand **Annotations Panel**.
- Displays the screen `title`, `kind`, documentation `notes`, incoming transition sources, and outgoing triggers.

### Positions Export Tool
- Moving screen nodes on the canvas modifies runtime positions.
- Clicking the **Export Positions** button generates a copyable TypeScript snippet:
  ```ts
  positions: {
    "login": { x: 80, y: 120 },
    "dashboard": { x: 540, y: 120 },
    "reset-password": { x: 80, y: 620 }
  }
  ```
- Pasting this object back into `defineFlow({ ..., positions })` commits the exact layout into source code.

---

## 4. Exporting High-Resolution Images

`microwireflow` provides programmatic utilities to export the canvas for documentation reviews or presentations:

```tsx
import { exportAsPng, exportAsSvg } from "@microx-technologies/microwireflow";

// Export as PNG image file download
await exportAsPng({
  filename: "auth-wireflow.png",
  scale: 2, // 2x retina resolution
});

// Export as vector SVG file download
await exportAsSvg({
  filename: "auth-wireflow.svg",
});
```
