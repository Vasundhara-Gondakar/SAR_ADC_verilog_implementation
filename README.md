**SAR_ADC_verilog_implementation**

The Concept: "Guess My Number"
A SAR ADC uses a **Binary Search** algorithm to find an unknown analog voltage. It's exactly like playing "Guess My Number" between 0 and 255:
1. You start by guessing the exact middle: 128.
2. If the comparator says the real voltage is "Higher", you guess the middle of the upper half:192.
3. If it says "Lower", you guess the middle of the lower half: 64.

This hardware takes exactly 8 clock cycles as it consists of only 1 comparator to play this game and find the 8 bit digital answer. Hence, no. of clock cycles required = no. of bits.

**1. Architecture Overview**

 **a. Sample and Hold (S&H)** : 
Captures and freezes the input voltage so it doesn't change mid conversion.

**2. SAR control logic and two row register architecture**:
It consists of two main parts:
**Row 1: The Sequencer (Ring Counter)**
The first register, referred to as the sequencer, functions as a moving pointer that determines which bit is currently being evaluated. It is initialized with its most significant bit set and shifts right with each clock cycle, effectively stepping through each bit position from MSB to LSB. 
For example : Considering, 8 bit register is considered where a single `1` shifts from left to right on every clock cycle. It tells the next row the exact bit that is being currently guessed. 
**Row 2: The Data Register (The Guesser)**
The Data Register is the part of the circuit that makes and corrects the digital guesses for the SAR ADC. 
   **1: Making the guess (setting the bit)**
      It sets a bit to 1 based on where the Sequencer points. This is its 'guess' for that bit position.
   2: Checking the Result (Comparator Feedback)
      On the next clock cycle, it checks the Comparator to see if its guess was too high. 
      If the guess was too high: The Comparator says "low" (Input voltage is less than the guessed Voltage), so the Data Register clears that bit back to 0. 
      If the guess was less than or equal to the target: the Comparator says "high"(Input voltage is greater than the guessed Voltage), so the Data Register keeps that bit as 1. 
      In the same clock cycle, it sets the next bit to 1, repeating the process. It doesn't wait.
The logic can be visualized as given in the table below:
| Step (Assumption) | $B_3$ (MSB) | $B_2$ | $B_1$ | $B_0$ (LSB) | Comparator Decision |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **1** | 1 | 0 | 0 | 0 | Determines $A_3$ |
| **2** | $A_3$ | 1 | 0 | 0 | Determines $A_2$ |
| **3** | $A_3$ | $A_2$ | 1 | 0 | Determines $A_1$ |
| **4** | $A_3$ | $A_2$ | $A_1$ | 1 | Determines $A_0$ |
| **Final Result**| $A_3$ | $A_2$ | $A_1$ | $A_0$ | Conversion Complete |


