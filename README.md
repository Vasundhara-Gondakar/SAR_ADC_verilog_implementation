# SAR_ADC_verilog_implementation
The main focus is to understand the two row SAR logic.

###  The Concept: "Guess My Number"
A SAR ADC uses a **Binary Search** algorithm to find an unknown analog voltage. It's exactly like playing "Guess My Number" between 0 and 255:
1. You start by guessing the exact middle: **128**.
2. If the comparator says the real voltage is "Higher", you guess the middle of the upper half: **192**.
3. If it says "Lower", you guess the middle of the lower half: **64**.

The hardware takes exactly n clock cycles  (as it consists of only 1 comparator) to play this game and find the n-bit digital answer. Hence, no. of clock cycles required = no. of bits.

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
  ### **Comparator:**
  Compares the Voltage received from DAC and the input voltage and feeds to the control logic block.

---

##  NOTE: 
The code fakes the analog world using digital math:

1. **The "Analog" Target:** Instead of a real voltage, a 16-bit binary integer (for example, the number `200`) is taken as input to the ADC.
2. **The "DAC" (The Padding Trick):** The simulated DAC just uses a Verilog concatenation trick: `assign dac_voltage = {8'b0, data_reg};`. This simply glues eight `0`s to the front of the 8 bit guess (like `128`), turning it into a 16-bit number without changing its actual decimal value.
3. **The Comparator:** Because the target input and the padded DAC guess are now both exactly 16 bits, it just does a basic math check: `if (held_voltage >= dac_voltage)` (e.g., *Is 200 >= 128?*). 

