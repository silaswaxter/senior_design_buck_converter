# Design Decisions

1. Basing the circuit off of "Figure 10: Typical Application Schematic" (see
   page 14 of TP54... datasheet).

   - Compare/Contrast requirements:
     - Vin:
       - mine: 8-15V
       - theirs: 5-15V
     - Vout
       - mine: 5V
       - theirs: 2.5V
     - Vout-ripple:
       - mine: 300 mV
       - theirs: 30 mV
     - Inominal
       - mine: 1.5A
       - theirs: ~2A

1. Since Vin > Vout + 2, external VIN UVLO hystersis circuit is not required.
   (ie resistors 1 and 2 on reference schematic can be eliminated); see page 11
   of TP54... datasheet).

1. An output rise-time configuring capacitor must be chosen using the equations
   (list on p. 11/12). From what I understand, a lower output rise-time can
   reduce the noise on the output and should probably be greater than the
   converter's switching period (see this
   [blog](https://www.analog.com/en/analog-dialogue/articles/preventing-start-up-issues-due-to-output-inrush-in-switching-converters.html)).
   The governing equations are:

   ```Math
   Tss(ms) = (Css(nF) * Vref (V)) / Iss (uA)

   where Vref = 0.8 V, Iss = 2 uA, and 1 ms<=Tss<=10 ms.
   ```

   I select 8 ms as my target. Then

   ```Math
   8 = Css (nF) * 0.4
   20 = Css (nF)
   Css = 0.02 (uF)
   ```

   What should the tolerances on this value be? Worst case 10 ms is Tss, so

   ```Math
   10 = Css_max (nF) * 0.4
   25 = Css_max (nF)
   ```

   This aligns with the datasheet that says it should be \<= 27 nf.

   So, I can spec a 0.020 uF capacitor with upto +/-25%.

1. Specifying the Vsense voltage divider resistors, R5 and R6:

   The datasheet specifies that R5 should be fixed at 10k.

   It provides an equation for R6:

   ```Math
   R6 = (R5 - Vout) / (Vout - Vref)
   ```

   In another section it says Vref = 0.8V

   Therefore:

   R5 = 10k R6 = 1.905k = 1.91k (of which there are multiple SMD parts on
   digikey)

1. Specifying the output inductor:

   Using the datasheet equations--pretty self explanatory.

   I found:

   Lmin = 7.073 uH ILpp = 0.52 A ILrms = 1.51 A ILpk = 1.76 A

   Therefore an inductor was chosen such that:

   L_min_with_tolerance > 7 uH I_L_saturation_current > 1.76 A I_L_RMS > 1.51 A

1. Specifying the output capacitor:

   $$
   C_{out} > \frac{1}{2\Pi*R_o*F_{CO_{max}}
   $$

   Since $R_{o} = \frac{V_{o}}{I_{o}} = \frac{5}{1.5} = 3.5 => 4$ and
   $F_{CO_{max}}$ is $\frac{1}{5} F_{switching}$, but clamped at 70kHz:

   $$
   C_{out} > 0.57 uF
   $$

   Also:

   $$
   V_{out_{pp}} = I_{L_{pp}} \frac{D - 0.5}{4 F_{sw} C_{out}}
   $$

   $V_{out_{pp}}$ is greatest with maximized with largest $D$, `0.9`, and the
   ESR will be a driving parameter of the capacitor as well.

   $$
   ESR_{C_{out}} < \frac{V_{O_{pp_{max}}}}{I_{L_{pp}}} - \
      \frac{D - 0.5}{4 * F_{sw} * C_{out}}
   $$

   $$
   ESR(C_{out}) < 1.154 - \frac{0.4}{280k * C_{out}}
   $$

   $$
   ESR(0.57u) < 1.154 - 2.51 \implies \text{DOESN'T WORK}
   $$

   $$
   ESR(1.5u) < 0.202 \Omega
   $$

   I found
   [this](https://www.digikey.com/en/products/detail/w%C3%BCrth-elektronik/885012207023/5453510)
   capacitor which should work. It doesn't specify the ESR, but instead
   specifies the dissipation factor (DF). Working it out on paper, its ESR is
   $75.8 \Omega$.

1. Specifying the catch diode:

   - Must Have the following:
     - Absolute Maximum Ratings:
       - reverse voltage > V_in_max + 0.5
         - reverse voltage > 15.5 V
       - peak current > I_out_max + (i_l_pkpk/2)
         - peak current > 2.5 A
     - Capable of dissipating power losses.

1. Specifying the Compensation Components:

   1. Choose closed loop crossover frequency.
      - $= \frac{1}{8} * F_{min_{operating}}$ (I assume operating frequency is
        the same as switching frequency.)
      - $F_{CO} < 75 kHz$
