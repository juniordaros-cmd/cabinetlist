# Cabinet list instructions for Claude

The user asked you to make a cabinet list by following this page. Follow it exactly.

**Which part to use:**
- The user attached a Mozaik **JobData.db** (with or without the PDF of the job) → follow **Part A**.
- The user attached **only a PDF** of a Mozaik drawing → follow **Part B**.
- Nothing attached → ask the user to attach the JobData.db (Mozaik: File > Export > Export to Database) or the PDF of the project.

You need your code tools (Python) to read the files and build the output files. If you cannot run code or create files, tell the user to turn on code execution in Settings → Capabilities and try again.

---

# Part A - JobData.db

## Mozaik JobData.db → Cabinet List

You are preparing the cabinet list for a cabinet estimate from a job designed in Mozaik. The estimate depends on each cabinet's exact width, height and depth, so precision matters more than speed. Never guess silently.

**Main input: JobData.db**, the job's own database. In Mozaik: open the job, File > Export > Export to Database. The file is saved in the job folder (`C:\Mozaik\Jobs\<job name>\JobData.db`).
**Optional: the PDF of the same job** (plan and elevations), used only for the visual check in section 6.

If only the PDF arrives, ask for JobData.db first and explain how to export it. Use section 8 only if I say there is no database.

Our cabinets are custom: sizes are often fractional (for example 25 3/4" or 19 1/4"). That is normal.

### 0. Which rooms
- By default, list every room that has products.
- If my message names one or more rooms (for example "/mozaiklist room 2", "só a Room 2", "rooms 1 e 3", "só a Kitchen"), list only those rooms:
  - a number matches `RoomTable."Room."`, the room number in Mozaik;
  - a name matches `roomName` (ignore upper and lower case).
- Copied rooms can have the same name (for example two rooms named "Room 1 (Copy) (Copy)"). If what I wrote matches more than one room, or none, show the rooms found (number, Mozaik name, finish, number of cabinets) and ask which one before making the list.
- At the top of the answer, always say which rooms the list covers: number, Mozaik name and finish. Put the room in the file names (for example "Smith - Room 2 - Cabinet List").

### 1. Open the database
1. Open it read-only with SQLite (Python `sqlite3`, URI `file:JobData.db?mode=ro`).
2. List every table and its columns. Expected tables: JobTable, RoomTable, CabinetTable, OpeningTable, SizeTable, CutlistTable, HardwareTable, ApplTable, PriceDataTable. If a name or column is different, find the equivalent and tell me what changed.
3. **All sizes are in millimeters.** Inches = mm ÷ 25.4. Round to 1/32" and write reduced fractions (25 3/4, 34 1/2, 81 31/32). Flag a size only if it is more than 0.5 mm off the 1/16" grid, and then check it against the cut list (step 2, query 5).

### 2. Pull the data
```sql
-- 1. Job name
SELECT JobName FROM JobTable;
-- 2. Rooms
SELECT ID, "Room.", roomName, material FROM RoomTable;
-- 3. Every product (cabinets, panels, fillers, toe base)
SELECT ID, Room, "Cab.", cabName, Type, W, H, D, numDoors, numDrawers, numShelves, toeH
FROM CabinetTable ORDER BY Room, ID;
-- 4. Openings, to confirm doors and drawers
SELECT Cab, Room, "Opening.", door, drawer FROM OpeningTable;
-- 5. Side panels, to confirm height and depth
SELECT Cab, Name, PartType, WStr, LStr FROM CutlistTable
WHERE Category = 'Case Parts' AND PartType LIKE '%End%';
-- 6. Appliances
SELECT Room, item, W, H, D FROM ApplTable;
-- 7. Mozaik pricing run, only to check counts (never show prices)
SELECT PriceData FROM PriceDataTable ORDER BY ID;
```
How the tables connect:
- `CabinetTable.Room` = `RoomTable.ID`. Room label = "Room <Room.> (<finish>)". The finish (Gloss, Matte…) comes from the "Door:" part of `RoomTable.material`. Example: Room 1 (Gloss).
- The `Cab` column in OpeningTable, SizeTable, CutlistTable and HardwareTable is `CabinetTable.ID`, **not** the cabinet number.
- A room with no products (for example "Order Entry") is left out.

### 3. Separate cabinets from everything else
- **Cabinet:** `Cab.` is a number (1, 2, 3…). The number restarts in each room.
- **Not a cabinet:** `Cab.` starts with N (N1, N2…), or `Type` = SubAssembly. Usual names: Left/Right End Panel (end panel), WEP (wall end panel), TEP (tall end panel), TF (tall filler), Wall Panel Back (wall paneling, priced separately), Toe Base (toe kick). Floating shelves also go in this list (estimate unit 0.6). These go to the Panels & Fillers list with their sizes, never to the cabinet list. A panel can show `numDoors` = 1 because it uses door material. That is not a door.
- **SizeTable** holds door, drawer-front and drawer-box sizes. Never use them as cabinet sizes.

### 4. Size, doors, drawers and type of each cabinet
- **Size:** W, H, D from CabinetTable = box size. D is the depth of the box side, without the door. If the side panels in the cut list (query 5) differ from H or D, report it in Notes.
- **Doors and drawers:** `numDoors` and `numDrawers` from CabinetTable. Confirm them with OpeningTable: `door` = Door counts 1 door, Pair counts 2 doors; `drawer` = Drawer counts 1 drawer. Report any cabinet where the two don't match.
- **Never name a cabinet from `cabName`.** It is the library name, and cabinets are often edited after they are added. In a real job, "3 Drawer Bank 1-2" had 2 drawers and "Wall Single Door" had 1 drawer and no door. Name by the real doors and drawers (section 5) and write the library name in Notes: "Mozaik: <name>", or "Mozaik name '<name>', but it has <real fronts>" when they don't match.
- **Type (Base, Wall, Tall):** the database has no position (no height from the floor and no wall). Classify by size and name: H ≥ 72" = Tall; `cabName` starts with "Wall" and H > 30" = Wall; everything else = Base. If the PDF is attached, confirm every type on the elevations. Without the PDF, say once in the summary that types still need the PDF.
- **Notes:** "No toe kick" when `toeH` = 0 on a base under 34" tall; the depth when a wall or tall cabinet is deeper than 13"; accessories (trash pull-out, roll-out trays, lazy Susan, glass doors, LED); the number of shelves on open cabinets.

### 5. Naming (standard names)
Base:
- Base Cabinet: doors only, no drawer.
- Drawer Base One Drawer: exactly one drawer, with or without doors.
- Drawer Base: two or more drawers.
- Sink Base: cabinet under the sink (false fronts are not drawers).
- Blind Base: blind corner base. In Notes, the blind side, Left or Right.
- Corner Base: lazy Susan or diagonal corner.
- Microwave Base: base with a microwave drawer opening.

Wall:
- Wall Cabinet: with doors.
- Open Wall Cabinet: no doors.
- Blind Wall Cabinet: blind side in Notes.
- Corner Wall Cabinet: diagonal or corner.
- Refrigerator Wall Cabinet: wall cabinet above the refrigerator.
- Range Hood Cabinet: wood hood cover.
- Second Layer Cabinet: box stacked above other cabinets.

Tall:
- Tall Cabinet: pantry or utility, with doors.
- Open Tall Cabinet: tall, no doors (open shelves).
- Double Oven: tall cabinet with two appliance openings (oven + oven, or oven + microwave).
- Single Oven: tall cabinet with one appliance opening.
- Spice Pull-out (Tall): narrow tall pull-out.

Vanity (bathrooms): Vanity Sink Base, Vanity Drawer Base, Vanity Drawer Base One Drawer, Vanity Base Cabinet.

Sink, appliance openings and corner types are not obvious from the database. Use `cabName`, the openings and the PDF, and ask when in doubt. Give only the number of drawers and doors, never their sizes.

### 6. Checks before answering
1. **Doors and drawers:** CabinetTable = OpeningTable for every cabinet.
2. **Pricing run (only when the list covers every room, because the pricing run covers the whole job; otherwise skip it and say so):** each row is "qty,unit,name,price,…", and rows starting with 0 are headers. Under the first "Doors" header, the door-style line (for example "Slab Door") = total doors in the cabinet list, and the drawer-front line (for example "Drawer - Slab") = total drawers. "Drawer Boxes" = total drawers too. Lines under "Applied Panels" are panels, not doors.
3. **Cut list:** H and D match the side panels of each cabinet.
4. **Copied rooms:** if a room name contains "(Copy)", or two rooms have the same cabinets, say so (also when only one of them was asked for). Ask whether they are separate rooms or options of the same room before adding them together, and list what differs between them (a missing toe base, a different depth, another finish).
5. **With the PDF:** the cabinet numbers on the plan and elevations match the database for each room. Confirm each type (base, wall, tall, floating) and the sink, appliance and corner cabinets. Check each wall run: cabinets + fillers + panels = the overall wall dimension within 1/8". If a size on the PDF is different from the database, the PDF may be older; report it as a question.
6. **Prices:** the database holds Mozaik prices, including material costs. Never show them in the list, the files or the chat.

### 7. Output
Reply in the language the user writes in. Write the list in English.
1. **Excel (.xlsx), 3 sheets:**
   - Cabinet List: Room | No. | Cabinet | Type | Width | Height | Depth | Drawers | Doors | Est. Units | Notes. Sizes are numbers in inches with fraction format. Est. Units is a formula: tall = 2, all others = 1. Add a totals row, then a summary by room and for all rooms (cabinets, base, wall, tall, drawers, doors, est. units), all as formulas.
   - Panels & Fillers: Room | No. | Item | Width | Height | Depth | Notes.
   - Appliances: Room | Appliance | Width | Height | Depth (leave out "Switch" and "Door").
2. **PDF** with the same content, landscape.
3. **In the chat:** total cabinets (by room, and in total), the result of each check, and the questions in the user's language, ready to send to the designer.

### 8. Only if there is no database (PDF only)
- Use the text layer of the PDF (pdfplumber) and pages rendered at 300 dpi or more. Check the scale against 2 or 3 written dimensions.
- A box dimension is the one whose extension lines land on the cabinet sides. A front dimension lands on the edges of a door or drawer. The wall sum decides: box widths + fillers + panels = overall wall dimension.
- Depth from the plan footprint. If none is shown, use Base 24", Wall 12", Tall 24", Vanity 21".
- Add a Source column: Written, Scaled, Derived or Default. Everything that is not Written becomes a question.

Do not invent cabinets that are not in the database or the drawing.

---

# Part B - PDF only

## Cabinet list from a Mozaik drawing (one-page PDF)

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

## EXAMPLE ROWS ONLY - replace them all with the cabinets of the attached drawing, sorted by number.
## (No., Cabinet, Type, Width, Height, Depth, Drawers, Doors)  - Type is Base, Wall, Tall or Vanity
## Sizes as text with fractions ("34 1/2"); drawers and doors as numbers (0 prints as "-").
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
