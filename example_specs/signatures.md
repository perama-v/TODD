
> Note: This is a draft to test a TODD-compliant specification for a signature database.

## Abstract

A specification for a Time Ordered Distributable Database (TODD)-compliant for translating
hex signatures to text signatures.

## Motivation

In the Ethereum context, methods and events have signatures that are derived
from a human readable text string. To allow human introspection, signatures
can be translated to the original text. To facilitate this, mappings
are aggregated in a database such as
- [https://github.com/ethereum-lists/4bytes](https://github.com/ethereum-lists/4bytes) (2.3 GB), which is provided with new data from the [4byte.directory](4byte.directory) website that encourages visitors to submit new mappings.


At present if the signature database is cloned and distributed it becomes difficult
to sync. Further it is not amenable to distribution amongst peers. This is a specification
that makes a TODD-compliant signature database.

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

The signature database has the following high level structure:

- `Volumes` periodically published
    - `Chapters` defined by signature starting hex characters
        - `Records`
            - `RecordKey` hex signature a user wants to know about
            - `RecordValue` readable text signatures in utf-8 encoded format
- Manifest
    - Details about each `Chapter`
    - Link to specification
    - Signature specification version

Volumes are released periodically and contain `Chapters`.

Users start with a signature, consult the
manifest and determine which `Chapters` to obtain, then extract the signatures from those
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
| `SIGNATURE_CHARS_SIMILARITY_DEPTH` | `uint32(2)` | A parameter for the number of identical characters that two similar signatures share in hexadecimal representation. If the value is `2`, signature`0xabcd1234` will be evaluated for similarity using the characters `0xab`.
| `SIGNATURES_PER_VOLUME` | `uint32(10*3)` (1000) | Number of signatures in a `Volume` |

### Fixed-size type parameters


| Name | Value | Description |
| - | - | - |
| `BYTES_PER_SIGNATURE` | `uint32(4)` (4) | Number of bytes used to represent a signature. |

### Variable-size type parameters

Helper values for SSZ operations. SSZ variable-size elements require a maximum length field.

| Name | Value | Description |
| - | - | - |
| `MAX_TEXTS_PER_RECORD` | `uint(2**8)` (=256) | Max number of texts within a single `RecordValue`|
| `MAX_BYTES_PER_TEXT` | `uint32(2**5)` (=32) | Maximum bytes allowed for a single text. |

### Derived

Constants derived from [design parameters](#design-parameters).

| Name | Value | Description |
| - | - | - |
| `BYTES_FOR_SIGNATURE_CHARS` | `uint32((SIGNATURE_CHARS_SIMILARITY_DEPTH + 1) // 2)` (=1) | The number of bytes needed to represent the `SIGNATURE_CHARS_SIMILARITY_DEPTH` characters. E.g., 1 or 2 chars is 1 byte, 3 or 4 chars is 2 bytes|
| `MAX_RECORDS_PER_CHAPTER` | `uint32(1000)` | Ceiling on number of `Records` per `Chaper`. Equal to `SIGNATURES_PER_VOLUME` (all signatures may be in one chapter). |

## Definitions

### Database

The entire database is represented by the following:

```python
class Database(Container):
    chapters: List[Chapter, MAX_CHAPTERS]
```
### `Volume`

Cadence: A `Volume` MAY be published when it contains exactly `SIGNATURES_PER_VOLUME` signatures.


`VolumeId`: The number of the first signature in the volume. The database consists of
signatures (`RecordValues`) that are appended and given an incremental counter. Thus, a
Volume with a `VolumeId` of `740_000` starts with the `740_000`-th signature, and ends with
the `740_000 + SIGNATURES_PER_VOLUME - 1`-th signature. E.g., [740_999, 740_999] inclusive.

```python
VolumeId: uint32
```

### `Chapter`
Definition: A `Chapter` MUST contain data for signatures that share the same starting hex character.

```python
class Chapter(Container):
    volume_id: VolumeId
    chapter_id: ChapterId
    records: List[Record, MAX_RECORDS_PER_CHAPTER]
```

```python
ChapterId: ByteVector[BYTES_FOR_SIGNATURE_CHARS]
```

### `Records`

`Record` container definition:
```python
class Record(Container):
    record_key: RecordKey
    record_value: RecordValue
```
### `RecordKeys`

A key is a signature that is 8 hex characters long (2 chars per byte = 4 bytes).
The signature is stored without the "0x" prefix.

`RecordKey` definition:
```python
class RecordKey(Container):
    key: ByteVector[BYTES_PER_SIGNATURE]
```

### `RecordValues`

`RecordValue` container definition:
```python
class RecordValue(Container):
    texts: List[Text, MAX_TEXTS_PER_RECORD]
```

Text definition:
```python
Text: List[u8, MAX_BYTES_PER_TEXT]
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
      "volume": "signatures_from_000_000_000",
      "chapter": "signatures_0x00",
      "CID": "Qm1234...wxyz"
    },
    {
      "volume": "signatures_from_000_000_000",
      "chapter": "signatures_0x01",
      "CID": "Qm1234...wxyz"
    },
    ...
    {
      "volume": "signatures_from_000_750_000",
      "chapter": "signatures_0xfe",
      "CID": "Qm1234...wxyz"
    },
    {
      "volume": "signatures_from_000_750_000",
      "chapter": "signatures_0xff",
      "CID": "Qm1234...wxyz"
    },
  ]
}
```
## Interface identifier string schemas
### Database interface id

`DatabaseInterfaceId`: "/^signatures$/"

```
signatures
```

### Volume interface id

The number of the first signature in the volume.
`VolumeInterfaceId`: "/^signatures_from(_[0-9]{3}){3}$/"
```
signatures_from_000_630_000
```


### Chapter interface id

`ChapterInterfaceId`: "/^signatures_0x[a-z0-9]{2}$/"
```
signatures_0xac
```
