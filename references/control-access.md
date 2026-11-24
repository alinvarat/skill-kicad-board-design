# Control access: placement acceptance gate

Apply this review before accepting a control placement, and whenever adjacent connectors, components, mounting hardware, an enclosure, or a 3D model changes. A visible actuator and nonintersecting component bodies do not establish operability.

## Establish the action before packing parts
1. Identify the exact fitted control and intended operator action: press, slide, rotate or insert. Identify the actual actuation surface from its manufacturer drawing and calibrated model. Do not infer actuation from the footprint center, reference text, shell or camera view.
2. Record the actuation direction, start/end positions, full travel including relevant tolerances, and force when it affects operation. Read the exact part drawing; an unverified model orientation cannot establish the direction. A side-operated slide switch needs side access and lateral travel; viewing it from above does not make it a top-operated button.
3. Define the intended means of operation. Normal controls must work with the intended finger/hand; do not silently substitute a screwdriver, fingernail-only maneuver, unplugging cables or removing the board. A service tool is acceptable only when consistent with the product's service scenario. Keep USER controls primary; BOOT/RESET remain service controls unless the user specifies otherwise.
4. Reserve a three-dimensional approach path, contact/grip space and swept volume for the complete action before packing other parts. Check height as well as plan view. Use actual hardware, a measured mockup or explicitly labeled project assumptions for fingers/tools; do not invent a universal clearance value. Include both switch states and reaching the far end of travel. Keep this reserved operating space distinct from the part courtyard and the moving actuator itself.

## Check the assembled operating state
- Evaluate the board in its normal mounting orientation with fasteners, all relevant mating plugs, header sockets, cable exits/bends and enclosure walls present. Bare-board renders omit these obstacles. Check label visibility and identification in that same state.
- Walk through approach → contact → full actuation → release/withdrawal. State how the user reaches the control without contacting exposed pins, moving another control or unplugging a normal connection. Inspect useful side/oblique views or sections. A top view alone cannot establish the approach path.
- Check the controls again when a nearby tall model is repaired or a connector is moved. Improved packing or a newly correct model can invalidate an earlier access assumption even when the switch itself did not move.

## Accept or reject with evidence
Keep a compact project record: control/reference → normal/service action → exact actuator/direction/travel source → intended finger/tool → installed-state obstacles → approach and swept-space evidence → result and assumptions.

Use these result boundaries:
- **Rejected placement:** the intended operation is obstructed, requires an unagreed workaround, or contradicts the part's actuation direction. Rework the floorplan, orientation or control choice before accepting the placement. Do not report the layout as usable because DRC or body clearance passed.
- **Unverified access:** a material actuator, mating-part, mounting or operating-envelope dimension is missing. Preserve it as an open design issue; do not convert an assumption or a render into a verified usability claim.
- **Model-reviewed access:** documented assembled-state geometry supports the complete intended action, with assumptions disclosed. This is still distinct from physical validation.
- **Physically validated access:** the stated operation was actually exercised with the intended assembled hardware or an appropriate documented mockup. Record the setup and result; do not claim an unperformed trial.

For generated or tightly packed layouts, record the PCB revision/hash with this review. Placement, model, connector, enclosure or cable changes invalidate affected access results. A placement solver's objective must preserve established operating space; if it cannot fit the layout, revisit the floorplan rather than shrinking that space. Completion reports must distinguish control-access findings from electrical and fabrication checks.
