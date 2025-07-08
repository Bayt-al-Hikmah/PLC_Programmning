## Objectives
- Logic Control Instructions
- Working with Counters
- Working with Timers
## Logic Control Instructions
### Introduction
Logic control instructions, also known as jump instructions, are fundamental tools in programming. They allow us to control the program's execution flow by jumping to specific sections of code when a certain condition is met. Using these instructions, we can build sophisticated program logic, creating different execution paths that depend on system inputs and states. This enables our program to react dynamically instead of just running from top to bottom.  
The core components for using jump instructions are:
- **The Jump Instruction:** The command that tells the CPU to go to another part of the code (e.g., `JC`).
- **The Destination Label:** A named marker in the code that the jump instruction targets (e.g., `PART2`).
### Understanding Key Concepts
Before we look at the specific instructions, it's important to understand the signals the PLC uses to make jump decisions.  
- RLO (Result of Logic Operation): The RLO is a single-bit memory that stores the result of the last logic operation (like A for AND, O for OR). It's the primary driver for conditional jumps. If the result of a logic string is TRUE, the RLO is 1. If it's FALSE, the RLO is 0.
- Status Word: This is a special register in the CPU that contains several bits of information about the result of the last operation. Jumps can directly check these bits. Key bits include:
    - **/FC (First Check):** The RLO is copied to the /ER bit at the beginning of a logic string.
    - BR (Binary Result): This bit makes the state of the RLO available for further processing. A SAVE instruction can be used to store the RLO into the BR bit, preserving it for later checks.
    - OV (Overflow): Set to 1 if the result of a mathematical calculation is too large or too small for the data type.
    - OS (Stored Overflow): The OV bit is saved to the OS bit if an error occurs.

### Jump Instructions
#### Unconditional Jump
This instruction jumps to a label every time it is executed, without checking any conditions.
- **JU \<label\>** (Jump Unconditionally): Immediately jumps to the specified destination label.
    - Example: **JU SKIP_CALCS** will always redirect the program flow to the line marked SKIP_CALCS:.
- **JL \<label\>** (Jump via Jump List): This is an advanced instruction that enables multiple jumps, acting like a CASE or SWITCH statement. It selects a jump destination from a list based on a number in the accumulator.
	- The value in ACCU 1-L-L determines which JU instruction to execute from a list that follows the JL instruction.
	- If ACCU 1-L-L = 0, it jumps to the first JU instruction in the list.
	- If ACCU 1-L-L = 1, it jumps to the second JU instruction, and so on.
	- If ACCU 1-L-L = > then the numbers of Jump destiation, it jumps to the Label that come after the instruction.
	- The list can contain up to 255 `JU` jump destinations. If the value in the accumulator is larger than the number of available jumps, the program jumps to the instruction immediately following the list.
```
L MB0 //Load jump destination number into ACCU 1-L-L. 
JL LSTX //Jump destination if ACCU 1-L-L > 3. 
JU SEG0 //Jump destination if ACCU 1-L-L = 0. 
JU SEG1 //Jump destination if ACCU 1-L-L = 1. 
JU COMM //Jump destination if ACCU 1-L-L = 2. 
JU SEG3 //Jump destination if ACCU 1-L-L = 3. 
LSTX: JU COMM 
SEG0: 
* //Permitted instruction 
* 
JU COMM 
SEG1: 
* //Permitted instruction 
* 
JU COMM 
SEG3: 
* //Permitted instruction. 
* 
JU COMM 
COMM: 
* 
* 
```
#### Jumps Based on Logic Result (RLO)
These are the most common jumps. They check the current value of the RLO.
- **JC \<label\>** (Jump if Condition true): Jumps to the destination label if the RLO is 1 (TRUE).If the RLO is 0, the jump is not executed. The RLO is set to 1, and the program scan continues with the next statement.
- **JCN \<label\>** (Jump if Condition Not true): Jumps to the destination label if the RLO is 0 (FALSE).If the RLO is 1, the jump is not executed. The program scan continues with the next statement.
- **JCB \<label\>** (Jump if Condition and BR ): Jumps if the RLO is 1 and the BR bit is 1.If the RLO 0, the jump is not executed. The RLO is set to 1, and the program scan continues with the next statement. Independent of the RLO, the RLO is copied into the BR for the JCB instruction.
- **JNB \<label\>** (Jump if Not RLO and BR ): Jumps if the RLO is 0 .If the RLO is 1, the jump is not executed. The RLO is set to 1 and the program scan continues with the next statement. Independent of the RLO, the RLO is copied into the BR when there is a JNB instruction.
#### Jumps Based on the Status Word
These instructions check the status bits directly, often to handle errors or saved states.
- **JBI \<label\>** (Jump if Binary Result is 1): Jumps if the BR bit is 1.
- **JNBI \<label\>** (Jump if Not Binary Result is 1): Jumps if the BR bit is 0.
- **JO \<label\>** (Jump on Overflow): Jumps if the overflow bit (OV) is 1, typically used for math error handling.
- **JOS \<label\>** (Jump on Stored Overflow): Jumps if the stored overflow bit (OS) is `1`.
#### Jumps Based on Calculation Results
These jumps are used immediately after arithmetic operations (like addition, subtraction) to check the nature of the result in Accumulator 1.
- **JZ \<label\>** (Jump if Zero): Jumps if the result is 0.
- **JN \<label\>** (Jump if Not Zero): Jumps if the result is not 0.
- **JP \<label\>** (Jump if Positive): Jumps if the result is positive (not including zero).
- **JM \<label\>** (Jump if Minus): Jumps if the result is negative.
- **JPZ \<label\>** (Jump if Positive or Zero): Jumps if the result is positive or zero.
- **JMZ \<label\>** (Jump if Minus or Zero): Jumps if the result is negative or zero.
- **JUO \<label\>** (Jump if Unordered): Used after floating-point comparisons. Jumps if the numbers are not comparable (e.g., NaN - Not a Number).
### Examples
#### Example 1
We want to control our system such that the Heater (Q0.0) is turned ON only if:
- The **Safety Interlock** (I0.0) is active, **AND**
- The **Temperature** (I0.1) is low.
```
A I0.0 // Check if "Safety Interlock" is ON 
A I0.1 // AND check if "Temperature" is LOW 
JCN SKIP_HEAT // Jump to the label SKIP_HEAT if RLO is 0

// This part of the code only runs if the JCN condition is NOT met 
L 255 // Load the value 255 
T QB1 // Transfer it to the heater's power byte (e.g., for analog control) 
S Q0.0 // Turn ON the "Heater" output 

SKIP_HEAT: NOP 0 // Label: The program jumps here if conditions are not met, 
```
The programme jump if the conditions arn't met and don't active the heater, if condition are met it active heater.  
The **NOP 0** is a "No Operation" instruction, a common way to place a label.
#### Example 2
We want to implement a **safety check** where an alarm (Q0.5) is triggered if the **tank level** drops **below 20**.
```
L MW10 // Load current tank level (e.g., 15) into Accumulator 1 
L 20 // Load the setpoint value (20) into Accumulator 2 
-I // Subtract integers (ACCU1 = ACCU2 - ACCU1).. 
JM ALARM // Jump if Minus (if the result is negative). 

// --- Normal Operation Code ---
// This code runs if the level is 20 or higher. 
JU END_CHK // Unconditionally jump to the end of this check. 

// --- Alarm Routine --- 
ALARM: S Q0.5 // Label: Activate the "Low Level Alarm" output.
END_CHK: NOP 0 // Label: Program continues here regardless of the outcome.
```
## Counter Instructions 
### Introduction
A counter is a programming element designed specifically to count events. Each counter uses a 16-bit word in the CPU's memory to store its current value, which can range from 0 to 999.    
Counters are essential for tasks like tracking parts on a conveyor belt, counting machine cycles, or controlling a process after a specific number of events.  
The primary actions you can perform with a counter are:
- **Set/Preset:** Load a starting value into the counter.
- **Count Up/Down:** Increment or decrement the current value.
- **Reset:** Instantly clear the counter's value to zero.
- **Load Value:** Read the counter's current value for use in comparisons or calculations.
### Setting and Resetting a Counter
These instructions prepare the counter for operation or clear its value.
#### S <C_no> (Set Counter)
This instruction **presets** the counter with a specific starting value. The value must first be loaded into Accumulator 1. S is **edge-triggered**, meaning it only executes on the rising edge of the input signal (when the RLO goes from 0 to 1).
- **Example:** Load counter C1 with a preset value of 15.    
    ```
    A   I0.0      // Check for a signal from the "Preset" button
    L   C#15      // Load the constant value 15 (as BCD) into ACCU 1
    S   C1        // On the rising edge of I0.0, set counter C1 to 15
    ```
#### R <C_no> (Reset Counter)
This instruction resets the counter's value to **0**. It executes whenever the input condition (RLO) is `1`. It is **not** edge-triggered.
- **Example:** Reset counter `C1` when input `I0.1` is active.
    ```
    A   I0.1      // Check for a signal from the "Reset" button
    R   C1        // As long as I0.1 is TRUE, reset C1 to 0
    ```
### Counting Instructions
These are the core instructions for incrementing or decrementing the counter's value. Both are **edge-triggered**.
#### CU <C_no> (Count Up)
Increments the counter's value by 1 on a rising signal edge. The counter stops counting when it reaches its maximum value of 999.
- **Example:** Use a sensor at `I0.2` to count products moving by.
    ```
    A   I0.2      // Check for a signal from the product sensor
    CU  C1        // On the rising edge of I0.2, increment C1 by 1
    ```
#### CD <C_no> (Count Down)
Decrements the counter's value by 1 on a rising signal edge. The counter stops counting when it reaches **0**.
- **Example:** Use a sensor at `I0.3` to count products being removed. 
    ```
    A   I0.3      // Check for a signal from the removal sensor
    CD  C1        // On the rising edge of I0.3, decrement C1 by 1
    ```
### Reading the Counter Value
These instructions allow us to access the counter's current value to make decisions in your program.
#### L <C_no> (Load Counter Value)
Loads the counter's current value (0-999) into Accumulator 1 as a standard **integer** (binary format).
#### LC <C_no> (Load Counter Value as BCD)
Loads the counter's current value into Accumulator 1 in **Binary Coded Decimal (BCD)** format. This is often used when interfacing with displays or other devices that use BCD.
- **Example:** Compare the counter value to a target and activate an output.
    ```
    L   C1        // Load the current value of C1 into ACCU 1
    L   0         // Load the value 0 into ACCU 2
    ==I           // Compare the two integers
    =   Q4.0      // If they are equal (C1 has reached 0), turn on Q4.0
    ```
### FR <C_no> (Enable Counter / Free)
We know that The CU, CD, and S instructions are edge-triggered; they look for a 0 to 1 transition in the RLO (raising edge) to work, The `FR` instruction manually clears the internal flag that tracks this edge detection allowing it to see a new rising edge, even if the signal didn't change as expected.
- **Example:** Manually re-enable edge detection for counter `C3` with input `I2.0`.
    ```
    A   I2.0      // Check for the "re-enabling" signal
    FR  C3        // On a rising edge of I2.0, clear the internal
                  // edge-detection logic for counter C3.
    ```
### Example
This PLC (Programmable Logic Controller) program counts 10 boxes as they pass a sensor on a conveyor belt.
- Once **10 boxes** are counted, a **"Box Full" indicator light** turns ON.
- A **Reset button** clears the count and turns OFF the light so the system can start again.
#### Inputs and Outputs

| Symbol | Description                |
| ------ | -------------------------- |
| I0.0   | "Start Cycle" button       |
| I0.1   | Box detection sensor       |
| I0.2   | Reset button               |
| Q4.0   | "Box Full" indicator light |
| C10    | Counter register           |

#### Network 1: Initialize the Counter with the "Start Cycle" Button
```
A I0.0     // "Start Cycle" button pressed 
L C#10     // Load the constant value 10 
S C10      // Set the counter C10 with initial value 10
```
- When the operator presses the **Start Cycle button**, the system:
    - Loads the number **10** (this is how many boxes to count).
    - **Initializes counter C10** with this value.
    - Now the counter is ready to start counting **down** with each box.
#### Network 2: Count Each Box Detected
```
A I0.1     // "Box Sensor" is triggered 
CD C10     // Countdown the counter by 1
```
- Each time a **box passes the sensor**, input I0.1 becomes **true**.
- The instruction CD C10 (Count Down) decreases the counter value **by 1**.
- This continues until the counter reaches **0**, meaning 10 boxes have passed.
#### Network 3: Turn ON the "Box Full" Light
```
A C10      // Checks if counter value is NOT zero 
NOT        // Invert the result; now true only when counter is zero 
= Q4.0     // Set output light Q4.0
```
- When **10 boxes have passed**, the counter reaches **zero**.
- A C10 reads the **status bit** of counter C10:
    - This is **true** when the counter **is not zero**.
- NOT inverts it → becomes **true** only when **C10 = 0**.
- = Q4.0 turns ON the **"Box Full" light**.
#### Network 4: Reset the System
```
A I0.2     // "Reset" button is pressed 
R C10      // Reset counter to 0 
R Q4.0     // Turn off "Box Full" light
```
- When the operator presses the **Reset button**:
    - The counter C10 is **cleared to zero**.
    - The **"Box Full" light** is also turned OFF.
    - Now the system is ready for the next batch of 10 boxes after restarting with I0.0.
## Timer Instructions 

### Introductiom
 **Timer** is a fundamental programming element that introduces the element of time into our logic. We can think of it as a versatile digital stopwatch that we can start, stop, and read within our program. Timers are stored in a reserved memory area of the CPU, with each timer occupying a 16-bit word.  
Timers are essential for any process that isn't instantaneous, such as:
- Running a motor for a specific duration.
- Flashing a warning light.
- Delaying an action until a process has settled.
- Keeping a fan on for a few minutes after a machine has been shut down.
### Time: Value and Base
A timer's duration is defined by two components stored in its 16-bit memory word:
1. **Time Value:** A number from 0 to 999 that acts as the quantity of time.
2. **Time Base:** The unit or "resolution" of the time value. This determines how fast the time value is counted down. The available time bases are:
    - 10 milliseconds (0.01s)
    - 100 milliseconds (0.1s)
    - 1 second
    - 10 seconds

To make programming easier, we load a preset time using a specific format that combines these two elements automatically:

- **S5T#aH_bM_cS_dMS**
    - Where a, b, c, d are the hours, minutes, seconds, and milliseconds.
    - **Example:** S5T#1M_30S for 1 minute and 30 seconds.
    - The system automatically selects the best time base to represent this value. The maximum time we can enter is 2 hours, 46 minutes, and 30 seconds.
- **W#16#txyz** 
	- Where t = the time base (that is, the time interval or resolution) 
	- Where xyz = the time value in binary coded decimal format
### Choosing the Right Timer
The most important step is selecting the timer that matches the behavior we need. Each timer type is started by a corresponding instruction (`SD`, `SP`, etc.).
#### SD (On-Delay Timer)
The On-Delay timer waits for a set time after the input turns ON before its output turns ON. If the input turns off before the time has elapsed, the timer resets.
- **Use Case:** Sounding a warning siren for 5 seconds before a large machine starts.
- **Behavior:** Input ON -> Wait for preset time -> Output ON.
#### SP (Pulse Timer)
The Pulse timer turns its output ON immediately with the input. It stays ON for the preset time but will turn OFF early if the input signal disappears.
- **Use Case:** Activating a glue dispenser for a maximum of 2 seconds while a part is present.
- **Behavior:** Input ON -> Output ON -> Output stays ON for the preset time or until the input goes OFF.
#### SE (Extended Pulse Timer)
The Extended Pulse timer turns its output ON immediately with the input and stays ON for the _full_ preset time, regardless of whether the input signal is cut short. A new rising edge on the input will restart the timer.
- **Use Case:** Firing a camera flash for a fixed duration of 50ms, even if triggered by a very brief button press.
- **Behavior:** Input ON -> Output ON -> Output stays ON for the full preset time.
#### SF (Off-Delay Timer)
The Off-Delay timer's output turns ON with the input. When the input turns OFF, the timer starts, keeping the output ON for the preset time before it finally turns OFF.
- **Use Case:** Keeping a ventilation fan running for 3 minutes after a room light is switched off.
- **Behavior:** Input OFF -> Output stays ON -> Wait for preset time -> Output OFF.
#### SS (Retentive On-Delay Timer)
This is a "memory" timer. It behaves like an On-Delay timer but **retains its elapsed time** if the input signal goes OFF. When the input comes back ON, it continues timing from where it left off. It requires a separate R (Reset) instruction to be cleared.
- **Use Case:** Tracking the total cumulative runtime of a pump that is frequently started and stopped.
- **Behavior:** Accumulates time while the input is ON; requires a manual reset.
### Basic Timer Instructions
These instructions are used to manage and monitor your timers.
#### R <T_no> (Reset Timer)
Stops the addressed timer immediately and clears its time value to zero. This is essential for resetting retentive timers (SS) or manually stopping any other timer.
#### L <T_no> (Load Timer Value)
Loads the _current_ time value of the timer into Accumulator 1 as a standard **integer**. This is useful for calculations or displaying the remaining time in a non-BCD format.
#### LC <T_no> (Load Timer Value as BCD)
Loads the current time value _and_ its time base into Accumulator 1 in **Binary Coded Decimal (BCD)** format. This is the standard way to read a timer value if you need to see both the value and its resolution.
### FR <T_no> (Enable Timer / Free)
This is a utility instruction for special cases. It manually **re-triggers a running timer** if its start condition is still active (RLO = 1). Essentially, it "frees" the timer to start its time interval over from the beginning, even if it hasn't finished its cycle. This is not needed for most standard timer applications.
### Example
When a "Start" button (I0.0) is pressed
- A lubricating pump (Q4.0) runs for **5 seconds**
- Then the main motor (Q4.1) starts
- Emergency Stop (I0.1) resets everything
```
// Network 1: Start the timer when "Start" is pressed
A   I0.0       // Start button pressed
L   S5T#5S     // Load 5-second preset
SD  T1         // Start On-Delay timer T1

// Network 2: Turn on lubrication pump immediately
A   I0.0
=   Q4.0       // Activate lubrication pump

// Network 3: Turn on main motor after 5 seconds
A   T1         // Timer output bit is TRUE after delay
=   Q4.1       // Activate main motor

// Network 4: Emergency stop resets everything
A   I0.1       // Emergency Stop pressed
R   T1         // Reset timer
R   Q4.0       // Stop lubrication pump
R   Q4.1       // Stop main motor
```

