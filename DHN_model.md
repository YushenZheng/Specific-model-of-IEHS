# District Heating Network OTF Model

This file reproduces Appendix A.3 of the manuscript: the specific operation thermal flow (OTF) model of the district heating network (DHN).

## Notation

### Sets and Indices

| Symbol | Description |
|---|---|
| `T` | Set of dispatch periods |
| `t` | Index of a dispatch period |
| `b` | Index of a heating pipeline |
| `i` | Index of a DHN node |
| `g` | Index of a heat source |
| `P_i^-` | Set of pipelines ending at node `i` |
| `G_i` | Set of heat sources connected to node `i` |

### Superscripts

| Symbol | Description |
|---|---|
| `NS`, `NR` | Nodal temperature in the supply and return networks |
| `PS,in`, `PS,out` | Inlet and outlet temperatures of supply-side pipelines |
| `PR,in`, `PR,out` | Inlet and outlet temperatures of return-side pipelines |
| `GS`, `GR` | Source-side supply and return temperatures |
| `DS`, `DR` | Load-side supply and return temperatures |
| `A` | Ambient-temperature-related quantity |
| `G` | Heat-source-related quantity |
| `D` | Heat-load-related quantity |
| `H` | DHN-owned heat source or DHN internal variable |
| `E` | External or EPN-side heat source boundary variable |

### Operators and Matrix Notation

| Symbol | Description |
|---|---|
| `col(.)` | Column-vector concatenation |
| `diag(.)` | Diagonal matrix formed from a vector |
| `otimes` | Kronecker product |
| `I_T` | Identity matrix with dimension equal to the number of time periods |
| `underline(.)`, `overline(.)` | Lower and upper bounds |
| `A_+`, `A_-` | Incidence matrices describing starting and ending nodes of pipelines |
| `A_G`, `A_D` | Incidence matrices mapping heat sources and heat loads to DHN nodes |
| `K`, `J` | Pipeline temperature propagation and heat-loss coefficient matrices |
| `Omega^(.)` | Mass-flow weighting matrix used in temperature mixing |
## Variable Concatenation and Matrix Construction

In DHN, the constant mass flow and variable temperature (CF-VT) operation is investigated, which has been widely adopted, e.g., by China and some European countries. The mass flow is pre-scheduled while the temperature of circulating water is adjusted to meet the load. Variables are concatenated into a column vector as:

```math
\mathbf{x}
=
\mathrm{col}_{t}
\left[
\mathrm{col}_{b}
\left(
x_{b,t}
\right)
\right].
```

For example,

```math
\mathbf{\tau}^{\mathrm{PS,out}}
=
\left[
\tau_{1,1}^{\mathrm{PS,out}},
\tau_{2,1}^{\mathrm{PS,out}},
\ldots,
\tau_{E,1}^{\mathrm{PS,out}},
\tau_{1,2}^{\mathrm{PS,out}},
\ldots,
\tau_{E,2}^{\mathrm{PS,out}},
\ldots,
\tau_{E,T}^{\mathrm{PS,out}}
\right]^{T}.
\qquad (82)
```

Vectors `\mathbf{\tau}^{\mathrm{NS}}`, `\mathbf{\tau}^{\mathrm{NR}}`, `\mathbf{\tau}^{\mathrm{GR}}`, `\mathbf{\tau}^{\mathrm{DS}}`, `\mathbf{\tau}^{\mathrm{DR}}`, `\mathbf{\tau}^{\mathrm{PS,in}}`, `\mathbf{\tau}^{\mathrm{PR,in}}`, `\mathbf{\tau}^{\mathrm{PR,out}}`, `\mathbf{h}`, `\mathbf{d}`, and `\hat{\mathbf{\tau}}^{A}` are defined similarly.

Coefficient matrices are constructed as:

```math
\mathbf{M}^{G}
=
\mathrm{diag}
\left\{
\mathrm{col}_{t}
\left[
\mathrm{col}_{g}
\left(
m_{g,t}^{G}
\right)
\right]
\right\}.
```

Matrices `\mathbf{M}^{D}` and `\mathbf{J}` are defined similarly. 

To describe the weights of mass flow in temperature mixing at nodes, the mass factor matrices are defined as:

```math
\mathbf{\Omega}^{\mathrm{PS}}
=
\mathrm{diag}
\left\{
\mathrm{col}_{t}
\left[
\mathrm{col}_{b}
\left(
\omega_{b,t}^{\mathrm{PS}}
\right)
\right]
\right\},
```

```math
\mathbf{\Omega}^{\mathrm{GS}}
=
\mathrm{diag}
\left\{
\mathrm{col}_{t}
\left[
\mathrm{col}_{n}
\left(
\omega_{n,t}^{\mathrm{GS}}
\right)
\right]
\right\}.
```

The elements `\omega_{b,t}^{\mathrm{PS}}` and `\omega_{g,t}^{\mathrm{GS}}` are:

```math
\omega_{b,t}^{\mathrm{PS}}
=
\frac{
m_{b,t}^{\mathrm{PS}}
}{
\sum_{b'\in\mathcal{P}_{i}^{-}}m_{b',t}^{\mathrm{PS}}
+
\sum_{g\in\mathcal{G}_{i}}m_{g,t}^{G}
},
```

where node `i` is the ending node of pipeline `b`, and

```math
\omega_{g,t}^{\mathrm{GS}}
=
\frac{
m_{g,t}^{G}
}{
\sum_{b\in\mathcal{P}_{i}^{-}}m_{b,t}^{\mathrm{PS}}
+
\sum_{g'\in\mathcal{G}_{i}}m_{g',t}^{G}
},
```

where node `i` is the node where source `g` is connected. Mass factor matrices for the return network `\mathbf{\Omega}^{\mathrm{PR}}` and `\mathbf{\Omega}^{\mathrm{DR}}` are defined similarly. These weighting matrices ensure that nodal mixing temperatures are calculated as mass-flow-weighted averages of incoming pipeline flows and connected source or load flows. To simply notations, the topology of return networks is assumed to be identical to that of supply networks. Therefore, `\mathbf A_{-}=max(-\mathbf A,0)` and `\mathbf A_{+}=max(\mathbf A,0)` represent downstream pipelines `\mathcal{P}_{i}^{+} (\mathcal{P}_{i}^{-})` and upstream pipelines `\mathcal{P}_{i}^{-} (\mathcal{P}_{i}^{+})` in supply (return) networks respectively. Asymmetric topology can be trivially handled by using two sets of incidence matrices for supply and return networks, respectively.

## Model Description

In this paper, the DHN operates under a constant mass flow and variable temperature mode. The mass flow rates of pipes are predetermined as constants, while the circulating water temperature varies over time.

The heat sources of the DHN consist of three components: heat from its own heat boilers, the thermal output of CHP units, and the recovered waste heat from DCs. The operation of heat boilers is regulated by the DHN, whereas the thermal output of CHP units and the recovered waste heat from DCs are scheduled by the CHP units and DCs, respectively. Therefore, `\mathbf{h}` includes `\mathbf{h}_{H}`, `\mathbf{h}_{g}`, and `\mathbf{h}_{re}`, where

```math
\mathbf{h}_{re}
=
\mathrm{col}_{t}
\left[
\mathrm{col}_{i}
\left(
h_{i,t}^{re}
\right)
\right].
```

The vector `\mathbf{x}_{H}` collects other optimization variables in the DHN, including `\mathbf{\tau}^{\mathrm{NS}}`, `\mathbf{\tau}^{\mathrm{NR}}`, `\mathbf{\tau}^{\mathrm{GS}}`, `\mathbf{\tau}^{\mathrm{GR}}`, `\mathbf{\tau}^{\mathrm{DS}}`, `\mathbf{\tau}^{\mathrm{DR}}`, `\mathbf{\tau}^{\mathrm{PS,in}}`, `\mathbf{\tau}^{\mathrm{PS,out}}`, `\mathbf{\tau}^{\mathrm{PR,in}}`, and `\mathbf{\tau}^{\mathrm{PR,out}}`.

For easier understanding, the structure of nodes and pipelines in a DHN is shown in the following picture:

<img src="file:///D:/study/2026_Energy%20Internet/Fig2structureofDHN.png" title="" alt="Structure of nodes and pipelines in a DHN" data-align="center">

## OTF Formulation

```math
\min_{\mathbf{h}_{H},\mathbf{x}_{H}}
\quad
\mathbf{c}_{H}^{T}\mathbf{h}_{H}+k^{ex}\sum_{t\in\mathcal{T}}\Delta ex^H_{t}
\qquad (83)
```

subject to

```math
\mathbf{h}
=
c\mathbf{M}^{G}
\left(
\mathbf{\tau}^{\mathrm{GS}}
-
\mathbf{\tau}^{\mathrm{GR}}
\right)
\qquad (84)
```

```math
\mathbf{d}
=
c\mathbf{M}^{D}
\left(
\mathbf{\tau}^{\mathrm{DS}}
-
\mathbf{\tau}^{\mathrm{DR}}
\right)
\qquad (85)
```

```math
\underline{\mathbf{h}}_{E}
\le
\mathbf{h}_{E}
\le
\overline{\mathbf{h}}_{E}
\qquad (86)
```

```math
\underline{\mathbf{h}}_{H}
\le
\mathbf{h}_{H}
\le
\overline{\mathbf{h}}_{H}
\qquad (87)
```

```math
\mathbf{\tau}^{\prime\mathrm{PS,out}}
=
\mathbf{K}\mathbf{\tau}^{\mathrm{PS,in}}
+
\hat{\mathbf{\tau}}^{\mathrm{PS,out}}
\qquad (88)
```

```math
\mathbf{\tau}^{\prime\mathrm{PR,out}}
=
\mathbf{K}\mathbf{\tau}^{\mathrm{PR,in}}
+
\hat{\mathbf{\tau}}^{\mathrm{PR,out}}
\qquad (89)
```

```math
\mathbf{\tau}^{\mathrm{PS,out}}
=
\hat{\mathbf{\tau}}^{A}
+
\mathbf{J}
\left(
\mathbf{\tau}^{\prime\mathrm{PS,out}}
-
\hat{\mathbf{\tau}}^{A}
\right)
\qquad (90)
```

```math
\mathbf{\tau}^{\mathrm{PR,out}}
=
\hat{\mathbf{\tau}}^{A}
+
\mathbf{J}
\left(
\mathbf{\tau}^{\prime\mathrm{PR,out}}
-
\hat{\mathbf{\tau}}^{A}
\right)
\qquad (91)
```

```math
\mathbf{A}_{-}
\otimes
\mathbf{I}_{T}
\cdot
\mathbf{\Omega}^{\mathrm{PS}}
\mathbf{\tau}^{\mathrm{PS,out}}
+
\mathbf{A}_{G}
\otimes
\mathbf{I}_{T}
\cdot
\mathbf{\Omega}^{\mathrm{GS}}
\mathbf{\tau}^{\mathrm{GS}}
=
\mathbf{\tau}^{\mathrm{NS}}
\qquad (92)
```

```math
\mathbf{A}_{+}
\otimes
\mathbf{I}_{T}
\cdot
\mathbf{\Omega}^{\mathrm{PR}}
\mathbf{\tau}^{\mathrm{PR,out}}
+
\mathbf{A}_{D}
\otimes
\mathbf{I}_{T}
\cdot
\mathbf{\Omega}^{\mathrm{DR}}
\mathbf{\tau}^{\mathrm{DR}}
=
\mathbf{\tau}^{\mathrm{NR}}
\qquad (93)
```

```math
\mathbf{A}_{+}^{T}
\otimes
\mathbf{I}_{T}
\cdot
\mathbf{\tau}^{\mathrm{NS}}
=
\mathbf{\tau}^{\mathrm{PS,in}}
\qquad (94)
```

```math
\mathbf{A}_{-}^{T}
\otimes
\mathbf{I}_{T}
\cdot
\mathbf{\tau}^{\mathrm{NR}}
=
\mathbf{\tau}^{\mathrm{PR,in}}
\qquad (95)
```

```math
\mathbf{A}_{D}^{T}
\otimes
\mathbf{I}_{T}
\cdot
\mathbf{\tau}^{\mathrm{NS}}
=
\mathbf{\tau}^{\mathrm{DS}}
\qquad (96)
```

```math
\mathbf{A}_{G}^{T}
\otimes
\mathbf{I}_{T}
\cdot
\mathbf{\tau}^{\mathrm{NR}}
=
\mathbf{\tau}^{\mathrm{GR}}
\qquad (97)
```

```math
\underline{\mathbf{x}}_{H}
\le
\mathbf{x}_{H}
\le
\overline{\mathbf{x}}_{H}
\qquad (98)
```

and constraints (29)-(45) in the main manuscript.

## Explanation

The objective function in (83) minimizes the total cost in the DHN, including the generation cost of internal DHN-owned heat sources and the exergy loss cost of the DHN. The water temperatures in heat sources and loads are described by constraints (84) and (85), where `c` denotes the specific heat capacity of water. Specifically, (84) calculates heat injection from the temperature difference between source supply and return water, while (85) calculates heat demand from the temperature difference between load supply and return water. Constraints (86) and (87) confine the heat output of EPN-owned and DHN-owned heat sources to the technical limit intervals, respectively.

According to the node method, which is widely applied to describe quasi-dynamics of pipelines, the outlet temperatures of pipelines without heat loss are first estimated via constraints (88) and (89) for the supply and return networks, respectively. In the second step of the node method, the outlet temperatures are modified to account for heat losses in constraints (90) and (91) for the supply and return networks, respectively.

Constraints (92) and (93) represent the temperature of mass-flow mixing at nodes in the supply and return networks, respectively. In (92), incoming supply-pipeline flows and connected heat-source flows are mixed to form the nodal supply temperature. In (93), incoming return-pipeline flows and connected load return flows are mixed to form the nodal return temperature.

Constraints (94) and (95) determine the inlet temperatures of pipelines using the nodal temperatures at the starting end in the supply and return networks, respectively. Similarly, inlet temperatures of loads and sources are calculated in constraints (96) and (97), respectively. Constraint (98) keeps the internal states of the DHN within operational limits. The phrase "and constraints (29)-(45)" indicates that the OTF model is used together with the energy and exergy constraints of DC waste-heat recovery and DHN exergy flow presented in the main manuscript.






