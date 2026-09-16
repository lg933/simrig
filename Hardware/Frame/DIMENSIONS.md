# F-GT Elite 160: dimensions and measurement sheet

Source data for modelling the Next Level Racing F-GT Elite 160 (Wheel Plate / bottom-mount edition).
Vendor figures are copied from NLR's product page and manual. Everything else is measured on the
rig with a tape and filled in here before any CATPart is drawn in `CAD/`.

## Vendor data (NLR product page, fetched 2026-09-16)

Page: https://nextlevelracing.com/products/elite-160-wheel-plate-edition/
Manual: https://cdn.nlr.sh/wp-content/uploads/2023/03/FINAL-FOR-PRINT-F-GT-ELITE-160-BOTTOM-MOUNT-07102022.pdf

| Item | Value |
|---|---|
| Part number | NLR-E025 |
| Extrusion sections | 40/40, 40/80, 40/120, 40/160 aluminium, custom NLR profile, 8 mm T-slot (M8 T-nuts) |
| Product dimensions (as listed) | 136 cm (L) x 84.2 cm (W) x 94.2 cm (H) |
| Boxed dimensions | 126.5 cm (L) x 31.5 cm (W) x 44 cm (H) |
| Net weight | 57.1 kg (marketing copy says 60 kg) |
| Boxed weight | 63.6 kg |
| Finish | powder coat / anodised, laser-engraved alignment lines |
| Pedal plates | 2 x slotted 5 mm carbon steel |
| Seat brackets | 5 mm carbon steel |

The listed 136 cm length is shorter than an assembled rig with the pedal arms and seat sliders in a
normal driving position. Treat it as the compact/base figure and use the tape measure for the
as-built envelope. The manual's parts pages are marked "not to scale" and contain no lengths.

## Frame members (from the manual's parts list, sections inferred from the end caps)

Fill in the length column on the rig. Length is the raw extrusion length, end cap to end cap, not
including caps. Note the cut angle for mitred members.

| # | Member | Qty | Section | Length (mm) | End cut | Notes |
|---|---|---|---|---|---|---|
| 1 | Base Left | 1 | 40 x 160 | | square | handed end caps (left/right) |
| 2 | Base Right | 1 | 40 x 160 | | square | |
| 3 | Rear Base | 1 | 40 x 160 | | square | bolts between the two base members |
| 4 | Vertical Post Left | 1 | 40 x 120 | | angled | angle to measure |
| 5 | Vertical Post Right | 1 | 40 x 120 | | angled | |
| 6 | Pedal Arm Left | 1 | 40 x 80 | | angled | |
| 7 | Pedal Arm Right | 1 | 40 x 80 | | angled | |
| 8 | Seat Slider Member | 2 | 40 x 80 | | square | |
| 9 | Shifter Arm Support | 1 | 40 x 120 or 40 x 80 | | angled | confirm section on the rig |
| 10 | Shifter Arm | 1 | 40 x 80 | | square | |
| 11 | Buttkicker Mount Pole | 1 | 40 x 40 | | square | |
| 12 | Front Panel | 1 | sheet | | | perforated steel, measure W x H x t |

End cap count in the box: 6 x (40 x 160, handed), 6 x (40 x 120 angled), 6 x (40 x 80),
6 x (40 x 80 angled), 4 x (40 x 40). Use it to check the section assignments above.

## Plates and brackets (5 mm steel unless noted)

| Part | Qty | Overall (mm) | Hole / slot pattern | Notes |
|---|---|---|---|---|
| Wheel Plate | 1 | | | bottom mount, MOZA R12 bolt pattern |
| Pedal Plates | 2 | | | slotted, double fold |
| Pedal Brackets | 2 | | | |
| Pedal Adjustment Fins | 2 | | | |
| Seat Brackets | 2 | | | |
| Seat Slider Tabs | 4 | | | |
| Upright Connection Plates | 2 | | | |
| Shifter Arm Brackets | 3 | | | |
| Shifter Arm Plate / Shifter Plate | 1 + 1 | | | |
| Motion V3 Brackets | 2 | | | present if platform is NLR Motion V3 |
| Safety Step | 1 | | | |

## As-built positions (needed for the assembly, not the parts)

| Setting | Value | How measured |
|---|---|---|
| Driving position | Formula / Hybrid / GT | which shoulder-bolt slot is in use |
| Upright angle from vertical | | digital level on the post face |
| Seat slider position | | from rear base to seat bracket front edge |
| Seat bracket slot in use | | |
| Pedal arm angle | | level on the arm |
| Pedal plate position | | slot number, height from floor |
| Wheel plate height from floor | | |
| Wheel plate distance from seat back | | |
| Overall footprint L x W x H | | tape, as assembled |

## Peripherals to model as envelopes

Block models only, no detail. Overall dimensions plus mounting bolt pattern.

| Item | Model | L x W x H (mm) | Mount pattern | Source |
|---|---|---|---|---|
| Wheelbase | MOZA R12 | | | MOZA spec sheet |
| Pedals | MOZA CRP | | | |
| Seat | Sparco (model on the label) | | side-mount hole spacing | |
| Monitor | Samsung Odyssey Neo G9 LS49AG95 | | VESA 100 x 100 | Samsung spec sheet |
| Monitor stand | | | | |
| Motion platform | see README, identity to confirm | | | |
| Stream Deck XL | | | | |
