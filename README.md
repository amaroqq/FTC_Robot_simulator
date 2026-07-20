# FTC Robot Simulator & Web IDE

A browser-native FTC (FIRST Tech Challenge) Robot Simulator and Interactive IDE. Write standard Java LinearOpMode code inside a Monaco editor and execute it instantly in a 3D WebGL physics simulation—no local installation, compilers, or Java SDK setup required.

---

## Key Features

* **Browser-Native Java Transpiler**: Transpiles standard FTC Java LinearOpMode code into JavaScript Generator Functions (`function*`). This yields execution back to the browser thread on every frame, maintaining a locked 60 FPS physics and WebGL rendering pipeline without freezing the UI.
* **3D Physics Engine**: Powered by Rapier.js (Rust-compiled WebAssembly physics) and Three.js (WebGL graphics), featuring real chassis mass, motor torque limits, wheel friction vectors, and dynamic object collisions.
* **Monaco Code Editor**: Full-featured code editor with syntax highlighting, line formatting, in-memory state preservation across tab switches, and dynamic layout recovery.
* **Driver Station HUD**: Authentic FTC Driver Station interface supporting INIT, START, and STOP lifecycle states, alongside real-time telemetry logging.
* **Configurable Hardware**:
  - Motor Models: GoBilda 5202 Series (312 RPM, 435 RPM, 1150 RPM) and NeveRest Classic 40.
  - Drive Architectures: Mecanum (holonomic), Traction, and Omni-wheel configurations.
  - Gear Ratios: Custom gear reduction options (1:1, 2:1, 1:2, 3:2).
  - Zero Power Behaviors: BRAKE (active physics dampening) and FLOAT (coast mode).
* **Input Support**: Full keyboard emulation (WASD + QE) as well as HTML5 Gamepad API mapping.

---

## System Architecture

+-------------------------------------------------------------------+
|                       FTC Virtual Lab IDE                         |
+--------------------------------+----------------------------------+
|           Left Pane            |            Right Pane            |
|  +--------------------------+  |  +----------------------------+  |
|  |    Monaco Java Editor    |  |  |    Three.js WebGL Scene    |  |
|  +--------------------------+  |  +----------------------------+  |
|  |  Java -> Generator AST   |  |  |   Rapier3D Physics World   |  |
|  +--------------------------+  |  +----------------------------+  |
|  | Telemetry Terminal Output |  |  | Impulse & Drive Kinematics |  |
|  +--------------------------+  |  +----------------------------+  |
+--------------------------------+----------------------------------+

### Generator Compiler Mechanics

Single-threaded browser environments freeze if exposed to blocking infinite loops like `while (opModeIsActive())`. This project solves that by converting Java blocks into JavaScript Generator Functions:

1. Replaces standard loop constructs (`while`, `for`) with yield-injected iterators.
2. Maps `sleep(ms)` and `waitForStart()` to yield-based non-blocking execution pauses.
3. Advances generator step-by-step frame-by-frame inside the main `requestAnimationFrame` loop.

---

## FTC SDK Hardware API Reference

The simulator exposes a mocked FTC Hardware API compatible with standard FTC SDK methods.

### Motors (`frontLeft`, `frontRight`, `backLeft`, `backRight`)

* **`setPower(double power)`**: Sets motor power target between -1.0 and 1.0.
* **`getPower()`**: Returns current motor power setting.
* **`getCurrentPosition()`**: Returns current encoder count (ticks) calculated from motor rotation.
* **`setTargetPosition(int target)`**: Sets target encoder ticks for `RUN_TO_POSITION` mode.
* **`setMode(DcMotor.RunMode mode)`**: Sets motor mode (`RUN_WITHOUT_ENCODER`, `RUN_USING_ENCODER`, `RUN_TO_POSITION`, `STOP_AND_RESET_ENCODER`).
* **`isBusy()`**: Returns true if target position has not yet been reached.
* **`setZeroPowerBehavior(...)`**: Sets zero power behavior to `DcMotor.ZeroPowerBehavior.BRAKE` or `DcMotor.ZeroPowerBehavior.FLOAT`.

### Gamepad Controls (`gamepad1`)

* **`gamepad1.left_stick_y`**: Range -1.0 (forward) to 1.0 (reverse). Keyboard mapping: W / S
* **`gamepad1.left_stick_x`**: Range -1.0 (left) to 1.0 (right). Keyboard mapping: A / D
* **`gamepad1.right_stick_x`**: Range -1.0 (turn left) to 1.0 (turn right). Keyboard mapping: Q / E

### Telemetry & Timing

* **`telemetry.addData(key, value)`**: Appends key-value pairs to the Driver Terminal pane.
* **`telemetry.update()`**: Renders pending telemetry entries to the screen console.
* **`sleep(long milliseconds)`**: Non-blocking pause for specified duration (Autonomous Mode).
* **`waitForStart()`**: Holds sequence execution until the Start button is clicked.
* **`opModeIsActive()`**: Returns true while the OpMode state is actively running.

---

## Holonomic Kinematics

For Mecanum wheel configurations, omnidirectional vector calculations map motor speeds as follows:

* **frontLeft** = drive + strafe + turn
* **frontRight** = drive - strafe - turn
* **backLeft** = drive - strafe + turn
* **backRight** = drive + strafe - turn

Power normalization is applied in code to prevent clipping above 1.0:

    double max = Math.max(Math.abs(fl), Math.abs(fr));
    max = Math.max(max, Math.abs(bl));
    max = Math.max(max, Math.abs(br));

    if (max > 1.0) {
        fl /= max; 
        fr /= max; 
        bl /= max; 
        br /= max;
    }

---

## Getting Started

### Option 1: Run Locally (Zero Build Step)

1. Clone this repository to your computer.
2. Open `index.html` directly in any modern Web Browser (Chrome, Firefox, Edge, Safari).

### Option 2: Host via GitHub Pages

1. Push the code to a GitHub repository.
2. Navigate to Settings > Pages.
3. Set the source branch to `main` (or `master`) and directory to `/ (root)`.
4. Click Save to get a live hosted simulator URL.

---

## Technology Stack

* **Frontend Engine**: Standard HTML5, CSS3, ES6 JavaScript
* **3D Graphics**: Three.js (r160)
* **Physics Simulation**: Rapier3D (WebAssembly Build)
* **Code Editor Integration**: Monaco Editor
* **Math Rendering**: MathJax 3

---

## License

Distributed under the MIT License.
