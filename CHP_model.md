# CHP Unit Operation Model

This file reproduces Appendix A.2 of the manuscript: the operation model of combined heat and power (CHP) units.

## Model Description

CHP units simultaneously produce electricity and heat. In the proposed IEHS dispatch framework, CHP units sell electricity to the EPN and provide heat to the DHN. Their feasible operation is represented by a polyhedral electricity-heat operating region.

## Notation

### Sets and Indices

| Symbol | Description |
|---|---|
| `G^CHP` | Set of CHP units |
| `T` | Set of dispatch periods |
| `g` | Index of a CHP unit |
| `t` | Index of a dispatch period |

### Superscripts and Subscripts

| Symbol | Description |
|---|---|
| `CHP` | CHP-unit-related quantity |
| `CHP,E` | Electricity-output-related CHP cost coefficient |
| `CHP,H` | Heat-output-related CHP cost coefficient |
| `CHP,sell` | Revenue or payment obtained by selling electricity to the EPN |
| `g,t` | Variable of CHP unit `g` at time period `t` |

### Variables and Operators

| Symbol | Description |
|---|---|
| `p_g` | Electricity output vector of CHP unit `g` over the scheduling horizon |
| `h_g` | Heat output vector of CHP unit `g` over the scheduling horizon |
| `p_g,t`, `h_g,t` | Electricity and heat outputs of CHP unit `g` at time `t` |
| `P_g`, `H_g` | Electricity and heat coordinates of extreme points in the CHP feasible operating region |
| `alpha_g,t` | Convex-combination coefficient vector for the CHP operating point |
| `1`, `0` | All-one and all-zero vectors with compatible dimensions |
## Formulation

```math
\min_{\mathbf{\alpha}_{g,t}}
\quad
\sum_{g\in\mathcal{G}^{\mathrm{CHP}}} C_g^{\mathrm{CHP}}
-
\sum_{g\in\mathcal{G}^{\mathrm{CHP}}} C_g^{\mathrm{CHP,sell}}
\qquad (79)
```

subject to

```math
C_g^{\mathrm{CHP}}
=
\mathbf{p}_{g}^{T}
\mathbf{Q}_{g}^{\mathrm{CHP,E}}
\mathbf{p}_{g}
+
\mathbf{c}_{g}^{\mathrm{CHP,E}}
\mathbf{p}_{g}
+
\mathbf{p}_{g}^{T}
\mathbf{Q}_{g}^{\mathrm{CHP}}
\mathbf{h}_{g}
```

```math
\quad
+
\mathbf{h}_{g}^{T}
\mathbf{Q}_{g}^{\mathrm{CHP,H}}
\mathbf{h}_{g}
+
\mathbf{c}_{g}^{\mathrm{CHP,H}}
\mathbf{h}_{g}
+
c_g^{\mathrm{CHP,0}},
\quad
\forall g\in\mathcal{G}^{\mathrm{CHP}}
\qquad (80)
```

```math
p_{g,t}
=
\mathbf{\alpha}_{g,t}^{T}\mathbf{P}_{g},
\quad
h_{g,t}
=
\mathbf{\alpha}_{g,t}^{T}\mathbf{H}_{g},
```

```math
\mathbf{1}^{T}\mathbf{\alpha}_{g,t}
=
1,
\quad
\mathbf{0}
\le
\mathbf{\alpha}_{g,t}
\le
\mathbf{1},
\quad
\forall g\in\mathcal{G}^{\mathrm{CHP}},
\ t\in\mathcal{T}
\qquad (81)
```

## Explanation

The objective function (79) minimizes the generation cost of CHP units and maximizes the profit from selling electricity to the EPN. Matrices `\mathbf{Q}_g^{\mathrm{CHP,E}}`, `\mathbf{Q}_g^{\mathrm{CHP}}`, and `\mathbf{Q}_g^{\mathrm{CHP,H}}`, together with vectors `\mathbf{c}_g^{\mathrm{CHP,E}}`, `\mathbf{c}_g^{\mathrm{CHP,H}}`, and scalar `c_g^{\mathrm{CHP,0}}`, are the cost coefficients of CHP units.

Constraint (80) specifies the generation cost of CHP units. The first quadratic term represents the cost associated with electricity output, the second quadratic term represents the cost associated with heat output, and the cross term captures the coupling between electricity and heat production.

In (81), the electricity and heat generation of CHP units are formulated by a linear combination of extreme points in the polyhedral operating region. The constraint `\mathbf{1}^{T}\mathbf{\alpha}_{g,t}=1` and the bound `\mathbf{0}\le\mathbf{\alpha}_{g,t}\le\mathbf{1}` ensure that the operating point is a convex combination of feasible extreme points. Therefore, the generated pair `(p_{g,t},h_{g,t})` remains inside the CHP feasible operation region.





