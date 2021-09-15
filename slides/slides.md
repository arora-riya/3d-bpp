---
theme: seriph
title: '3D-BPP'
titleTemplate: '%s - Slides'
background: https://www.publicdomainpictures.net/pictures/200000/velka/plain-red-background.jpg
class: 'text-center'
highlighter: 'prism'
lineNumbers: false
colorSchema: 'light'
drawings:
  persist: false
---

<style>
  p {
    opacity: 1.0 !important;
  }
</style>

# 3D Bin Packing

Artificial Intelligence in Industry - Final Project <br> 
University of Bologna - Academic Year 2020/21

<div class="pt-12">
  Leonardo Calbi, Lorenzo Cellini, Alessio Falai
</div>

<div class="abs-br m-6 flex gap-2">
  <a href="https://github.com/Wadaboa/3d-bpp" target="_blank" alt="GitHub"
    class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

---

# 3D-BPP

## Problem definition

- We are given a set of $n$ rectangular-shaped _items_, each characterized by width $w_j$, depth $d_j$ and height $h_j$ $(j\in J=\{1,\dots,n\})$
- We are also given an unlimited number of identical 3D containers (_bins_) having width $W$, depth $D$ and height $H$
- The _three-dimensional bin-packing problem_ (3D-BPP) consists in orthogonally packing all the items into the minimum number of bins

## Assumptions

- Items may not be rotated
- All input data are positive integers satisfying $w_j\le W, d_j\le D, h_j\le H$

---

# Lower bounds

1. $L_0=\left\lceil\frac{\sum_{j=1}^n v_j}{B}\right\rceil$, where $v_j=w_j\times d_h\times h_j$
    - Continuous lower bound: measures the overall liquid volume
    - Worst-case performance ratio of $\frac{1}{8}$
    - Time complexity: $O(n)$
1. $L_1=\max\{L_1^{WH},L_1^{WD},L_1^{HD}\}$
    - Obtained by reduction to the one-dimensional case
    - Worst-case performance arbitrarily bad
    - Time complexity: $O(n)$
1. $L_2=\max\{L_2^{WH},L_2^{WD},L_2^{HD}\}$
    - Explicitly takes into account the three dimensions of the items and dominates $L_1$
    - Worst-case performance ratio of $\frac{2}{3}$
    - Time complexity: $O(n^2)$

---

# Dataset

## Distributions
$$
\begin{array} {|l|l|l|}\hline 
  \textbf{Characteristic} & \textbf{Distribution} & \textbf{Parameters}        \\ \hline
\text{Depth/width sratio } R_{DW}       & \text{Normal}                & (0.695,0.118)  \\
\text{Height/width ratio } R_{HW}      & \text{Lognormal}             & (-0.654,0.453) \\
\text{Repetition } F             & \text{Lognormal}             & (0.544,0.658)  \\
\text{Volume } V     & \text{Lognormal}             & (2.568,0.705) \\ 
\text{Weight } L     & \text{Lognormal}             & (2,2) \\ \hline
\end{array}
$$

## Reasoning
1. Volumes: $V\sim LN (\mu, \sigma), \mu=\frac{\sum_{j=1}^{N}\log v_j}{N}, \sigma=\frac{\sum_{j=1}^{N}(\log v_j-\mu)^2}{N},N=166407, j\in\{C_1,\dots C_5\}$
1. Widths: $W=(\frac{V}{R_{DW} \times R_{HW}})^{\frac{1}{3}}$
1. Depths: $D=W\times R_{DW}$
1. Heights: $H=W\times R_{HW}$

---

# Superitems

![superitems](/superitems.png)

- A superitem is a collection of individual items that are compactly stacked together
- Building procedure: single, then horizontal and then vertical
- Possibility to restrict horizontal superitems (either _2W_ or _2D_ or _4_ or none)
- Horizontal superitems are composed of items having the exact same dimensions
- Vertical superitems are composed of items s.t. the ones on top have an area support of at least $70\%$ thanks to the ones underneath
- Vertical superitems can be composed of at maximum $M$ superitems (either single or horizontal)

---

# Layers

<p align="center">
  <img src='/layer-pool-example.png'/>
</p>

- A layer is defined as a two-dimensional arrangement of items within the horizontal boundaries of a bin with no superitems stacked on top of each other

---

# Workflow

<p align="center">
  <img src='/flow.png' width='340'/>
</p>

---
layout: two-cols
---

# Baseline

<p align="left">
  <img src='/sppsi.png' width="400"/>
</p>

::right::

# Constraints

- (5): ensure that every item is included in a layer
- (6): define the height of layer $l$
- (7): redundant valid cuts that force the area of a layer to fit within the area of a bin
- (8): enforce at least one relative positioning relationship between each pair of items in a layer
- (9) and (10): ensure that there is at most one spatial relationship between items $i$ and $j$ along each of the width and depth dimensions 
- (11) and (12): non-overlapping constraints
- (13) and (14): ensure that items are placed within the boundaries of the bin

---

# Maxrects

<p align="center">
  <img src='/maxrects.png' width="500"/>
</p>

---
layout: two-cols
---

# Column generation

<p align="center">
  <img src='/cg.png' width="200"/>
</p>

<p align="center">
  <img src='/rmp.png' width="400"/>
</p>

<p align="center">
  <img src='/sp.png' width="400"/>
</p>

::right::

## Models

_RMP_
  - $\lambda$ are the dual variables corresponding to constraints (18)

_SP_
  - $o^l-\sum_i\sum_s\lambda_i f_{si} z_{sl}$ is the reduced cost of a new layer $l$
  - No-placement and placement strategy

## Details

- Warm start: maxrects vs single item layers
- Optimality: no branch-and-price scheme

---

# Layer filtering

1. _Discard layers by densities_ (given a minimum density $d_m$)
1. _Discard layers by item coverage_
      - If at least one item in layer $l$ was already selected more times than $M_a$, discard $l$
      - If at least $M_s$ items in layer $l$ are already covered by previously selected layers, discard $l$
2. _Remove duplicated items_
      - Remove superitems in different layers containing the same item (remove the ones in less dense layers)
      - Remove superitems in the same layer containing the same item (remove the ones with less volume)
      - Re-arrange layers (using maxrects) in which at least one superitem was removed (if $d_l > d_m$)
3. _Remove empty layers_
4. _Sort layers by densities_


---

# Bin packing

<p align="center">
  <img src='/bin-example.png' width="300"/>
</p>

1. Uncovered items: create new layers filled with items that were not covered in the previous procedures and add them to the layer pool
1. Bins building procedure: stack layers on top of each other until a bin is full and open new bins as needed
1. Compact bins: let items "fall" to the ground as much as possible, without allowing intersections


---

# Future improvements

- Allow items to be rotated on the 3 axis
- Integrate the branch-and-price scheme into column generation to prove optimality
- Handle weight constraints and bin load capacity
- Improve item support through MP models (as described in the paper)

---
layout: center
---

# Demo

```python
python3 -m streamlit run src/dashboard.py
```

---
layout: end
---

Bye bye