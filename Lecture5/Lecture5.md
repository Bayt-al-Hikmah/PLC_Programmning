## Objectives
- Data types 
- Analog Input and Output
- Comparison Operators and Arithmetic Operators
- Translating from SFC to Ladder
## Data Types
### BOOL
The `bool` data type can have only two possible values: `1` (true) or `0` (false). It takes 1 byte of memory space. This is the most commonly used data type in ladder diagrams, primarily for input instructions and output coils.
### BYTE
The second data type is `byte`. It is used to store up to 8 bits, allowing values from 0 to 255. It is useful for representing small numbers and is commonly used when working with counters.
### WORD
The `word` data type occupies 16 bits (2 bytes) of memory space. It can store integer values ranging from 0 to 65,535 (in unsigned form) or from -32,768 to 32,767 (in signed form). This data type is commonly used for  analog values, and situations where larger numeric ranges are required compared to `byte`.
### DWORD
The `DWORD` (Double Word) data type occupies 32 bits (4 bytes) of memory space. It can store integer values ranging from 0 to 4,294,967,295 (unsigned) or from -2,147,483,648 to 2,147,483,647 (signed). It is typically used when dealing with large counters, long timer values, or data that requires a wide numeric range.
### INT
The `INT` (Integer) data type occupies 16 bits (2 bytes) of memory. It can store signed integer values ranging from -32,768 to 32,767. It is commonly used for arithmetic operations, counting, and representing values that can go negative or positive within that range.
### DINT
The `DINT` (Double Integer) data type uses 32 bits (4 bytes) of memory. It can store signed integers ranging from -2,147,483,648 to 2,147,483,647. It is useful for calculations that require large numbers or when dealing with high-resolution data in industrial automation.
### REAL
The `REAL` data type is a 32-bit (4-byte) floating-point type based on the IEEE 754 standard. It is used to represent decimal or fractional values, such as **3.14** or **-0.75**. This type is essential when precision is required in calculations, like in measurements, scaling, or analog value processing.
### TIMER
The `TIMER` data type is a special structured type used to represent and control timers in ladder logic programming. It typically includes elements such as:
- **.EN** (Enable): Indicates if the timer is currently active.
- **.TT** (Timer Timing): True while the timer is counting.
- **.DN** (Done): Becomes true when the timer completes.
- **.PRE** (Preset): The target time to count to.
- **.ACC** (Accumulated): The current elapsed time.
### Addressing Data in PLCs
PLCs use a structured system to access data stored in their memory. The way data is addressed depends on its size (bit, byte, word, double word) and type (input, output, memory). The specific syntax can vary between PLC manufacturers (e.g., Siemens, Allen-Bradley, Mitsubishi), but the concepts are similar. Here, we'll use a common notation (similar to Siemens) for illustration:
#### Bit Addressing
This is the smallest addressable unit, representing a single `BOOL` value,Bit used for discrete inputs (sensors, buttons), discrete outputs (coils, lamps), or internal memory bits (flags, status bits).  
**Syntax Example:**
- `%I0.0`: Input bit 0 of byte 0 (e.g., first digital input).
- `%Q0.1`: Output bit 1 of byte 0 (e.g., second digital output).
- `%M1.0`: Memory bit 0 of byte 1 (an internal relay or flag)
.
The `I` denotes input, `Q` output, and `M` internal memory. The first number usually indicates the byte address, and the second (after the dot) indicates the bit position within that byte (typically 0-7).
#### Byte Addressing: 
A byte consists of 8 bits. This is used for the `BYTE` data type or to access a group of 8 bits collectively.it used for storing small integers (0-255), ASCII characters, or handling 8 digital signals as a group.  
**Syntax Example:**
- `%IB0`: Input byte 0 (includes bits `%IX0.0` to `%IX0.7`).
- `%QB1`: Output byte 1.
- `%MB2`: Memory byte 2.

#### Word Addressing:
A word consists of 16 bits (2 bytes). This is used for `WORD` and `INT` data types. it help us to store larger integers, raw analog input values (often 0-27648 or similar), timer presets/accumulated values.  
 **Syntax Example:**
- `%IW0`: Input word starting at byte 0 (includes `%IB0` and `%IB1`).
- `%QW2`: Output word starting at byte 2 (includes `%QB2` and `%QB3`).
- `%MW4`: Memory word starting at byte 4.

When a word is addressed (e.g., `%IW0`), it automatically refers to two consecutive bytes.
#### Double Word Addressing:
A double word consists of 32 bits (4 bytes). This is used for `DWORD`, `DINT`, and `REAL` data types, it store very large integers, floating-point numbers, high-resolution analog values, long timer/counter values.  
 **Syntax Example:**
- `%ID0`: Input double word starting at byte 0 (includes `%IB0`, `%IB1`, `%IB2`, and `%IB3`).
- `%QD4`: Output double word starting at byte 4.
- `%MD8`: Memory double word starting at byte 8.

 When a double word is addressed (e.g., `%ID0`), it automatically refers to four consecutive bytes.
#### Special Addressing
Data types like `TIMER` or other structured data types are often addressed by their identifier (e.g., `T1` for Timer 1), and their individual elements are accessed using a dot notation (e.g., `T1.DN`, `T1.ACC`), as described in the `TIMER` data type section.  
It's crucial to consult the specific PLC manufacturer's documentation for the exact addressing syntax and memory organization.
#### Example
Let’s suppose we want to create a simple system where a user can turn a light on and off using two buttons: one for turning the light on and another for turning it off. This system will require two input buttons and one output lamp.  
Since the inputs and outputs can be represented using bits, we can use the bit data type in our PLC program. 
- The **ON button** is connected to address `%I0.0`.
- The **OFF button** is connected to address `%I0.7`.
- The **lamp** is connected to output address `%Q1.1`.

<img src="./attachments/address_wiring.png" height="300px" >  
The ladder logic will be structured as follows:
<img src="./attachments/address_ladder.png" height="200px" >  
## Analog Input and Output
### Introduction
So far, we have worked with the `bool` data type, where all inputs and outputs were Boolean values either `0` (false) or `1` (true). These signals are typically generated by digital input devices such as push buttons and presence sensors.  
However, in industrial environments, some input devices and sensors provide analog values, such as temperature, distance, or motor rotation speed. To handle these types of signals, we need to use analog input and output instructions, which allow us to read and control continuous (non-binary) values.
### Analog Signal
In a PLC system, analog input modules typically receive signals in the range of 0–10V or 4–20 mA. These are standard signal formats used by most analog sensors to transmit physical values such as temperature, pressure, distance, or flow rate. 
However, these signals are just raw electrical values. To use them meaningfully in our program, we need to convert them into real-world engineering values. In TIA Portal, this process involves two key functions: `NORM_X` and `SCALE_X`.
#### Normalization `NORM_X`:
The analog input module converts the voltage or current signal into a digital value. For example, a 4–20 mA signal is typically mapped to a digital range of **0 to 27648** in Siemens PLCs.  
The `NORM_X` function is used to normalize the raw input value, it converts the raw signal into a value between 0.0 and 1.0.  
We need to provide:        
- `IN`: the address of the analog input (e.g., `%IW64`)       
- `HI`: the high limit of the input range (e.g., 27648)        
- `LO`: the low limit of the input range (e.g., 0)

<img src="./attachments/img1.png" height="200px" >  

#### Scaling `SCALE_X`:
The `SCALE_X` function scales the normalized value to the desired engineering units.  
We need to provide:
    - `VALUE`: the output of `NORM_X`
    - `MIN`: minimum value in engineering units (e.g., 0 PSI)
    - `MAX`: maximum value in engineering units (e.g., 100 PSI)

The result is a scaled value that represents the actual measurement.  
<img src="./attachments/img2.png" height="200px" >  
#### Example
Suppose we have a 4–20 mA pressure sensor, and we want to display the pressure in PSI.
1. Raw Input Range:
    - 4–20 mA corresponds to 0–27648 in the PLC’s digital input.    
2. Use `NORM_X`:  

<img src="./attachments/img3.png" height="200px" >  

3. **Use `SCALE_X`:**

<img src="./attachments/img4.png" height="200px" >  


Now `Scaled_Pressure` holds the actual pressure value in PSI, which can be used directly in our ladder logic for comparison, control, or display on an HMI.
### Analog Output
We can set an analog output in a similar way to how we handle analog input. First, we need to configure and assign the analog output module. Then, we use the `NORM_X` and `SCALE_X` functions to process and output the signal.  
For example, suppose we want to control the rotation speed of a motor, where the speed range is from 0 to 1000 RPM. To achieve this:
1. Get the desired speed from the user and store it in a variable.
2. Use the `NORM_X` function to normalize this value between 0 and 1. Set the LO/MIN input to `0` and the HI/MAX input to `1000`.
3. Pass the normalized value to `SCALE_X`, where we set the minimum to `0` and the maximum to `27648`. This scales the value to match the analog output range.
4. Send this output to the analog module, which will convert the signal to a voltage or current typically 0–10 V or 4–20 mA  depending on our hardware configuration.
5. Finally, we can use this analog signal in our electrical circuit to control the motor’s rotation speed.

This strategy is important because it allows us to go beyond simple on/off control. We can receive analog input from sensors (e.g., temperature, pressure, position) and use it to dynamically adjust analog outputs — enabling closed-loop control of more complex systems.
## Comparison Operators and Arithmetic Operators
### Introduction
Ladder logic provides us with comparison and arithmetic operators, allowing us to perform more advanced operations with analog inputs after converting them into real-number values.  
Using comparison operators, we can evaluate analog values against thresholds or other variables, and based on the result, control the state of outputs.  
With arithmetic operators, we can perform mathematical operations such as addition, subtraction, multiplication, and division on these values. This enables us to build custom mathematical functions and add more sophisticated control logic to our system.
### Comparison Operator
#### Equal `(==)` Operator
This operator allows us to compare two values to check if they are equal.
- If the two values are equal, the output is 1 (true).
- If they are not equal, the output is 0 (false).

<img src="./attachments/img5.png" height="200px">



#### Not Equal (!=) Operator
This operator checks if two values are not equal.
- If the values are different, the output is 1 (true).
- If they are equal, the output is 0 (false).

<img src="./attachments/img6.png" height="200px">

#### Less Than (<) Operator
This operator checks if the first value is less than the second.
- If the first value is smaller, the output is 1 (true).
- Otherwise, the output is 0 (false).

<img src="./attachments/img8.png" height="200px">  

#### Greater Than (>) Operator
This operator checks if the first value is greater than the second.
- If the first value is larger, the output is 1 (true).
- Otherwise, the output is 0 (false).

<img src="./attachments/img7.png" height="200px">  

#### Less Than or Equal (<=) Operator
This operator checks if the first value is less than or equal to the second
- If it is, the output is 1 (true).
- If the first value is greater, the output is 0 (false).

<img src="./attachments/img9.png" height="200px">  

#### Greater Than or Equal (>=) Operator
This operator checks if the first value is greater than or equal to the second.
- If it is, the output is 1 (true).
- If the first value is smaller, the output is 0 (false).

<img src="./attachments/img10.png" height="200px"> 

#### In-Range Operator
This operator checks if a value is within a specific range, inclusive.
- If the value is between or equal to the minimum and maximum limits, the output is 1 (true).
- If it falls outside the range, the output is 0 (false).

<img src="./attachments/img11.png" height="200px">  

### Arithmetic Operators
#### Addition (+) Operator
This operator adds two numerical values together.   
We assign the addresses of the operands (the values to be added), and then specify the address where the result (sum) will be stored.  
<img src="./attachments/img12.png" height="220px">  
#### Subtraction (−) Operator
This operator subtracts the second numerical value from the first.  
We assign the addresses of the operands, and then specify the address where the result (difference) will be stored.  
<img src="./attachments/img13.png" height="220px">  
### Multiplication (×) Operator
This operator multiplies two numerical values.  
We assign the addresses of the operands, and then specify the address where the result (product) will be stored.
<img src="./attachments/img14.png" height="220px">  
#### Division (÷) Operator
This operator divides the first numerical value by the second.  
We assign the addresses of the operands, and then specify the address where the result (quotient) will be stored.  
<img src="./attachments/img15.png" height="220px">  
#### Modulo (MOD) Operator
This operator returns the remainder after dividing the first value by the second.  
We assign the addresses of the operands, and then specify the address where the result (remainder) will be stored.  
<img src="./attachments/img16.png" height="220px">  

## Translating from SFC to Ladder
### Introduction
We have created programs using Ladder Logic to control our systems and learned how to build them step by step.  
But what if the system is represented using an SFC (Sequential Function Chart) diagram? How can we convert that into a Ladder Logic program?  
We can do that Using the set and reset Technique
### Set and Reset Technique
To convert an SFC diagram into Ladder Logic, the first step is to derive the logic equations that represent each transition condition.  
Next, we create the equations that will activate and deactivate each step in the sequence.  
Finally, we link each step to its corresponding actions to complete the control logic.  
In Ladder Logic:
- Activating a step is typically represented using a Set coil (SET).
- Deactivating a step is represented using a Reset coil (RESET).
### Examples
#### Example 1
Let’s suppose we have the following SFC diagram:  
<img src="./attachments/example1.png" height="280px">  

To convert it into Ladder Logic, we follow these steps:  
We begin by identifying the transitions between steps:
- `Tr1 = S1`
- `Tr2 = S2`
- `Tr3 = S3`

Then we define step Activation and deactivation equations, a step is activated if the previous step is active and the preceding transition is valid.  
A step is deactivated if the next step becomes active.  
Using this logic, we can write the following equations:
- `SET(X2) = X1 · Tr1 = X1 · S1`
- `RESET(X1) = X1 · Tr1 = X1 · S1`
- `SET(X3) = X2 · Tr2 = X2 · S2`
- `RESET(X2) = X2 · Tr2 = X2 · S2`
- `SET(X1) = X3 · Tr3 = X3 · S3`
- `RESET(X3) = X3 · Tr3 = X3 · S3`

Now we associate actions (like turning on outputs) with the corresponding steps:
- `KM1 = X2`
- `KM2 = X3`

This means:
- When step `X2` is active, output `KM1` is ON.
- When step `X3` is active, output `KM2` is ON.

**Ladder Logic:**  
<img src="./attachments/solution1.png">  
#### Example 2
Let’s suppose we have the following SFC diagram:  
<img src="./attachments/example2.png" height="280px">   

To convert it into Ladder Logic, we follow these steps:  
We begin by identifying the transitions between steps:
- `Tr1 = S1`
- `Tr2 = S2`
- `Tr3 = S3`
- `Tr4 = S4`
- `Tr5 = S5`

Then we define step Activation and deactivation equations, a step is activated if the previous step is active and the preceding transition is valid.  
A step is deactivated if the next step becomes active.  
Using this logic, we can write the following equations:
- `SET(X2) = X1 · Tr1 = X1 · S1`
- `RESET(X1) = X1 · Tr1 = X1 · S1`
- `SET(X3) = X1 · Tr3 = X1 · S3`
- `RESET(X1) = X1 · Tr3 = X1 · S3`
- `SET(X4) = (X3 · Tr4) + (X2 . Tr2) = (X3 · S4) + (X2 . S2)`
- `RESET(X3) = X3 · Tr4 = X3 · S4`
- `RESET(X3) = X2 · Tr2 = X2 · S2`
- `SET(X1) = X4 . Tr5 = X4 . S5`
- `RESET(X4) = X4 · Tr5 = X4 · S5`

Now we associate actions (like turning on outputs) with the corresponding steps:
- `KM1 = X2`
- `KM2 = X3`
- `KM3 = X4`

This means:
- When step `X2` is active, output `KM1` is ON.
- When step `X3` is active, output `KM2` is ON.
- When step `X4` is active, output `KM3` is ON.

<img src="./attachments/solution2.png">  
