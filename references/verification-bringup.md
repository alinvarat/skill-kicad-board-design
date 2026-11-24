# Verification, release evidence and bring-up

## Final-file checks
For a label-only change with unchanged circuit/copper, inspect the diff, save/reopen and check label readability, orientation and relevant silk/clearance rules. Do not rerun unrelated electrical analysis or ERC solely because text changed. For a complete-board review or production handoff, apply all relevant checks below.

1. Save and reopen the edited project with the intended KiCad version. Inspect for load warnings, lost footprints/models, changed nets, references or geometry. Compare the actual diff against scope. For generated work, verify source and output agree without destroying manual changes.
2. Refill all copper zones after geometry/net/rule changes and save the filled board. Do not assume a DRC command performs the refill: check installed command capabilities, otherwise use the PCB Editor. Run ERC on the root hierarchical schematic and DRC on the saved board with the project's libraries/settings available. Test schematic parity when a corresponding schematic exists.
3. Read the reports themselves: errors, warnings, exclusions, unconnected items and parity results. A successful process exit can mean only that the report was generated. Explain each accepted finding with component/net/location and consequence; do not weaken rules, add fake PWR_FLAGs or no-connect marks to hide a real problem.
4. Review at least the actual changed regions plus their affected power/return/interface paths in 2D; use 3D for assembly/mechanical review. Geometry counts, file existence and metadata validation cannot establish visual or functional correctness. For affected controls, retain the control-access review separately from DRC and 3D body-clearance results. Recheck the installed/plugged operating state after moving adjacent parts; previous usability conclusions are then stale.
5. Keep report paths, KiCad version, revision/hash and material assumptions with the project. After an edit, repeat affected checks against that final state; stale reports do not qualify. For production artifacts use pcb-fab-prep and inspect the exported files.

## CLI examples
Check `kicad-cli version`, `kicad-cli sch erc --help` and `kicad-cli pcb drc --help` first. These options were checked on KiCad 9.0.2; use the installed version's supported options. Its help may print an input-argument complaint while still displaying usage.

The following Python example runs in the project directory. Replace the two illustrative filenames with the actual root schematic and PCB filenames. Refill/save the board first. Use a new report directory for each run.

```python
from datetime import datetime
from pathlib import Path
import subprocess

reports = Path('reports') / datetime.now().strftime('%Y%m%d-%H%M%S-%f')
reports.mkdir(parents=True, exist_ok=False)
commands = [
    ['kicad-cli', 'sch', 'erc', '--format', 'json', '--severity-all',
     '--exit-code-violations', '--output', str(reports / 'erc.json'),
     'board.kicad_sch'],
    ['kicad-cli', 'pcb', 'drc', '--format', 'json', '--severity-all',
     '--schematic-parity', '--exit-code-violations', '--output',
     str(reports / 'drc.json'), 'board.kicad_pcb'],
]
for command in commands:
    result = subprocess.run(command, capture_output=True, text=True)
    print(command, 'exit:', result.returncode)
    print(result.stdout, result.stderr)
```

Read both generated JSON reports. A nonzero code can indicate violations or an execution failure; distinguish using the messages and actual report. Missing input, missing report or failed parity evaluation is a failed/unavailable check, not zero violations. If no schematic exists, omit ERC/parity and explicitly record the missing coverage. If CLI is unavailable, use the GUI equivalent and retain the actual result; if neither works, state verification is blocked.

## Staged bring-up for a new/substantially changed board
Write a project-specific table: test → setup/instrument → stimulus → expected range/pass criterion → stop condition → evidence. Derive limits from the design requirements. Do not invent universal rail tolerances or current limits.

- Unpowered: inspect orientation/assembly (check gold-plated pads for full silver wetting/fillets, especially on QFNs and castellated modules), verify intended isolation and check rail resistance/shorts with awareness of capacitors and semiconductor junctions. Probe continuity leg-to-leg (IC pin to IC pin) rather than pad-to-pad so open or cold solder joints are caught.
- First power: use a suitable current-limited source and safe load state; verify input current, rails, sequencing, ripple and unexpected heating. Stop for abnormal current/voltage/temperature before connecting valuable loads.
- Control: verify reset/clock/boot/programming (including RC reset/boot delay timing and differential-pair phase/bus decoding with a triggered scope or logic analyzer), communications, interlocks and default output states before enabling power stages. During rework, limit iron dwell time to avoid pad delamination, avoid repeated full-board re-reflow, and cut outer-layer traces cleanly with magnet-wire jumpers to pads or test points when patching nets.
- Load: increase load in stages; capture droop/ripple, switching overshoot/ringing, sense/trip behavior and efficiency with appropriate probing. Specify probe reference/bandwidth and avoid measurement-loop artifacts. Never connect an earth-referenced scope ground to a live switching or hazardous node.
- Corners/faults: test justified supply/load/temperature extremes, startup/shutdown, stalled load or short-circuit response using a controlled setup. Record actual duration, recovery and thermal steady-state criteria; a brief successful run is not a continuous current rating.
- SI/RF/EMC where applicable: measure the required timing/eye/impedance/RF/ESD/emission behavior with the proper setup and acceptance criteria. Simulation or visual review does not replace compliance testing.

Handoff distinguishes design checks passed, simulations completed, physical tests completed and remaining release blockers. Never report unperformed measurements as passed.

## Tool references
[KiCad CLI manual](https://docs.kicad.org/9.0/en/cli/cli.html) and [PCB Editor manual](https://docs.kicad.org/9.0/en/pcbnew/pcbnew.html) cover ERC/DRC, forward annotation, zones and inspection. Match documentation to the installed version; the 9.0 web manual can describe later 9.0 patch releases than a local installation.
