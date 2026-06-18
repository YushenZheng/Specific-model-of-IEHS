# Electric Power Network DCOPF Model

This file introduces the elaborate form of the optimization model in the electric power network (EPN).

## Model Description

The EPN is modeled by a direct current optimal power flow (DCOPF) formulation. The EPN operator schedules conventional thermal units, wind farms, and electricity purchased from CHP units to meet the nodal power demand while satisfying network security constraints. Since CHP units are operated by independent entities, the EPN purchases electricity from CHP units according to the electricity purchase price determined by the EPN operator.

## Notation

### Sets and Indices

| Symbol | Description |
|---|---|
| `T` | Set of dispatch periods, indexed by `t` |
| `G^TU` | Set of conventional thermal units |
| `G^WF` | Set of wind farms |
| `G^CHP` | Set of CHP units connected to the EPN |
| `g` | Index of a generation unit, wind farm, or CHP unit |
| `L` | Set or vector dimension associated with transmission branches |

### Superscripts and Subscripts

| Symbol | Description |
|---|---|
| `TU` | Thermal-unit-related quantity |
| `WF` | Wind-farm-related quantity |
| `CHP` | CHP-unit-related quantity |
| `CHP,sell` | Revenue or payment associated with electricity sold by CHP units to the EPN |
| `E` | EPN-side boundary signal or price |
| `D` | Electricity demand vector |
| `G` | Stacked generation vector |
| `car,bus` | Nodal carbon intensity at EPN buses |
| `car,G` | Carbon intensity of generation sources |
| `ref` | Reference bus used in the DC power-flow model |

### Operators and Matrix Notation

| Symbol | Description |
|---|---|
| `p_g` | Bold lowercase symbols denote vectors |
| `B` | Bold uppercase symbols denote matrices |
| `underline(.)`, `overline(.)` | Lower and upper bounds |
| `col(.)` | Column-vector concatenation |
| `odot` | Hadamard product, i.e., element-wise product |
| `1`, `0` | All-one and all-zero vectors with compatible dimensions |
| `Gamma^G` | Node-generator incidence matrix |
| `GSF` | Generation shift factor matrix |
## Variable Definitions and Explanation

The vector `\mathbf{p}_g` is formed by stacking `p_{g,t}` over the time horizon, i.e., 

```math
\mathbf{p}_g
=
\mathrm{col}_{t\in\mathcal{T}}
\left(
p_{g,t}
\right).
```

The vector `\mathbf{p}_{t}^{G}` is obtained by stacking the generation outputs of all units at time `t`:

```math
\mathbf{p}_{t}^{G}
=
\mathrm{col}_{g\in\mathcal{G}^{\mathrm{TU}}\cup\mathcal{G}^{\mathrm{CHP}}\cup\mathcal{G}^{\mathrm{WF}}}
\left(
p_{g,t}
\right).
```

The reserve vectors `\mathbf{rd}_g`, `\mathbf{ru}_g`, `\mathbf{rd}_t`, and `\mathbf{ru}_t` are defined similarly. The vector `\mathbf{x}_{E}` contains most of the optimization variables of the EPN, including `\mathbf{p}_g`, `\mathbf{\theta}_t`, `\mathbf{rd}_g`, and `\mathbf{ru}_g`. The electricity purchase price `\mathbf{c}_{E}` is passed to other agents as a boundary variable. To distinguish it from the internal decision variables, it is not included in `\mathbf{x}_{E}`.

The objective function (66) minimizes the total cost in the EPN. The first two terms are the thermal-unit generation cost and the wind-curtailment penalty cost, which are defined in (67) and (68), respectively. The third term is the cost of electricity purchased from CHP units, as shown in (69). Constraint (70) gives the range of the CHP electricity purchase price. Constraints (71) and (72) are the generation limits of thermal units and wind farms.

Constraint (73) consists of three reserve-related requirements. The first inequality reserves upward and downward regulation margins within the output range of each thermal unit. The second inequality requires the aggregated upward and downward reserves to satisfy system reserve demands. The third inequality limits the reserve capacities offered by each thermal unit. Constraint (74) ensures nodal power balance, where `\mathbf{B}` is the susceptance matrix and `\mathbf{\Gamma}^{G}` is the node-generator incidence matrix. Constraint (75) represents the branch transfer limit, where `\overline{\mathbf{P}}_L` is the line capacity limit. Constraint (76) specifies the inter-temporal ramping limit of the aggregated generation vector. Constraint (77) fixes the reference bus angle to remove the degree of freedom in DC power flow. Constraint (78) gives the average nodal carbon-emission-intensity calculation.

## DCOPF Formulation

```math
\min_{\mathbf{x}_{E},\mathbf{c}_{E}}
\quad
\sum_{g\in\mathcal{G}^{\mathrm{TU}}} C_g^{\mathrm{TU}}
+
\sum_{g\in\mathcal{G}^{\mathrm{WF}}} C_g^{\mathrm{WF}}
+
\sum_{g\in\mathcal{G}^{\mathrm{CHP}}} C_g^{\mathrm{CHP,sell}}
\qquad (66)
```

subject to

```math
C_g^{\mathrm{TU}}
=
\mathbf{p}_g^{T}\mathbf{Q}_g^{\mathrm{TU}}\mathbf{p}_g
+
\mathbf{c}_g^{\mathrm{TU}}\mathbf{p}_g
+
c_g^{\mathrm{TU,0}},
\quad
\forall g\in\mathcal{G}^{\mathrm{TU}}
\qquad (67)
```

```math
C_g^{\mathrm{WF}}
=
\mathbf{\rho}_g^{T}
\left(
\overline{\mathbf{p}}_g-\mathbf{p}_g
\right),
\quad
\forall g\in\mathcal{G}^{\mathrm{WF}}
\qquad (68)
```

```math
C_g^{\mathrm{CHP,sell}}
=
\mathbf{c}_{E}^{T}\mathbf{p}_g,
\quad
\forall g\in\mathcal{G}^{\mathrm{CHP}}
\qquad (69)
```

```math
\underline{\mathbf{c}}_{E}
\le
\mathbf{c}_{E}
\le
\overline{\mathbf{c}}_{E}
\qquad (70)
```

```math
\underline{\mathbf{P}}_{g}
\le
\mathbf{p}_{g}
\le
\overline{\mathbf{P}}_{g},
\quad
\forall g\in\mathcal{G}^{\mathrm{TU}}
\qquad (71)
```

```math
\mathbf{0}
\le
\mathbf{p}_{g}
\le
\overline{\mathbf{P}}_{g},
\quad
\forall g\in\mathcal{G}^{\mathrm{WF}}
\qquad (72)
```

```math
\underline{\mathbf{P}}_{g}
+
\mathbf{rd}_{g}
\le
\mathbf{p}_{g}
\le
\overline{\mathbf{P}}_{g}
-
\mathbf{ru}_{g},
\quad
\forall g\in\mathcal{G}^{\mathrm{TU}}
```

```math
\mathbf{1}^{T}\mathbf{ru}_{t}
\ge
SR_t^{\mathrm{up}},
\quad
\mathbf{1}^{T}\mathbf{rd}_{t}
\ge
SR_t^{\mathrm{down}},
\quad
\forall t\in\mathcal{T}
```

```math
\mathbf{0}
\le
\mathbf{ru}_{g}
\le
\overline{\mathbf{ru}}_{g},
\quad
\mathbf{0}
\le
\mathbf{rd}_{g}
\le
\overline{\mathbf{rd}}_{g},
\quad
\forall g\in\mathcal{G}^{\mathrm{TU}}
\qquad (73)
```

```math
\mathbf{B}\mathbf{\theta}_{t}
=
\mathbf{\Gamma}^{G}\mathbf{p}_{t}^{G}
-
\mathbf{p}_{t}^{D},
\quad
\forall t\in\mathcal{T}
\qquad (74)
```

```math
-
\overline{\mathbf{P}}_{L}
\le
\mathbf{GSF}
\cdot
\left(
\mathbf{\Gamma}^{G}\mathbf{p}_{t}^{G}
-
\mathbf{p}_{t}^{D}
\right)
\le
\overline{\mathbf{P}}_{L},
\quad
\forall t\in\mathcal{T}
\qquad (75)
```

```math
-
\mathbf{RD}
\le
\mathbf{p}_{t}^{G}
-
\mathbf{p}_{t-1}^{G}
\le
\mathbf{RU},
\quad
\forall t\in\mathcal{T}
\qquad (76)
```

```math
\theta_{\mathrm{ref}}
=
0
\qquad (77)
```

## Note on Carbon Intensity

The nodal carbon intensity `\mathbf{e}_{t}^{\mathrm{car,bus}}` is not an independent dispatch variable. It is calculated from the solved EPN dispatch and power-flow results and then sent to DCs as an accounting-based carbon-emission signal:

```math
\mathbf{1}^{T}
\left(
\mathbf{e}_{t}^{\mathrm{car,bus}}
\odot
\mathbf{p}_{t}^{D,*}
\right)
=
\mathbf{1}^{T}
\left(
\mathbf{e}_{t}^{\mathrm{car,G}}
\odot
\mathbf{p}_{t}^{G,*}
\right)
\qquad (78)
```





