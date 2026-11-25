# Finishing a board: brief, budget and convergence

Read this before placement or routing of a new/substantially changed board, when resuming an unfinished board, and whenever a routing session ends with unconnected items. Its purpose is a finished board, not more experiments.

## 1. Project brief (one file, in the project)
Keep `DESIGN_BRIEF.md` in the project root. Create it if missing; read it first on every resume. It holds:
- **Product and use**: what the board is for, who uses it, how it is mounted, powered, plugged and programmed in real use (from project evidence and user statements, not guesses).
- **Frozen spec**, one line each with the date the user confirmed it: supply range, current per output (continuous / peak / duration / simultaneous?), copper weight, layer count, outline and mounting holes, which side is machine-assembled and which is hand-soldered, fab/assembler rule set, cost priority, features that must stay.
- **User rulings**: every "do not" and "must" the user stated (for example: do not enlarge the board, do not remove protection circuits, indicator LEDs sit next to what they indicate). Quote the user's words with a date.
- **Open questions** that block a decision.

Never change a frozen item yourself (for example, trying a 4-layer stackup when the user fixed 2 layers). You may test an alternative in a separate folder only to show the user its numbers. Never promote it to the main file without the user's explicit yes. A spec change (current, layers, copper, outline, assembly side) restarts the feasibility budget in §2 for the affected blocks. Record the change and its date in the brief before continuing; do not patch it into a half-routed board silently.

## 2. Feasibility budget before placement
Do these quick calculations and write the results into the brief. If one fails, present the user one decision with your recommendation (shrink feature set, more layers, other package, larger board, other assembly split) before placing parts.
1. **Area**: per side, sum of courtyard areas plus an escape margin (fine-pitch IC: perimeter × 0.6 mm or more; others: 0.25 mm around). Compare with usable area (outline minus keepouts, connector envelopes, mounting hardware, antenna keepout). Treat about 60 % on a 2-layer board or 75 % on 4 layers as a warning, not a pass line (unverified rule of thumb; a 2-layer motor board with one side mostly power copper already failed at 57 %). Count the side carrying power pours as only partly available for signals.
2. **Current paths**: for each high-current path, required copper width (IPC-2152 or IPC-2221, stated ΔT and copper weight, layers sharing current) against the narrowest real corridor, including connector pad pitch, the gaps between pads and the via field. A path that cannot fit is a spec conflict, not a routing problem.
3. **Escape**: for each QFN/BGA/fine-pitch connector, count nets that must leave each side and the channels available at the chosen track/clearance/via sizes.
4. **Layer decision**: if 1–3 are tight, compare layer counts now. Check the live fab price and rules, never prices from memory.

## 3. Status file bound to the board
Keep `STATUS.md` in the project root. Put the SHA-256 of the main `.kicad_pcb` at the top, and under it the latest fresh DRC numbers (unconnected, errors by type, parity), the report path and the next three actions. Update it on every promote. On resume, compare the hash with the actual file before believing anything else; a mismatch means the status is stale, so re-run DRC first.

## 4. Convergence rules for routing
1. **One baseline, one metric.** Work on one candidate file. The metric is a fresh DRC after zone refill: unconnected count, then error count (track width below fab minimum, clearance, dangling vias, starved thermals and hole clearance all count). Save a numbered checkpoint after each accepted step.
2. **Every attempt must improve the metric or be reverted.** Write the before/after numbers into `STATUS.md`. Do not keep side variants alive.
3. **Fixed order.** Route and lock: (a) power and return copper with planes/pours, (b) switching hot loops, gate-drive and bootstrap loops, (c) decoupling, (d) USB/fast/sensitive nets, (e) remaining slow signals. An autorouter only gets (e), with (a)–(d) locked; re-add any locked copper that an SES import deleted.
4. **Stop-loss.** After two routing strategies stall (no metric improvement), do not write another router, placer or autorouter variant. Choose one: close the remaining items per net (§5), move or rotate the specific blocking parts, or bring one constraint decision to the user (§2).
5. **Autorouter setup checks before each run** (Specctra DSN / Freerouting):
   - Rip up by an explicit net list. Keep power, ground and their vias; count vias per net before and after the rip-up and stop if a power/GND count changed.
   - Remove ground/power pour planes and the ground net from the DSN copy. Pours exported as full-board planes block routing; re-pour after import.
   - Mark all kept copper as fixed (`(type fix)`), add an edge keepout ring at the fab copper-to-edge distance, and use the via size of the real stackup.
   - After importing the SES, re-add any kept copper the import deleted, refill zones, then run a fresh DRC.
6. **Router output acceptance.** A router pass counts only when its tracks are at or above the fab minimum width, have no dangling vias, and the refilled zones still reach every thermal pad. Otherwise the pass is rejected, not "nearly done".

## 5. Closing the last unconnected items
Work through the fresh DRC unconnected list grouped by net:
1. **Power/GND fragments**: a pour island or stub that does not reach its plane needs a via to the plane layer next to the pad, a widened neck, or a zone priority/clearance change. Check after refill that it is gone.
2. **Signal near a dense IC**: crop the search to the smallest box holding both endpoints and the real blockers; add a via on the escape side; use an inner layer; route under a 0603/0805 body; use an existing series resistor or a 0 Ω jumper as a bridge (signals only, rating checked); swap pins/functions only where the datasheet and firmware allow it, and update the schematic.
3. **Long cross-board signal**: assign it a dedicated channel instead of weaving: an inner layer on multilayer boards; on 2 layers, a reserved strip on the side with less power copper (for example between connector rows), crossing to the other side only with short hops.
4. After each net: refill, DRC, confirm the count fell and nothing new appeared. When an item truly cannot close, list it with the reason and the exact decision needed.

## 6. Session hand-off
End every session with `STATUS.md` updated (hash, fresh numbers, what changed, next actions) and old trial folders either deleted after confirmation or listed as obsolete. The next session must be able to continue from `DESIGN_BRIEF.md` + `STATUS.md` alone.
