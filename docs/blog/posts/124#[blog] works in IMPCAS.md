---
number: 124
title: "[blog] works in IMPCAS"
state: open
url: https://github.com/konosubakonoakua/blog/issues/124
author: konosubakonoakua
createdAt: 2024-12-02T09:41:37Z
updatedAt: 2025-10-22T07:22:26Z
labels: []
---

# 124 [blog] works in IMPCAS

## Beam Loss Monitor (v1.0)

> Contributions:
> - test bench setup
> - automatic test program (python)
> - post-data processing (python)
> - softioc program (C++)

- Test bench for beam loss monitor
  <details>
  
  ![image](../assets/124/1.png)
  <img src=https://github.com/user-attachments/assets/62d50cb1-3d63-4454-ba13-8a40ef5a74ad width="70%">
  ![image](../assets/124/2.png)
  
  </details>
- 6ch waveform retrieve (keysight oscilloscope control, qt5+python+qcodes)
  <details>
  
  ![image](../assets/124/3.png)

  </details>
- frequency sweeping (tek function generator control, qt5+python+qcodes)
  <details>
  
  ![11392eb0482cb9bd9d33113fb6f7b12](../assets/124/4.png)

  </details>
- softioc (running on zynq7015, C++ version)
  <details>

  - publish PVs (counts, interlock, ADC raw data...)
  - calibration (ADC readout)
  - autosave (save key parameters)

  </details>
- diagrams showcase (jupyter notebook)
  <details>

  ![image](../assets/124/5.png)
  ![image](../assets/124/6.png)
  ![image](../assets/124/7.png)

  </details>


---

## Comments

### konosubakonoakua · 2024-12-02T10:04:12Z

## cfc test equipment (128ch)
we need an equipment for 128 Channel Charge Integrator which can route our current (pA) signal to it  one by one.

<details>

![image](../assets/124/8.png)

</details>

> Contributions:
> - test equipment develop
>   - fpga program (verilog)
>   - softioc program (python)
>   - hardware design (designed with kicad, manufactured by JLC PCB)
> - keithley current source control (python)
> - automatic test program (jupyter notbook)
> - data processing (python data retrieve, diagram plotting)

- test equipment
  - first version (not my hardware design but I developed its FPGA program. There're so many HW bugs, even relays are all flipped, fixed by hand...🙄)
    <details>
    
    ![image](../assets/124/9.png)
    ![image](../assets/124/10.png)
    ![8938ac82c08360e185bbf33143d6729](../assets/124/11.jpg)
    
    </details>
  - second version (basically the same with first version with HW issues fixed but still inconvenient, can only switch channels in order)

    <details>
  
    ![8813a5ea0e63b7947aa0282c9e5c23a](../assets/124/12.jpg)
  
    </details>

  - third version (use raspberry pi SBC with a 128ch io-expander hat, python+softioc inside, easy to use, fully designed by myself)
    <details>
  
    ![image](../assets/124/13.png)
    ![image](https://github.com/user-attachments/assets/2632d545-c4e1-4615-a65a-aeeaaedfefc3)
    ![PixPin_2024-12-02_17-14-05](https://github.com/user-attachments/assets/ffdddc62-3a59-4812-a080-eb3697e8e616)
  
    </details>
- keithley current source control (softioc based on pcaspy with phoebus opi)

  <details>

  ![image](../assets/124/16.png)

  </details>

- diagrams showcase (jupyter notebook)
  <details>
  
  ![image](../assets/124/17.png)
  ![image](../assets/124/18.png)
  ![image](../assets/124/19.png)

  </details>


### konosubakonoakua · 2024-12-02T13:10:21Z

## machine protect
> Contributions:
> - data processing (python data retrieve, parsing binary data with pandas then render to csv file)

## Magnet Test
> Contributions:
> - data processing (python data retrieve, render to html)

<details>

![image](../assets/124/20.png)

</details>



### konosubakonoakua · 2024-12-02T18:13:41Z


## non-linear fitting tool (based on scipy & sympy & jupyterlab, beaucage model)
<details>

![image](../assets/124/21.png)
![recording](../assets/124/22.gif)

</details>


### konosubakonoakua · 2024-12-19T00:39:10Z

## Hardware Development
### [Zynq7020 Core Board](https://github.com/konosubakonoakua/hw-zynq7020-core-board/)
<details>

<summary> screenshots </summary>

![top](https://github.com/user-attachments/assets/de350671-9043-4a9b-b9fe-817ccbea7607)
![bottom](../assets/124/24.png)

</details>

### [TF Card Connector](https://github.com/konosubakonoakua/hw-tf-card-connector)
<details>

<summary> screenshots </summary>

![top](../assets/124/25.png)
![bottom](../assets/124/26.png)


</details>

### konosubakonoakua · 2024-12-19T04:47:34Z

## Tools
### [FPGA downloader](https://github.com/konosubakonoakua/openFPGALoader-GUI)
A python program which can be running on raspberry pi or PC.
Engineers handle over FPGA flash files (*.mcs) to non-programmers via U-disk, then they can attach JTAG to raspberry pi to perform FPGA flash downloading independently which saves the time of engineers.

<details>

<summary> screenshot </summary>

![openFPGALoader](../assets/124/27.png)


</details>
