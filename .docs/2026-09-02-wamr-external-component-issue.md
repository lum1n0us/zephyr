# Add Wasm-micro-runtime as an external component

Issue draft for zephyrproject-rtos/zephyr, following the format of
https://github.com/zephyrproject-rtos/zephyr/issues/102282

Suggested labels: `TSC`, `area: Modules`

---

### Origin

https://github.com/wasm-micro-runtime/wasm-micro-runtime

### Purpose

WebAssembly Micro Runtime (WAMR) is a lightweight standalone WebAssembly runtime
targeting embedded and resource constrained devices. It allows a Zephyr
application to load and execute WebAssembly (WASM) modules at runtime:

- Interpreter (classic and fast) and AOT execution modes
- Built-in libc and WASI subset, plus host-defined native APIs
- Sandboxed execution as defined by the WebAssembly specification
- Configurable memory model (global heap pool or system allocator)

### Mode of Integration

This library should be integrated as **an external project module**.

WAMR already supports Zephyr through a dedicated platform layer and ships the
Zephyr module glue (`zephyr/module.yml`, `zephyr/Kconfig`,
`zephyr/CMakeLists.txt`) in its own repository, so an application only has to
add the West project and select a few `CONFIG_WAMR_*` options.

The main motivation is to let Zephyr applications update or extend
functionality after deployment without reflashing the firmware, and to run
untrusted or third-party logic inside a sandbox.

### Maintainership

@lum1n0us @TianlongLiang @srberard

### Pull Request

(documentation PR adding `doc/develop/manifest/external/wamr.rst` — to be linked)

### Description

#### Primary functionality of library

WAMR provides a WebAssembly runtime plus its toolchain support: module
loading and validation, interpretation and AOT execution, a native API
registration mechanism to expose host functionality to WASM modules, and a
small footprint memory allocator suitable for MCU-class devices.

#### What problem does it solve

It gives Zephyr a portable, sandboxed application container. Device logic can be
shipped and updated as WASM modules independently of the firmware image,
which is useful for field-updatable behaviour, multi-tenant edge workloads and
for isolating third-party code from the rest of the system.

#### Why the best integration is as an external component

- WAMR has its own release cadence, upstream community and maintainers; keeping
  it external allows an independent lifecycle
- The Zephyr port lives upstream in WAMR, so no fork or downstream patches are
  needed
- Most applications do not need a WASM runtime, so it should not be part of the
  default manifest
- Matches existing Zephyr practice for optional, externally maintained runtimes
  and language toolchains

### Security

- WASM modules execute inside the sandbox defined by the WebAssembly
  specification: linear memory bounds, control flow integrity and no access to
  host resources beyond the native APIs explicitly registered by the
  application.
- WAMR validates a `.wasm` module while loading it: a malformed or invalid
  binary is rejected with an error and loading is aborted, so a corrupted or
  crafted module cannot reach execution. `.wasm` modules may therefore come
  from an untrusted source, but the application is responsible for checking
  that loading actually succeeded before running them. AOT modules are not
  validated this way and must be provisioned from a trusted source
  (signature verification, secure transport, secure storage) by the
  application.
- Accesses to the linear memory are bounds-checked, so a memory overflow in a
  WASM module is trapped inside the sandbox instead of corrupting runtime or
  host state.
- Native APIs exported to WASM modules widen the attack surface and must be
  written defensively; the runtime validates pointer arguments against the
  module's linear memory.

### Dependencies

The WAMR Zephyr port relies solely on Zephyr-provided subsystems (kernel,
threading, timing, heap). AOT compilation is done on the host with `wamrc`,
which is not part of the Zephyr build.

### Version or SHA

main

### License (SPDX)

Apache-2.0 WITH LLVM-exception
