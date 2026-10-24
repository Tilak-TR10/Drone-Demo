# Drone-Demo
# Drone mission — quick start

Run the mission on the **laptop**, starting with the drone centred on the fixed
charging pad and disarmed.

## What the program does

1. Confirm charging, sufficient battery, pad alignment and charger release.
2. Rise vertically to the calculated clearance height, then exit sideways through the opening.
3. Fly a circle outside the structure while retrieving photographs over Wi-Fi and saving them on the laptop.
4. Return to the outside staging point. Detect the AprilTag, establish the entry height, re-detect, approach and align.
5. Hand final descent to the autopilot, confirm touchdown and disarming, then confirm charging again.
6. Repeat if requested. A fault can pause or end the mission; an aborted flight does not count as successful docking.

The circle and its return path stay outside the structure's modelled front edge.
The same height band is used for exit, survey and return. Takeoff and landing
necessarily pass below that band.

## Where to start

All commands below run from the extracted **RB_Drone** folder.

| File | Purpose |
| --- | --- |
| `mission_config.py` | Edit measured dimensions, network settings, circle, battery thresholds and charging mode here. |
| `mission_controller.py` | Main program to launch. |
| `mission_logic.py` | Read this for the commented flight sequence and conditions for changing stages. |
| `MISSION_README.md` | Detailed setup, controls and automatic charger interface. |

The mission reuses `test4_velocity_control.py`, `detect_apriltag_image.py` and
`wifi_cam_stream.py` unchanged. Keep them and the calibration folder together.
**Do not run the old landing script alongside the mission controller.**

## Preliminaries

- **Measure the geometry:** fill every required `None` in `mission_config.py`, including vehicle dimensions with propellers, minimum roof gap, opening/pad dimensions, entry heading, resting reference height and circle radius. The estimated 40–50 cm gap is not the commanded flight height.
- **Verify vision/localisation:** check tag ID and size, tag-to-pad offset, camera calibration and coordinate frames. The tag must be visible during approach; survey flight needs a reliable continuous local position estimate.
- **Check the route:** keep the full survey disk, sideways corridor and potential emergency landing areas clear. The code does not detect obstacles.
- **Check connections:** laptop Wi-Fi must reach the camera; MAVLink telemetry must reach the laptop. Defaults are camera `192.168.0.208` and UDP port `14553`; adjust them to the actual setup.
- **Check charging and flight behavior:** configure battery reserves and verify charger release, takeoff/landing behavior and autopilot link-loss/Offboard-loss actions. A failsafe climb could conflict with the roof.

## Install and check — Windows PowerShell

Use Python 3.10 or newer:

```powershell
py -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements_mission.txt
.\.venv\Scripts\python.exe mission_controller.py --check-dependencies
.\.venv\Scripts\python.exe -m unittest discover -s tests -v
.\.venv\Scripts\python.exe mission_controller.py --check-config
```

These checks do **not** connect to or fly the drone. The configuration check
deliberately fails until the missing measurements are supplied. On Linux/macOS,
use `python3` to create the environment and `.venv/bin/python` for the commands.

## Start a mission after vehicle validation

One flight, followed by final charging confirmation:

```powershell
.\.venv\Scripts\python.exe mission_controller.py --fly --cycles 1
```

Use `--cycles 0` for repeated flights. Without `--fly`, the program only checks
configuration.

Charging defaults to **manual confirmation**: press **C** after observing actual
charging, then **G** when charged, centred and physically released. Battery and
readiness checks still apply. Unattended repetition requires a real charger
bridge using the JSON interface described in `MISSION_README.md`.

During flight: **R** returns, **P** pauses, **J** retries after a pause, and
**ESC** requests landing **at the current location** and stops repetition.
The default pause timeout is 15 seconds, after which the program requests landing
at the current location.

Pictures and pose/time metadata are saved under
`RB_Drone/mission_output/<session>/photos/`. Pose metadata is not synchronised to
the exact camera exposure time.

**Validation status:** 40 offline checks passed, including two consecutive
simulated runtime cycles. These were not PX4/SITL or real-flight tests. Physical
clearance, landing precision and charging contact still require validation on
your actual drone and station.
