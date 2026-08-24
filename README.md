# workflow

CI and reproducible verification workflows for the Windows CE 6.0 toolchain projects.

## Repository roles

- `kagurasumusun/wince-build` — Platform Builder-independent CE 6.0 build platform.
- `kagurasumusun/wince-source` — complete CE 6.0 source input.
- `kagurasumusun/winde` — standalone CE development IDE.
- `kagurasumusun/workflow` — CI/verification orchestration only.

## Current verification model

The workflows must test the current repository architecture rather than historical prototype paths.

```text
wince-source/ce600
       +
       │ source input
       ▼
wince-build
       │
       ├─ configuration / feature model
       ├─ NMAKE-compatible build frontend
       ├─ deterministic dependency graph
       └─ Ninja execution
       │
       ▼
compiler / assembler / resource / linker
       │
       ▼
SYSGEN → REG / DAT / BIB → ROMIMAGE → NK.BIN
       │
       ▼
structural and reproducibility verification
```

## Workflow policy

1. Do not reference the removed `engine/` directory.
2. Do not rely on prototype-specific executable locations.
3. Do not use a GitHub PAT when public repository access is sufficient.
4. Do not download or reconstruct the CE source from installation media in the build-platform workflows; use `kagurasumusun/wince-source` as the source boundary.
5. Do not silently accept partial image generation as a successful complete CE build.
6. A workflow may use GitHub Actions for verification, but the repository itself must remain buildable outside CI using the documented local commands.
7. Do not pin the project to an old CeGCC fork merely because an older workflow happened to use it. Toolchain selection is an engineering decision based on real CE source compatibility, target ABI, linker behavior and reproducibility.
8. Keep workflow diagnostics tied to explicit artifacts and recorded source/build revisions.

## Workflows

### `wince-build.yml`

Cross-host configure/build/test verification on Linux, macOS and Windows. It also checks that the professional repository boundaries are present and that stale `engine/` references have not returned.

### `full-ce6-build.yml`

Manual real CE 6.0 build verification. It consumes the existing `wince-source/ce600` tree and invokes the unified build pipeline through configuration, build graph generation and image generation.

### `full-os-build-as-test.yml`

Manual end-to-end OS build verification. It requires the real pipeline artifacts and `NK.BIN`; it does not treat a partial `--no-rom` build as completion.

## Toolchain evaluation

`salman-javed-nz/cegcc-build` is a candidate modern CeGCC baseline. The workflow repository must not silently switch to it or another fork. Candidate toolchains should be evaluated against the real CE 6.0 corpus for:

- ARM/WinCE PE-COFF correctness
- ARMv5/ARM926EJ-S and ARMEL/soft-float requirements where applicable
- C/C++ ABI compatibility
- assembler and linker behavior
- imports/exports
- resource compilation
- runtime support
- full-tree build closure
- reproducibility

The current workflow architecture deliberately leaves toolchain provisioning to the build platform's supported configuration instead of hard-coding a historical compiler archive into every workflow.

## CI is not the implementation

A green workflow is evidence for the exact commands it executed. It is not permission to weaken the build platform, add compatibility hacks solely for CI, or claim that an unimplemented subsystem is complete.

The authoritative engineering requirements live in `kagurasumusun/wince-build/docs/HANDOFF.md`.
