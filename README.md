# AM5 DDR5 Overclocking Guide

A comprehensive guide to overclocking DDR5 memory on the AM5 platform.

## Disclaimer

This guide is based on my personal experience with the following hardware:
* **Memory Kit:** G.Skill 24GBx2 Single-Rank (Hynix M-die)

Please be aware that overclocking can potentially damage your hardware. Proceed at your own risk.

---
*A special thanks to all the pioneers and community members for sharing their knowledge.*


# Ryzen 9 9800X3D Memory Overclocking Guide: A Journey to 6400MT/s CL28

This report provides a systematic and detailed guide for achieving a DDR5-6400MT/s overclock with a CAS Latency (CL) of 28 in a 1:1 synchronous mode (MCLK=UCLK). The hardware used for this guide is the AMD Ryzen 9 9800X3D processor, an MSI MAG X870 Tomahawk Wifi motherboard, and a G.Skill F5-6000J2636H24GX2-TR5NS memory kit.

This document is not merely a theoretical procedure but a practical record reflecting the real-world challenges, analysis, and solutions encountered during the actual tuning process to achieve extreme low latency.

---

## Section 1: Foundation for High-Performance Overclocking

This initial phase is critical and cannot be skipped, as it determines the stability of the entire subsequent tuning process. Insufficient preparation here will make it significantly harder to diagnose problems that arise in later stages.

### 1.1. Required Software

Effective overclocking is supported by precise monitoring, benchmarking, and stress testing. The following applications are industry-standard tools for memory tuning.

* **AIDA64 Extreme:** The primary tool for measuring memory bandwidth and latency. It is used to quantitatively assess performance improvements.
* **TestMem5 (TM5) with Anta777 Absolut / 1usmus v3 Config:** The current gold standard for memory error detection. In addition to the base TM5 application, it is crucial to acquire a stricter testing profile. Running with administrator privileges is recommended.
* **OCCT:** An essential tool for verifying the stability of the CPU's Integrated Memory Controller (IMC) and Infinity Fabric (FCLK). It can expose instabilities under complex loads that memory-only tests might miss.
* **HWiNFO64:** Indispensable for real-time monitoring of crucial data points such as CPU SOC voltage, DRAM VDD/VDDQ voltages, and memory module temperatures.
* **ZenTimings:** A utility that displays all memory timings, related clocks, and voltages in detail for Ryzen systems in real-time. It is extremely useful for confirming that BIOS settings have been correctly applied within the OS.

### 1.2. Initial BIOS Setup (Optimized Defaults)

Before beginning to overclock, it is essential to start from a clean, stable, default state, ensuring no previous settings remain.

**Steps:**
1.  Enter the BIOS and execute `Load Optimized Defaults` (usually the F6 key).
2.  Enable `Re-Size BAR Support`.
3.  Disable any CPU auto-overclocking features like `MSI Game Boost`, as they can interfere with manual tuning.

---

## Section 2: Establishing a Performance Baseline

Before attempting a manual overclock, you must quantitatively measure the system's performance in its default and manufacturer-certified overclocked states. This data will serve as the reference point for evaluating your subsequent tuning efforts.

### 2.1. Benchmark 1: JEDEC Default Profile

After loading optimized defaults, the memory will operate at its JEDEC base specification (e.g., 5200MT/s). Run the AIDA64 Cache & Memory Benchmark and record this baseline latency value.

### 2.2. Benchmark 2: EXPO Certified Profile

Enabling the EXPO profile will automatically load the memory manufacturer's tested optimal settings. If you encounter boot failures or instability at this stage, it may indicate a fundamental issue beyond the scope of manual tuning that needs to be addressed first. Run the AIDA64 benchmark again and record your "EXPO baseline".

---

## Section 3: The Challenge: Achieving 6400MT/s

This is the most challenging and critical phase of the process. It requires an analytical approach—not just inputting values, but observing system behavior and identifying the root cause of any issues.

### 3.1. Optimizing AM5 Clock Ratios and Platform Settings

Optimal latency is achieved when the memory controller clock (UCLK) runs at the same frequency as the memory's effective clock (MCLK), known as a 1:1 ratio. Additionally, the target for the Infinity Fabric Clock (FCLK), which connects the CPU cores to the memory controller, is 2133MHz, forming an ideal 3:2 synchronous ratio for 6400MT/s operation.

**Steps:**
1.  In the BIOS OC menu, set `DRAM Frequency` to **DDR5-6400MT/s** and `UCLK DIV1 Mode` to **UCLK=MCLK**.
2.  To prioritize stability initially, set `FCLK Frequency` to **2000MHz**.
3.  Configure the following platform settings to maximize performance:
    * `Power Down Enable`: **Disabled**. This improves stability during overclocking.
    * `Memory Context Restore`: **Disabled**. This helps in finding stable settings during the initial tuning phase.
    * `Data Scramble`: **Disabled**. May slightly increase latency, so it's disabled.
    * `SVM (Secure Virtual Machine) & IOMMU`: **Disabled**. While useful for virtualization, disabling these can sometimes slightly improve latency if not needed.
    * `TSME (Transparent Secure Memory Encryption)`: **Disabled**. This is essential, as enabling it can incur a 5-10ns latency penalty.

### 3.2. Voltage Stabilization: Finding the IMC's Limit

The stability of a 6400MT/s 1:1 mode is heavily dependent on the quality of the CPU's Integrated Memory Controller (IMC)—the "silicon lottery"—and the key to this is the **CPU SOC Voltage (VSOC)**.

**Tuning Methodology:**
1.  Start by attempting to boot with very loose timings (e.g., tCL 36, tRCD/tRP 46, tRAS 80).
2.  Begin with `DRAM Voltage (VDD/VDDQ)` at **1.40V** and `CPU SOC Voltage` at **1.25V**.
3.  If TestMem5 produces errors, first try increasing the DRAM voltage to 1.42V.
4.  > **IMPORTANT:** If increasing the DRAM voltage worsens the errors or causes them to appear sooner, this strongly suggests the issue lies with the CPU's IMC, not the DRAM chips. In this case, revert the DRAM voltage and increase the `CPU SOC Voltage` in 0.02V increments (e.g., to 1.27V).
5.  Through this process, stability with loose timings was established for this specific hardware at **CPU SOC 1.27V** and **DRAM VDD/VDDQ 1.40V**. This becomes the foundation for all further tuning.

---

## Section 4: Optimizing Primary Timings - Achieving CL28

With a stable voltage foundation, it's time to tighten the primary timings.

**Steps:**
1.  **Adjusting tCL:** Lower tCL in steps of 2, from 36 -> 32 -> 30 -> to the target of **28**. Run a TM5 stability test at each step.
2.  **Adjusting tRCD and tRP:** Once tCL 28 is stable, begin tightening tRCD and tRP. During this process, it was discovered that `tRCDRD` hit a wall at **38**, a critical finding that identified the memory's limit. However, `tRCDWR` and `tRP` could be tightened further, ultimately achieving excellent values of **20** and **34**, respectively.
3.  **Adjusting tRAS and tRC:** Using the general rules of thumb `tRAS ≈ tCL + tRCDRD` and `tRC ≈ tRAS + tRP`, very tight and stable values of **tRAS 66** and **tRC 48** were achieved.

---

## Section 5: Final Settings

### 5.1. Nitro Mode Settings

As the final step, after all timings and voltages are stable, enable `Nitro Mode` to optimize memory training.

* `Robust training Enable`: `1/2/0` -> `1/2/1` -> `1/3/1`. (Lower values on the left are better).

---
## References

1.  [MAG X870 TOMAHAWK WIFI | Gaming Motherboards｜Best Motherboard for AI PC - MSI](https://www.msi.com/Motherboard/MAG-X870-TOMAHAWK-WIFI)
2.  [AMD Ryzen 7000/9000 DDR5 RAM OC Guide - OCinside.de](https://www.ocinside.de/workshop_en/amd_ryzen_7000_9000_ddr5_oc_guide/3/)
3.  [Fool-Proof DDR5 Overclocking Guide for AM5 (Focus on 64GB, Dual Rank) - Reddit](https://www.reddit.com/r/overclocking/comments/1kdhqbm/foolproof_ddr5_overclocking_guide_for_am5_focus/)
4.  [AMD UCLK DIV1 Mode - SkatterBencher](https://skatterbencher.com/amd-uclk-div1-mode/)
5.  [AM5 - DDR5 Tuning Cheat Sheet, observations and notes : r/overclocking - Reddit](https://www.reddit.com/r/overclocking/comments/1k3o7qe/am5_ddr5_tuning_cheat_sheet_observations_and_notes/)
