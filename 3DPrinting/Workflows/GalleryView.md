# GalleryView Workflow

Create a visual gallery page for a 3D print project. Shows multi-angle preview renders, an interactive STL viewer, design notes, and version history.

## When to Use

When the user wants to see their design visually, compare versions, or set up a project page for tracking progress.

## Step 1: Generate Preview Renders

Use OpenSCAD CLI to render the model from multiple angles. These become static preview images.

**Camera angles:**

```bash
# Isometric (3/4 view)
xvfb-run -a openscad --autocenter --viewall \
    --colorscheme='Tomorrow Night' \
    --camera=0,0,0,55,0,35,0 \
    --imgsize=800,600 \
    -o preview-iso.png design.scad

# Top view (bird's eye)
xvfb-run -a openscad --autocenter --viewall \
    --colorscheme='Tomorrow Night' \
    --camera=0,0,0,90,0,0,0 \
    --imgsize=800,600 \
    -o preview-top.png design.scad

# Front view (slight angle for depth)
xvfb-run -a openscad --autocenter --viewall \
    --colorscheme='Tomorrow Night' \
    --camera=0,0,0,15,0,15,0 \
    --imgsize=800,600 \
    -o preview-front.png design.scad
```

**Camera format:** `--camera=tx,ty,tz,rx,ry,rz,dist` (translate, rotate, distance). Use `--autocenter --viewall` to auto-frame the model. Set distance to 0 to let OpenSCAD calculate it.

For multi-part designs, render each part separately using the RENDER selector:
```bash
openscad -D 'RENDER="part_a"' --autocenter --viewall \
    --camera=0,0,0,55,0,35,0 --imgsize=800,600 \
    -o preview-part-a-iso.png design.scad
```

If `xvfb-run` is not available, check if the system has a display (`echo $DISPLAY`). If not, install xvfb: `sudo apt install xvfb`.

## Step 2: Interactive STL Viewer

Use Three.js to create an interactive 3D viewer that loads and displays STL files in the browser. The user can rotate, zoom, and pan the model.

**Three.js setup (CDN):**

```html
<script src="https://cdn.jsdelivr.net/npm/three@0.145.0/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.145.0/examples/js/loaders/STLLoader.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.145.0/examples/js/controls/OrbitControls.js"></script>
```

**Viewer function:**

```javascript
function createViewer(containerId, stlFile, color) {
    const container = document.getElementById(containerId);
    const width = container.clientWidth;
    const height = container.clientHeight;

    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x12121c);

    const camera = new THREE.PerspectiveCamera(45, width / height, 0.1, 2000);
    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(width, height);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
    container.appendChild(renderer.domElement);

    const controls = new THREE.OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    controls.dampingFactor = 0.08;

    // 3-point lighting
    scene.add(new THREE.AmbientLight(0x404040, 2));
    const key = new THREE.DirectionalLight(0xffffff, 1);
    key.position.set(100, 200, 150);
    scene.add(key);
    const fill = new THREE.DirectionalLight(0xffffff, 0.5);
    fill.position.set(-100, -50, 100);
    scene.add(fill);
    const rim = new THREE.DirectionalLight(0xffffff, 0.3);
    rim.position.set(0, -100, -100);
    scene.add(rim);

    // Load STL
    new THREE.STLLoader().load(stlFile, function(geometry) {
        const material = new THREE.MeshPhongMaterial({
            color: color,
            specular: 0x222222,
            shininess: 60
        });
        const mesh = new THREE.Mesh(geometry, material);
        geometry.computeBoundingBox();
        mesh.position.sub(geometry.boundingBox.getCenter(new THREE.Vector3()));
        scene.add(mesh);

        // Auto-frame camera
        const size = geometry.boundingBox.getSize(new THREE.Vector3());
        const maxDim = Math.max(size.x, size.y, size.z);
        const fov = camera.fov * (Math.PI / 180);
        const dist = (maxDim / (2 * Math.tan(fov / 2))) * 1.8;
        camera.position.set(dist * 0.8, dist * 0.6, dist * 0.8);
        camera.lookAt(0, 0, 0);
        controls.target.set(0, 0, 0);
        controls.update();
    });

    (function animate() {
        requestAnimationFrame(animate);
        controls.update();
        renderer.render(scene, camera);
    })();

    // Responsive
    new ResizeObserver(() => {
        const w = container.clientWidth, h = container.clientHeight;
        camera.aspect = w / h;
        camera.updateProjectionMatrix();
        renderer.setSize(w, h);
    }).observe(container);
}
```

**Usage:**
```javascript
// Color is a hex integer. Pick whatever fits the project.
createViewer('viewer-v2', 'project-v2.stl', 0x2dd4bf);
createViewer('viewer-v1', 'project-v1.stl', 0x12c2e9);
```

The viewer container needs a fixed height and width. A `div` with `width: 100%; height: 400px` works as a starting point.

## Step 3: Assemble the Page

**Check for an existing gallery page first.** If the project already has an `index.html`, read it and add the new version to it. The gallery grows over time. Each version gets its own section with renders, viewer, and notes. The newest version goes at the top.

**Page structure:**

1. **Project header**: Name, brief description
2. **Latest version section** (newest first, each version has):
   - Version label (v1, v2, v3...)
   - Preview render grid: 3-column grid showing isometric, top, and front views
   - Interactive STL viewer
   - Design notes: what changed, measurements, print settings
   - Downloads: links to this version's STL and SCAD files
3. **Previous versions**: Same structure, stacked below. Don't remove old versions.

For multi-part designs, give each part its own viewer with a different color for visual distinction.

**File naming by version:**
```
preview-iso-v1.png, preview-iso-v2.png, ...
preview-top-v1.png, preview-top-v2.png, ...
preview-front-v1.png, preview-front-v2.png, ...
project-v1.stl, project-v2.stl, ...
project-v1.scad, project-v2.scad, ...
```

**Where to save the page:**

Check for a portal directory first. If `~/portal/` exists, save there:
```
~/portal/{project-name}/
    index.html
    preview-iso-v1.png
    preview-top-v1.png
    preview-front-v1.png
    project-v1.stl
    project-v1.scad
```
The portal serves these as web pages, so the user can view the gallery from any device. Provide the URL when done.

If no portal exists, save everything to a project directory and serve it locally:
```
{project-directory}/
    index.html
    preview-iso-v1.png
    preview-top-v1.png
    preview-front-v1.png
    project-v1.stl
    project-v1.scad
```
Then start a local server so the STL viewer works (browsers block `file://` loading via fetch):
```bash
cd {project-directory}
python3 -m http.server 8080
# Open http://localhost:8080 in a browser
```
The user gets the same gallery experience either way. The only difference is how it's served.

**STL file paths:** Use relative paths in the HTML so the STL files just need to be in the same directory as the page.

## Notes

- For large STL files (>5MB), the Three.js viewer may take a few seconds to load. Consider adding a loading indicator.
