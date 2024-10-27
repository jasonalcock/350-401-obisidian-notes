---
aliases: 
publish: 
---
%%
date:: [[2024-10-20]]
parent:: 
%%
# [[Wireless Signals & Modulation]]

## Electromagnetic Spectrum Overview

The electromagnetic spectrum consists of waves defined by three main properties:

- **Wavelength**: The distance between two peaks of a wave, measured in meters.
- **Frequency**: The number of wave cycles passing a point per second, measured in Hertz (Hz).
- **Energy**: Related to frequency, with higher frequencies corresponding to higher energies.

### Key Types of Electromagnetic Waves (Lowest to Highest Frequency)

|Type of Wave|Approximate Frequency Range|Applications|
|---|---|---|
|**Radio Waves**|< 300 GHz|Communication (Wi-Fi, radio)|
|**Microwaves**|300 MHz - 300 GHz|Wireless links, radar, heating|
|**Infrared (IR)**|300 GHz - 400 THz|Remote controls, thermal images|
|**Visible Light**|430 THz - 770 THz|Human vision, fiber optics|
|**Ultraviolet (UV)**|750 THz - 30 PHz|Sterilization, fluorescence|
|**X-rays**|30 PHz - 30 EHz|Medical imaging, security scans|
|**Gamma Rays**|> 30 EHz|Cancer treatment, astronomy|

---

### Key Relationships

- **Wavelength and Frequency**: Inversely proportional. Higher frequencies mean shorter wavelengths.
- **Frequency and Energy**: Directly proportional. Higher frequency waves carry more energy.

![[Pasted image 20241027184054.png]]

### Wireless Network Frequencies

- **Operating Range**: Most wireless networks operate within the **2.4 GHz to 6 GHz** microwave range.
    
- **Wi-Fi Bands**: Commonly referred to as the **2.4 GHz** and **5 GHz** bands, these numbers represent the centre frequencies.
    
    > _Note_: Each band is broken down into smaller segments, known as channels, by regulatory bodies like the **Unlicensed National Information Infrastructure (U-NII)**.
    

---

### Key Concepts in Wireless Frequency & Bandwidth

- **Band**: A range of frequencies grouped together for specific applications. For example, the **AM band** ranges from **530 kHz to 1710 kHz**.
    
- **Channel**: A segment within a band with a designated center frequency. Wireless channels don’t operate on a single frequency—they have a bandwidth that overlaps into adjacent frequencies.
    
- **Bandwidth**: The frequency range needed to operate a channel. In Wi-Fi, this is the amount of spectrum a channel occupies.
    

---

### 2.4 GHz Wi-Fi Band (Details)

- **Frequency Range**: 2.400 GHz to 2.4835 GHz, spanning **0.0835 GHz**.
- **Channels**: Divided into **11 channels**, each with a **22 MHz bandwidth**.
    - **Channel Spacing**: Channels are spaced **5 MHz apart**, resulting in frequency overlap.
    - **Non-Overlapping Channels**: Channels **1, 6, and 11** are non-overlapping, which minimises interference.

> **Tip**: Understanding channel placement and overlap is crucial for network configuration and interference management in Wi-Fi networks.

### 5 GHz Wi-Fi Band (Details)

- **Frequency Range**: Covers **5.150 GHz to 5.925 GHz**, split into multiple sub-bands by U-NII.
- **Channels**:
    - Channels are **20 MHz wide** but can be bonded to create **40 MHz, 80 MHz,** or **160 MHz** channels for higher data throughput.
    - Unlike 2.4 GHz, the 5 GHz band has **non-overlapping channels** due to its wider frequency range, reducing interference potential.
- **Sub-Bands (U-NII Designations)**:
    - **U-NII-1 (5.150–5.250 GHz)**: Indoor, low-power use.
    - **U-NII-2A (5.250–5.350 GHz)**: Indoor/outdoor, requires DFS (Dynamic Frequency Selection) to avoid radar interference.
    - **U-NII-2C (5.470–5.725 GHz)**: Same as U-NII-2A with DFS.
    - **U-NII-3 (5.725–5.850 GHz)**: Outdoor, higher power allowed.
    - **U-NII-4 (5.850–5.925 GHz)**: Primarily for vehicle-to-everything (V2X) applications.

> **Non-Overlapping Channels**: In 5 GHz, the wider frequency range allows for more non-overlapping channels, improving network performance and minimizing interference compared to the 2.4 GHz band.

> **DFS Requirement**: Dynamic Frequency Selection (DFS) helps avoid conflicts with radar signals, a requirement in certain 5 GHz sub-bands.



![[Pasted image 20241027184031.png]]