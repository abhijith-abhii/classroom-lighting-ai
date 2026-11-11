# Code walkthrough and interview guide

Start in app.py: it reads project.json, creates the Flask application and optionally opens a recorded video. lab/web.py defines the request boundary and serializes inference. lab/inference.py converts pixels into boxes or probabilities; it does not decide what a physical actuator should do. lab/tracking.py links boxes by class and overlap, expires tracks and counts line crossings. lab/runtime.py combines inputs, advances the explicit clock and calls the controller. lab/core.py contains deterministic state transitions without a camera or browser dependency. lab/hal.py supplies clocks, sensors, outputs and SQLite persistence. static/app.js displays state and sends explicit user actions.

Read one scenario JSON next. Each object is a command, such as setting sensor values, checking, or advancing time. Run the matching test in tests/test_core.py and change one input. Then inspect evidence/results.json and the raw trace to connect a rule to a measured outcome. The projects share a controller library and regression suite; the repeated 56-test result is the same suite run in each standalone package, not 168 unique tests.

## Decisions and alternatives

Higher resolution increased F1 from 0.632 to 0.736 but lowered precision from 0.867 to 0.829 and increased detector median host latency from 13.31 to 48.50 ms. The combined policy kept lights on throughout the difficult one-hour scripted trace: occupied-off time fell from 1200 to 0 seconds, while assumed energy rose from 40 to 120 Wh. This experiment does not demonstrate energy savings.

Python and ONNX make desktop replay practical. A bare-metal RTOS or emulator would not validate camera models without a target-specific peripheral model. A class-aware IoU tracker is simple to inspect but can switch IDs during occlusion; an appearance tracker could improve association at added cost and still would not establish identity or liveness. SQLite keeps events and state atomic, while physical side effects require a separate acknowledgement design.

## Ten interview questions

### 1 Why are stationary people retained?

Motion is an event trigger, not occupancy truth. Visual person candidates keep lights on even if the motion signal is false.

### 2 Why not solve posters with tracking?

A static depiction can produce a stable track. This project found person candidates in 9/10 synthetic framed photos; known zones mark uncertainty, but liveness remains unsolved.

### 3 What does a ten-minute interval miss?

A short visit can start and end between checks. Event triggers request checks sooner; reliable door coverage and physical tests are still needed.

### 4 Did the final controller save energy?

No. On the adversarial hour it kept lights on continuously: 120 estimated Wh versus 40 Wh. Comfort and conservative uncertainty increased energy use.

### 5 How do you prevent test leakage?

Freeze selection using validation before reading final test scores. Exact duplicates are grouped for TrashNet; COCO images are disjoint. Scene identifiers are missing, so independence is limited. Future video splits must hold out entire recordings and locations.

### 6 How do you distinguish confidence from accuracy?

Confidence is one model score. Accuracy is a frequency measured against held-out labels. The dashboard displays scores while the evidence files report measured performance.

### 7 Why use an explicit simulation clock?

It makes long timeouts deterministic and replayable. A 600-second step integrates one-second controller ticks. This does not emulate Pi scheduling, camera timing or electrical behavior.

### 8 What happens when saving fails?

SQLite commits an event ID and state together. Failed commits restore controller and inference/tracker snapshots. A real actuator cannot be rolled back this way, so physical integration needs an outbox and acknowledgement protocol.

### 9 How is a retry different from a new event?

Reusing an event_id is idempotent. A new event ID is a new command. Waste item IDs prevent repeat counts within a bounded history; a production system needs a durable item ledger across unlimited retention.

### 10 What would you measure on a Raspberry Pi?

End-to-end latency including capture, missed deadlines under CPU load, thermal throttling, process memory, sensor stale time, restart behavior, supply voltage and motor/LED timing. Host measurements cannot be relabeled as Pi results.

## Modifications to implement yourself

1. Add a minimum time separation between negative confirmations and test two rapid Check now clicks.
2. Implement a door-triggered shorter follow-up cadence and compare both occupied-off time and empty-on time on the preserved trace.
3. Label a new, consented scene set containing empty chairs, bags, screens and stationary occupants; hold out entire rooms before experimenting with depiction regions.

## Evidence backed resume bullets

- Built a Raspberry Pi–targeted occupancy simulator with YOLO11n inference, IoU tracking, event-triggered checks, manual override and persistent decision logs.
- Compared 320- and 640-pixel inference on 78 held-out COCO images, improving person F1 from 0.632 to 0.736 while documenting latency and precision costs.
- Reduced occupied-room light-off time from 1200 to 0 seconds in a 3600-second scripted trace; documented the increase from 40 to 120 Wh under an assumed 120 W load.

Use these after you can explain and reproduce the implementation. Keep simulation and AI assistance clear in the project discussion.
