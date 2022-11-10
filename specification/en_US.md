# Specification `BDSF`[^1] 0.3
## `1.` Targets
1. Files in BDSF should take up as little space as possible.
   > Any file in this format should take up the least amount of disk space. Preferably, it should take up the least amount of disk space of `BSON` format.
2. BDSF files should be easy to read and write in all senses (software code is implied).
   > The program executing the BDSF file **MUST NOT** be a heavy load on the system.
3. The file must have no size limitations.
   > Despite the `#1.1.` target, this format should have no data size limitations.
4. Must be suitable for several purposes.
   > This format must be suitable for at least the following purposes: `Storage of data (multiple documents)` and `Transmission of data over a network.

## `2.` Storage
### `2.1.` Basic format
The first byte of the file is responsible for specifying the type of data storage.

If its value is `01` (1), then the multi-document storage format is used:
```py
File {
    Byte type (1);
    Path[] paths (N);
    Document[] documents (N);
}
```
> *Table `#1`
> Block name | Type | Length (bytes : bits) | Description
> ---------- | ---- | --------------------- | ------------
> `type`     | `Byte` | `1 : 8` | 1 byte indicating storage type 
> `paths`    | `Path[]` | `N : B * 8`[^2] | A list of paths to documents (written without type)
> `documents` | `Document[]` | `N : B * 8 `[^2] | List of documents (written without type indication)

If this byte is nulled (value `00`, 0), then the network transmission format (one document) is used:
```py
File {
    Byte type (1);
    Document document (N);
}
```

### `2.2.2.` Initial types (`Path`, `Document`)
#### `2.2.1.` Path
> `BDSFT:<type>` denotes any type from `#2.4.`
```py
Path {
    BDSFT:TV key (1 + N);
    BDSFT:TV[N] coordinates (1 + N);
}
```
> *Table `#2`*
> Block name | Type | Length (bytes : bits) | Description
> -------------- | ------------ | ---------------------- | ---------------------------------------------------------------------------------------
> `key` | `BDSFT:TV` | `1 + N : 8 + (B * 8)`[^2] | Type key from `2.3.`, also denotes document name
> `paths` | `Path[]` | `1 + N : 8 + (B * 8)`[^2] | The number of bytes after which the block with this document begins

#### `2.2.2.` Document
> `BDSFT:<type>` denotes a type from `2.4.`
```py
Document {
    Byte BOD = 0 (1);
    BDSFT:KTV data (N);
    Byte BOD = 0 (1);
```
> *Table `#3`*
> Block name | Type | Length (bytes : bits) | Description
> ---------- | ---- | --------------------- | ---------------------------------------------------------------------------------------
> `BOD`      | `Byte` | `1 : 8` | Always `00`, document end and edge (sides) signature
> `data`     | `BDSFT:KTV` | `N : B * 8`[^2] | key:value pairs (`key:value`)

### `2.3.` Data types
> `BDSFT:<type>` denotes a type from `2.4.`
>
Type               | Length (bytes : bits) | Range                     | Bytecode | Format
------------------ | --------------------- | ------------------------- | -------- | ----------------------------
`Byte`             | `1 : 8`               | `-128 - 127`              | `01`     | `-`
`UInt8`            | `1 : 8`               | `0 - 255`                 | `02`     | `-`
`Int16`            | `2 : 16`              | `-2¹⁵+1 - 2¹⁵-1`          | `03`     | `HEX (big endian, signed)`
`UInt16`           | `2 : 16`              | `0 - 2¹⁶-1`               | `04`     | `HEX (big endian, unsigned)`
`Int32`            | `4 : 32`              | `-2³¹+1 - 2³¹-1`          | `05`     | `HEX (big endian, signed)`
`UInt32`           | `4 : 32`              | `0 - 2³²-1`               | `06`     | `HEX (big endian, unsigned)`
`Int64`            | `8 : 64`              | `-2⁶³+1 - 2⁶³-1`          | `07`     | `HEX (big endian, signed)`
`UInt64`           | `8 : 64`              | `0 - 2⁶⁴-1`               | `08`     | `HEX (big endian, unsigned)`
`Int128`           | `16 : 128`            | `-2¹²⁷+1 - 2¹²⁷-1`        | `09`     | `HEX (big endian, signed)`
`UInt128`          | `16 : 128`            | `0 - 2¹²⁸-1`              | `0A`     | `HEX (big endian, unsigned)`
`Float`            | `4 : 32`              | `~ 6 - 9 .` (`2.3/1`)     | `0B`     | `IEEE 754-2019`
`Double`           | `8 : 64`              | `~ 15 - 17 .` (`2.3/1`)   | `0C`     | `IEEE 754-2019`
`Decimal`          | `16 : 128`            | `28 - 29 .` (`2.3/1`)     | `0D`     | `IEEE 754-2019`
`Boolean`          | `1 : 8`               | `1 - 8`                   | `0E`     | `00 = false; 01 = true`
`String`           | `N : B * 8`[^2]       | `-`                       | `0F`     | `c-string`
`Array`            | `N : B * 8`[^2]       | `-`                       | `10`     | `2.3/2`
`Dictionary`       | `N : B * 8`[^2]       | `-`                       | `11`     | `2.3/3`
`Timestamp`        | `4 : 32`              | `0 - 2147483647`          | `12`     | `Unix Timestamp`
`Timestamp64`      | `8 : 64`              | `0 - 9223372036854775807` | `13`     | `64-bit Timestamp`
`Array[Type]`      | `N : B * 8`[^2]       | `-`                       | `14`     | `2.3/4`
`Dictionary[Type]` | `N : B * 8`[^2]       | `-`                       | `15`     | `2.3/5`
`Null`             | `0 : 0`               | `0 - 0`                   | `16`     | `-`
`ItemID`           | `16 : 128`            | `Byte[16]`                | `17`     | `For now not used`
`PNG Image`        | `N : B * 8`[^2]       | `-`                       | `18`     | `2.3/6`

> #### Note `2.3/1`.
> `.` - Denotes "numbers after the dot"

> #### Note `2.3/2`
> This type consists of two parts
> 1. The content of the list in the format `BDSFT:TV`
> 2. Signature of the end of the list (always `00`)

> #### Note `2.3/3`.
> This type consists of two parts
> 1. The contents of the dictionary in the format `BDSFT:KTV`.
> 2. Signature of the end of the dictionary (always `00`)

> #### Note `2.3/4`.
> This type consists of three parts
> 1. 1 byte, which denotes the type, which will be all the contents of the array
> 2. The contents of the array in the format `BDSFT:V`.
> 3. Signature of the end of the list (always `00`)

> #### Note `2.3/5`.
> This type consists of three parts
> 1. 1 byte, which denotes the type, which will be all the contents of the dictionary
> 2. The contents of the dictionary in the format `BDSFT:KV`.
> 3. Signature of the end of the dictionary (always `00`)

> #### Note `2.3/6`.
> This type is the same PNG, but
> 1. There must not be a PNG signature at the beginning
> 2. There are no fields in the `IHDR` block: `Compression method` and `Filtering method`.
> 3. Block `IEND` is missing.

### `2.4.` Recording format for key-value pairs
> All documents have the following notation: KTV
> 
> Entry format: `<chars>[\[<mtype>\]]`

**Values:**
- `chars` - Characters indicating data blocks
- `mtype` - Basic type

**Characters (`chars`):**
- `K` - Key, TV format of the given point, N bytes
- `T` - Type of value, 1 byte
- `V` - Value itself, N bytes

**Basic types (`mtype`):**
- N` - `Int16`, `Int32`, `Int64`, `Int128`, `UInt8`, `UInt16`, `UInt32`, `UInt64` and `UInt128`
- `F` - `Float`, `Double` and `Decimal128`

[^1]: BDSF *(**Binary Data Storage Format)* - Binary Data Storage Format.
[^2]: `N` = Any number; `B` = Number of bytes.
