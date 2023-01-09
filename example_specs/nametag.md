
> Note: This is a draft to test a TODD-compliant specification for a nametag database.

## Abstract

A specification for a Time Ordered Distributable Database (TODD)-compliant for mapping
Ethreum addresses to names and tags.

## Motivation

Ethereum addresses may be associated with human-relatable tags and names ("nametags" henceforth).
These nametags can be aggregated and used to create front ends for explorers and wallets. One
such database is RolodETH (https://github.com/verynifty/RolodETH).

At present if the nametag database is cloned and distributed it becomes difficult
to sync. Further it is not amenable to distribution amongst peers. This is a specification
that makes a TODD-compliant nametag database.

By following this specification, different client implementations can be written
to manage the creation, distribution, maintenance and consumption of the distributed
database.

## Table of Contents
  - [Abstract](#abstract)
  - [Motivation](#motivation)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
    - [General Structure](#general-structure)
    - [Notation](#notation)
  - [Constants](#constants)
    - [Design parameters](#design-parameters)
    - [Fixed-size type parameters](#fixed-size-type-parameters)
    - [Variable-size type parameters](#variable-size-type-parameters)
    - [Derived](#derived)
  - [Definitions](#definitions)
    - [Database](#database)
    - [`Volume`](#volume)
    - [`Chapter`](#chapter)
    - [`Records`](#records)
    - [`RecordKeys`](#recordkeys)
    - [`RecordValues`](#recordvalues)
  - [Manifest](#manifest)
  - [Interface identifier string schemas](#interface-identifier-string-schemas)
    - [Database interface id](#database-interface-id)
    - [Volume interface id](#volume-interface-id)
    - [Chapter interface id](#chapter-interface-id)

## Overview

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted
as described in RFC 2119 and RFC 8174.

### General Structure

The nametag database has the following high level structure:

- `Volumes` periodically published
    - `Chapters` defined by address starting hex characters
        - `Records`
            - `RecordKey` address a user wants to know about
            - `RecordValue` nametags in JSON format
- Manifest
    - Details about each `Chapter`
    - Link to specification
    - Nametag specification version

Volumes are released periodically and contain `Chapters`.

Users start with an address, consult the
manifest and determine which `Chapters` to obtain, then extract the nametags from those
`Chapters`.

### Notation
Code snippets appearing in `this style` are to be interpreted as Python 3 psuedocode. The
style of the document is intended to be readable by those familiar with the
Ethereum [consensus](#ethereum-consensus-specification) and
[ssz](#ssz-spec) specifications. Part of this
rationale is that SSZ serialization is used in this index to benefit from ecosystem tooling.
No doctesting is applied and functions are not likely to be executable.


## Constants

### Design parameters

| Name | Value | Description |
| - | - | - |
| `ADDRESS_CHARS_SIMILARITY_DEPTH` | `uint32(1)` | A parameter for the number of identical characters that two similar addresses share in hexadecimal representation. If the value is `1`, address`0xabcd...1234` will be evaluated for similarity using the first character `0xa`.
| `NAMETAGS_PER_VOLUME` | `uint32(10*3)` (1000) | Number of nametags in a `Volume` |

### Fixed-size type parameters


| Name | Value | Description |
| - | - | - |
|-|-|-|

### Variable-size type parameters

Helper values for SSZ operations. SSZ variable-size elements require a maximum length field.

| Name | Value | Description |
| - | - | - |
| `MAX_NAMES_PER_RECORD` | `uint(2**8)` (=256) | Max number of names within a single `RecordValue`|
| `MAX_TAGS_PER_RECORD` | `uint(2**8)` (=256) | Max number of tags within a single `RecordValue`|
| `MAX_BYTES_PER_NAME` | `uint32(2**5)` (=32) | Maximum bytes allowed for a single name. |
| `MAX_BYTES_PER_TAG` | `uint32(2**5)` (=32) | Maximum bytes allowed for a single tag. |

### Derived

Constants derived from [design parameters](#design-parameters).

| Name | Value | Description |
| - | - | - |
| `BYTES_FOR_ADDRESS_CHARS` | `uint32((ADDRESS_CHARS_SIMILARITY_DEPTH + 1) // 2)` (=1) | The number of bytes needed to represent the `ADDRESS_CHARS_SIMILARITY_DEPTH` characters. E.g., 1 or 2 chars is 1 byte, 3 or 4 chars is 2 bytes|
| `MAX_RECORDS_PER_CHAPTER` | `uint32(1000)` | Ceiling on number of `Records` per `Chaper`. Equal to `NAMETAGS_PER_VOLUME` (all nametags may be in one chapter). |

## Definitions

### Database

The entire database is represented by the following:

```python
class Database(Container):
    chapters: List[Chapter, MAX_CHAPTERS]
```
### `Volume`

Cadence: A `Volume` MAY be published when it contains exactly `NAMETAGS_PER_VOLUME` nametags.


`VolumeId`: The number of the first nametag in the volume. The database consists of
nametags (`RecordValues`) that are appended and given an incremental counter. Thus, a
Volume with a `VolumeId` of `740_000` starts with the `740_000`-th nametag, and ends with
the `740_000 + NAMETAGS_PER_VOLUME - 1`-th nametag. E.g., [740_999, 740_999] inclusive.

```python
VolumeId: uint32
```

### `Chapter`
Definition: A `Chapter` MUST contain data for addresses that share the same starting hex character.

```python
class Chapter(Container):
    volume_id: VolumeId
    chapter_id: ChapterId
    records: List[Record, MAX_RECORDS_PER_CHAPTER]
```

```python
ChapterId: ByteVector[BYTES_FOR_ADDRESS_CHARS]
```

### `Records`

`Record` container definition:
```python
class Record(Container):
    record_key: RecordKey
    record_value: RecordValue
```
### `RecordKeys`

`RecordKey` definition:
```python
RecordKey: ByteVector[BYTES_FOR_ADDRESS_CHARS]
```

### `RecordValues`

`RecordValue` container definition:
```python
class RecordValue(Container):
    names: List[Name, MAX_NAMES_PER_RECORD]
    names: List[Tag, MAX_TAGS_PER_RECORD]
```

Name definition:
```python
Name: List[MAX_BYTES_PER_NAME]
```
Tag definition:
```python
Tag: List[MAX_BYTES_PER_TAG]
```

## Manifest

Version: SemVer
Schemas: Link to this document.

Chapters appear as a list in the format below, where "volume" and "chapter"
values are the `VolumeInterfaceId` and `ChapterInterfaceId` respectively.

Example:

```json
{
  "version": "0.0.1",
  "schemas": "url.todo",
  "chapters": [
    {
      "volume": "nametags_from_000000000",
      "chapter": "addresses_0xa",
      "CID": "Qm1234...wxyz"
    },
    {
      "volume": "nametags_from_000000000",
      "chapter": "addresses_0xb",
      "CID": "Qm1234...wxyz"
    },
    ...
    {
      "volume": "nametags_from_000750000",
      "chapter": "addresses_0xe",
      "CID": "Qm1234...wxyz"
    },
    {
      "volume": "nametags_from_000750000",
      "chapter": "addresses_0xf",
      "CID": "Qm1234...wxyz"
    },
  ]
}
```
## Interface identifier string schemas
### Database interface id

`DatabaseInterfaceId`: "/^nametags$/"

```
nametags
```

### Volume interface id

The number of the first nametag in the volume.
`VolumeInterfaceId`: "/^nametags_from(_[0-9]{3}){3}$/"
```
nametags_from_000_630_000
```


### Chapter interface id

`ChapterInterfaceId`: "/^addresses_0x[a-z0-9]{1}$/"
```
addresses_0xa
```
