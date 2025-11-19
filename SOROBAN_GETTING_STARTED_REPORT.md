# Soroban Getting Started Tutorial – Comprehensive Field Report

> **Scope**  
> Hands-on execution of every instruction in `docs/build/smart-contracts/getting-started`, behaving strictly as a first-time Soroban developer with no external knowledge. This document records the full timeline, command outcomes, observations, gaps, and recommendations.

---

## 1. Environment & Methodology

- **Host details**: Ubuntu (kernel `linux 6.1.147`), repo root `/workspace`, git branch `gpt-5_1-codex`.
- **Shell state management**: Single persistent Bash shell per instructions. All commands executed from `/workspace` unless stated.
- **Tooling constraints**: Only documentation from `docs/build/smart-contracts/getting-started/**`. No prior Stellar/Soroban knowledge applied.
- **Verification approach**:
  - Execute commands verbatim; if interactive, capture prompts/outcomes.
  - Record exit codes, key log lines, produced artifacts.
  - Stop and note when instructions lack information required to proceed.
  - Track alignment between docs and actual CLI behavior (versions, warnings).

---

## 2. Execution Timeline

### 2.1 Setup (`setup.mdx`)

| Step | Command(s) Run | Result | Notes |
| --- | --- | --- | --- |
| Install Rust via rustup | `curl ... | sh` | **Failed initially** – installer refused non-interactive mode | Docs omit `-y`; reran after user guidance not provided. Eventually succeeded by installing later via updated rustup pipeline after Homebrew step. |
| Update toolchain | `rustup update stable`, `rustup default stable`, `rustc --version` | **Success** – Rust `1.91.1` active | Required to gain `wasm32v1-none` support (doc mandates Rust ≥1.84). |
| Add wasm target | `rustup target add wasm32v1-none` | **Success** | Previously failed on Rust `1.82.0`; after upgrade, installed smoothly. |
| Install Stellar CLI | `/bin/bash -c "$(curl ... brew install)"`, `brew install stellar-cli`, `stellar --version` | **Success after installing Homebrew** | Docs only mention `brew install stellar-cli`; had to deduce Homebrew installation path. |
| Editor tooling | Not explicitly verifiable | Not attempted | Instructions describe VS Code plugins; not enacted due to environment scope. |
| Autocompletion | `source <(stellar completion --shell bash)` and appended to `~/.bashrc` | **Success** | Matches doc suggestion for bash sessions. |

### 2.2 Hello World (`hello-world.mdx`)

| Step | Command(s) | Outcome | Evidence |
| --- | --- | --- | --- |
| Initialize project | `stellar contract init soroban-hello-world` | **Success** | Workspace with `contracts/hello-world` scaffold created. |
| Inspect structure | (Implicit via file reads) | **Aligned** | Files match documentation’s depiction. |
| Run tests | `cd soroban-hello-world && cargo test` | **Success** | Unit test `test::test` passed. |
| Build contract | `stellar contract build` | **Success** | Produced `hello_world.wasm` (620 bytes, hash `fc704269...`). |
| Optional optimize | `stellar contract optimize ...` | **Success w/ warning** | CLI warns command is deprecated → doc outdated. |

### 2.3 Deploy to Testnet (`deploy-to-testnet.mdx`)

| Step | Command(s) | Outcome | Notes |
| --- | --- | --- |
| Generate keypair | `stellar keys generate alice --network testnet --fund` | **Success** | Friendbot funded account. |
| Inspect keys | `stellar keys address alice`, `stellar keys ls -l` | **Success** | Confirmed alias + file location. |
| Deploy Hello World | `stellar contract deploy --wasm ... --source-account alice --network testnet --alias hello_world` | **Success** | Created contract `CDM4QYCCFZAFKVAO...`. CLI output included install + deploy tx hashes. |
| Invoke contract | `stellar contract invoke --id <id> --source-account alice --network testnet -- hello --to RPC` | **Success** | Response `["Hello","RPC"]`. CLI indicated read-only simulation. |

### 2.4 Storing Data (`storing-data.mdx`)

| Step | Command(s) | Outcome | Observations |
| --- | --- | --- | --- |
| Add `increment` contract | `stellar contract init . --name increment` | **Success** | Created new crate under `contracts/increment`. |
| Replace code | Updated `src/lib.rs` and `src/test.rs` per doc | **Success** | Implemented counter logic + tests. |
| Build workspace | `stellar contract build` | **Success** | Output included `increment.wasm` (641 bytes). |
| Verify artifacts | `ls target/wasm32v1-none/release/*.wasm` | **Success** | Both hello_world + increment .wasm present. |
| Run tests | `cargo test` and `cargo test -- --nocapture` | **Success** | Logged diagnostic events showing counter increments 0→2. |

### 2.5 Deploy Increment Contract (`deploy-increment-contract.mdx`)

| Step | Command(s) | Outcome | Notes |
| --- | --- | --- | --- |
| Upload WASM | `stellar contract upload --network testnet --source-account alice --wasm increment.wasm` | **Success** | Returned hash `95baf5fbc...`. |
| Deploy via hash | `stellar contract deploy --wasm-hash ... --alias increment` | **Success** | Instantiated contract `CDBLELTO5QQ3BZOA...`. |
| Invoke increment | `stellar contract invoke ... -- increment` (x3) | **Success** | Outputs `1`, `2`, `3` on consecutive runs, confirming stateful counter. |

### 2.6 Hello World Frontend (`hello-world-frontend.mdx`)

| Step | Command(s) | Outcome | Remarks |
| --- | --- | --- | --- |
| Verify Node.js | `node --version` | `v22.21.1` | Meets doc requirement (≥20). |
| Create Astro app | `npm create astro@latest hello-world-frontend -- --template basics --install --no-git --yes` | **Success** | Needed non-interactive flags (docs omit). |
| Generate bindings | `stellar contract bindings typescript --network testnet --contract-id hello_world --output-dir packages/hello_world` | **Success** | CLI pulled contract spec from Testnet. |
| Build bindings | `npm install && npm run build` inside `packages/hello_world` | **Success** | TypeScript compiled. |
| Update frontend page | Replaced `src/pages/index.astro` with doc snippet | **Success** | Imports bindings, calls contract. |
| Run dev server | `npm run dev` + `curl http://localhost:4321` | **Success** | Received `Hello Devs!` HTML; dev server logs captured. |

### 2.7 Scaffold Stellar (Doc Addendum)

| Step | Command | Result | Gap |
| --- | --- | --- | --- |
| Install scaffold CLI | `cargo binstall stellar-scaffold-cli` | **Failed** (`error: no such command: binstall`) | Docs assume `cargo-binstall` exists; not installed by default. No alternative instructions provided. |

---

## 3. Detailed Observations & Evidence

1. **Homebrew prerequisite** – Linux instructions only show `brew install stellar-cli`. Installing Homebrew required running the script manually, adding PATH lines, and rerunning the shell. Without these actions, `brew` commands fail (`command not found`). Documentation should acknowledge this initial requirement explicitly.
2. **Rust toolchain version** – Default toolchain was `1.82.0`, which cannot add `wasm32v1-none`. The docs state “For Rust v1.84.0 or higher…” but offer no steps for upgrading. Running `rustup update stable` resolved this; the doc should mention the upgrade command explicitly.
3. **`stellar contract optimize` warning** – CLI output: “`stellar contract optimize` is deprecated and will be removed... use `stellar contract build --optimize`.” Tutorial still promotes deprecated command.
4. **CLI deployment output** – Both deploy commands print simulation hashes and final explorer URLs, matching expectations. Useful detail for doc to highlight: `stellar contract deploy` automatically uploads when passed `--wasm`.
5. **Increment TTL values** – Code snippet uses `extend_ttl(50, 100)` while the narrative later talks about extending by 100 ledgers; potential mismatch.
6. **Front-end scaffolding** – Without the `--template`, `--yes`, `--no-git`, `--install` flags, `npm create astro@latest` halts awaiting user input. Tutorial assumes interactive answers but never lists them, forcing guesswork. The final command used replicates the sample project described earlier.
7. **Dev server verification** – `curl http://localhost:4321` returned an HTML snippet containing `<h1>Hello Devs!</h1>`, confirming the binding invocation succeeded and that RPC data flowed correctly.
8. **Scaffold Stellar** – Attempted command produced:  
   ```
   error: no such command: `binstall`
   help: a command with a similar name exists: `install`
   ```  
   Without `cargo-binstall`, the instructions dead-end. No mention of fallback `cargo install --locked stellar-scaffold-cli`.

---

## 4. Code & Artifact Inventory

| Artifact | Path | Purpose |
| --- | --- | --- |
| Hello World contract | `soroban-hello-world/contracts/hello-world/src/lib.rs` | Generated by CLI; unchanged except for build/test operations. |
| Increment contract implementation | `soroban-hello-world/contracts/increment/src/lib.rs` | Matches tutorial’s storage example; uses `symbol_short!`, `Env.storage().instance()`, TTL extension. |
| Increment tests | `soroban-hello-world/contracts/increment/src/test.rs` | Exercises `increment()` thrice, asserting returned counts. |
| WASM outputs | `target/wasm32v1-none/release/hello_world(.optimized).wasm`, `increment.wasm` | Used for deployment + CLI bindings. |
| Bindings package | `hello-world-frontend/packages/hello_world` (built via npm) | Contains TypeScript client for the deployed Hello World contract. |
| Frontend page | `hello-world-frontend/src/pages/index.astro` | Imports bindings, calls `hello`, renders greeting. |

---

## 5. Documentation Gaps & Recommendations

| # | Area | Gap Description | Impact | Recommendation |
| --- | --- | --- | --- | --- |
| 1 | Setup – CLI install | Linux section assumes Homebrew exists; no install steps or alternative channel. | Blocks entire tutorial for fresh Linux users. | Provide explicit Homebrew install instructions (script + PATH updates) or list other install methods (binary tarball, cargo command with version). |
| 2 | Setup – Rust install/update | `curl ... | sh` fails non-interactively; target instructions require ≥1.84 but upgrade path unspecified. | Users stuck with old toolchain cannot add wasm target. | Document `curl ... | sh -s -- -y` for unattended setups and add “Update Rust” step: `rustup update stable` + `rustup default stable`. |
| 3 | Build section | `stellar contract optimize` command deprecated. | Following tutorial produces warning; future removal will break step. | Update docs to use `stellar contract build --optimize` and explain difference vs. `stellar contract build`. |
| 4 | Deploy recap | `deploy-to-testnet.mdx` claims CLI configuration already done, but Setup never described RPC config aside from install. | Novices might not know how to set network defaults or RPC URLs. | Add explicit instructions for `stellar config network add ...` and explain how `--network testnet` works. |
| 5 | Storing Data explanation | TTL narrative references `(100, 100)` after showing `(50, 100)`. | Creates confusion about recommended TTL parameters. | Align code and prose or explain rationale for two values. |
| 6 | Frontend scaffolding | `npm create astro@latest` prompts undocumented; tutorial appears deterministic but isn’t. | Non-interactive environments hang; novices unsure what to choose. | Provide the exact command with `--template`, `--install`, `--yes`, `--no-git`, or list prompt answers. |
| 7 | Bindings prerequisites | Step assumes contract alias `hello_world` exists; not restated. | Readers who failed earlier stage won’t know to swap actual contract ID. | Add reminder to deploy with `--alias hello_world` or show `--contract-id <actual-id>`. |
| 8 | Scaffold Stellar | Requires `cargo-binstall`, which isn’t included or mentioned. | Step impossible on fresh machines. | Add prerequisite instruction (`cargo install cargo-binstall`) or present `cargo install --locked stellar-scaffold-cli` alternative. |
| 9 | General troubleshooting | Docs assume perfect execution; no guidance when commands fail (e.g., missing brew, outdated Rust). | Users may abandon tutorial on first error. | Add sidebar with common issues and fixes (Homebrew missing, wasm target errors, CLI not on PATH). |

---

## 6. Final Assessment

- **Completeness**: All primary tutorial flows (Setup, Hello World, Deploy, Storing Data, Deploy Increment, Frontend) were executed successfully after resolving undocumented prerequisites. The optional Scaffold Stellar section remains blocked due to missing tooling instructions.
- **First-time developer viability**: Moderate. With the documentation as written, a true newcomer is likely to stall during Setup (Rust and CLI installation) or Astro project creation. Additional friction arises from deprecated commands and unstated alias requirements.
- **Critical blockers**:
  1. Absence of Linux-specific CLI installation guidance (no Homebrew instructions).
  2. No mention of upgrading Rust or running the installer non-interactively.
  3. Scaffold Stellar instructions referencing `cargo binstall` without detailing how to get it.
- **High-impact improvements**:
  - Provide turnkey scripts or detailed steps for installing prerequisites (Homebrew, rustup upgrade, CLI configuration).
  - Refresh build/deploy instructions to match current CLI behavior (e.g., `stellar contract build --optimize`).
  - Document interactive tool prompts (Astro scaffolding) and include fallback commands for users who already have deployed contracts without aliases.
  - Expand troubleshooting/FAQ sections to cover common CLI/toolchain errors encountered in this run.

With these changes, the Getting Started path would be significantly more approachable for new developers and more resilient to environment variability.
