# SAR_ADC_verilog_implementation

###  The Concept: "Guess My Number"
A SAR ADC uses a **Binary Search** algorithm to find an unknown analog voltage. It's exactly like playing "Guess My Number" between 0 and 255:
1. You start by guessing the exact middle: **128**.
2. If the comparator says the real voltage is "Higher", you guess the middle of the upper half: **192**.
3. If it says "Lower", you guess the middle of the lower half: **64**.

This hardware takes exactly 8 clock cycles as it consists of only 1 comparator to play this game and find the 8-bit digital answer. Hence, no. of clock cycles required = no. of bits.

---

###  Architecture Overview

### **a. Sample and Hold (S&H):**
Captures and freezes the input voltage so it doesn't change mid-conversion.

###  b. SAR Control Logic & Two-Row Register Architecture
It consists of two main parts:

#### **Row 1: The Sequencer (Ring Counter)**
The first register, referred to as the sequencer, functions as a moving pointer that determines which bit is currently being evaluated. It is initialized with its most significant bit set and shifts right with each clock cycle, effectively stepping through each bit position from MSB to LSB.  

*For example:* Considering an 8-bit register, a single `1` shifts from left to right on every clock cycle. It tells the next row the exact bit that is being currently guessed. 

#### **Row 2: The Data Register (The Guesser)**
The Data Register is the part of the circuit that makes and corrects the digital guesses for the SAR ADC.  
  
* **Step 1: Making the guess (setting the bit)** It sets a bit to 1 based on where the Sequencer points. This is its 'guess' for that bit position.
  
* **Step 2: Checking the Result (Comparator Feedback)** On the next clock cycle, it checks the Comparator to see if its guess was too high.  
  * **If the guess was too high:** The Comparator says "low" (Input voltage is less than the guessed Voltage), so the Data Register clears that bit back to `0`.  
  * **If the guess was less than or equal to the target:** The Comparator says "high" (Input voltage is greater than the guessed Voltage), so the Data Register keeps that bit as `1`.  
  
  In the same clock cycle, it sets the next bit to `1`, repeating the process. It doesn't wait.

---

###  The Logic Trace

The logic can be visualized as given in the table below:

| Step (Assumption) | B<sub>3</sub> (MSB) | B<sub>2</sub> | B<sub>1</sub> | B<sub>0</sub> (LSB) | Comparator Decision |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **1** | 1 | 0 | 0 | 0 | Determines A<sub>3</sub> |
| **2** | A<sub>3</sub> | 1 | 0 | 0 | Determines A<sub>2</sub> |
| **3** | A<sub>3</sub> | A<sub>2</sub> | 1 | 0 | Determines A<sub>1</sub> |
| **4** | A<sub>3</sub> | A<sub>2</sub> | A<sub>1</sub> | 1 | Determines A<sub>0</sub> |
| **Final Result**| A<sub>3</sub> | A<sub>2</sub> | A<sub>1</sub> | A<sub>0</sub> | Conversion Complete |

  ### **DAC:** 
  Converts the digital guess back to an analog scale.
  Once the input is captured, the SAR logic begins generating trial values that are passed through the DAC. In real hardware, the DAC converts a digital code into a corresponding analog voltage that can be compared against the input signal. 
