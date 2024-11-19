### **I/O Mapping and Variable Naming**

Based on the terminal assignments you've provided, here's a list of inputs and outputs with reasonably named variables:

#### **Inputs (Sensors and Push-buttons)**

| Terminal No. | Variable Name                   | Description                                   |
|--------------|---------------------------------|-----------------------------------------------|
| I1 (5)       | `Slider1_Front_PB`              | Push-button Slider 1 Front                    |
| I2 (6)       | `Slider1_Rear_PB`               | Push-button Slider 1 Rear                     |
| I3 (7)       | `Slider2_Front_PB`              | Push-button Slider 2 Front                    |
| I4 (8)       | `Slider2_Rear_PB`               | Push-button Slider 2 Rear                     |
| I5 (9)       | `Slider1_PhotoSensor`           | Phototransistor Slider 1                      |
| I6 (10)      | `MillingMachine_PhotoSensor`    | Phototransistor Milling Machine               |
| I7 (11)      | `LoadingStation_PhotoSensor`    | Phototransistor Loading Station               |
| I8 (12)      | `DrillingMachine_PhotoSensor`   | Phototransistor Drilling Machine              |
| I9 (13)      | `OutputConveyor_PhotoSensor`    | Phototransistor Output Conveyor Belt          |

#### **Outputs (Motors and Actuators)**

| Terminal No. | Variable Name                   | Description                                   |
|--------------|---------------------------------|-----------------------------------------------|
| Q1 (15)      | `Slider1_Motor_Forward`         | Motor Slider 1 Forward                        |
| Q2 (16)      | `Slider1_Motor_Backward`        | Motor Slider 1 Backward                       |
| Q3 (17)      | `Slider2_Motor_Forward`         | Motor Slider 2 Forward                        |
| Q4 (18)      | `Slider2_Motor_Backward`        | Motor Slider 2 Backward                       |
| Q5 (19)      | `FeedConveyor_Motor`            | Motor Feed Conveyor Belt                      |
| Q6 (20)      | `MillingConveyor_Motor`         | Motor Conveyor Belt Milling Machine           |
| Q7 (21)      | `MillingMachine_Motor`          | Motor Milling Machine                         |
| Q8 (22)      | `DrillingConveyor_Motor`        | Motor Conveyor Belt Drilling Machine          |
| Q9 (23)      | `DrillingMachine_Motor`         | Motor Drilling Machine                        |
| Q10 (24)     | `OutputConveyor_Motor`          | Motor Output Conveyor Belt                    |

---

### **Structured Text Programming**

Below, I'll provide a sample Structured Text program that outlines the control logic for the indexed line. This includes loading a workpiece, processing it through the milling and drilling machines, and moving it to the output conveyor.

#### **Variable Declarations**

First, declare your variables in the **Global Variables** or at the beginning of your program:

```pascal
// Inputs
VAR
    Slider1_Front_PB        : BOOL; // I1
    Slider1_Rear_PB         : BOOL; // I2
    Slider2_Front_PB        : BOOL; // I3
    Slider2_Rear_PB         : BOOL; // I4
    Slider1_PhotoSensor     : BOOL; // I5
    MillingMachine_PhotoSensor : BOOL; // I6
    LoadingStation_PhotoSensor : BOOL; // I7
    DrillingMachine_PhotoSensor : BOOL; // I8
    OutputConveyor_PhotoSensor  : BOOL; // I9
END_VAR

// Outputs
VAR
    Slider1_Motor_Forward   : BOOL; // Q1
    Slider1_Motor_Backward  : BOOL; // Q2
    Slider2_Motor_Forward   : BOOL; // Q3
    Slider2_Motor_Backward  : BOOL; // Q4
    FeedConveyor_Motor      : BOOL; // Q5
    MillingConveyor_Motor   : BOOL; // Q6
    MillingMachine_Motor    : BOOL; // Q7
    DrillingConveyor_Motor  : BOOL; // Q8
    DrillingMachine_Motor   : BOOL; // Q9
    OutputConveyor_Motor    : BOOL; // Q10
END_VAR

// Internal Variables
VAR
    System_Enabled          : BOOL := TRUE; // System Start Control
    Workpiece_Loaded        : BOOL := FALSE;
    Milling_Complete        : BOOL := FALSE;
    Drilling_Complete       : BOOL := FALSE;
    Milling_Timer           : TON; // Timer for Milling Operation
    Drilling_Timer          : TON; // Timer for Drilling Operation
END_VAR
```

#### **Control Logic Implementation**

```pascal
// Initialize Outputs
Slider1_Motor_Forward   := FALSE;
Slider1_Motor_Backward  := FALSE;
Slider2_Motor_Forward   := FALSE;
Slider2_Motor_Backward  := FALSE;
FeedConveyor_Motor      := FALSE;
MillingConveyor_Motor   := FALSE;
MillingMachine_Motor    := FALSE;
DrillingConveyor_Motor  := FALSE;
DrillingMachine_Motor   := FALSE;
OutputConveyor_Motor    := FALSE;

// Main Control Logic
IF System_Enabled THEN
    // **Stage 1: Load Workpiece onto Slider 1**
    IF NOT Workpiece_Loaded THEN
        FeedConveyor_Motor := TRUE;
        IF LoadingStation_PhotoSensor THEN
            FeedConveyor_Motor := FALSE;

            // Move Slider 1 Forward
            IF NOT Slider1_Front_PB THEN
                Slider1_Motor_Forward := TRUE;
            ELSE
                Slider1_Motor_Forward := FALSE;

                // Move Slider 1 Backward
                IF NOT Slider1_Rear_PB THEN
                    Slider1_Motor_Backward := TRUE;
                ELSE
                    Slider1_Motor_Backward := FALSE;
                    Workpiece_Loaded := TRUE;
                END_IF;
            END_IF;
        END_IF;
    END_IF;

    // **Stage 2: Milling Process**
    IF Workpiece_Loaded AND NOT Milling_Complete THEN
        // Start Milling Conveyor
        MillingConveyor_Motor := TRUE;
        IF MillingMachine_PhotoSensor THEN
            MillingConveyor_Motor := FALSE;

            // Start Milling Machine
            MillingMachine_Motor := TRUE;

            // Start Milling Timer (e.g., 5 seconds)
            Milling_Timer(IN := TRUE, PT := T#5S);

            IF Milling_Timer.Q THEN
                MillingMachine_Motor := FALSE;
                Milling_Complete := TRUE;
                Milling_Timer(IN := FALSE);
            END_IF;
        END_IF;
    END_IF;

    // **Stage 3: Drilling Process**
    IF Milling_Complete AND NOT Drilling_Complete THEN
        // Start Drilling Conveyor
        DrillingConveyor_Motor := TRUE;
        IF DrillingMachine_PhotoSensor THEN
            DrillingConveyor_Motor := FALSE;

            // Start Drilling Machine
            DrillingMachine_Motor := TRUE;

            // Start Drilling Timer (e.g., 5 seconds)
            Drilling_Timer(IN := TRUE, PT := T#5S);

            IF Drilling_Timer.Q THEN
                DrillingMachine_Motor := FALSE;
                Drilling_Complete := TRUE;
                Drilling_Timer(IN := FALSE);
            END_IF;
        END_IF;
    END_IF;

    // **Stage 4: Move Workpiece to Output Conveyor**
    IF Drilling_Complete THEN
        OutputConveyor_Motor := TRUE;
        IF OutputConveyor_PhotoSensor THEN
            OutputConveyor_Motor := FALSE;

            // Reset Process Flags for Next Workpiece
            Workpiece_Loaded    := FALSE;
            Milling_Complete    := FALSE;
            Drilling_Complete   := FALSE;
        END_IF;
    END_IF;
ELSE
    // If System is Disabled, Ensure All Motors are Stopped
    Slider1_Motor_Forward   := FALSE;
    Slider1_Motor_Backward  := FALSE;
    Slider2_Motor_Forward   := FALSE;
    Slider2_Motor_Backward  := FALSE;
    FeedConveyor_Motor      := FALSE;
    MillingConveyor_Motor   := FALSE;
    MillingMachine_Motor    := FALSE;
    DrillingConveyor_Motor  := FALSE;
    DrillingMachine_Motor   := FALSE;
    OutputConveyor_Motor    := FALSE;
END_IF;
```

#### **Explanation of Control Logic**

- **System Enable Control:**
  - The entire operation is enclosed within an `IF System_Enabled THEN` condition, allowing you to start or stop the system as needed.

- **Stage 1 - Loading the Workpiece:**
  - The feed conveyor brings the workpiece to the loading station.
  - When the `LoadingStation_PhotoSensor` detects a workpiece, the conveyor stops.
  - Slider 1 moves forward until `Slider1_Front_PB` is activated.
  - Then, it moves backward until `Slider1_Rear_PB` is activated.
  - Once back at the rear position, `Workpiece_Loaded` is set to `TRUE`.

- **Stage 2 - Milling Process:**
  - The milling conveyor starts moving the workpiece to the milling machine.
  - When `MillingMachine_PhotoSensor` detects the workpiece, the conveyor stops.
  - The milling machine motor starts, and a timer (`Milling_Timer`) runs for the duration of the milling process.
  - After the timer elapses, the milling machine stops, and `Milling_Complete` is set to `TRUE`.

- **Stage 3 - Drilling Process:**
  - The drilling conveyor moves the workpiece to the drilling machine.
  - Upon detection by `DrillingMachine_PhotoSensor`, the conveyor stops.
  - The drilling machine motor starts, controlled by `Drilling_Timer`.
  - After drilling is complete, the machine stops, and `Drilling_Complete` is set to `TRUE`.

- **Stage 4 - Output Conveyor:**
  - The output conveyor moves the finished workpiece out of the system.
  - When `OutputConveyor_PhotoSensor` detects that the workpiece has exited, the conveyor stops.
  - Process flags are reset to allow the next workpiece to be processed.

- **Safety and Reset Conditions:**
  - All motors are set to `FALSE` when the system is disabled to ensure safety.
  - Timers are reset after their respective processes are complete.

---
