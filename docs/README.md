# Documentation

| Folder | Contents |
|---|---|
| [`specs/`](specs/) | T1 specifications: interfaces between components, such as the [faceplate interface](specs/faceplate-interface.md), and the T1's own behaviour and values, such as [controls and the status LED](specs/controls-and-status-led.md) the [experience budgets](specs/experience-budgets.md), and the T1 [electronics integration and verification](specs/electronics-integration-verification.md) (its modules, interaction matrix and evaluation rows). The methods every OpenDAP device shares (the [integration and verification method](https://github.com/opendap-project/toolkit-hardware/blob/main/docs/specs/integration-and-verification.md), the [PCB design constraints method](https://github.com/opendap-project/toolkit-hardware/blob/main/docs/specs/pcb-design-constraints-method.md)) are in toolkit-hardware. Generic software specifications (audio profiles, devices and sync) are in [system](https://github.com/opendap-project/system/tree/main/docs/specs), and the interface mockups in [blackbird-ui](https://github.com/opendap-project/blackbird-ui/tree/main/renders/web) (RFC 0008). |
| [`plans/`](plans/) | Work plans with their task lists, such as the [electronics plan](plans/2026-10-electronics-plan.md), which applies the [hardware engineering system plan](https://github.com/opendap-project/toolkit-hardware/blob/main/docs/plans/2026-10-hardware-engineering-system-plan.md). |
| [`adr/`](adr/) | Architecture Decision Records: decisions already taken. |
| [`research/`](research/) | Research that informs requirements and decisions. T1 applications of shared research name their source, such as the [design for compliance](research/2026-10-design-for-compliance.md) on the [compliance regime catalog](https://github.com/opendap-project/toolkit-hardware/blob/main/docs/research/2026-10-compliance-regimes.md). |
| [`requirements/`](requirements/) | Product requirements (PRD). Each requirement has an ID, a priority (**must** / **should**) and a status. |

Only information confirmed by the maintainers is recorded here. Earlier experiments are not a source.
