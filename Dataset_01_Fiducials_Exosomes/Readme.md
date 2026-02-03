# Cryo-ET Reconstruction Tutorials  
**Automated IMOD · ETOMO · AreTomo**

## Welcome 👋

This repository accompanies the cryo-electron tomography (Cryo-ET) part and provides **guided tutorials for tomogram reconstruction using three commonly used workflows**.

The goal is not only to obtain a reconstructed tomogram, but to understand:
- **how to curate your data prior to alignment**
- **what each pipeline does**
- **how much user interaction is expected**
- **when to choose which approach**

The three options below represent **different philosophies of tomography processing**, all of which are widely used in the field.

---
## 🔴 Tilt Series Inspection and Curation
**Quality control · Subset selection · Manual decisions**

Before alignment and reconstruction, it is essential to inspect the tilt series manually. Visual inspection allows you to assess data quality and to identify entire tilt series or individual views that should be excluded. This step is especially useful if you want to
ensure reliable downstream alignment and reconstruction.

This step typically includes:
- import tilt series
- motion and CTF correct the tilt series
- manually go through each tilt series to check acquisition quality and problems (e.g. tracking failure, incorrect tilting, missing target)
  - excluding individual views
  - optionally creating a subset of tilt series for parameter tuning and testing
- understand common data-quality issues

👉 **Best choice for:**
Quality control and preparing clean input for automated workflows.

➡️ **Continue here:** [TiltseriesPreparation](./TiltseriesPreparation.md)

---
# Tilt Series Alignment and Reconstruction Workflows
There are many methods to align and reconstruct tilt series. In this course we will cover three methods. 
Click on the following links to continue the Tutorial. In this course, you are encouraged to **try more than one approach on the same dataset** and compare the results.

---

## How to choose?

| Your goal | Recommended option | Link to Tutorial|
|---------|-------------------|-----------------|
| Fast, reproducible reconstruction |  🔵 **IMOD**   |  [IMOD](automatic-imod.md)
| Learn and understand alignment | 🟣 **ETOMO**  | [ETOMO](ETOMO.md)
| No fiducials / modern workflows | 🟢 **AreTomo**  | [AreTomo](aretomo.md)

---

## 🔵 Option 1: IMOD Pipeline  
**Fast · Reproducible · Minimal interaction**

The automated IMOD workflow runs a full reconstruction pipeline with little to no user input, typically including:
- preprocessing and coarse alignment  
- automatic fiducial model generation  
- fiducial-based alignment  
- CTF correction  
- tomogram reconstruction  

This option is ideal if you want to:
- process data efficiently
- obtain reliable results for most datasets
- focus on downstream analysis rather than alignment details

👉 **Best choice for:**  
High-throughput processing, standardized workflows, and learning how automation is used in practice.

➡️ **Start here:** [Automated IMOD tutorial](./automated-imod.md)

---

## 🟣 Option 2: ETOMO (Interactive IMOD)  
**Step-by-step · Educational · Full control**

ETOMO is the interactive graphical interface of IMOD. It allows you to:
- manually pick and fix fiducials  
- inspect alignment quality at each step  
- adjust parameters and tomogram orientation  
- understand how alignment decisions affect the final reconstruction  

This option is ideal if you want to:
- learn *how* tomogram alignment works
- troubleshoot difficult datasets
- gain intuition for fiducial-based alignment

👉 **Best choice for:**  
Learning, teaching, and understanding the principles behind tomography reconstruction.

➡️ **Start here:** [ETOMO tutorial](./etomo.md)

---

## 🟢 Option 3: AreTomo  
**Fiducial-free · Robust · Modern**

AreTomo is an alignment and reconstruction tool that does **not require fiducials**. It uses patch tracking and correlation-based alignment instead.

This option is ideal if you want to:
- work with datasets without gold beads
- compare fiducial-based vs. fiducial-free approaches
- explore modern alternatives to classical IMOD workflows

👉 **Best choice for:**  
Lamellae, cellular tomography, and datasets where fiducials are sparse or absent.

➡️ **Start here:** [AreTomo tutorial](./aretomo.md)

---

## Take-home message

There is no single “correct” pipeline. Knowing how to get started with data processing is an important step. This is also a base to learn **when to trust automation**, **when to interact**, and **when to switch tools**.
