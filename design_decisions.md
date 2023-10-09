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

1.
