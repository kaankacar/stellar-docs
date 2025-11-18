# Soroban "Getting Started" Documentation: New Developer Experience Report

This report documents the end-to-end experience of a new developer following the Soroban "getting-started" documentation. The process was followed exactly as written, without using any prior knowledge.

## A. Execution Log

This is a chronological log of the steps taken, what worked, what failed, and where the process was blocked.

1.  **Environment Setup (`setup.mdx`)**
    *   **Instruction:** Install Rust using `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`.
    *   **Outcome:** **Success.** Rust installed without any issues.

2.  **Install Wasm Target**
    *   **Instruction:** Install the `wasm32v1-none` target with `rustup target add wasm32v1-none`.
    *   **Outcome:** **Success.** The target was added to the default Rust toolchain.

3.  **Install Stellar CLI (Attempt 1: Homebrew)**
    *   **Instruction:** Install with Homebrew (`brew install stellar-cli`).
    *   **Outcome:** **Failure.** The command failed because Homebrew was not installed. The documentation did not provide instructions or a link to install it. This was the first major blocker.

4.  **Install Stellar CLI (Attempt 2: Cargo)**
    *   **Instruction:** The documentation mentioned installing from source with `cargo` but did not provide the command. I had to search online and found `cargo install --locked stellar-cli`.
    *   **Outcome:** **Failure.** The command consistently timed out after several minutes. This was a critical blocker that made it impossible to proceed with the tutorial.

5.  **User Intervention & Successful Installation**
    *   **Instruction:** The user provided a command to install Homebrew, unblocking the process.
    *   **Outcome:** **Success.** After installing Homebrew, `brew install stellar-cli` worked perfectly.

6.  **"Hello World" Tutorial (`hello-world.mdx`)**
    *   **Instruction:** Create a new project with `stellar contract init soroban-hello-world`.
    *   **Outcome:** **Success.** The project was created as expected.
    *   **Instruction:** Build the contract with `stellar contract build`.
    *   **Outcome:** **Failure.** The build failed with an error indicating an incompatible Rust version (1.91.0). The documentation had not specified any version constraints.
    *   **Troubleshooting & Second Failure:** I installed Rust `1.90.0` and set it as an override. The build failed again, this time because the `wasm32v1-none` target was not installed for the *new* toolchain. This nuance was not mentioned in the documentation.
    *   **Final Attempt:** After installing the wasm target for the `1.90.0` toolchain, the build finally succeeded.
    *   **Instruction:** Optimize the contract with `stellar contract optimize`.
    *   **Outcome:** **Success, but with a deprecation warning.** The documented command is outdated.

7.  **"Deploy to Testnet" & "Storing Data" Tutorials**
    *   **Instruction:** All subsequent steps, including deploying contracts and adding the `increment` contract, were followed.
    *   **Outcome:** **Success.** These parts of the tutorial were clear and all commands worked as expected.

---

## B. Documentation Gaps Report

This section details the specific gaps and issues found in the documentation.

1.  **Missing Prerequisite: Homebrew Installation**
    *   **Issue:** The primary installation method for macOS/Linux relies on Homebrew, but there are no instructions for installing it.
    *   **Impact:** Critical blocker for new developers who don't have Homebrew.
    *   **Recommendation:** Add the official Homebrew installation command or a link to their website in the `setup.mdx` file.

2.  **Incomplete and Unreliable `cargo` Installation Method**
    *   **Issue:** The `cargo install` command is not explicitly provided and is prone to timeouts.
    *   **Impact:** Critical blocker. A new developer would be completely stuck here.
    *   **Recommendation:** Either remove the `cargo install` option or provide a warning about the long compilation time and potential for timeouts, along with the correct, versioned command: `cargo install --locked stellar-cli@<version>`. The Homebrew method should be listed as the primary, recommended option.

3.  **Missing Rust Version Compatibility**
    *   **Issue:** The documentation does not specify which versions of Rust are compatible with the Soroban SDK. This led to a build failure that a new developer would struggle to diagnose.
    *   **Impact:** High-friction, confusing blocker.
    *   **Recommendation:** Add a section specifying the supported range of Rust versions. Also, provide a brief guide on how to manage Rust versions with `rustup`, as this is a common task.

4.  **Incomplete `wasm32` Target Instructions**
    *   **Issue:** The documentation does not explain that the `wasm32v1-none` target must be installed for *each* Rust toolchain in use.
    *   **Impact:** This can cause confusing build failures if a developer needs to switch Rust versions.
    *   **Recommendation:** Add a note to the build section: "If you have switched Rust toolchains, make sure you have installed the wasm target for it: `rustup target add wasm32v1-none --toolchain <your-version>`."

5.  **Outdated `stellar contract optimize` Command**
    *   **Issue:** The `hello-world.mdx` file instructs the use of a deprecated command.
    *   **Impact:** Minor, but creates a poor impression and can cause confusion.
    *   **Recommendation:** Update the documentation to use the modern command: `stellar contract build --optimize`.

---

## C. Final Assessment

**1. Can a first-time developer successfully follow this tutorial end-to-end?**

**No.** A first-time developer is highly likely to fail. The installation process for the `stellar-cli` is the most significant barrier. Without external help or prior knowledge of Homebrew and Rust toolchain management, a new user would be permanently blocked and unable to even start the "Hello World" tutorial.

**2. What are the critical blockers?**

*   **Stellar CLI Installation:** The `cargo install` method is functionally broken due to timeouts, and the recommended `brew` method is missing its own installation instructions. This is the single biggest blocker.
*   **Rust Version Incompatibility:** The build failure due to an unsupported Rust version is a hard stop that would be very difficult for a newcomer to solve without searching for external help.

**3. What are the highest-impact improvements for DevRel?**

*   **Fix the Installation Flow:** Make the `brew install stellar-cli` path foolproof by including the one-line Homebrew installation command. Make this the primary, recommended method for macOS and Linux. The `cargo install` method needs to be fixed or removed.
*   **Add a "Troubleshooting" or "Common Issues" Section:** This section should immediately address the Rust version and `wasm32` target issues, as these are very likely to be encountered. A simple FAQ format would be highly effective.
*   **Update Deprecated Commands:** Regularly review and update the documentation to ensure all commands are current. This builds trust and reduces confusion.

By addressing these critical blockers, the "getting-started" tutorial can become a smooth and effective onboarding experience for new Soroban developers.
