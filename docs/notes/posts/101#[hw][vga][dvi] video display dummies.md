---
number: 101
title: "[hw][vga][dvi] video display dummies"
state: open
url: https://github.com/konosubakonoakua/blog/issues/101
author: konosubakonoakua
createdAt: 2024-09-29T09:54:18Z
updatedAt: 2024-09-29T16:41:40Z
labels: []
---

# 101 [hw][vga][dvi] video display dummies

## 75 Ohm Resistor (Any resistor between 50 and 150 ohms is okay)
> [!TIP]
> simulating load on one signal, without soldering, or any extra hardware. I simply insert the resistor —75 Ω is the ideal spec— in the DVI output port to make contact between C1-C5 (analog red - analog ground), or C2-C5 (green - ground), or C3-C5 (blue - ground).

![image](../assets/101/1.png)
<img src="https://github.com/user-attachments/assets/fc212b02-dc6e-4391-a288-664f6ccbcd06" width="30%" />
![image](../assets/101/2.png)

## not working with rgb connect with 75 Ohm, but need also field & row sync.
> [!NOTE]
> Not working with external video card, but ok with internal video card.

![image](../assets/101/3.png)
<img src="https://github.com/user-attachments/assets/a9fbed90-55ba-4691-8f05-4a7f87c63881" width="50%" />
<img src="https://github.com/user-attachments/assets/377777c7-3290-42ee-99d7-98e562195bd2" width="50%" />




---

## Comments

### konosubakonoakua · 2024-09-29T10:01:17Z

## References
- https://blog.zorinaq.com/the-5-second-vga-dummy-plug/
