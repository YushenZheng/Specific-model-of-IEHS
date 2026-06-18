# Supplementary Model Formulations for DC-IEHS Dispatch

This repository provides the supplementary formulations of the integrated electricity-heat system (IEHS) optimization model used in the manuscript.

The model details are organized into three files:

- [Electric Power Network DCOPF Model](./EPN_model.md)
- [CHP Unit Operation Model](./CHP_model.md)
- [District Heating Network OTF Model](./DHN_model.md)

## GitHub Rendering Notes

The equations are written in GitHub-supported fenced math blocks, i.e., code fences marked with `math`. Inline mathematical symbols in the explanatory text are written as code-style notation to avoid GitHub's math-sanitizer issues with some LaTeX macros inside tables or paragraphs.

## Model Structure

| File           | Main content                                                                           |
| -------------- | -------------------------------------------------------------------------------------- |
| `EPN_model.md` | Direct-current optimal power flow (DCOPF) model of the electric power network (EPN)    |
| `CHP_model.md` | Operation model and feasible region formulation of combined heat and power (CHP) units |
| `DHN_model.md` | Operation thermal flow (OTF) model of the district heating network (DHN)               |

## Coupling Among Models

The EPN purchases electricity from CHP units, while the DHN receives heat from boilers, CHP units, and recovered DC waste heat. The CHP units therefore couple the electric and thermal subsystems through their joint electricity-heat output region.

The nodal carbon intensity used by DCs is calculated after solving the EPN dispatch problem according to the resulting generation dispatch and power-flow distribution. It is an accounting-based signal rather than a direct optimization variable.




