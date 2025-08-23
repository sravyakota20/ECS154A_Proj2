# Project 2 README

* Student 1: Varshini Gurushankar (922249127)
* Student 2: Sravya Kota (919987625)

## Project Status
* Aug 21st 8:00 am - partially finished panel have to still workout on output V
* Aug 21st 9:40 am - Finished Panel
* Aug 21st 1:20 pm - Finished Input Buffer
* Aug 21st 10:40 pm - Completed Part 3 halfway through
* Aug 22nd 9:50 am - Made some more progress on part 3 but still not done 

## Known Issues
The project fails to function properly in ...

## References
* [Build an XOR gate from AND/NOT](https://electronics.stackexchange.com/questions/487529/build-an-xor-gate-from-and-not)
* [Some other link](https://stackoverflow.com)

## Generative AI Use
I used ChatGPT for Prompt 1, and ZZZZZ for Prompt 2....

### Prompt 1
How do you create an XOR from only and/or/not gates?

### Response 1
To create an XOR (exclusive OR) gate using only AND, OR, and NOT gates, you can follow these steps. The XOR gate outputs true only when the inputs are different.

The truth table for XOR is:

| A | B | A XOR B |
|---|---|---------|
| 0 | 0 |    0    |
| 0 | 1 |    1    |
| 1 | 0 |    1    |
| 1 | 1 |    0    |

The Boolean expression for XOR can be derived as:\
A &oplus; B = (A &and; &not; B) &or; (&not; A &and; B)

This can be broken down into the following steps:

1. Create \(&not; A\) and \(&not; B\) using NOT gates.
1. Create \(A &and; &not; B\) and \(&not; A &and; B\) using AND gates.
1. Combine the results of the two AND gates using an OR gate.

Here is the step-by-step logic construction:

1. **NOT gates** to get \(&not; A\) and \(&not; B\):\
   NOT(A) &rarr; &not; A\
   NOT(B) &rarr; &not; B

1. **AND gates** to get \(A &and; &not; B\) and \(&not; A &and; B\):\
   A &and; &not; B &rarr; AND(A, &not; B)\
   &not; A &and; B &rarr; AND(&not; A, B)

1. **OR gate** to get the final result:\
   (A &and; &not; B) &or; (&not; A &and; B) &rarr; OR(AND(A, &not; B), AND(&not; A, B))

Putting it all together in a circuit diagram:

1. Input \(A\) goes to a NOT gate to produce \(&not; A\).
1. Input \(B\) goes to a NOT gate to produce \(&not; B\).
1. Input \(A\) and the output of the NOT gate for \(B\) go to an AND gate to produce \(A &and; &not; B\).
1. Input \(B\) and the output of the NOT gate for \(A\) go to an AND gate to produce \(&not; A &and; B\).
1. The outputs of the two AND gates go to an OR gate to produce the final output \(A &oplus; B\).

In terms of Boolean logic:\
XOR(A, B) = \(A &and; &not; B\) &or; \(&not; A &and; B\) 

This way, you have constructed an XOR gate using only AND, OR, and NOT gates.

### Changes 1
I needed XNOR, so changed it to:\
&not;\(\(A &and; &not; B\) &or; \(&not; A &and; B\)\)

### Our Changes 

### Prompt 1

How can I implement a 2-bit “+1” counter without an adder, and run it only while a control signal is 1?

### Response 1

A 2-bit binary up-counter has next-state equations:

* $q_0^+ = \lnot q_0$
* $q_1^+ = q_1 \oplus q_0$

To enable counting only when **EN=1**, drive the D inputs through 2:1 MUXes:

* $D_0 = \mathrm{EN} ? (\lnot q_0) : 0$
* $D_1 = \mathrm{EN} ? (q_1 \oplus q_0) : 0$

Thus, when EN=1 it sequences 00→01→10→11; when EN=0, next state is forced to 00. A “done” pulse for “four ticks elapsed” is simply $q_1 \land q_0$.



### Changes 1

Set **EN = isLocking ∨ isUnlocking**. Named the register **K\[1:0]**, D wiring exactly as above, and defined **kDone = K1 ∧ K0** to exit the 4-cycle spin.

---

### Prompt 2

What’s a clean way to select between two 2-bit “next-state” sources for one register?

### Response 2

Use a single **2:1 MUX with Data Bits=2**. Bundle the two candidate 2-bit values as buses on **In0** and **In1**, and drive **Select** with the condition choosing which source is active. The MUX output goes to the register’s D\[1:0]. Keep MSB on the upper wire and LSB on the lower wire consistently.

### Changes 2

Set **Select = isUnlocked**; **In1 = PHU\[1:0]** (Unlocked branch), **In0 = PHL\[1:0]** (Locked branch). I inserted a second 2-bit MUX in front with **Select = enterSpin** to force **PH=00** the cycle a spin begins.

---

### Prompt 3

How do I encode prioritized state transitions with only basic gates/MUXes (no big PLA)?

### Response 3

Chain small 2:1 MUXes per state bit from **lowest** to **highest** priority, using:

* **In0** = “previous” result (start with current bit $q$)
* **In1** = target bit for this rule
* **Select** = rule condition

After cascading all rules, the final MUX output is the next-state bit $D$. Repeat for each bit in the state. This implements “first true wins” priority cleanly without large decoders.


### Changes 3

Applied to **Top\[1:0]** with priority:

1. `isLocking & kDone` → **10**
2. `isUnlocking & kDone` → **00**
3. `enterLocking` → **01**
4. `enterUnlocking` → **11**
   else hold.
   I built two MUX chains (one per bit) in that order.

---

### Prompt 4

Give me binary test vectors at the pins (C3..C0 → L/U) for a lock that: locks on double-L or L+3 digits; unlocks on U+correct 3 digits; spins last 4 clocks; inputs may be ignored during spins.

### Response 4

Legend: `1010=L`, `1011=U`, `1111=idle`, digits `0000..1001`.

**Idle:**
C: `1111,1111,1111,1111` → L=0,U=0 (steady)

**Lock via double-L (from Unlocked):**
C: `1111,1010,1111,1010,1111,1111,1111,1111,1111`
Expect L: `0,0,0,0,1,1,1,1,0`; U: all 0

**Program via L+3 digits (e.g., 3,5,7):**
C: `1111,1010,1111,0011,1111,0101,1111,0111,1111,1111,1111,1111,1111`
Expect L high for exactly 4 clocks after the third digit.

**Unlock (default combo 0,0,2):**
C: `1111,1011,1111,0000,1111,0000,1111,0010,1111,1111,1111,1111,1111`
Expect U: `0,0,0,0,0,0,0,0,1,1,1,1,0`; L: all 0

**Mismatch at digit 1:**
C: `1111,1011,1111,1001` (if d1≠1001) → no spin; stays Locked

**Cancel with L during verify:**
C: `1111,1011,1111,d1,1111,1010` → cancel; no spin

**Restart with U during verify:**
C: `1111,1011,1111,d1,1111,1011,1111,d1,1111,d2,1111,d3,1111` → U spins 4 clocks at end


### Changes 4

I plugged in my actual combo: initially `0000,0000,0010` (0,0,2). After I ran the L+3-digit case with `0011,0101,0111`, the stored combo became 3,5,7; I re-ran the unlock vectors with those binaries.



