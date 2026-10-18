---
name: cabinet-list
description: Make the cabinet list as a one-page PDF from a Mozaik cabinet drawing (PDF with plan, wall elevations and cabinet numbers) - each cabinet with width, height, depth, drawers and doors. Use when the user attaches a Mozaik drawing and asks for the cabinet list.
---

# Cabinet list from a Mozaik drawing (one-page PDF)

I will attach a cabinet design PDF made in Mozaik. It has a plan view, wall elevations ("Wall #2", "Wall #3"…) and 3D views, with cabinet numbers like #1, #2 and panel numbers like #N1. Make the cabinet list and deliver it as a one-page PDF in exactly the format below. The list is used to price the job, so width, height and depth must be exact. Use your code tools (Python) to read and measure the PDF and to build the PDF. Reply to me in the language I write in; the PDF is always in English.

STEP 1 — Read the drawing
1. Extract the text word by word with its positions (pdfplumber extract_words; plain text extraction mixes overlapping labels) and render every page at 300 dpi. Zoom into each elevation and each part of the plan; never read sizes from a full-page view.
2. Find the project name, the drawing/room title (for example R12) and the address and unit (title block and notes), the plan page, each wall elevation and the 3D page.
3. Check the scale. The scale is in the title block (usually 1/2" = 1'-0"). Measure 2 or 3 written dimensions on the vector lines of the PDF and work out the real factor. At 1/2" = 1'-0" printed to scale, 1 real inch = 3 pt. Use this factor for every size that is not written.
4. If the PDF has more than one room or drawing, make one list (one PDF) per room, unless I name one.
5. How to read the elevations: a box with an X is the side of a cabinet that sits on the next wall (not a cabinet of this wall). A letter "F" on a door marks a finished end on that side; the door is still a door. Small numbers next to panels (13, 24, 25…) are depth callouts.

STEP 2 — Find the cabinets
- Every cabinet has a number (#1, #2…) on the plan and on its wall elevation.
- Items numbered #N1, #N2… are panels and fillers: never put them in the list.
- A numbered item that is only 3/4" thick (for example a panel or shelf fixed under a cabinet) is not a cabinet: leave it out.
- If two numbers overlap (for example #4 printed over #5), two items sit in the same place. Use the vector lines to tell them apart: usually one is a cabinet and the other a 3/4" piece under or behind it.
- Keep the original cabinet numbers, even if that leaves a gap (#1, #2, #3, #5…).
- Gaps at the ends of a wall (for example 2 1/2" or 1 15/16"), open corners, and appliances (range, hood, refrigerator, dishwasher) are not cabinets.

STEP 3 — Measure each cabinet (inches, fractions to 1/16")
- Width: the box width from the dimension string of its wall: the top string for wall cabinets, the bottom string for base cabinets. The small 3/4" values between boxes are panels.
- Height: base and tall = toe kick + box, without the countertop (from the vertical string, for example 4 + 30 1/2 = 34 1/2). Wall = from the vertical string when it is written; otherwise box height = door height + 1/4" (door 28 1/2 → box 28 3/4), confirmed with the scale.
- Depth: the box depth without the door, measured on the plan to the box front line (not the door or the countertop edge). Usually wall 12" and base 24"; a deep cabinet over the refrigerator is often 24".
- Custom cabinets often have fractional sizes. Never round a written dimension.

STEP 4 — Doors, drawers and name
- Count the fronts on the elevation. Each door shows its size inside it. A drawer is a short front (for example "6 A") above a door or other drawers. False fronts are not drawers. In a blind cabinet, the blind part (no hinge marks) is not a door.
- Name each cabinet by its real fronts, not by the library name:
  Base: Base Cabinet (doors only) · Drawer Base One Drawer (exactly one drawer, with or without doors) · Drawer Base (two or more drawers) · Sink Base (only when a sink is drawn or noted above it) · Blind Base · Corner Base (lazy Susan or diagonal) · Microwave Base.
  Wall: Wall Cabinet (with doors; also the cabinet above a metal hood) · Open Wall Cabinet (no doors) · Blind Wall Cabinet · Corner Wall Cabinet · Refrigerator Wall Cabinet (over the refrigerator space, even when no refrigerator is drawn) · Range Hood Cabinet (only a wood hood cover) · Second Layer Cabinet (stacked above other cabinets).
  Tall: Tall Cabinet (with doors) · Open Tall Cabinet (no doors) · Double Oven (two appliance openings, oven + oven or oven + microwave) · Single Oven (one appliance opening) · Spice Pull-out (Tall).
  Vanity: Vanity Sink Base · Vanity Drawer Base · Vanity Drawer Base One Drawer · Vanity Base Cabinet.
- For blind cabinets, add the blind side as seen from the front: "Blind Wall Cabinet (blind right)".

STEP 5 — Check before making the PDF
- Each wall, top run and base run: cabinets + panels + gaps = the overall wall dimension.
- The heights stack up to the overall height shown on the elevation.
- The cabinet numbers on the plan match the numbers on the elevations.
- The doors and drawers match the elevations.
Fix what you can.

STEP 6 — Build the PDF with exactly this code
Only change PROJECT, ROOM, ADDRESS, FILENAME and ROWS. Do not change anything else. Install reportlab only if it is missing (pip install reportlab). Save the PDF in your output folder so you can give it to me as a downloadable file.

```python
from reportlab.lib.pagesizes import letter
from reportlab.platypus import SimpleDocTemplate, Table, TableStyle, Paragraph, Spacer
from reportlab.lib.styles import ParagraphStyle
from reportlab.lib import colors
from reportlab.lib.units import inch as IN

PROJECT = "Client Name - Project"            # project name from the title block
ROOM = "R1"                                  # drawing / room title
ADDRESS = "123 Main St, City - Apt 1"        # format: "Number Street, City - Unit" (leave out " - Unit" if there is none)
FILENAME = "Client Project R1 - Cabinets.pdf"  # "<Project> <Room> - Cabinets.pdf"

# EXAMPLE ROWS ONLY - replace them all with the cabinets of the attached drawing, sorted by number.
# (No., Cabinet, Type, Width, Height, Depth, Drawers, Doors)  - Type is Base, Wall, Tall or Vanity
# Sizes as text with fractions ("34 1/2"); drawers and doors as numbers (0 prints as "-").
ROWS = [
    (1, "Base Cabinet", "Base", "30", "34 1/2", "24", 0, 2),
    (2, "Wall Cabinet", "Wall", "30", "30", "12", 0, 2),
]

ACCENT = colors.HexColor("#8B3A22")
counts = [(t.lower(), sum(1 for r in ROWS if r[2] == t)) for t in ("Base", "Wall", "Tall", "Vanity")]
parts = ", ".join(f"{n} {t}" for t, n in counts if n)
total_drawers = sum(r[6] for r in ROWS)
total_doors = sum(r[7] for r in ROWS)

doc = SimpleDocTemplate(FILENAME, pagesize=letter, leftMargin=0.7*IN, rightMargin=0.7*IN,
                        topMargin=0.7*IN, bottomMargin=0.7*IN,
                        title=f"{PROJECT} ({ROOM}) - Cabinet List")
title_style = ParagraphStyle("t", fontName="Helvetica-Bold", fontSize=16, leading=20)
sub_style = ParagraphStyle("s", fontName="Helvetica", fontSize=9.5, leading=12, textColor=colors.HexColor("#595959"))

data = [["#", "Cabinet", "Width", "Height", "Depth", "Drawers", "Doors"]]
for no, name, typ, w, h, d, dr, do in ROWS:
    data.append([str(no), name, w, h, d, str(dr) if dr else "-", str(do) if do else "-"])
data.append(["", f"Total: {len(ROWS)} cabinets ({parts})", "", "", "", str(total_drawers), str(total_doors)])

table = Table(data, colWidths=[w*IN for w in [0.4, 2.6, 0.75, 0.75, 0.65, 0.75, 0.65]], repeatRows=1)
table.setStyle(TableStyle([
    ("FONT", (0,0), (-1,0), "Helvetica-Bold", 10), ("TEXTCOLOR", (0,0), (-1,0), colors.white),
    ("BACKGROUND", (0,0), (-1,0), ACCENT),
    ("FONT", (0,1), (-1,-1), "Helvetica", 10), ("VALIGN", (0,0), (-1,-1), "MIDDLE"),
    ("ALIGN", (0,0), (0,-1), "CENTER"), ("ALIGN", (2,0), (-1,-1), "CENTER"),
    ("ROWBACKGROUNDS", (0,1), (-1,-2), [colors.white, colors.HexColor("#F6F1EE")]),
    ("LINEBELOW", (0,0), (-1,-2), 0.4, colors.HexColor("#D9D2CD")),
    ("FONT", (0,-1), (-1,-1), "Helvetica-Bold", 10), ("LINEABOVE", (0,-1), (-1,-1), 1, ACCENT),
    ("TOPPADDING", (0,0), (-1,-1), 5), ("BOTTOMPADDING", (0,0), (-1,-1), 5),
]))
doc.build([
    Paragraph(f"{PROJECT} ({ROOM}) - Cabinet List", title_style),
    Paragraph(f"{ADDRESS} &nbsp;|&nbsp; Dimensions in inches (W x H x D, box depth)", sub_style),
    Spacer(1, 12),
    table,
])
```

STEP 7 — Answer
Send me the PDF and show the same table in the chat (columns: #, Cabinet, Width, Height, Depth, Drawers, Doors, and the total line). Nothing else. Only if something could not be confirmed, add up to 3 short bullets under "To confirm", in the chat, never in the PDF.

Optional: if a Mozaik JobData.db file is also attached, take width, height and depth from its CabinetTable (values are in millimeters: divide by 25.4; numbers starting with N are not cabinets) and use the drawing only to confirm doors, drawers and names.
