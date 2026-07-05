# GALasm - Technical Code Description

## Project Overview

**GALasm** is a portable, open-source GAL (Generic Array Logic) assembler written in ANSI-C. It converts GAL assembly language source files into multiple output formats suitable for programming GAL devices. The project is based on the original GALer assembler (Amiga platform) but has been ported to be platform-independent and compilable with modern C compilers like GCC and SAS/C.

### Key Information
- **Version**: 2.1
- **Copyright**: © 1998-2003 Alessandro Zummo (port), © 1991-1996 Christian Habermann (original)
- **License**: Proprietary with restrictions on commercial use
- **Platform Support**: ANSI-C based, Linux, Amiga, and other Unix-like systems
- **Primary Language**: C (ANSI-C standard)

---

## Supported GAL Device Types

The assembler supports four major GAL architectures:

1. **GAL16V8** - 16-pin GAL with 8 outputs
   - Logic array: 2048 fuses (LOGIC16)
   - XOR array: 2056
   - Pin count: 20 (including VCC/GND)

2. **GAL20V8** - 20-pin GAL with 8 outputs
   - Logic array: 2560 fuses (LOGIC20)
   - XOR array: 2560
   - Pin count: 24 (including VCC/GND)

3. **GAL22V10** - 22-pin GAL with 10 outputs
   - Logic array: 5808 fuses (LOGIC22V10_SIZE)
   - Total fuses: 5892
   - Pin count: 24 (including VCC/GND)

4. **GAL20RA10** - 20-pin GAL with registered outputs
   - Logic array: 3200 fuses (LOGIC20RA10_SIZE)
   - Total fuses: 3274
   - Pin count: 24 (including VCC/GND)

---

## Architecture & Core Components

### Source File Organization

The codebase consists of four main C modules:

#### 1. **galasm.c / galasm.h** - Main Assembler Engine
   - **Purpose**: Core GAL assembly logic and fuse mapping
   - **Key Responsibilities**:
     - Parse GAL source files (assembly language)
     - Process logic equations and pin declarations
     - Manage fuse matrix computations
     - Support multiple pin modes (input/output configurations)
   
   - **Key Data Structures**:
     - Pin-to-fuse mappings for each GAL type and mode
     - Pin naming and declaration tracking
     - Output logic macrocell (OLMC) configuration
   
   - **Pin Mapping Arrays**: For each GAL type, arrays map physical pins to fuse matrix columns:
     - `PinToFuse16Mode1/2/3[]` - GAL16V8 configurations
     - `PinToFuse20Mode1/2/3[]` - GAL20V8 configurations
     - `PinToFuse22V10[]` - GAL22V10 mapping
     - `PinToFuse20RA10[]` - GAL20RA10 mapping
   
   - **Fuse Array Constants**:
     - Defines precise bit offsets for various functional units within JEDEC files
     - Includes locations for logic arrays, XOR arrays, signature fields, and architecture control words

#### 2. **jedec.c** - JEDEC File Format Handler
   - **Purpose**: Generate and manipulate JEDEC format files
   - **Key Responsibilities**:
     - Calculate JEDEC file checksums
     - Write output files in JEDEC standard format
     - Handle line ending conversions (CRLF vs. native)
     - Support output file generation (.jed files)
   
   - **Key Functions**:
     - `FileChecksum()` - Computes 16-bit checksums with STX/ETX markers
     - `WriteOutputWithCRLFLineEndings()` - Platform-independent output
     - `WriteOutputWithNativeLineEndings()` - OS-specific line endings

#### 3. **support.c** - Utility Functions
   - **Purpose**: General-purpose helper functions
   - **Key Responsibilities**:
     - Filename manipulation (base name extraction)
     - File extension handling
     - Memory allocation wrappers
     - String operations
   
   - **Key Functions**:
     - `GetBaseName()` - Extract filename without extension
     - File I/O support routines

#### 4. **localize.c** - Error Handling & Localization
   - **Purpose**: Error messages and localization strings
   - **Key Content**:
     - `ErrorArray[]` - General application error messages
     - `AsmErrorArray[]` - Assembly-specific error messages
   
   - **Error Categories**:
     - File I/O errors (file not found, save failures)
     - Memory errors
     - JEDEC format errors
     - Assembly syntax errors (type declaration, pin names, equations)
     - Pin configuration errors (illegal assignments, duplicates)

---

## Input/Output Format Support

### Input Format
- **Source Files**: GAL assembly language (.pld or .chp files)
- **Syntax Features**:
  - GAL type declaration (GAL16V8, GAL20V8, GAL22V10, GAL20RA10)
  - Pin declarations with names and assignments
  - Logic equations using Boolean expressions
  - Pin suffixes: T (tristate), R (register), E (enable), CLK (clock), APRST/ARST (async reset)

### Output Formats (Optional)
Command-line flags control file generation:
- `.chp` - Chip file (fuse array representation)
- `.fus` - Fuse listing
- `.pin` - Pin configuration
- `.jed` - JEDEC file (standard programming format)

### Command-Line Interface
```
GALasm [-scfpa] <filename>
  -s  Enable security fuse
  -c  Do not create .chp file
  -f  Do not create .fus file
  -p  Do not create .pin file
  -a  Restrict checksum to fuse array only
```

---

## Fuse Matrix Architecture

### Logic Array Organization
The fuse matrix is organized with specific bit locations for each GAL type:

- **Row Size** (input terms):
  - GAL16V8/20V8: 64 bits per row
  - GAL22V10: 132 bits per row
  - GAL20RA10: 80 bits per row

- **Functional Components**:
  - **LOGIC array**: Main programmable AND array
  - **XOR array**: Output polarity/inversion control (8 bits typical)
  - **SIG field**: Signature/identification field (64 bits)
  - **AC1 field**: Architecture control bits (8 bits)
  - **PT field**: Power-up state (64 bits)
  - **SYN field**: Synchronous preset (1 bit)
  - **AC0 field**: Additional control (1 bit)
  - **ACW field**: Architecture control word (82 bits total)

### Pin-to-Matrix Mapping
Multiple pin modes support different input/output configurations:
- **Mode 1, 2, 3** (GAL16V8/20V8): Different output pin assignments for flexibility
- **Fixed Mode** (GAL22V10/20RA10): Single defined configuration
- **-1 values** indicate unused or reserved pin positions in the matrix

---

## Build System

### Makefile Configuration
- **Compiler**: GCC with Wall and O2 optimization flags
- **Object Files**: `galasm.o`, `support.o`, `jedec.o`, `localize.o`
- **Target Executable**: `galasm`
- **Build Commands**:
  - `make` or `make all` - Build the assembler
  - `make clean` - Remove object files and executable
  - `make cleantilde` - Remove editor backup files (~)

### Compilation
Standard ANSI-C compilation with no external dependencies beyond libc.

---

## Key Features

1. **Multi-Device Support**: Handles 4 major GAL architectures with distinct architectures
2. **Flexible Pin Modes**: Multiple configuration modes for input/output pin assignments
3. **JEDEC Standard Compliance**: Outputs standard JEDEC format for industry-standard programmers
4. **Security Fuse Support**: Optional security fuse programming
5. **Selective Output Generation**: Command-line control over which output files to create
6. **Cross-Platform**: ANSI-C ensures portability across Unix-like systems
7. **Checksum Validation**: Built-in JEDEC file checksum computation for data integrity
8. **Comprehensive Error Reporting**: Detailed error messages for assembly debugging

---

## Data Flow

```
Source File (.pld)
    ↓
[galasm.c - Parser & Lexer]
    ├─ Parse GAL type declaration
    ├─ Process pin declarations
    ├─ Parse logic equations
    └─ Build fuse array
    ↓
[galasm.c - Fuse Mapper]
    ├─ Map pins to fuse matrix columns
    └─ Compute Boolean logic into fuse patterns
    ↓
[jedec.c - JEDEC Generator]
    ├─ Format fuse data
    ├─ Calculate checksum
    └─ Write output
    ↓
Output Files (.chp, .fus, .pin, .jed)
```

---

## Technical Highlights

### Memory Management
- Dynamic memory allocation for filename handling
- Buffer management for JEDEC file output
- Efficient string operations for pin name tracking

### Error Handling
- Comprehensive error codes for different failure scenarios
- Line-by-line assembly error reporting
- File I/O error detection and reporting

### Platform Independence
- Platform-agnostic line ending handling (CRLF vs. native)
- Portable C code with no system-specific dependencies
- Support for both 16-bit and 32-bit systems

---

## Example Applications

The project includes example files demonstrating GAL designs:
- **Counter.pld** - Simple counter logic
- **Gatter.pld** - Gate logic design
- **Tristate.pld** - Tri-state output demonstration
- **GAL16V8.pld**, **GAL20RA10.pld**, **GAL22V10.pld** - Device-specific examples

---

## Documentation

Comprehensive HTML documentation provided in `/docs/galer/` folder covering:
- Assembly language syntax
- JEDEC file format specifications
- Device-specific architecture details
- Error codes and troubleshooting
- Complete GAL programming guide
