# LOD-control-with-threeJS

This repository provides a proof-of-concept for hosting and viewing complex 3D models directly in a web browser without specialized hardware or software. By utilizing Level of Detail (LOD) control and mesh optimization, the project achieves significant performance gains while maintaining visual context.

![LOD Controlled Model](reports/img/first-working-lod-model.gif)

---
🚀 Overview

The goal is to eliminate the need for heavy-duty software in construction visualization. By implementing dynamic mesh swapping in three.js, this project enables:

- 3x average GPU performance improvement (measured by triangle count).
- Zero increase in draw calls despite dynamic mesh swapping.
- Cross-platform accessibility via standard web browsers.

[View Live Demo](https://suryashch.github.io/LOD-control-with-threeJS/)
---

🛠️ Key Technical Concepts

1. Mesh Density & Optimization

3D models are composed of vertices (points) and edges (connections). Complex geometries, specifically cylinders (pipes) common in construction, are heavy on GPU resources due to high vertex counts.

- Decimation: We use Blender's Decimate modifier to reduce mesh density while preserving the overall shape.
- Distance-Based Perception: At significant distances, the human eye cannot distinguish between a high-poly and low-poly mesh. This project exploits this by reducing quality for distant objects.

2. Level of Detail (LOD) Implementation

Using the three.LOD class, multiple versions of the same object are loaded into a single container.

- Hi-Res (Green): Rendered when the camera is close.
- Low-Res (Red): Rendered when the camera moves beyond a specific distance threshold.

3. Per-Object Traversal

For complex scenes (e.g., the 303-object Piperack), the system uses the .traverse() function to loop through the scene graph. A Map() object stores object names and their corresponding mesh resolutions, allowing for automated LOD assignment across large-scale models.

---
📊 Performance Metrics

| Metric |Improvement (Avg) | Peak Improvement |
| Triangles (GPU) | 3x Reduction | 5x Reduction |
| Draw Calls (CPU) | Constant | - |

---
📂 Repository Structure

    [/scripts](scripts/): Contains the core three.js implementation and logic.

    [reports/img](reports/img/): Documentation assets and performance comparison captures.

    [reports/per-object-lod-control-with-threejs.md](reports/3d-model-LOD.md): The full research paper and technical breakdown.

🔧 Getting Started

    Clone the repo: git clone https://github.com/suryashch/LOD-control-with-threeJS

    Run a local server: Use Live Server or python -m http.server.

    Open index.html: Navigate to the local address in any modern browser.

Summary provided by Google Gemini.