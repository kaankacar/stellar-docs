# Soroban "Getting Started" Documentation: New Developer Experience Report

This report documents the end-to-end experience of a new developer following the Soroban "getting-started" documentation. The process was followed exactly as written, without using any prior knowledge.

## What Went Well

Despite the challenges, several aspects of the documentation were very effective and well-designed for a new developer:

*   **Clear Project Structure:** The `stellar contract init` command creates a well-organized and intuitive project structure. The separation of contracts into their own directories within a workspace is a clean and scalable approach.
*   **Excellent Code Explanations:** The inline explanations of the Rust code in the `hello-world.mdx` and `storing-data.mdx` files were excellent. The documentation did a great job of explaining the purpose of each line of code, from `#![no_std]` to the contract data keys.
*   **Seamless Testnet Interaction:** The process of deploying and interacting with the contract on the Testnet was very smooth. The `stellar contract deploy` and `stellar contract invoke` commands are powerful and easy to use. The automatic funding of the test account with `--fund` is a great touch.
*   **Good Use of Examples:** The "Hello World" and "Increment" examples are well-chosen. They are simple enough for a beginner to understand but complex enough to demonstrate key concepts like contract invocation and data storage.

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
        *   **Error Message:** `-bash: brew: command not found`

4.  **Install Stellar CLI (Attempt 2: Cargo)**
    *   **Instruction:** The documentation mentioned installing from source with `cargo` but did not provide the command. I had to search online and found `cargo install --locked stellar-cli`.
    *   **Outcome:** **Failure.** The command consistently timed out after several minutes. This was a critical blocker that made it impossible to proceed with the tutorial.
        *   **Error Message:** `The command timed out after 401.2746195793152 seconds.`

5.  **User Intervention & Successful Installation**
    *   **Instruction:** The user provided a command to install Homebrew, unblocking the process.
    *   **Outcome:** **Success.** After installing Homebrew, `brew install stellar-cli` worked perfectly.

6.  **"Hello World" Tutorial (`hello-world.mdx`)**
    *   **Instruction:** Create a new project with `stellar contract init soroban-hello-world`.
    *   **Outcome:** **Success.** The project was created as expected.
    *   **Instruction:** Build the contract with `stellar contract build`.
    *   **Outcome:** **Failure.** The build failed with an error indicating an incompatible Rust version (1.91.0). The documentation had not specified any version constraints.
        *   **Error Message:** `error: use a rust version other than 1.81, 1.82, 1.83 or 1.91.0 to build contracts (got 1.91.0)`
    *   **Troubleshooting & Second Failure:** I installed Rust `1.90.0` and set it as an override. The build failed again, this time because the `wasm32v1-none` target was not installed for the *new* toolchain. This nuance was not mentioned in the documentation.
        *   **Error Message:** `error[E0463]: can't find crate for 'core'`
    *   **Final Attempt:** After installing the wasm target for the `1.90.0` toolchain, the build finally succeeded.
    *   **Instruction:** Optimize the contract with `stellar contract optimize`.
    *   **Outcome:** **Success, but with a deprecation warning.** The documented command is outdated.
        *   **Warning Message:** ` `stellar contract optimize` is deprecated and will be removed in future versions of the CLI. Use `stellar contract build --optimize` instead.`

7.  **"Deploy to Testnet" & "Storing Data" Tutorials**
    *   **Instruction:** All subsequent steps, including deploying contracts and adding the `increment` contract, were followed.
    *   **Outcome:** **Success.** These parts of the tutorial were clear and all commands worked as expected.

---

## B. Documentation Gaps Report

This section details the specific gaps and issues found in the documentation.

1.  **Missing Prerequisite: Homebrew Installation**
    *   **Issue:** The primary installation method for macOS/Linux relies on Homebrew, but there are no instructions for installing it.
    *   **Impact:** This is a critical blocker for any new developer who does not already have Homebrew installed. It creates an immediate dead-end and forces the user to search for external resources, breaking the flow of the tutorial.
    *   **Recommendation:** Add the official Homebrew installation command or a link to their website directly in the `setup.mdx` file.

2.  **Incomplete and Unreliable `cargo` Installation Method**
    *   **Issue:** The `cargo install` command is not explicitly provided in the documentation and is prone to timeouts.
    *   **Impact:** This is another critical blocker. Even if a user finds the command, the repeated timeouts make it seem like the installation is broken. A new developer is unlikely to wait more than a few minutes for a command to complete without assuming something has gone wrong.
    *   **Recommendation:** The Homebrew method should be listed as the primary, recommended option. If the `cargo` method is to be kept, it must be fixed to be more reliable, or at the very least, include a prominent warning about the long compilation time and the correct, versioned command: `cargo install --locked stellar-cli@<version>`.

3.  **Missing Rust Version Compatibility**
    *   **Issue:** The documentation does not specify which versions of Rust are compatible with the Soroban SDK. This led to a build failure that a new developer would struggle to diagnose.
    *   **Impact:** This is a high-friction, confusing blocker. The error message is clear if you know what to look for, but a new developer might not understand why their freshly installed Rust version is incompatible. It undermines confidence in the tutorial.
    *   **Recommendation:** Add a section specifying the supported range of Rust versions. Also, provide a brief guide on how to manage Rust versions with `rustup`, as this is a common and necessary task for any Rust developer.

4.  **Incomplete `wasm32` Target Instructions**
    *   **Issue:** The documentation does not explain that the `wasm32v1-none` target must be installed for *each* Rust toolchain in use.
    *   **Impact:** This can cause confusing build failures if a developer needs to switch Rust versions to solve the compatibility issue mentioned above. The error message (`can't find crate for 'core'`) is not immediately obvious to a newcomer.
    *   **Recommendation:** Add a note to the build section: "If you have switched Rust toolchains, make sure you have installed the wasm target for it: `rustup target add wasm32v1-none --toolchain <your-version>`."

5.  **Outdated `stellar contract optimize` Command**
    *   **Issue:** The `hello-world.mdx` file instructs the use of a deprecated command.
    *   **Impact:** This is a minor issue, but it creates a poor impression and can cause confusion. It makes the documentation seem out of date.
    *   **Recommendation:** Update the documentation to use the modern command: `stellar contract build --optimize`.

---

## C. Final Assessment

**Can a first-time developer successfully follow this tutorial end-to-end?**

**No.** A first-time developer is highly likely to fail. The installation and setup process has critical blockers that would prevent a new user from even beginning the first tutorial.

**Critical Blockers:**

1.  **Stellar CLI Installation:** The primary installation path is broken for new users. The `cargo install` method is unreliable due to timeouts, and the recommended `brew` method is missing instructions for installing Homebrew itself.
2.  **Rust Version Incompatibility:** The lack of versioning information for the Rust toolchain leads to a build failure that is difficult for a newcomer to diagnose and solve.

**Highest-Impact Recommendations for DevRel:**

1.  **Fix the Installation Flow:**
    *   **Make Homebrew the primary, recommended method** for macOS and Linux.
    *   **Include the one-line Homebrew installation command** in the documentation to make the process self-contained.
    *   **Fix or remove the `cargo install` method.** If it is kept, it needs to be reliable and provide clear expectations about compilation time.

2.  **Create a "Troubleshooting" Section:**
    *   This section should be prominently linked in the "Setup" guide.
    *   It should immediately address the **Rust version incompatibility** and **`wasm32` target installation** issues, as these are the next most likely blockers. A simple FAQ format would be highly effective.

3.  **Perform a Command Audit:**
    *   Regularly review the documentation to ensure all commands are up-to-date.
    *   Replace deprecated commands like `stellar contract optimize` with their modern equivalents. This builds trust and reduces confusion.

By addressing these critical blockers, the "getting-started" tutorial can become a smooth and effective onboarding experience for new Soroban developers.
