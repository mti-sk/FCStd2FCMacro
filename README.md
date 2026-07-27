# FCStd2FCMacro

Export a FreeCAD document's feature tree into a self-contained, re-runnable
FreeCAD Python macro.

`FCStd2FCMacro` runs **inside FreeCAD** and walks the active document via the
API, so it reads the *live* objects — exact sketch geometry, placements and
feature parameters. This means the generated macro cannot drift from the model
the way hand-written or externally-parsed reconstructions do.

Typical uses: keeping a plain-text "master" build of a model, versioning a
design in git, sharing a parametric model as a single script, or getting back a
clean rebuild after heavy GUI editing.

## Requirements

- FreeCAD 1.0 or newer (developed and tested against the 1.1 line).
- A document built mainly with the **PartDesign** and **Sketcher** workbenches.

## Installation

Copy `FCStd2FCMacro.FCMacro` into your FreeCAD macro directory. You can find it
via **Macro → Macros… → the folder path shown at the top**, or open the
directory with `App.getUserMacroDir(True)` in the Python console.

## Usage

1. Open the document you want to export.
2. Open **View → Panels → Report view** (so you can see the output log).
3. Run **Macro → Macros… → `FCStd2FCMacro` → Execute**.
4. The generated macro is written next to your macros as
   `<DocName>_rebuild.FCMacro`. Its full path is printed in the Report view
   (`=== SAVED: … ===`).
5. Run the generated `*_rebuild.FCMacro` on a **new/empty document** to
   reconstruct the model.

## What is reproduced

| Item | Behaviour |
|------|-----------|
| Sketches | Exact geometry (lines, circles, arcs, points, ellipses), placement, construction flags. Fixed with a `Block` constraint (0 DOF). |
| Profile features | Pad, Pocket, Revolution, Groove, Loft, Pipe, … recreated generically with their parameters. |
| Holes | `PartDesign::Hole` with diameter, countersink/counterbore, depth, etc. |
| Text | Draft `ShapeString` + its consuming Pocket. |
| Patterns / mirrors | Linear/Polar patterns and mirrors, with originals and numeric parameters. |
| Multiple bodies | Supported. |

## Known limitations (by design)

- **Dress-ups are notes only.** `Fillet`, `Chamfer`, `Draft` and `Thickness`
  address their edges through FreeCAD's topological naming (mapped edge names).
  Those names are not stable on a freshly rebuilt shape, so they are exported as
  a comment with the radius/size and edge count. **Re-apply them in the GUI as
  the final step.**
- **Sketches are fixed with `Block`, not dimensioned.** The shape is exact and
  fully constrained, but not editable by dimensions until you un-block the
  sketch you want to change.
- **Unknown object types** are emitted as a `# TODO … add manually` comment and
  listed in the Report view — nothing is silently dropped, but exotic features
  may need manual work.
- Ellipses/splines are best-effort; verify orientation after rebuild.

## Configuration

Two flags at the top of the macro:

- `BLOCK_SKETCHES` (default `True`) — fully fix sketch geometry with `Block`.
- `DRESSUP_AS_NOTE` (default `True`) — export Fillet/Chamfer/Draft/Thickness as
  comments instead of attempting (unreliable) reconstruction.

## Disclaimer

This software is provided **"as is", without warranty of any kind**, express or
implied. It generates and executes Python code that creates and modifies
FreeCAD documents.

By using it you agree that:

- **You use it entirely at your own risk.** The authors and contributors accept
  **no liability whatsoever** for any direct, indirect, incidental, special or
  consequential damages, including but not limited to loss of data, corrupted or
  incorrect models, lost work, or any other loss arising from the use of, or
  inability to use, this software — to the maximum extent permitted by
  applicable law.
- **Always keep independent backups** of your `.FCStd` files. Run the exporter
  and the generated rebuild macros on copies first. Reconstruction is
  best-effort and may be inexact or incomplete for some models.
- This software is **not affiliated with or endorsed by** the FreeCAD project or
  its developers. "FreeCAD" is the property of its respective owners.
- Nothing here constitutes professional, engineering or legal advice. You are
  responsible for verifying that any generated model is correct and fit for your
  purpose before relying on it.

## License

Released under the [MIT License](LICENSE).
