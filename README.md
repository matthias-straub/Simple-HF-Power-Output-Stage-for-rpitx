# Simple HF Power Output Stage for rpitx using the IXDN602PI

** this is only for licensed radio amateurs **

This is a very simple and very cheap HF power output stage for [`rpitx`](https://github.com/F5OEO/rpitx) on a Raspberry Pi.

The idea is slightly unconventional: instead of building a traditional RF power amplifier, an **IXDN602PI MOSFET gate driver** is used directly as the final RF output stage.

In my build it works very well at **7 MHz and 14 MHz**. With an **18 V supply** I get approximately **0.5 W RF output** in my setup. This is a significant boost over the 10mW to expect from the Pi alone.

I use this successfully with my JS8 implementation  [`js8pitx`](https://github.com/matthias-straub/js8pitx) into a dipole for 20m and can easily reach USA and Australia from Europe. 

The circuit contains only simple parts, no RF transformer, no bias network and no RF choke.

## Circuit

```text
GPIO4 ───── INA   Pin 2

              IXDN602PI
          +---------------+
NC    1 --|               |-- 8 NC
INA   2 --|               |-- 7 OUTA ---- 1 Ω ---- 100 nF ----> LPF
GND   3 --|               |-- 6 VCC
INB   4 --|               |-- 5 OUTB
          +---------------+

OUTA (pin 7)
     |
     1R
     |
   100 nF
     |
  50 ohm LPF
     |
  RF output
```

Only **channel A** is used.

Connect the unused **INB (pin 4) to GND** and leave OUTB unconnected.

## Power supply decoupling

Put a **100 nF capacitor directly between VCC pin 6 and GND pin 3**. Keep these connections as short as possible.

I also use additional local supply buffering:

```text
+18 V
  |
  +--- 100 nF ----- GND    <- directly at the IC
  |
  +--- 470 nF ----- GND
  |
  +--- 10...47 uF - GND    <- optional bulk capacitor
  |
 IXDN602PI
```

The **100 nF capacitor directly at the IC is the important one**.

## Parts

| Part | Value |
|---|---|
| U1 | IXDN602PI |
| R1 | 10 ohm, GPIO input series resistor |
| R2 | 1 ohm, output series resistor |
| C1 | 100 nF, VCC bypass |
| C2 | 470 nF / 0.47 uF, additional VCC bypass |
| C3 | 10–47 uF, optional bulk capacitor |
| C4 | 100 nF, output DC-blocking capacitor |
| Supply | 18 V |
| Output | 50 ohm low-pass filter |

Use a suitably voltage-rated capacitor for C4. I use parts rated comfortably above the supply voltage.

## Raspberry Pi connection

rpitx outputs RF on:

```text
GPIO4 = physical header pin 7
```

Connect the Raspberry Pi ground to the output-stage ground.

In my installation the GPIO4 and GND wires are about 12 cm long and are **twisted together**:

```text
Pi GPIO4  ))))))))))  power stage input
Pi GND    ((((((((((  power stage ground
```

The 10-ohm resistor at the GPIO input helps damp fast edges and ringing.

## Signal path

The complete RF path is very simple:

```text
Raspberry Pi
     |
   GPIO4
     |
 IXDN602PI
     |
   1 ohm
     |
  100 nF
     |
  50-ohm LPF
     |
  RF output
```

The waveform at the driver output does not have to remain a perfect square wave.

At higher frequencies it may become trapezoidal or even approach a triangular waveform. This is not necessarily a problem. The useful signal is the fundamental frequency, while the following low-pass filter removes the unwanted harmonics.

## Filtering is extremely important

**Do not use this output stage without a proper low-pass filter.**

In my experience, the rpitx fundamental itself is surprisingly clean, but GPIO-based RF generation and especially the power output stage inevitably produce harmonics.

Once the signal is amplified, those harmonics are amplified as well.

The LPF is therefore an essential part of the design:

```text
GPIO4
   |
IXDN602PI
   |
fundamental + harmonics
   |
  LPF
   |
filtered RF output
```

Use a low-pass filter designed for the frequency range or amateur band you intend to use and capable of handling the expected RF power.

### Cheap HF LPF modules

There are inexpensive Chinese HF low-pass filter boards sold on eBay and similar marketplaces.

They can work well, but I would **not trust the printed frequency specification without measuring them**.

A **NanoVNA is extremely useful** for checking these filters.

At minimum, verify:

- insertion loss in the desired band
- cutoff frequency
- attenuation at the second harmonic
- attenuation at the third harmonic
- input and output matching

In my case, some inexpensive filters required adjustment. I found that the **inductor coils needed to be shortened** to move the response into the desired frequency range.

So treat inexpensive prebuilt LPFs as a starting point rather than calibrated RF equipment.

After modifying a filter, measure it again with the NanoVNA.

## Shielding is strongly recommended

I put the complete RF output stage and filtering inside a **grounded metal enclosure**.

This is highly recommended.

```text
        grounded metal enclosure
+----------------------------------+
|                                  |
| GPIO input -> IXDN602 -> LPF     |
|                                  |
+----------------------------------+
                 |
                GND
```

Connect the enclosure to circuit ground using a **short, low-inductance connection**.

The enclosure helps reduce direct radiation from:

- the IXDN602 output
- PCB wiring
- fast switching edges
- harmonics before the LPF
- supply and ground wiring

It also helps prevent RF from coupling back into the Raspberry Pi or nearby electronics.

The LPF should ideally be physically close to the output stage so that the unfiltered signal does not travel through long wires before reaching the filter.

## Results

My current setup:

```text
Frequencies tested:  7 MHz and 14 MHz
Driver supply:       18 V
Driver channels:     one
Output resistor:     1 ohm
RF output:           approximately 0.5 W in my setup
```

The exact output power depends on:

- supply voltage
- frequency
- filter losses
- load impedance
- physical layout
- measurement method

An interesting observation from testing was that **using both IXDN602 channels in parallel was less efficient in my application**.

At the same supply voltage, the IC drew substantially more current and became hotter, while the improvement into an approximately 50-ohm load was small.

For this application, a **single channel works better**.

## Layout matters

For best results:

- keep OUTA -> resistor -> coupling capacitor -> LPF short
- place the 100 nF VCC capacitor directly between pins 6 and 3
- keep ground paths short
- twist GPIO4 with a ground wire if the connection to the Pi is relatively long
- avoid long breadboard jumpers at the RF output
- place the LPF close to the power stage
- put the complete RF section inside a grounded metal enclosure

## Important notes

The IXDN602PI is a **MOSFET gate driver**, not an RF power amplifier.

Using it as a final RF power output stage is therefore outside its normal intended application.

Monitor the IC temperature when experimenting with higher supply voltages or frequencies.

**Proper filtering is mandatory.** A clean-looking fundamental signal does not mean that the transmitted spectrum is clean.

Measure your LPF and, if possible, verify the actual RF spectrum with suitable test equipment.

For initial testing, use a **50-ohm dummy load** rather than connecting the circuit directly to a radiating system.

Use only frequencies and power levels that you are legally permitted to transmit on.

## Why?

Because sometimes the simplest RF power stage is not an RF amplifier at all.

```text
GPIO4 -> gate driver -> LPF -> RF output
```

A handful of components, proper filtering and shielding, and approximately half a watt of HF output.
