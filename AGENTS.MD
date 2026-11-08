# MORSE — AGENTS.md

> **Project:** MORSE  
> **Codename:** MORSE  
> **Type:** Next-generation microkernel operating system  
> **Primary language:** Rust  
> **Initial architecture:** x86_64  
> **Kernel model:** Capability-based microkernel  
> **Development philosophy:** Correctness first. Security by design. Performance through measurement.

---

# 1. Project Identity

MORSE is an independent, next-generation microkernel project written primarily in Rust.

Its objective is to explore and build a modern operating-system foundation that combines:

- The modularity and isolation of microkernel architectures.
- The performance and hardware control of low-level systems programming.
- Strong memory safety guarantees through Rust.
- Capability-based security.
- Efficient inter-process communication.
- Predictable scheduling and low-latency execution.
- Modern multicore and heterogeneous hardware support.
- A foundation suitable for an advanced graphical desktop environment.

MORSE is not a Linux distribution, a Linux fork, a Linux compatibility layer, or a direct reimplementation of XNU.

It must have its own architecture, abstractions, interfaces and design principles.

The project should learn from existing operating systems without blindly reproducing their implementation.

## Core philosophy

1. Minimal trusted kernel.
2. Memory safety wherever technically possible.
3. Explicit ownership and resource management.
4. Isolation by default.
5. Capability-based access control.
6. Asynchronous and efficient communication.
7. Predictable system behavior.
8. Hardware-independent abstractions where appropriate.
9. Measurable performance.
10. Long-term maintainability.

MORSE must prioritize architectural quality over superficial feature completeness.

---

# 2. Instructions for AI Agents

You are an engineer contributing to MORSE.

You must behave as a systems programmer, kernel engineer and software architect.

Your responsibility is not simply to produce code that compiles. Your responsibility is to preserve the integrity of the operating system.

## Mandatory rules

- Read the existing architecture and documentation before modifying code.
- Understand the execution context of every kernel component.
- Never invent APIs, structures, functions or interfaces without checking the codebase.
- Never assume that an operation is safe in interrupt context.
- Never introduce undefined behavior without a documented and justified requirement.
- Never use unsafe Rust when a safe alternative is reasonably available.
- Never introduce unnecessary dependencies into the kernel.
- Never duplicate existing abstractions.
- Never silently change public interfaces.
- Never replace a working implementation with a less complete placeholder.
- Never hide compiler errors, warnings or failed tests.
- Never claim that a feature works without testing it.
- Never fabricate benchmark results.
- Never sacrifice correctness for an unmeasured performance improvement.

Before implementing a feature, determine:

1. What problem does it solve?
2. Which architectural layer owns it?
3. What are its invariants?
4. What are its security implications?
5. What are its performance implications?
6. How can it be tested?
7. What happens when it fails?

If a design decision has major architectural consequences, document the alternatives and trade-offs before implementing it.

When the existing architecture is unclear, investigate rather than guessing.

---

# 3. Architectural Principles

MORSE must follow a genuine microkernel architecture.

The kernel should contain only mechanisms that require privileged execution or direct management of fundamental hardware resources.

## Kernel responsibilities

The initial kernel should provide:

- CPU initialization.
- Privilege and execution context management.
- Thread and scheduling primitives.
- Address-space management.
- Virtual memory primitives.
- Interrupt and exception handling.
- Inter-process communication mechanisms.
- Capability management.
- Minimal hardware abstraction.
- Timer and clock primitives.
- Kernel memory allocation.
- Synchronization primitives.
- Multicore coordination.

## User-space responsibilities

The following should be implemented outside the privileged kernel whenever possible:

- Device drivers.
- Filesystem implementations.
- Network stack.
- Storage management.
- Process management policies.
- Graphical compositor.
- Window manager.
- Audio services.
- Application runtime.
- System configuration services.

A component must not enter kernel space merely because doing so appears convenient.

The trusted computing base must remain as small as reasonably possible.

## Architecture overview

    Applications
          |
    User-space Runtime
          |
    System Services
      |    |    |
    VFS  Network  Drivers
      \    |    /
       Capability IPC
             |
        MORSE Kernel
      /      |       \
   Scheduler VM     Interrupts
             |
          Hardware

This diagram represents logical responsibilities, not a requirement that every service use an identical communication path.

---

# 4. Programming Language and Toolchain

Rust is the primary implementation language.

## Rust requirements

- `#![no_std]` for kernel code.
- `#![no_main]` where required by the binary entry point.
- Explicit ownership and lifetime management.
- Strong typing for hardware and kernel interfaces.
- Minimal and justified unsafe code.
- No accidental allocation in critical paths.
- No hidden blocking operations.
- Explicit synchronization.
- Clear separation between privileged and unprivileged code.

Use stable Rust whenever technically possible.

Nightly Rust features are permitted when required by kernel development, but every unstable feature must have a documented justification.

Do not adopt nightly features simply because they are convenient.

## Unsafe Rust

Unsafe code must be treated as a security-sensitive boundary.

Every unsafe block must:

- Have a clear safety justification.
- Document its required invariants.
- Minimize the scope of unsafety.
- Avoid exposing unsound abstractions to safe callers.
- Be covered by appropriate tests where possible.

Never use `transmute`, raw pointer manipulation or unchecked operations as shortcuts around a poorly designed abstraction.

Do not use `unwrap()` or `expect()` in production kernel paths unless failure is demonstrably impossible under documented invariants.

## Dependencies

Dependencies must be evaluated according to:

- Security.
- Maintenance status.
- Compatibility with `no_std`.
- Runtime overhead.
- Code size.
- Licensing.
- Auditability.
- Long-term maintenance costs.

Do not introduce a dependency for functionality that is trivial to implement safely.

Do not rewrite mature and complex functionality merely to avoid dependencies.

---

# 5. Initial Hardware Architecture

The first implementation target is x86_64.

The architecture must not become permanently coupled to x86_64.

## Initial x86_64 requirements

Investigate and progressively implement:

- UEFI boot.
- Long mode initialization.
- GDT and TSS.
- IDT.
- CPU exception handling.
- Local APIC.
- I/O APIC.
- HPET or suitable hardware timers.
- Paging.
- NX support.
- SMEP and SMAP where available.
- PCID where available.
- SYSCALL/SYSRET or an appropriately justified syscall mechanism.
- SMP initialization.
- ACPI discovery.
- PCI enumeration.
- Interrupt routing.

Hardware capabilities must be detected at runtime.

Never assume that every processor supports every optional feature.

Architecture-specific implementations must remain isolated behind explicit interfaces.

Future architecture targets should include:

- AArch64.
- RISC-V 64-bit.

Do not compromise the initial architecture to prematurely implement every target.

---

# 6. Memory Management

Memory management is one of MORSE's most critical subsystems.

It must be designed for safety, predictable behavior and scalability.

## Physical memory management

Implement a physical frame allocator supporting:

- Boot-time memory map ingestion.
- Reserved memory regions.
- Frame allocation.
- Frame deallocation.
- Alignment requirements.
- Reference tracking where appropriate.
- Multiple memory zones where justified.
- SMP-safe allocation.

Memory ownership must be explicit.

No physical frame may be returned to the allocator while it is still accessible through an active mapping.

## Virtual memory

The virtual memory subsystem must eventually support:

- Independent address spaces.
- User/kernel separation.
- Page permissions.
- Guard regions.
- Demand allocation where appropriate.
- Copy-on-write.
- Shared memory.
- Memory mapping.
- Address-space switching.
- TLB invalidation.
- SMP-safe mapping changes.

Avoid unnecessary TLB flushes, but never compromise memory isolation to optimize them.

## Heap allocator

The kernel allocator must:

- Support the required allocation patterns.
- Avoid unnecessary fragmentation.
- Have documented synchronization behavior.
- Be usable in the intended execution contexts.
- Provide clear failure handling.

Different allocation strategies may be used for different workloads.

Do not assume one allocator is optimal for every purpose.

---

# 7. Scheduler and Execution Model

The scheduler must prioritize fairness, responsiveness, predictability and multicore scalability.

A scheduler is not considered good simply because it achieves high throughput in synthetic benchmarks.

## Required properties

- Preemptive scheduling.
- Multicore support.
- Thread priorities.
- Efficient context switching.
- Low scheduling latency.
- Fair CPU distribution.
- CPU affinity.
- Idle-thread handling.
- Timer integration.
- Explicit synchronization.
- Protection against priority inversion.

## Future capabilities

Investigate:

- Per-CPU run queues.
- Work stealing.
- Load balancing.
- Deadline scheduling.
- Real-time scheduling classes.
- Interactive workload prioritization.
- Heterogeneous CPU awareness.

Scheduling policies must remain separate from fundamental scheduling mechanisms.

Never introduce arbitrary priority boosts or heuristics without a measurable justification.

## Performance metrics

Track:

- Context-switch latency.
- Wake-up latency.
- Scheduling latency.
- Throughput.
- Tail latency.
- CPU utilization.
- Cross-core migration overhead.
- Lock contention.

Report distributions, not just averages.

---

# 8. IPC and Capability-Based Security

IPC is a fundamental part of MORSE.

It must be designed as a first-class kernel subsystem.

## IPC requirements

Investigate and implement:

- Synchronous message passing.
- Asynchronous message passing.
- Notifications.
- Endpoint abstractions.
- Shared-memory communication.
- Efficient message transfer.
- Blocking and non-blocking operations.
- Timeout support.
- Cancellation.
- Backpressure.
- SMP-safe communication.

The design must avoid unnecessary copies where safely possible.

However, zero-copy must never become an excuse for unsafe shared memory.

## Capability model

Resources must be accessed through explicit capabilities.

Capabilities should represent authority over resources such as:

- IPC endpoints.
- Memory objects.
- Address spaces.
- Threads.
- Interrupt sources.
- Device resources.
- Shared-memory regions.

Implement:

- Capability creation.
- Capability transfer.
- Capability revocation.
- Capability derivation.
- Rights restriction.
- Explicit authority validation.

Never rely exclusively on globally accessible integer identifiers as a security mechanism.

The design should follow the principle of least authority.

---

# 9. Interrupts and Concurrency

Interrupt handling must be carefully separated from ordinary execution.

Interrupt handlers must remain short and predictable.

They should defer expensive work to appropriate execution contexts.

## Mandatory considerations

- Interrupt masking.
- Interrupt routing.
- Nested interrupts where supported.
- Per-CPU interrupt state.
- Deferred work.
- Atomic operations.
- Memory ordering.
- Locking discipline.
- Deadlock prevention.
- Interrupt-safe synchronization.

Every synchronization primitive must explicitly document whether it is safe in:

- Ordinary thread context.
- Interrupt context.
- NMI context, where applicable.

Never acquire a potentially blocking lock from an interrupt handler.

Avoid global locks in frequently executed kernel paths.

---

# 10. Drivers and Hardware Isolation

Drivers should run in user space whenever practical.

The kernel must expose minimal, controlled mechanisms for accessing hardware.

## Driver architecture

Investigate:

- Device discovery.
- PCI and PCIe enumeration.
- Device resource assignment.
- Interrupt delegation.
- DMA abstractions.
- IOMMU integration.
- Device ownership.
- Driver lifecycle.
- Driver isolation.
- Driver recovery and restart.

DMA must be treated as a security boundary.

Never assume that a device cannot access memory outside its intended region.

Where hardware supports it, investigate IOMMU-based isolation.

A failed user-space driver should not automatically compromise the kernel.

---

# 11. System Services

System services must be independently executable and restartable where appropriate.

The system should eventually provide:

- Process manager.
- Service manager.
- Virtual filesystem service.
- Storage services.
- Network services.
- Device manager.
- Time service.
- Logging service.
- Configuration service.
- Security authority service.

Service discovery and lifecycle management must use explicit interfaces.

Do not create circular dependencies between foundational services.

Boot ordering must be deterministic and documented.

---

# 12. Graphics and Desktop Architecture

The long-term objective includes a modern graphical environment.

The graphical architecture must be treated as a major subsystem, not an afterthought.

The goal is an experience comparable to or better than the responsiveness and animation quality of modern macOS.

## Display architecture

Investigate:

- User-space compositor.
- Layer-based rendering.
- GPU command submission.
- Damage tracking.
- Buffer management.
- Frame scheduling.
- Presentation timing.
- VSync.
- Variable refresh rate.
- Multi-monitor support.
- Hardware acceleration.

## Animation engine

The animation engine should support:

- Independent animation timelines.
- Time-based interpolation.
- Bézier timing functions.
- Spring animations.
- Layer transformations.
- Opacity transitions.
- Efficient composition.
- Cancellation and interruption.
- Deterministic animation state.
- Frame deadline awareness.

Animations must not depend unnecessarily on an application's main thread.

The system must minimize frame hitches and input-to-photon latency.

## Graphics performance metrics

Measure:

- Frame time.
- Frame-time variance.
- Missed presentation deadlines.
- Input-to-photon latency.
- GPU utilization.
- CPU composition overhead.
- Buffer submission latency.
- Memory bandwidth.

Do not equate high FPS with smoothness.

Consistent frame delivery is a first-class performance requirement.

---

# 13. Security Architecture

Security must be integrated into the architecture from the beginning.

It must not be treated as a feature to add later.

## Security requirements

- Strong kernel/user-space separation.
- Capability-based authorization.
- Address-space isolation.
- Memory permission enforcement.
- Controlled IPC.
- Explicit resource ownership.
- Minimal privileged code.
- Safe syscall validation.
- Protection against integer overflow.
- Validation of untrusted input.
- DMA isolation where available.
- Secure boot support as a future objective.

Never trust user-space pointers.

Validate every user-provided address, length and resource identifier.

Security checks must remain effective across process boundaries and privilege transitions.

Document threat assumptions and security boundaries.

---

# 14. Performance Engineering

Performance must be measured scientifically.

Never optimize based solely on intuition.

## Required principles

1. Establish a baseline before optimization.
2. Identify the actual bottleneck.
3. Change one relevant variable at a time.
4. Benchmark before and after.
5. Repeat measurements.
6. Account for measurement noise.
7. Track regressions.
8. Preserve correctness.

## Important metrics

- Boot time.
- Memory footprint.
- System-call latency.
- IPC throughput.
- IPC latency.
- Context-switch latency.
- Allocation latency.
- Page-fault latency.
- Scheduler responsiveness.
- Interrupt latency.
- Storage throughput.
- Network throughput.
- Graphics frame pacing.

Report:

- Mean.
- Median.
- P95.
- P99.
- Maximum where useful.
- Test conditions.

Never claim MORSE is faster than Linux, XNU or another operating system without reproducible comparative benchmarks.

Performance claims must specify the workload and hardware.

---

# 15. Testing and Validation

Every subsystem must have a testing strategy.

## Development environment

Use QEMU as the primary initial testing environment.

Use:

- Automated build checks.
- Unit tests.
- Integration tests.
- Boot tests.
- Architecture-specific tests.
- Fault injection.
- Stress tests.
- Fuzzing where applicable.
- Static analysis.
- Formal verification for selected critical components where feasible.

## Required validation

A feature is not complete merely because the kernel boots.

Its correctness must be tested under relevant conditions.

Examples:

- Memory allocator exhaustion.
- Invalid user-space addresses.
- Malformed IPC messages.
- Concurrent capability operations.
- Interrupt storms.
- Thread termination during communication.
- Address-space destruction.
- Driver failure.
- Multicore contention.

Tests must be deterministic wherever possible.

Failures must be reproducible and documented.

---

# 16. Code Quality and Architecture

Maintain a clear separation between:

- Architecture-specific code.
- Architecture-independent kernel logic.
- Memory management.
- Scheduling.
- IPC.
- Security.
- Drivers.
- User-space services.
- Shared interfaces.

Avoid monolithic modules.

Avoid unnecessary abstraction layers.

Do not introduce complex generic frameworks without a concrete requirement.

Use descriptive naming and explicit types.

Public interfaces must have documentation.

Important invariants must be documented near the code that enforces them.

Comments should explain why a design exists, not merely repeat what the code does.

---

# 17. Repository and Documentation Discipline

The repository must remain understandable.

Maintain:

- `README.md`: project overview and current status.
- `AGENTS.md`: engineering instructions.
- `ARCHITECTURE.md`: system architecture.
- `ROADMAP.md`: development phases.
- `SECURITY.md`: security model and vulnerability reporting.
- `CONTRIBUTING.md`: contribution guidelines.
- `docs/`: subsystem documentation.
- `benchmarks/`: benchmark definitions and results.

Do not claim implemented functionality in documentation when it is only planned.

Every significant architectural decision should be documented.

Use Architecture Decision Records (ADRs) for decisions with long-term consequences.

Each ADR should include:

- Context.
- Problem.
- Alternatives.
- Decision.
- Consequences.
- Reconsideration criteria.

---

# 18. Development Workflow

Before modifying the project:

1. Inspect the repository structure.
2. Read relevant documentation.
3. Inspect existing implementations.
4. Identify dependencies and interfaces.
5. Understand the current build and test process.

Before writing code:

1. Define the exact scope.
2. Identify invariants.
3. Identify failure cases.
4. Determine the testing strategy.
5. Consider security and performance implications.

After implementing:

1. Run formatting.
2. Run static checks.
3. Compile affected targets.
4. Execute relevant tests.
5. Review unsafe code.
6. Inspect the diff.
7. Update documentation.
8. Report actual results.

Never make unrelated modifications.

Never silently rewrite entire subsystems.

Preserve existing functionality unless the task explicitly requires changing it.

If a requested change conflicts with architectural invariants, explain the conflict and propose a safer design.

---

# 19. Roadmap

Development must proceed incrementally.

The following roadmap establishes architectural priorities, not permission to implement every subsystem simultaneously.

## Phase 0 — Foundations

- Repository structure.
- Rust toolchain.
- Build system.
- Boot protocol selection.
- QEMU environment.
- Architecture documentation.
- Initial CI.

## Phase 1 — Bootable kernel

- UEFI boot.
- Kernel entry point.
- Memory map.
- Serial logging.
- Panic handling.
- Basic CPU initialization.
- Initial physical memory allocator.

## Phase 2 — Core kernel

- GDT and IDT.
- Exceptions.
- Interrupt handling.
- Virtual memory.
- Kernel heap.
- Timer subsystem.
- Thread abstraction.
- Initial scheduler.

## Phase 3 — Microkernel primitives

- User/kernel separation.
- Address spaces.
- System calls.
- IPC.
- Capabilities.
- Thread management.
- Synchronization.

## Phase 4 — Multicore and security

- SMP initialization.
- Per-CPU structures.
- Concurrent scheduling.
- Capability isolation.
- User-space memory protection.
- Interrupt delegation.
- DMA security.

## Phase 5 — User-space foundation

- Initial user-space runtime.
- Process manager.
- Service manager.
- Driver framework.
- IPC-based service discovery.
- Initial filesystem service.

## Phase 6 — Advanced operating-system services

- Storage.
- Networking.
- Device management.
- Process lifecycle.
- Resource management.
- Driver recovery.

## Phase 7 — Graphics

- Display services.
- GPU integration.
- Compositor.
- Window management.
- Animation engine.
- Input pipeline.
- Frame scheduling.

## Phase 8 — Optimization and maturity

- Performance profiling.
- Memory optimization.
- Scheduler optimization.
- IPC optimization.
- Security auditing.
- Stress testing.
- Hardware compatibility.
- Long-term stability.

Do not skip foundational phases to produce visually impressive demonstrations.

---

# 20. Definition of Done

A feature is complete only when:

- Its implementation matches the intended architecture.
- Its invariants are documented.
- Its error paths are handled.
- Its security implications are addressed.
- Its tests pass.
- Its performance is measured where relevant.
- Its interfaces are documented.
- Its unsafe code is justified.
- Its integration does not introduce regressions.

A compiling kernel is not necessarily a correct kernel.

A bootable kernel is not necessarily a usable operating system.

A fast benchmark is not necessarily a fast operating system.

MORSE must be engineered as a complete system.

---

# 21. Final Engineering Principle

Never optimize for the appearance of technical sophistication.

Do not add unnecessary complexity, experimental algorithms, custom allocators, schedulers or abstractions simply because they sound advanced.

Prefer the simplest architecture that satisfies the requirements without compromising long-term extensibility.

Challenge existing design decisions when evidence justifies doing so.

Study Linux, XNU, seL4, Fuchsia, Redox OS, MINIX 3 and other operating systems for their engineering lessons.

Do not copy their limitations or assume their design decisions are universally correct.

Every major component should have a clear reason to exist.

Every optimization should have evidence.

Every security boundary should be explicit.

Every subsystem should have a defined owner.

**MORSE is built to be understood, measured, verified and improved.**

The ultimate objective is not merely to create another kernel.

It is to build a modern operating-system foundation where security, performance, responsiveness and architectural elegance are fundamental properties rather than afterthoughts.
