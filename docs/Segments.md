The configuration file for **splat** consists of a number of well-defined segments.

Most segments can be defined as a either a dictionary or a list, however the list syntax is only suitable for simple cases as it does not allow for specifying many of the options a segment type has to offer.

Splat segments' behavior generally falls under two categories: extraction and linking. Some segments will only do extraction, some will only do linking, some both, and some neither. Generally, segments will describe both extraction and linking behavior. Additionally, a segment type whose name starts with a dot (.) will only focus on linking.

## `asm`

**Description:**

Segments designated Assembly, `asm`, will be disassembled via [spimdisasm](https://github.com/Decompollaborate/spimdisasm) and enriched with Symbols based on the contents of the `symbol_addrs` configuration.

**Example:**

```yaml
# as list
- [0xABC, asm, filepath1]
- [0xABC, asm, dir1/filepath2]  # this will create filepath2.s inside a directory named dir1

# as dictionary
- name: filepath
  type: asm
  start: 0xABC
```

### `hasm`

**Description:**

Hand-written Assembly, `hasm`, similar to `asm` except it will not overwrite any existing files. Useful when assembly has been manually edited.

**Example:**

```yaml
# as list
- [0xABC, hasm, filepath]

# as dictionary
- name: filepath
  type: hasm
  start: 0xABC
```

## `bin`

**Description:**

The `bin`(ary) segment type is for raw data, or data where the type is yet to be determined, data will be written out as raw `.bin` files.

**Example:**

```yaml
# as list
- [0xABC, bin, filepath]

# as dictionary
- name: filepath
  type: bin
  start: 0xABC
```

## `code`

**Description:**

The 'code' segment type, `code` is a group that can have many `subsegments`. Useful to group sections of code together (e.g. all files part of the same overlay).

**Example:**

```yaml
# must be a dictionary
- name:  main
  type:  code
  start: 0x00001000
  vram:  0x80125900
  subsegments:
    - [0x1000, asm, entrypoint]
    - [0x1050, c, main]
```

## `c`

**Description:**

The C code segments have two behaviors:
- If the target `.c` file does not exist, a new file will be generated with macros to include the original assembly (macros differ for IDO vs GCC compiler).
- Otherwise the target `.c` file is scanned to determine what assembly needs to be extracted from the ROM.

Assembly that is extracted due to a `c` segment will be written to a `nonmatchings` folder, with one function per file.

**Example:**

```yaml
# as list
- [0xABC, c, filepath]

# as dictionary
- name: filepath
  type: c
  start: 0xABC
```

## `header`

**Description:**

This is platform specific; parses the data and interprets as a header for e.g. N64 or PS1 elf.

**Example:**

```yaml
# as list
- [0xABC, header, filepath]

# as dictionary
- name: filepath
  type: header
  start: 0xABC
```

## `data`

**Description:**

Data located in the ROM. Extracted as assembly; integer, float and string types will be attempted to be inferred by the disassembler.

**Example:**

```yaml
# as list
- [0xABC, data, filepath]

# as dictionary
- name: filepath
  type: data
  start: 0xABC
```

This will created `filepath.data.s` within the `asm` folder.

## `.data`

**Description:**

Data located in the ROM that is linked from a C file. Use the `.data` segment to tell the linker to pull the `.data` section from the compiled object of corresponding `c` segment.

**Example:**

```yaml
# as list
- [0xABC, .data, filepath]

# as dictionary
- name: filepath
  type: .data
  start: 0xABC
```

**NOTE:** `splat` will not generate any `.data.s` files for these `.` (dot) sections.

## `rodata`

**Description:**

Read-only data located in the ROM, e.g. floats, strings and jump tables. Extracted as assembly; integer, float and string types will be attempted to be inferred by the disassembler.

**Example:**

```yaml
# as list
- [0xABC, rodata, filepath]

# as dictionary
- name: filepath
  type: rodata
  start: 0xABC
```

This will created `filepath.rodata.s` within the `asm` folder.

## `.rodata`

**Description:**

Read-only data located in the ROM, linked to a C file. Use the `.rodata` segment to tell the linker to pull the `.rodata` section from the compiled object of corresponding `c` segment.

**Example:**

```yaml
# as list
- [0xABC, .rodata, filepath]

# as dictionary
- name: filepath
  type: .rodata
  start: 0xABC
```

**NOTE:** `splat` will not generate any `.rodata.s` files for these `.` (dot) sections.

## `bss`

**Description:**

`bss` is where variables are placed that have been declared but are not given an initial value. These sections are usually discarded from the final binary (although PSX binaries seem to include them!).

Note that the `bss_size` option needs to be set at segment level for `bss` segments to work correctly.

**Example:**

```yaml
- { start: 0x7D1AD0, type: bss, name: filepath, vram: 0x803C0420 }
```
## `.bss`

**Description:**

Links the `.bss` section of the associated `c` file.

**Example:**
```yaml
- { start: 0x7D1AD0, type: .bss, name: filepath, vram: 0x803C0420 }
```

## Images

**Description:**

**splat** supports most of the [N64 image formats](https://n64squid.com/homebrew/n64-sdk/textures/image-formats/):

- `i`, i.e. `i4` and `i8`
- `ia`, i.e. `ia4`, `ia8`, and `ia16`
- `ci`, i.e. `ci4` and `ci8`
- `rgb`, i.e. `rgba32` and `rgba16`

These segments will parse the image data and dump out a `png` file.

**Note:** Using the dictionary syntax allows for richer configuration.

**Example:**

```yaml
# as list
- [0xABC, i4, filename, width, height]
# as a dictionary
- name: filename
  type: i4
  start: 0xABC
  width: 64
  height: 64
  flip_x: yes
  flip_y: no
```