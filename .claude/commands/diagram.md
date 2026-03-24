---
description: Create a draw.io diagram using the cli-anything-drawio CLI
argument-hint: <description of what to diagram>
---

<objective>
Create a professional draw.io diagram for: $ARGUMENTS

Use the `cli-anything-drawio` CLI to build the diagram programmatically, then export to SVG with post-processing for correct edge routing.

Before building, determine the visual style and diagram type (see selection steps below).
</objective>

<context>
CLI location: `cli-anything-drawio` (on system PATH)
Default output directory: `docs/diagrams/` (create if needed)
</context>

<process>

## Step 1: Understand the request

Parse $ARGUMENTS for style and type keywords.

Style keywords:
- "corporate" -> Corporate style
- "modern-docs" or "modern docs" -> Modern Docs style
- "blueprint" -> Blueprint style

Type keywords:
- "architecture" -> Architecture / System Overview
- "flowchart" -> Flowchart / Decision
- "er" or "data model" -> ER / Data Model
- "process" or "pipeline" -> Process / Pipeline
- "freeform" -> Freeform

If a style or type keyword is detected, skip the corresponding prompt in the steps below.

## Step 2: Select visual style

If style was not detected in Step 1, ask the user:

> Which visual style?
> (a) Corporate -- muted blues/grays, consulting deck feel
> (b) Modern Docs -- high contrast, minimal, developer docs feel
> (c) Blueprint -- deep blues, engineering diagram feel

## Step 3: Select diagram type

If type was not detected in Step 1, ask the user:

> Which diagram type?
> (a) Architecture / System Overview
> (b) Flowchart / Decision
> (c) ER / Data Model
> (d) Process / Pipeline
> (e) Freeform

## Step 4: Plan the diagram

Using the selected type pattern (see `<diagram_patterns>`) and style preset (see `<style_presets>`):
- Identify all shapes needed, their labels, positions, and sizes
- Identify all edges: source, target, label, exit/entry points
- Plan groupings and swimlanes where the type pattern calls for them
- Compute bounding boxes for any groups before creating children

## Step 5: Create the project

```
mkdir -p docs/diagrams
cli-anything-drawio project new -o docs/diagrams/<name>.drawio -n "<title>"
```

Always pass `--project <path>` on every subsequent command.

## Step 6: Add shapes

Use `diagram add-shape` with `-l`, `-x`, `-y`, `-w`, `-h`, `-s`, `--fill`, `--stroke`, `--font-color`, `--font-size` from the selected preset.

Apply `--font-color` per-shape based on fill darkness (see Font Color Rule in `<style_presets>`).

Batch multiple add-shape calls in a single Bash invocation. Track each returned `id` value for wiring edges.

## Step 7: Add groups and swimlanes

Use a two-pass layout:
1. Plan all child positions and sizes first, then compute the bounding box
2. Create the group shape at (min_x - 40, min_y - 30) with size (bbox_w + 80, bbox_h + 60)
3. Create child shapes at their planned positions within the group bounds

Style groups with neutral fill, dashed or light stroke, and sublabel font size for the label.

## Step 8: Add edges with smart exit/entry points

Use the exit/entry lookup table below to choose connection points based on the relative position of source and target:

- Target to the right: exitX=1,exitY=0.5 -> entryX=0,entryY=0.5
- Target below: exitX=0.5,exitY=1 -> entryX=0.5,entryY=0
- Target to the left: exitX=0,exitY=0.5 -> entryX=1,entryY=0.5
- Target above: exitX=0.5,exitY=0 -> entryX=0.5,entryY=1
- Diagonal: use the nearest edge for both exit and entry

Staggering rule: When N edges leave the same side of a shape, distribute exit/entry points at 1/(N+1), 2/(N+1), ..., N/(N+1) along that side.

Full style override template:
```
--style "edgeStyle=orthogonalEdgeStyle;rounded=1;orthogonalLoop=1;jettySize=auto;html=1;endArrow=block;endFill=1;exitX=<val>;exitY=<val>;exitDx=0;exitDy=0;entryX=<val>;entryY=<val>;entryDx=0;entryDy=0;strokeColor=<hex>;"
```

Use the edge stroke color and edge label font color from the selected style preset.

## Step 9: Save and export

```
cli-anything-drawio --project <path> project save
cli-anything-drawio --project <path> export svg <output.svg> --overwrite
```

## Step 10: Post-process SVG (REQUIRED)

The native SVG exporter draws lines from shape centers to shape centers. Run this Python script to clip line endpoints to shape borders. Replace `<output.svg>` with the actual SVG path.

```python
python -c "
import re, math

with open('<output.svg>', 'r') as f:
    svg = f.read()

# Parse rectangles (includes rounded rects)
rects = []
for m in re.finditer(r'<rect x=\"([^\"]+)\" y=\"([^\"]+)\" width=\"([^\"]+)\" height=\"([^\"]+)\"', svg):
    rects.append(('rect', tuple(float(m.group(i)) for i in range(1, 5))))

# Parse ellipses
ellipses = []
for m in re.finditer(r'<ellipse cx=\"([^\"]+)\" cy=\"([^\"]+)\" rx=\"([^\"]+)\" ry=\"([^\"]+)\"', svg):
    ellipses.append(('ellipse', tuple(float(m.group(i)) for i in range(1, 5))))

all_shapes = rects + ellipses

def point_in_bbox(px, py, shape_type, dims, margin=10):
    if shape_type == 'rect':
        rx, ry, rw, rh = dims
        return (rx - margin <= px <= rx + rw + margin and
                ry - margin <= py <= ry + rh + margin)
    else:  # ellipse
        ecx, ecy, rx, ry = dims
        return ((ecx - rx - margin) <= px <= (ecx + rx + margin) and
                (ecy - ry - margin) <= py <= (ecy + ry + margin))

def find_shape(px, py):
    # First try: point inside bounding box (with margin)
    for shape_type, dims in all_shapes:
        if point_in_bbox(px, py, shape_type, dims):
            return (shape_type, dims)
    # Fallback: nearest shape center within a generous threshold
    best, best_dist = None, float('inf')
    for shape_type, dims in all_shapes:
        if shape_type == 'rect':
            cx, cy = dims[0] + dims[2]/2, dims[1] + dims[3]/2
        else:
            cx, cy = dims[0], dims[1]
        d = abs(cx - px) + abs(cy - py)
        if d < best_dist:
            best_dist, best = d, (shape_type, dims)
    return best if best_dist < 200 else None

def intersect_rect(ox, oy, cx, cy, rect):
    rx, ry, rw, rh = rect
    left, right, top, bottom = rx, rx + rw, ry, ry + rh
    dx, dy = cx - ox, cy - oy
    if dx == 0 and dy == 0:
        return cx, cy
    candidates = []
    if dx != 0:
        for edge_x in (left, right):
            t = (edge_x - ox) / dx
            if t > 0:
                iy = oy + t * dy
                if top - 1 <= iy <= bottom + 1:
                    candidates.append((edge_x, iy, t))
    if dy != 0:
        for edge_y in (top, bottom):
            t = (edge_y - oy) / dy
            if t > 0:
                ix = ox + t * dx
                if left - 1 <= ix <= right + 1:
                    candidates.append((ix, edge_y, t))
    if not candidates:
        return cx, cy
    candidates.sort(key=lambda c: c[2])
    return candidates[0][0], candidates[0][1]

def intersect_ellipse(ox, oy, cx, cy, ellipse):
    ecx, ecy, rx, ry = ellipse
    dx, dy = cx - ox, cy - oy
    if dx == 0 and dy == 0:
        return cx, cy
    a = (dx/rx)**2 + (dy/ry)**2
    b = 2*((ox - ecx)*dx/rx**2 + (oy - ecy)*dy/ry**2)
    c_val = ((ox - ecx)/rx)**2 + ((oy - ecy)/ry)**2 - 1
    disc = b**2 - 4*a*c_val
    if disc < 0:
        return cx, cy
    sqrt_disc = math.sqrt(disc)
    t1, t2 = (-b + sqrt_disc)/(2*a), (-b - sqrt_disc)/(2*a)
    ts = [t for t in (t1, t2) if t > 0.001]
    if not ts:
        return cx, cy
    t = min(ts)
    return ox + t*dx, oy + t*dy

def intersect_shape(ox, oy, cx, cy, shape):
    shape_type, dims = shape
    if shape_type == 'rect':
        return intersect_rect(ox, oy, cx, cy, dims)
    elif shape_type == 'ellipse':
        return intersect_ellipse(ox, oy, cx, cy, dims)
    return cx, cy

def fix_line(match):
    x1, y1 = float(match.group(1)), float(match.group(2))
    x2, y2 = float(match.group(3)), float(match.group(4))
    rest = match.group(5)
    src = find_shape(x1, y1)
    tgt = find_shape(x2, y2)
    nx1, ny1 = intersect_shape(x2, y2, x1, y1, src) if src else (x1, y1)
    nx2, ny2 = intersect_shape(x1, y1, x2, y2, tgt) if tgt else (x2, y2)
    return f'<line x1=\"{nx1:.1f}\" y1=\"{ny1:.1f}\" x2=\"{nx2:.1f}\" y2=\"{ny2:.1f}\"{rest}'

fixed = re.sub(r'<line x1=\"([^\"]+)\" y1=\"([^\"]+)\" x2=\"([^\"]+)\" y2=\"([^\"]+)\"([^/]*/>)', fix_line, svg)

with open('<output.svg>', 'w') as f:
    f.write(fixed)
print('Fixed line endpoints for rects and ellipses')
"
```

## Step 11: Verify

```
cli-anything-drawio --project <path> diagram list
```

Check that:
- All planned shapes exist with correct labels
- All planned edges exist with correct source/target
- Edge count matches the plan

Use `diagram info <id>` to inspect individual elements. Use `diagram update` or `diagram remove` to fix issues before re-exporting.

## Step 12: Add a legend (if appropriate)

If the diagram has multiple visual categories (e.g., different fill colors representing different layers or roles), add a legend at the bottom-left using small shapes. Skip if the diagram is self-explanatory.

## Step 13: Present the result

Tell the user:
- File paths created (.drawio and .svg)
- Style and type used
- Note that the .drawio file is editable in draw.io desktop or the VS Code draw.io extension

</process>

<style_presets>

Font Color Rule: Apply `--font-color` per shape based on the fill color of that shape.
- Dark fills -> use white font (#FFFFFF)
- Light/neutral fills -> use the preset-specific dark font color listed below

Do NOT apply font color globally. Set it on each shape individually.

### Corporate

- Primary fill:   #4472C4  ->  font #FFFFFF
- Secondary fill: #5B9BD5  ->  font #FFFFFF
- Accent fill:    #ED7D31  ->  font #FFFFFF
- Neutral fill:   #F2F2F2  ->  font #1F2937
- Primary stroke: #2F5496
- Neutral stroke: #BFBFBF
- Font sizes: Title 14pt, label 11pt, sublabel 9pt
- Borders: 1px
- Spacing: 80-100px gaps between shapes
- Edge stroke: #2F5496
- Edge label font: #1F2937

### Modern Docs

- Primary fill:   #18181B  ->  font #FFFFFF
- Secondary fill: #3B82F6  ->  font #FFFFFF
- Accent fill:    #10B981  ->  font #FFFFFF
- Neutral fill:   #F4F4F5  ->  font #09090B
- Primary stroke: #27272A
- Neutral stroke: #D4D4D8
- Font sizes: Title 13pt, label 11pt, sublabel 9pt
- Borders: 1.5px
- Spacing: 60-80px gaps between shapes
- Edge stroke: #27272A
- Edge label font: #09090B

### Blueprint

- Primary fill:   #1E3A5F  ->  font #FFFFFF
- Secondary fill: #2563EB  ->  font #FFFFFF
- Accent fill:    #F59E0B  ->  font #1E3A5F
- Neutral fill:   #DBEAFE  ->  font #1E3A5F
- Primary stroke: #1E40AF
- Secondary stroke: #93C5FD
- Font sizes: Title 13pt, label 11pt, sublabel 9pt
- Borders: 1.5px
- Spacing: 70-90px gaps between shapes
- Edge stroke: #1E40AF
- Edge label font: #1E3A5F

</style_presets>

<diagram_patterns>

### Architecture / System Overview

- Flow direction: Left-to-right or top-to-bottom
- Shapes:
  - `rounded` for services and components
  - `cylinder` for databases
  - `cloud` for external systems
  - `document` for files and artifacts
- Grouping: Use `group` shapes to contain related services (e.g., by layer or domain)
- Edges: Orthogonal style, labeled with protocols (e.g., "REST", "gRPC", "events")

### Flowchart / Decision

- Flow direction: Top-to-bottom
- Shapes:
  - `rounded` for process steps
  - `diamond` for decision points
  - `ellipse` for start and end terminals
  - `parallelogram` for input/output
- Grouping: Rarely needed; use swimlanes only if multiple actors are involved
- Edges: Orthogonal style, labeled on decision branches ("yes" / "no")

### ER / Data Model

- Flow direction: Left-to-right
- Shapes: Each entity is two stacked rectangles:
  1. Title rectangle: `rectangle` with `fontStyle=1` (bold), primary fill, title font size
  2. Fields rectangle: `rectangle` with neutral fill, smaller font, `align=left` for field list
  Both rectangles share the same width and are flush (no gap between them).
  Wrap both in a `group` shape.
- Edges: `entity-relation` style, labeled with cardinality ("1:N", "M:N")

### Process / Pipeline

- Flow direction: Left-to-right
- Shapes:
  - `rounded` for pipeline stages
  - `diamond` for gates and checkpoints
  - `document` for outputs and artifacts
- Grouping: Use `swimlane` for parallel tracks (e.g., different teams or environments)
- Edges: Orthogonal, sequential; labels on conditional paths only

### Freeform

- No prescribed flow direction or shape set
- Apply the selected style preset for consistent colors and fonts
- Use general layout principles: consistent spacing, logical grouping, clear flow

</diagram_patterns>

<gotchas>
1. No unicode in labels -- The CLI uses cp1252 encoding. Use plain ASCII only in `-l` values. No arrows, emoji, or special characters (e.g., use "to" instead of a right-arrow character).
2. Batch shape creation -- Chain multiple add-shape calls in a single Bash invocation to reduce round-trips and stay within tool call limits.
3. Track IDs carefully -- Each add-shape returns an `id:` line. Record all IDs before wiring edges.
4. Always post-process SVG -- The native exporter ignores exit/entry perimeter styles and draws center-to-center lines. Post-processing is not optional.
5. Save before export -- The export command reads from disk, not from in-memory state. Always run `project save` first.
6. Desktop renderer not installed -- Use `export svg` plus post-processing. The `export render` command requires draw.io desktop to be installed and is not available in most environments.
7. Group shapes must be created before children -- Position children within the group's bounds after creating the group shape.
8. Page index is 0-based -- The first page is index 0 in all page commands.
</gotchas>

<cli_reference>
```
cli-anything-drawio project new -o <path> -n <name>
cli-anything-drawio --project <path> diagram add-shape -l <label> -x <X> -y <Y> -w <W> -h <H> -s <shape> --fill <hex> --stroke <hex> --font-color <hex> --font-size <N> --style <override>
cli-anything-drawio --project <path> diagram add-edge --source <id> --target <id> -l <label> -s <style> --stroke <hex> --style <override>
cli-anything-drawio --project <path> diagram update <cell-id> -l <label> -x <X> -y <Y> -w <W> -h <H> --style <override>
cli-anything-drawio --project <path> diagram list [-t vertex|edge]
cli-anything-drawio --project <path> diagram info <cell-id>
cli-anything-drawio --project <path> diagram remove <cell-id>
cli-anything-drawio --project <path> page add -n <name>
cli-anything-drawio --project <path> page list
cli-anything-drawio --project <path> page remove <index>
cli-anything-drawio --project <path> page rename <index> -n <name>
cli-anything-drawio --project <path> project save
cli-anything-drawio --project <path> export svg <output-path> [--overwrite]
cli-anything-drawio --project <path> export render <output-path> -f <png|pdf|svg> [--overwrite] [--scale N] [--width N] [--height N] [--border N] [--transparent] [--quality N]
cli-anything-drawio --project <path> session undo
cli-anything-drawio --project <path> session redo
```
</cli_reference>

<success_criteria>
1. `.drawio` file created and saved (editable in draw.io desktop or VS Code)
2. `.svg` file exported with edges connecting at shape borders (not centers)
3. Visual style matches the selected preset (colors, fonts, and spacing consistent throughout)
4. Layout follows the selected diagram type pattern (correct flow direction, shape types, and grouping)
5. Groups and swimlanes used where the type pattern calls for them
6. No overlapping shapes or crossing edges where avoidable
7. Staggered exit/entry points when multiple edges share the same side of a shape
8. Legend included if multiple visual categories exist
9. Verification step passed -- all planned shapes and edges are accounted for
</success_criteria>
