---
title:  "SSD Controller Verification"
date:   2024-01-18 12:00:30 +0800
categories: SSD
toc: true
toc_icon: glasses
# layout: posts
---

This article provides insights into considerations for SSD Controller Functional Verification.

## Introduction

The content is a summarized compilation of knowledge gained from the presentation by *Vikas Tomar* at **Flash Memory Summit 2019**. The original PDF file can be downloaded [here](https://www.flashmemorysummit.com/Proceedings2019/08-07-Wednesday/20190807_TEST-202A-1_Tomar.pdf).

## Notes

### What to verify?

- Link Level verification (PCIe)
  - Interrupts (MSI, MSIx)
  - PCIe power management (Various Power saving states)
  - Resets
- NVMe Controller Register Level verification
  - Register values
  - Action on register access
- Queue Interface
  - Queue creation/deletion, Doorbell, Empty/Full conditions
  - Queue location and data access
  - Queue starving*
- Data transfer between Host and controller
  - Data Access direction
  - Extra RD/WR on PCIe interface*
- Command Level
  - Admin and IO command
  - Autonomous commands like Abort, Event notifications
  - Possible completion status
- Data structure access
  - PRP (Offsets for PRP1 and PRP2)
  - SGL (Various Descriptors)
- Data structure Values
  - Identify data structures
  - Name space data structures
  - Log pages access
- Feature verification
- Error handling verification

### Verification Challenges

- Large Configurations space
  - Behavior of a NVMe operation depends on the combination of various parameter
  - Similarly NVMe SSD can show different performance statistics depending upon
    - Feature enabled by host
    - Queues created by host
    - Parameters related to data transfer selected by host
- Extensive feature support
- Interoperability and Conformance affected by
  - Large configuration space
  - New features
  - Various platforms
- Stimulus generation
  - Impossible to create directed scenarios
  - Randomization helps but do not solve the problem
- Debug
  - Hard to investigate a suspicious transaction

### Test Plans

- Controller configuration Test plan
  - SSD Name space (NS DS)
  - Command support
- Host configuration Test plan
  - Covering the possible host configurations
- NVMe Protocol Events Test Plan
  - Specification mapped feature wise Test plan covering
    - Controller register space field access
    - Queue operations
    - PCIe Features
    - NVMe Features
- Standard Compliance test plan

## Summary

SSD controller verification is a complex task, encompassing various types of operations. It is considered challenging, even when focusing solely on functional correctness. Customizing SSD controllers for **entrepreneurial** purposes can further complicate the process to meet heightened performance requirements.
