# `ctrlc` — CTRLFlow Compiler

`ctrlc` is a lightweight, zero-dependency Interface Definition Language (IDL) compiler written in OCaml. It processes `.idl` interface definitions and generates lean, header-only C ABI definitions, C++ driver base classes, Lua bindings, and Protobuf schemas.
It performs the core job of tools like Microsoft's `MIDL`, but with a strict focus on minimalism: no complex inheritance, no templated arrays, and no hidden runtime dependencies.

---

## Output Targets

For a given set of IDL files, `ctrlc` generates a unified set of outputs:

| Generated File | Purpose |
| :--- | :--- |
| `<base>.h` | C ABI interface declarations, vtable definitions, struct layouts, and IID macros. |
| `<base>_s.h` | Header-only C driver framework. Provides ref counting, `QueryInterface`, vtable wiring, and default `CTRLFLOW_E_NOT_SUPPORTED` fallbacks for unimplemented driver slots. |
| `<base>_cxx.h` | Header-only C++ driver base classes. Exposes interface implementations as virtual methods and encapsulates object lifecycles[cite: 1]. |
| `<base>_lua.h` | Lua bindings (via `sol2`). Maps interfaces directly into Lua tables with reference management and enum tables[cite: 1]. |
| `<base>_<interface>.proto` | *(Optional via `--proto`)* Emits the interface as a gRPC service and its structs as Protobuf messages for RPC transport[cite: 1]. |

---

## Features

* **Include-Once Import Splicing:** Imports (`import "types.idl";`) are resolved during compilation and spliced into the generated headers exactly once, preventing duplicate definitions across multiple interfaces.
* **Built-In Signal Bridge:** Supports ABI-safe signals (`signal state_changed(Int32 state);`)[cite: 1]. Generates thread-safe connection tables, signal argument structs, and emit helpers for both C and C++ drivers].
* **Strict Type Mapping:** Unifies built-in types into explicit scalar representations (`Int8` through `Int64`, `UInt8` through `UInt64`, `Float`, `Double`, `Bool` mapped to `int32_t`, and borrowed `String` handling).
* **Compile-Time AST Verification:** Written in OCaml to ensure strict, type-safe validation of interfaces, methods, signals, and underlying types.

---

## Building from Source

### Prerequisites

* OCaml 4.14 or later (or OCaml 5.x)
* [Dune](https://dune.build/) build system
* [Menhir](https://gitlab.inria.fr/fpottier/menhir) parser generator

### Compiling

```bash
# Clone the repository
git clone [https://github.com/your-org/ctrlc.git](https://github.com/your-org/ctrlc.git)
cd ctrlc

# Build release binary
dune build --profile release

# Run tests
dune runtest
```

The compiled native executable will be located at `_build/default/bin/main.exe` (or `_build/default/bin/ctrlc`).

### Usage

```bash
ctrlc <idl-files...> --outdir <output-directory> [--name <basename>] [--proto <InterfaceName>]
```

### Command-Line Arguments

* `<idl-files...>` - One or more input `.idl` files listed in dependency order.
* `--outdir <dir>` - Directory where generated headers and schemas will be placed.
* `--name <basename>` - Base filename prefix for output files (defaults to `ctrlflow`).
* `--proto <Interface>` - Generates a corresponding `.proto` schema for the specified interface (repeatable).

### Example

```bash
ctrlc types.idl device.idl --outdir ./generated --name ctrlflow --proto DeviceControl
```

Generates:
```
generated/
├── ctrlflow.h
├── ctrlflow_s.h
├── ctrlflow_cxx.h
├── ctrlflow_lua.h
└── ctrlflow_devicecontrol.proto
```


### IDL Syntax Example

```
import "types.idl";

[ iid("ctrlflow.DeviceControl"), version("1.0"), helpstring("Hardware Control Interface") ]
interface DeviceControl : CtrlFlowObject {
    signal state_changed(Int32 new_state);

    Status open([in] String address);
    Status close();
    Status read([in] String tag, [out] Value* value);
    Status write([in] String tag, [in] Value* value);
};
```

### License

This project is licensed under the Apache-2.0 License.
