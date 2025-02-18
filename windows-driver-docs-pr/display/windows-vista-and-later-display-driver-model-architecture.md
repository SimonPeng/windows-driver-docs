---
title: WDDM Architecture
description: Windows Display Driver Model (WDDM) Architecture
keywords:
- display driver model WDK Windows Vista , architecture
- Windows Vista display driver model WDK , architecture
- architecture WDK display
- user-mode display drivers WDK Windows Vista , architecture
ms.date: 08/10/2021
---

# WDDM Architecture

The display driver model architecture for the Windows Display Driver Model (WDDM), available starting with Windows Vista, is composed of user-mode and kernel-mode parts. The following figure shows the architecture required to support WDDM.

![diagram illustrating the wddm architecture.](images/dx10arch.png)

A graphics hardware vendor must supply the user-mode display driver and the display miniport driver. The user-mode display driver is a dynamic-link library (DLL) that is loaded by the Microsoft Direct3D runtime. The display *miniport driver* communicates with the Microsoft DirectX graphics kernel subsystem. For more information about the user-mode display driver and display miniport driver, see the [Windows Display Driver Model (WDDM) Reference](/windows-hardware/drivers/ddi/_display/).
