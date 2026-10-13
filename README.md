# Drawing Generator

Local web application that creates ISO drawings from STEP files.
Each drawing has an ISO 5457 frame, an ISO 7200 title block, first-angle Front / Top / Right views, section A-A through the part centre and an isometric view.
Dimensions, notes and ISO 286 tolerances are added interactively.
Output is PDF, DXF, SVG and FCStd (FreeCAD / AstoCAD document with a TechDraw page).
AstoCAD is not used at run time. Projection and hidden-line removal use OpenCascade through build123d.

## 1. Installation (Windows)

1. Install Python 3.10 to 3.14 from python.org (64-bit). `py -0` lists the versions known to the py launcher.
2. Unzip the `drawgen` folder with all files in the same folder.
3. Double-click `start.bat`.

`start.bat` uses the py launcher and selects the newest installed version from 3.14 to 3.10.
The first start creates the environment in `%LOCALAPPDATA%\drawgen\venv` and installs `build123d`, `flask`, `ezdxf` and `reportlab` (approximately 400 MB, internet connection required).
Later starts use the existing environment.
The server runs at `http://localhost:8765` and the browser opens automatically.
Close the command window to stop the server.

Manual installation:

```
py -3.14 -m venv %LOCALAPPDATA%\drawgen\venv
%LOCALAPPDATA%\drawgen\venv\Scripts\python.exe -m pip install -r requirements.txt
%LOCALAPPDATA%\drawgen\venv\Scripts\python.exe run.py
```

`run.py --port 9000` sets a different port and `run.py --no-browser` does not open the browser.
The server listens on 127.0.0.1 only.

## 2. Files

| File | Function |
|---|---|
| `start.bat` | Creates the environment on first start and starts the server |
| `run.py` | Starts the server |
| `config.json` | Defaults: sheet, scale, views, line widths, sheet sizes, frame, title block layout and field values |
| `requirements.txt` | Python packages |
| `drawgen/project.py` | Hidden-line removal, section cut, hatching |
| `drawgen/drawing.py` | Views, layout, sheet and scale selection, title block values |
| `drawgen/titleblock.py` | Frame and title block |
| `drawgen/render.py` | SVG, PDF and DXF output |
| `drawgen/fcstd.py` | FCStd output |
| `drawgen/annotate.py` | Dimensions and notes |
| `drawgen/fits.py` | ISO 286 limits |
| `annotations/` | Saved dimensions and notes, one JSON file per drawing |
| `drawgen/web.py` | Web server |
| `drawgen/static/` | Web interface |

## 3. Use

1. Drop STEP files on **Models**, or enter a file or folder path in the path field and press **Open**. Each solid in a STEP file is listed as a part, named from the STEP labels.
2. Select the parts.
3. Set **Sheet size**, **Scale**, **Mode**, views and title block values.
4. Press **Preview**.
5. Press **Export**.

| Control | Function |
|---|---|
| **Sheet size** | A4 (portrait), A3 to A0 (landscape), or **Auto**. Default A3 |
| **Scale** | **Auto** or an ISO 5455 scale. **Auto** uses the largest scale that fits the sheet |
| **Mode** | **One drawing per part** or **One drawing per file** (all selected parts of a file on one drawing) |
| **Section A-A**, **Plane normal**, **Offset**, **Reverse** | Section plane normal to X, Y or Z, offset in mm from the bounding box centre, viewing direction reversed |
| **Hidden lines** | Hidden lines in the Front, Top and Right views |
| **Tangent edges** | Edges between tangent faces |
| **Isometric scale factor** | Isometric view scale relative to the drawing scale. A value other than 1 adds the isometric scale below the view |
| **Density (kg/m^3)** | Mass in the title block is volume x density when > 0 |
| Title block fields | Values for this run. Empty Title, Identification number and Date of issue are filled with the part name, the title in upper case and today's date |
| **Output folder** | Files are written to this folder. Empty: the browser downloads one file, or `drawings.zip` for several files |
| **Write to a drawings folder next to each opened STEP file** | Writes to `<STEP folder>\drawings` for files opened by path |
| **Save as defaults** | Writes the current settings and fields to `config.json`. Title, Supplementary title, Identification number, Date, Mass and Material are not stored |

Output file names are `<STEP file>.pdf` for a STEP file with one part and `<STEP file>_<part>.pdf` for several parts.
With **Auto** sheet size the smallest sheet from A4 to A0 is used where the scale is >= `auto_sheet_min_scale` (default 1:1).

## 4. Title block editor

The **Title block editor** tab edits `config.json` > `title_block` and `border`.

| Control | Function |
|---|---|
| **Width**, **Height** | Title block size in mm (default 180 mm x 40 mm) |
| **Label height** | Text height of the cell labels in mm |
| **Font** | Font family for SVG output. PDF uses Helvetica |
| **Logo cell**, **Projection cell** | `x, y, w, h` in mm |
| **Binding edge**, **Other edges** | Frame margins in mm (ISO 5457: 20 mm and 10 mm) |
| **Logo** | PNG or JPG placed in the logo cell (PDF and SVG only) |
| Cell table | One row per field: **Key** (field name), **Label**, **x**, **y**, **w**, **h**, **Text h** (value text height in mm) |
| **Add cell** | New row. A new key adds a field to the title block fields |
| **Update preview** | Shows the sheet and the title block |
| **Save title block** | Writes to `config.json` |

Cell coordinates are in mm from the top-left corner of the title block.

## 5. Views and layout

| View | Direction |
|---|---|
| Front | Looking along +Y, X to the right, Z up |
| Top | From +Z, placed below the front view |
| Right | From +X, placed left of the front view |
| Section A-A, plane normal X | Cutting plane shown on the front view, viewed in +X (**Reverse**: -X) |
| Section A-A, plane normal Y | Cutting plane shown on the top view, viewed in +Y (**Reverse**: -Y) |
| Section A-A, plane normal Z | Cutting plane shown on the front view, viewed in -Z (**Reverse**: +Z) |
| Isometric | View direction (1, -1, 1) |

Two layouts are evaluated and the one giving the larger scale is used:

```
A:  Right | Front | A-A          B:  Right | Front | A-A | Iso
          | Top   | Iso                    | Top   |     |
```

Line widths follow ISO 128 in `config.json` > `line_widths_mm` (visible 0.5 mm, hidden, chain and hatch 0.25 mm, frame 0.7 mm).
Hatching is 45 deg at 2.5 mm spacing on paper (`hatch_angle_deg`, `hatch_spacing_mm`).

## 6. Dimensions and notes

Press **Preview**, then **Dimensions and notes** on a drawing (or the **Dimensions and notes** tab).

| Tool | Key | Picks |
|---|---|---|
| **Select** | S | Click a dimension or note to edit it. Drag it to move it. A linear dimension follows the mouse across (dimension line position) and along (text position); text dragged beyond an extension line gets an extended dimension line. **Text centred between extension lines** returns the text to the centre |
| **Horizontal** | H | First point, second point, dimension line position |
| **Vertical** | V | First point, second point, dimension line position |
| **Aligned** | A | First point, second point, dimension line position |
| **Diameter** | D | Circle or arc, then text position. In a side view of a shaft or bore: first edge point, opposite edge point, dimension line position (Ø added to the measured distance) |
| **Radius** | R | Arc or circle, text position |
| **Note** | N | Text position, then the text in a text box (Enter = new line, Ctrl+Enter or OK = place) |
| **Leader note** | L | Arrow point, text position, then the text in a text box (Enter = new line, Ctrl+Enter or OK = place) |

Views are moved by dragging them with **Select**:

| View | Movement |
|---|---|
| Front | Moves Front, Top and Right together |
| Top | Vertical only (stays aligned below Front) |
| Right | Horizontal only (stays aligned beside Front) |
| Section, Isometric | Free |

Dimensions and notes move with their view.
View positions are saved in `annotations/<drawing name>.layout.json` as offsets in mm from the automatic layout and are used for PDF, DXF, SVG and FCStd export.
**Reset views** returns all views to the automatic layout.
Shift+drag or the middle mouse button pans.

Points snap to line end points (red square), line mid points (green square) and circle and arc centres (circle).
Both points of a dimension are picked in the same view.
The isometric view has no snap points.
Line widths in the editor are shown at a fixed screen width at every zoom level.
Curves stored as B-splines that lie on a circle are recognised as circles for Diameter and Radius.
Mouse wheel zooms, dragging empty space, Shift+drag or the middle button pans, F fits the sheet, Esc cancels the current pick, Delete removes the selected item, Ctrl+Z undoes.

Existing note text is edited in the **Text** field of the properties panel (Enter = new line). The markers `\n` and ` /n ` in note text are also converted to line breaks.

Properties of the selected dimension:

| Field | Function |
|---|---|
| **Prefix** | Text before the value, for example `4x ` |
| **Text override** | Replaces the measured value |
| **Tolerance** | **None** (general tolerance from the title block), **ISO 286 fit**, **Symmetric ±**, **Upper / lower deviation** |
| **Class** | ISO 286 tolerance class, for example `H7` (hole, upper case) or `g6` (shaft, lower case). Grades IT5 to IT11, letters D to S, sizes up to 500 mm |
| **Show deviations** | Adds the limit deviations of the class next to the class |

Dimensions are stored in view coordinates and follow the view when the sheet, scale or layout changes.
They are saved in `annotations/<drawing name>.json` and are included in PDF, DXF (layer `DIM`) and SVG exports.
Dimension text height is `dim_text_height_mm` (3.5 mm).

## 7. DXF

DXF R2010, units mm, drawn at sheet size 1:1 (model coordinates are scaled by the drawing scale).
Layers: `VISIBLE`, `HIDDEN` (ISO_HIDDEN), `CHAIN` (ISO_CHAIN), `HATCH`, `TEXT`, `FRAME`, `TITLE`, with lineweights from `line_widths_mm`.
Circles and arcs in the projection plane are written as CIRCLE and ARC.
Other curves are written as LWPOLYLINE.

## 8. FCStd

The .FCStd file contains:

| Object | Content |
|---|---|
| `Part` | The part (or the selected parts as a compound) as Part::Feature, hidden in the 3D view |
| `Page` | TechDraw page with the same sheet size and scale as the PDF |
| `Template` | SVG template with the frame and title block. Field values are editable texts (double-click the field on the page) |
| `Front`, `Top`, `Right`, `Isometric` | TechDraw views (DrawViewPart) at the PDF positions |
| `Section` | Section view A-A (DrawViewSection), base view `Front` |
| `SectionCaption` | Annotation "A-A" |

TechDraw computes the view geometry when the file is opened.
Dimensions, centre lines and annotations are added with the TechDraw tools.
The section cut surface is a grey fill.
A 45 deg hatch is set by changing **Cut Surface Display** of `Section` to **SvgHatch** (the hatch file is included).
View positions can differ from the PDF by a few mm because TechDraw centres views on the 3D bounding box.

## 9. Limitations

- Input is STEP only. AstoCAD models are exported with **File > Export** > STEP.
- Dimensions and notes are not written to the FCStd file.
- Dimensions in the DXF are lines and text, not DIMENSION entities.
- One section view per drawing.
- Projection time depends on the number of B-spline faces (approximately 10 s to 30 s per part for a 7 MB STEP part with 234 faces on a 2-core machine). Views are computed in parallel on machines with more than 2 cores.
