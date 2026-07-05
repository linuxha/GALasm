# GALasm 2.1 - Portable GAL Assembler

A portable, open-source GAL (Generic Array Logic) assembler written in ANSI-C. GALasm converts GAL assembly language source files into multiple output formats suitable for programming GAL devices.

## Features

- **Multi-device support**: GAL16V8, GAL20V8, GAL22V10, GAL20RA10
- **Cross-platform**: ANSI-C based, runs on Linux, macOS, and other Unix-like systems
- **Multiple output formats**: JEDEC (.jed), chip files (.chp), fuse listings (.fus), pin configurations (.pin)
- **JEDEC standard compliance**: Compatible with industry-standard GAL programmers
- **Security fuse support**: Optional security fuse programming
- **Flexible output control**: Command-line options to selectively generate output files

## Prerequisites

### Build Requirements
- **C Compiler**: GCC (or any ANSI-C compatible compiler)
- **Standard Libraries**: libc (included with most systems)
- **Build Tools**: GNU Make

### System Requirements
- Linux, macOS, BSD, or other Unix-like system
- Minimum 1 MB free disk space

### Installation on Common Systems

**Ubuntu/Debian:**
```bash
sudo apt-get install build-essential
```

**macOS (with Homebrew):**
```bash
brew install gcc make
```

**Fedora/RHEL:**
```bash
sudo dnf install gcc make
```

## Building

### Quick Start
```bash
cd src
make
```

This creates the `galasm` executable in the `src/` directory.

### Build Targets

| Target | Description |
|--------|-------------|
| `make` or `make all` | Build the GALasm assembler |
| `make clean` | Remove object files and executable |
| `make cleantilde` | Remove editor backup files (~) |

### Compilation Details
- **Compiler Flags**: `-Wall -O2` (warnings enabled, optimization level 2)
- **Output**: `galasm` executable (no installation prefix applied)

## Usage

### Basic Syntax
```bash
./galasm [options] <filename>
```

### Command-Line Options

| Option | Description |
|--------|-------------|
| `-s` | Enable security fuse (prevents unauthorized reprogramming) |
| `-c` | Do not create `.chp` (chip) file |
| `-f` | Do not create `.fus` (fuse) file |
| `-p` | Do not create `.pin` (pin configuration) file |
| `-a` | Restrict checksum to fuse array only (default includes all) |

### Examples

Assemble a GAL16V8 design with all output files:
```bash
./galasm examples/Counter.pld
```

Assemble with security fuse enabled:
```bash
./galasm -s examples/GAL22V10.pld
```

Assemble and generate only JEDEC output (no .chp, .fus, .pin):
```bash
./galasm -cfp examples/Gatter.pld
```

## Output Files

When assembling a source file `example.pld`, GALasm generates:

| File | Extension | Description |
|------|-----------|-------------|
| Chip Configuration | `.chp` | Readable chip fuse array representation |
| Fuse List | `.fus` | Detailed fuse programming list |
| Pin Configuration | `.pin` | Pin name and type definitions |
| JEDEC Format | `.jed` | Standard JEDEC file for programming GAL devices |

### JEDEC File Format
The `.jed` file is the standard format for GAL programmers. It contains:
- Fuse array data (0 = blown, 1 = intact)
- Device signature and architecture control words
- Checksum for data integrity verification
- CRLF line endings for cross-platform compatibility

## Source File Format

GALasm accepts assembly language files describing GAL logic. Basic structure:

```
GAL16V8          ; Device type declaration

CLK = 1          ; Pin assignments
RESET = 2
OUTPUT1 = 19
OUTPUT2 = 18

OUTPUT1.D = INPUT1 & INPUT2 & CLK
OUTPUT1.AR = RESET

OUTPUT2 = OUTPUT1 : OUTPUT2 | INPUT3
```

### GAL Type Declarations
- `GAL16V8` - 16-pin with 8 outputs
- `GAL20V8` - 20-pin with 8 outputs
- `GAL22V10` - 22-pin with 10 outputs
- `GAL20RA10` - 20-pin with registered outputs

### Pin Name Suffixes
| Suffix | Function |
|--------|----------|
| `.D` | Registered (D flip-flop) input |
| `.T` | Toggle/T flip-flop |
| `.R` | Reset state |
| `.E` | Output enable (tristate) |
| `.CLK` | Clock select |
| `.APRST` | Asynchronous preset/reset |
| `.ARST` | Asynchronous reset |

For complete syntax documentation, see the HTML manual in `docs/galer/main.html`.

## Examples

The `examples/` directory contains sample GAL designs:

| File | Type | Description |
|------|------|-------------|
| `Counter.pld` | GAL16V8 | Binary counter logic |
| `Gatter.pld` | GAL16V8 | Gate logic design |
| `Tristate.pld` | GAL16V8 | Tri-state output control |
| `GAL20RA10.pld` | GAL20RA10 | Device-specific example |
| `GAL22V10.pld` | GAL22V10 | Device-specific example |

Each example includes associated output files (.chp, .fus, .jed, .pin) showing expected results.

## Troubleshooting

### Build Issues

**Error: `gcc: command not found`**
- Install GCC: `sudo apt-get install gcc` (Ubuntu/Debian)
- Or install Xcode Command Line Tools on macOS: `xcode-select --install`

**Error: `make: command not found`**
- Install make: `sudo apt-get install make` (Ubuntu/Debian)
- Or `brew install make` on macOS

**Error: `Permission denied` when running galasm**
- Make the executable: `chmod +x src/galasm`

### Assembly Errors

**Error: "type of GAL expected"**
- First line of source file must declare a GAL type (GAL16V8, GAL20V8, etc.)

**Error: "pinname defined twice"**
- A pin name is assigned multiple times. Each pin can only be declared once.

**Error: "this pin can't be used as output"**
- Attempted to use an input-only pin as output. Check GAL device pinout.

**Error: "unknown pinname"**
- Pin name used in equation was not declared. Declare it with `PINNAME = N` syntax.

## Documentation

- **[description.md](description.md)** - Detailed technical code documentation
- **[docs/galer/](docs/galer/)** - Complete HTML reference manual
  - `main.html` - Start here for overview
  - `sourcefile.html` - Source file syntax guide
  - `jedecfile.html` - JEDEC format specification
  - `errors.html` - Complete error code reference
- **[ChangeLog](ChangeLog)** - Version history and changes

## Installation (Optional)

To install GALasm system-wide after building:

```bash
cd src
make
sudo cp galasm /usr/local/bin/
```

Now you can run `galasm` from anywhere:
```bash
galasm input.pld
```

To uninstall:
```bash
sudo rm /usr/local/bin/galasm
```

## Performance Notes

- Typical assembly time: < 1 second for standard GAL designs
- Memory usage: Minimal (< 10 MB)
- Supported file sizes: Up to typical GAL source file sizes (practical limit > 1 MB)

## License

```
Copyright (c) 1998-2003 Alessandro Zummo (port)
Original sources Copyright (c) 1991-1996 Christian Habermann
All Rights Reserved

Commercial use is strictly forbidden
```

**Important**: GALasm is provided AS-IS without warranty. The authors cannot be held responsible for any damage caused by faults in the assembler. Use at your own risk.

## Support & Contact

- **Original Author**: Christian Habermann (GALer)
- **Port Author**: Alessandro Zummo
- **Support Page**: http://www.towertech.it/azummo/ (no longer works)
- **Repository**: [GALasm on GitHub](https://github.com/daveho/GALasm)

## Credits

- **GALer**: Original GAL assembler for Amiga (Christian Habermann)
- **GALasm**: ANSI-C port for cross-platform compatibility (Alessandro Zummo)
- **GCC**: Free C compiler used for compilation

# Links

https://www.jedec.org/