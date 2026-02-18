# Design Workflow

Create 3D printable models using OpenSCAD. From concept through parametric design to export-ready STL files.

## Approach

1. **Start in 2D.** Before writing any code, describe the shape as an overhead or side view. Get the dimensions and spatial relationships clear in 2D first. This gives you something concrete to reason about before adding the third dimension.

2. **Write OpenSCAD.** OpenSCAD is a scripting language for 3D geometry. Variables, modules, boolean operations, all in text. Because it's code, it's parametric by nature: all dimensions live as variables, so you can tweak and re-render without re-explaining the design.

3. **Build incrementally.** Get the basic shape right before adding features. Each iteration should add one thing. Test at each step.

4. **Export STL when ready.** Work in OpenSCAD source during the design phase. Export STL files when the user is ready to take something to their slicer.

## OpenSCAD File Structure

```openscad
// === PARAMETERS ===
diameter = 100;
thickness = 1.5;
wall = 2.0;
$fn = 32; // Curve resolution: 32 for drafts, 100+ for final exports

// === MODULES ===
module part_a() {
    // geometry here
}

module part_b() {
    // geometry here
}

module assembled() {
    part_a();
    part_b();
}

// === RENDER SELECTION ===
RENDER = "preview";
if (RENDER == "preview") { assembled(); }
else if (RENDER == "part_a") { part_a(); }
else if (RENDER == "part_b") { part_b(); }
```

- All dimensions as variables at the top. Easy to adjust.
- Separate modules for each logical part.
- A render selector so individual parts can be exported as STL.
- Shape parts with `difference()`, `union()`, and `intersection()` (the core boolean operations).
- Set `$fn` for curved surfaces: `$fn=32` for drafts, `$fn=100` or higher for final exports. Higher values increase render time.

## CLI Commands

**Export STL:**
```bash
openscad -D 'RENDER="part_a"' -o part_a.stl design.scad
```

**Render preview image (headless):**
```bash
xvfb-run -a openscad --autocenter --viewall \
    --camera=0,0,0,55,0,35,0 \
    --imgsize=800,600 \
    -o preview.png design.scad
```

Camera format: `--camera=tx,ty,tz,rx,ry,rz,dist`. Use `--autocenter --viewall` to auto-frame the model.

If `xvfb-run` is not available: `sudo apt install xvfb`.

## Working with Existing STL Files

When modifying or building on top of an existing STL, analyze it first:

1. **Bounding box**: Overall dimensions.
2. **Solid or hollow?** Check for vertices at center at Z=0.
3. **Thickness profile**: Material thickness at different points.
4. **Feature locations**: Where do features attach? Analyze Z values at different radii.

**Importing STL into OpenSCAD:**
- Understand the geometry before doing boolean operations.
- Overlap for clean unions. Surfaces that merely touch can create non-manifold edges. Extend new geometry slightly into the existing mesh.
- Check "Simple: yes" on export. "No" means mesh issues.

## Manifold Geometry

Slicers need watertight meshes. Non-manifold geometry causes print failures.

- All meshes must be watertight (no holes, no non-manifold edges).
- Use consistent vertex counts when connecting rings or cylinders.
- Use fan triangulation from the center for caps.
- Always verify in the slicer before printing.

## Functional Parts

For parts that fit around existing objects (clips, holders, cradles, brackets):

- Measure with calipers. Don't estimate.
- Add 0.2-0.4mm clearance for press-fit connections.
- Print a test piece at the critical dimension before printing the full part.
- Get exact measurements from the user and ask about the mechanical behavior they want (clips on, slides in, snaps over, etc.).

## Versioning

Don't overwrite working designs. When iterating:

- Source files: `design.scad` -> `design-v2.scad`
- Exports: `part.stl` -> `part-v2.stl`
- Preview images: `preview.png` -> `preview-v2.png`

The user should be able to compare versions and go back to a previous one.
