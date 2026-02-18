---
name: 3DPrinting
description: 3D modeling and print preparation. USE WHEN user provides STL files, mentions .stl extension, discusses 3D prints, or any physical object design. LOAD PROACTIVELY when conversation involves modifying existing 3D models.
---

# 3DPrinting

Design and prepare 3D models for printing.

## Printer Specifications

| Spec | Value |
|------|-------|
| Printer | [Your printer model] |
| Build Volume | [X] x [Y] x [Z] mm |
| Nozzle | [diameter] mm |
| Default Layer Height | [height] mm |
| Fine Detail Layer | [range] mm |

## Workflow Routing

| Workflow | Trigger | File |
|----------|---------|------|
| **Design** | "design a part", "3D print", "OpenSCAD", "create a model" | `Workflows/Design.md` |
| **GalleryView** | "gallery page", "visual view", "render gallery", "show the model" | `Workflows/GalleryView.md` |

## Quick Reference

- **Start in 2D.** Describe overhead or side views before going to 3D.
- **Use OpenSCAD.** Parametric, readable, version-controllable. LLMs do well with it because it's code.
- **Test prints.** Print small sections first. Catch fit issues before full prints.
- **Keep versions.** Don't overwrite working designs. Use version suffixes (v2, v3).
- **Manifold geometry.** All meshes must be watertight. Verify in slicer before printing.

## Examples

**Example 1: Design a new part**
```
User: "I want to design a phone stand"
-> Load Workflows/Design.md
-> Discuss dimensions and requirements
-> Create 2D description, then OpenSCAD
```

**Example 2: See the model visually**
```
User: "Can I see what that looks like?"
-> Load Workflows/GalleryView.md
-> Render preview images and build interactive viewer
```

**Example 3: Iterate on fit**
```
User: "The clip is too tight on the first print"
-> Load Workflows/Design.md
-> Adjust clearance variables, re-render, new test piece
```
