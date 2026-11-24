# Dense-board and unrouted-net toolbox

Read this when a board will not route, has many unconnected items after autorouting, or placement is crowded. Evidence labels: **[verified]** observed in a real run; **[sourced]** from a vendor/datasheet/app note (URL at the end); **[unverified]** general practice, confirm before relying on it.

## 1. Diagnose before blaming density
1. Export the DRC JSON, group `unconnected_items` by net, and list the pad/track endpoints of each. **[verified]**
2. Split the list into: power/GND nets, signal nets, and LED/auxiliary nets. On one robot controller board, 28 of 81 unconnected items were VIN/GND top-side pads with no via to the bottom copper, a script gap rather than a density limit, and the 8 LED nets were under a tenth of the total.
3. Fix scripted power/GND gaps first (stitching vias from each isolated top-side pad cluster to the bottom bar or plane), then judge density from the remaining signal nets.

## 2. Choose placement by routed result, not by wirelength
- Run several placement seeds, route each one fully (same router settings), and keep the seed with the fewest unconnected items. Wirelength and zero overlap do not predict routability. **[sourced: pyplacer README, ICCAD 2011 SimPL, FieldPlacer]**
- Add a congestion term to the placement cost where possible. **[sourced]**
- Before placing parts near a fine-pitch QFN, reserve escape channels: escape the outer ring first, use dogbone vias at 0.5 mm pitch or more, and keep the surrounding area free of passives until fanout is planned. **[sourced]**
- Do not confine the placer to a bounding box that hides free corners. Measure free space first (grid occupancy, largest free rectangle) and place small auxiliary parts such as status LEDs by function in the real free areas. **[verified: a cluster term plus a bound box pushed all LEDs into the densest centre]**

## 3. Crossing and jumper tricks (two-layer boards)
- Use an existing series resistor (LED resistor, gate resistor, pull-up) as a free bridge over another net. **[unverified]**
- Add a 0 ohm SMD resistor only for signal nets that cannot be routed; never on motor/VIN current paths. Check the part's rated current in its datasheet before use. **[sourced: forum tip for keeping the ground plane intact; rating unverified]**
- Route under 0603/0402 bodies where mask and clearance allow, and through THT pads (header, JST) as layer-change points. **[unverified]**
- Put a GND via directly beside or in each decoupling-capacitor GND pad instead of routing GND. Via-in-pad needs a filled-and-capped process; open vias pull solder away. **[sourced]**
- Keep the bottom layer a continuous ground reference: cross-unders on it must be short and need a nearby return strap. Route mostly horizontal on one layer and mostly vertical on the other. **[sourced]**
- Reduce the number of nets: resistor arrays for LED resistors (check LCSC stock/class), pin/function swaps that the datasheet allows, and ESP32-S3 GPIO-matrix remapping of UART/I2C/SPI/PWM. Do not use strapping pins (GPIO0, 3, 45, 46) or GPIO33-37 on octal-PSRAM modules; confirm against the Espressif table, and update firmware when remapping. **[sourced]**

## 4. Layer count
- When a 2-layer board needs crowded switching loops plus logic routing, compare a 4-layer stackup: ground on the layer adjacent to the top (top / GND / power / bottom); a signal/power/ground/signal order lengthens return loops. Ground pours on 2-layer boards get cut into islands by routing and show up as starved thermal connections. **[sourced]**
- Check live JLCPCB quote and capability pages for the real price and via limits (multilayer minimum via hole 0.2 mm / pad 0.45 mm against 0.3 / 0.5 mm for 1-2 layers per the capabilities page). Do not quote a 4-layer price from memory. **[sourced, price not found]**

## 5. Autorouter handling (Freerouting via Specctra DSN)
- Rule count and router passes do not rescue a bad placement. **[sourced + verified]**
- Match the DSN via name to the layer count (`Via[0-1]_...` for two layers, `Via[0-3]_...` for four); a wrong name makes scripted DSN edits fail and the router run does nothing. **[verified]**
- Importing an SES deletes locked/wide power tracks; re-add them by script after every import, then refill zones. **[verified]**
- Check the actual number of routed items after each run; a run that only reports a low unrouted count because the DSN edit failed is not a result. **[verified]**

## 6. Motor-driver specifics
- DRV8701: 0.1 uF low-ESR ceramic rated for VM placed as close to the VM pin as possible, connected to GND by a thick trace or plane; a separate bulk capacitor of at least 10 uF; extra bulk near the external FETs; align the FETs as in the datasheet layout figure; connect SP and SN to the sense resistor with separate traces. **[sourced: DRV8701 datasheet]**
- Parallel FETs: one gate resistor per gate, FETs close together with identical layout, one gate trace split close to the gates. The switch node is the most critical net: short and wide. **[sourced: TI]**
- 1 oz copper at 10 A needs about 7 mm width for roughly 10 C rise (MPS); IPC-2221 at 20 C rise gives about 4.7 mm. Prefer pours to narrow traces and use large vias (0.3 mm vias conduct heat far worse than 1 mm vias). **[sourced]**

## Sources
- DRV8701 datasheet: https://www.ti.com/document-viewer/DRV8701/datasheet/layout
- TI SLVA959B motor-driver layout: https://www.ti.com/document-viewer/lit/html/SLVA959B/GUID-9223E498-2CDA-48C5-8ACF-273605795A5F
- MPS motor driver layout part 1: https://www.monolithicpower.com/en/motor-driver-pcb-layout-guidelines-part-1/
- Two-layer routing tips: https://www.hackster.io/news/pcb-friday-two-layer-pcb-routing-strategies-and-tips-1c6b8cfcf5d3 , https://www.signalintegrityjournal.com/blogs/12-fundamentals/post/1207-seven-habits-of-successful-2-layer-board-designers , https://www.edaboard.com/threads/2-layer-board-routing-guidelines.266271/
- Fanout and escape routing: https://jlcpcb.com/blog/effective-escape-routing-strategies , https://resources.altium.com/p/utilizing-fanout-strategies-to-maximize-pcb-real-estate
- Placement for routability: https://github.com/ajokela/pyplacer , https://web.eecs.umich.edu/~imarkov/pubs/conf/iccad11-simplr.pdf
- 2 vs 4 layer: https://www.elektormagazine.com/labs/why-compact-embedded-designs-use-4-layer-pcbs , https://resources.pcb.cadence.com/blog/2022-circuit-board-4-layer-pcb-stackup-planning
- ESP32-S3 pins: https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32s3/schematic-checklist.html
- JLCPCB capabilities: https://api.jlcpcb.com/capabilities/Capabilities , https://jlcpcb.com/blog/bga-pcb-design-complete-guide-layout-and-routing-guidelines
