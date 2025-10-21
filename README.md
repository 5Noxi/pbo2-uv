# PBO UV

Dynamically adjusts CPU voltage and frequency, modifying power limits based on cooling, workload, and VRM capability. Curve Optimizer in PBO enables per core undervolting that adapts to both light and heavy workloads. Each processor has different undervolting potential, with more headroom at low loads and less at high loads. It applies undervolting using counts instead of raw millivolts, with `1 count ≈ 3mV–5mV`. Users can adjust up to `+/- 30` counts (`90–150mV` total) on all cores or individual cores.

![](https://github.com/5Noxi/pbo2-uv/blob/main/images/pbointro1.png)
![](https://github.com/5Noxi/pbo2-uv/blob/main/images/pbointro2.png)

# PBO Settings Overview

| Setting | Description | Limitation / Effect |
|--------|-------------|----------------------|
| **PPT** (Package Power Tracking) | Maximum power the CPU can draw before boost levels off. | Limited by cooling. |
| **EDC** (Electrical Design Current) | Maximum peak current the VRM allows for short bursts. | Limited by VRM specs. |
| **TDC** (Thermal Design Current) | Maximum sustained current the VRM can deliver over time. | Limited by VRM cooling. |
| **Platform Thermal Throttle** | Sets the maximum CPU temperature before throttling occurs. | Lowering it too much reduces performance. |
| **Scalar** | Increases the FIT limit (`1x–10x`), enabling higher voltages. | May shorten CPU lifespan. |
| **Boost Clock Override** (Fmax Override) | Raises the maximum boost frequency (`-1000` to `+200 MHz`, `25 MHz` steps). | Some motherboards allow higher values, but they have no effect. |
| **Curve Optimizer** | Adjusts per-core voltage by modifying the VFT table. | Fine-tunes efficiency and boost behavior. |

![](https://github.com/5Noxi/pbo2-uv/blob/main/images/pbotechow.png)

Example scenarios ([*](https://skatterbencher.com/amd-precision-boost-overdrive-2/):

![](https://github.com/5Noxi/pbo2-uv/blob/main/images/scenarios.png)

# Premodifications

Download [Prime95](https://www.mersenne.org/download/), [HWiNFO](https://www.hwinfo.com/download/) & [Corecycler](https://github.com/sp00n/corecycler/releases). Optionally, download [PBO2 Tuner](https://drive.google.com/file/d/1OswZcZ72jhm_Neek9c7PV-aRhM1EuOrX/view) ([thread](https://www.overclock.net/threads/corecycler-tool-for-testing-single-core-stability-e-g-curve-optimizer-settings.1777398/page-45#post-28999750)), which allows you to view the current values/limits and edit the curve via a trigger during boot, but you will set them permanently via the UEFI anyway.

- Enter your BIOS and search for the `Precision Boost Overdrive` tab
  - You may have multiple ones. Find out which one overwrites the others by changing a limit and testing it e.g. with small P95 FFTs
  - Some duplications may include single PBO settings, ignore them
- Disable `PBO Fmax Enhancer`

![](https://github.com/5Noxi/pbo2-uv/blob/main/images/fmaxenh.png)

- Setting limits (will be edited at the end)
  - Look up the PPT, EDC, and TDC limits used on your CPU.
- Leave scalar at `Auto` (or `1x`/`2x` which is mostly the same as `Auto`), never set it to `10x`
- Don't change platform thermal throttle, unless you want to decrease the max temps
- Boost Clock Override (`Fmax Override`), as written above "Raises frequency ceiling (`-1000` to `+200 MHz`) - `25 MHz` steps"
  - Increase it to the desired value - you can start with `+100MHz`/`+200MHz`. Make sure that your temperatures are fine, so you don't throttle

![](https://github.com/5Noxi/pbo2-uv/blob/main/images/fmaxov.png)

- Set Curve Optimizer to `All Cores` & use `-30`

![](https://github.com/5Noxi/pbo2-uv/blob/main/images/magnitude.png)

# Stress Testing

Since you currently have all cores set to `-30`, many may be unstable/crash. Therefore, filter them with [Corecycler]((https://github.com/sp00n/corecycler/releases)).

- Open `Run CoreCycler.bat`
- Let it pass one iteration
  - You'll probably see errors on specific cores now
- Reboot into UEFI, set the curve optimizer to `Per Core`
  - Set the cores, which have trown a error to `-25`
  - Leave the other ones at `-30`
- Run corecycler again, repeat the steps
  - Once you have passed an iteration, let it run for a while, as the other cores could still throw an error
- If corecycler can run for `~5H` (or less/more - at least `1-2H`) without any crashes, you should be done with the curve optimizer

Afterwards you can use [`Prime95`](https://prime95.net/download/) (smallest/small FFTs), [`Y-Cruncher`](https://www.numberworld.org/y-cruncher/) (all), [`OCCT`](https://www.numberworld.org/y-cruncher/), [`LinPack`](https://github.com/BoringBoredom/Linpack-Extended).

# PPT, TDC & EDC Limits

You can either set `PPT Limit`, `TDC Limit` & `EDC Limit` to manual and their default value, or try to find them as shown below (may not be accurate as your system is probably not stable enough):

- Set them all to unlimited (e.g. 1000)
- Let [Prime95](https://www.mersenne.org/download/) (small FFTs) run for `5-10 min`
  - Open [HWiNFO](https://www.hwinfo.com/download/) during the test, as you should note the following values (under the `Enhanced CPU` section):

![](https://github.com/5Noxi/pbo2-uv/blob/main/images/limits.png)

- Go back into the BIOS, set the limits to manual & type in the maximum values
- Now you'll have to test each step with [PresentMon](https://github.com/GameTechDev/PresentMon) & any CPU heavy game
- Benchmark the game & capture your FPS with PresentMon (you can upload your `.csv` file [here](https://boringboredom.github.io/Frame-Time-Analysis/)), lower the limits & repeat
  - At the beginning you can decrease all of them at the same time
     - PPT `-10` 
     - EDC `-10`
     - TDC `-5`
- After noticing a noticeable performance loss, go back to the previous limits
  - Change each limit individually, start e.g. with PPT
  - Benchmark, check whether there is a loss of performance. If so, use the previous limit, do the same with EDC, then with TDC
- You should now have the lowest limits, which don't cause any performance loss
