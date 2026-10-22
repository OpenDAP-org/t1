# T1

Product integration of the OpenDAP T1: product requirements, the complete assembly and system simulations.

Part of [OpenDAP](https://github.com/opendap-project). This repository is open source.

## Working here

- Write everything in English: code, docs, commit messages.
- Record only information the maintainers confirmed. Do not infer specs, numbers or wording.
- Product requirements are in `docs/requirements/`. Update them with the `/opendap:requirement` skill; write an ADR (`/opendap:adr`) only for decisions taken.
- Build the assembly and simulations only from components' `export/` folders, pinned in `components.yaml`, following the [component export contract](https://github.com/opendap-project/toolkit-hardware/blob/main/docs/specs/component-export.md). Never copy component sources here.
- Licensing follows REUSE (`REUSE.toml`, `LICENSES/`). After adding files or folders, run the `/opendap:license-check` skill.
