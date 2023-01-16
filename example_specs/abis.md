
> Note: This is a draft to test a TODD-compliant specification for an ABI database.

## Abstract

A specification for a Time Ordered Distributable Database (TODD)-compliant for abi
Ethereum addresses to ABIs.

## Motivation

Ethereum contracts have interfaces that are used to interact with the code. For example,
deciphering the nature of an event observed in a historical transaction. Making ABIs
available readily could allow more peer-to-peer applications that do not rely on APIs or
single third parties.

Some considerations are:
- Some ABIs don't have parameter names.
- Some ABIs do not have a verified source.

The database does not make claims about correctness, it focuses on making data available
to users, who can themselves amplify the data. There are mechanisms to verify that
an ABI matches the source code at contract deployment, however this is beyond the scope of
this work. This is mostly because it requires using specific compiler versions and
the original source code.

Security:
- Given that the ABI is not source-code verified, it is possible for a publisher to
produce an incorrect ABI. For this reason, it is not recommended to use the ABIs
in the database for important actions, such as to create and send transactions.
Consider using Sourcify.dev
([https://docs.sourcify.dev/docs/intro/](https://docs.sourcify.dev/docs/intro/)) if this
is required.

Additional notes:
- The Sourcify database contains rigorously checked source code and metadata, which is critical
for some use cases. The data is hosted on IPFS with an IPNS pin, and users can download the
entire database (~16 GB). As new data is added (through [sourcify.dev](sourcify.dev)), the
data CIDs of the database directories change, but the IPNS still allows users to obtain
a specific contract ABI by directory traversal. The property of changing directory CIDs makes it
difficult for end users to host subsets
of the database for distribution amongst peers.

This is a specification that makes a TODD-compliant ABI database.
By following this specification, different client
implementations can be written to manage the creation, distribution, maintenance
and consumption of the distributed database.

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

The ABI database has the following high level structure:

- `Volumes` periodically published
    - `Chapters` defined by address starting hex characters
        - `Records`
            - `RecordKey` address a user wants to know about
            - `RecordValue` contract ABI in JSON encoded format
- Manifest
    - Details about each `Chapter`
    - Link to specification
    - Abi specification version

Volumes are released periodically and contain `Chapters`.

Users start with an address, consult the
manifest and determine which `Chapters` to obtain, then extract the ABI from one of those
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
| `ADDRESS_CHARS_SIMILARITY_DEPTH` | `uint32(2)` | A parameter for the number of identical characters that two similar addresses share in hexadecimal representation. If the value is `2`, address `0xabcd...1234` will be evaluated for similarity using the characters `0xab`.
| `ABIS_PER_VOLUME` | `uint32(10*3)` (1000) | Number of ABIs in a `Volume` |

### Fixed-size type parameters


| Name | Value | Description |
| - | - | - |
| `BYTES_PER_ADDRESS` | `uint32(20)` (20) | Number of bytes used to represent an address. |

### Variable-size type parameters

Helper values for SSZ operations. SSZ variable-size elements require a maximum length field.

| Name | Value | Description |
| - | - | - |
| `MAX_TEXTS_PER_RECORD` | `uint(2**8)` (=256) | Max number of texts within a single `RecordValue`|
| `MAX_BYTES_PER_ABI` | `uint32(2**8)` (=256) | Maximum bytes allowed for a single ABI. |

### Derived

Constants derived from [design parameters](#design-parameters).

| Name | Value | Description |
| - | - | - |
| `BYTES_FOR_ADDRESS_CHARS` | `uint32((ADDRESS_CHARS_SIMILARITY_DEPTH + 1) // 2)` (=1) | The number of bytes needed to represent the `ADDRESS_CHARS_SIMILARITY_DEPTH` characters. E.g., 1 or 2 chars is 1 byte, 3 or 4 chars is 2 bytes|
| `MAX_RECORDS_PER_CHAPTER` | `uint32(1000)` | Ceiling on number of `Records` per `Chaper`. Equal to `ABIS_PER_VOLUME` (all ABIs may be in one chapter). |

## Definitions

### Database

The entire database is represented by the following:

```python
class Database(Container):
    chapters: List[Chapter, MAX_CHAPTERS]
```
### `Volume`

Cadence: A `Volume` MAY be published when it contains exactly `ABIS_PER_VOLUME` ABIs.


`VolumeId`: The number of the first ABI in the volume. The database consists of
ABIs (`RecordValues`) that are appended and given an incremental counter. Thus, a
Volume with a `VolumeId` of `740_000` starts with the `740_000`-th ABI, and ends with
the `740_000 + ABIS_PER_VOLUME - 1`-th ABI. E.g., [740_999, 740_999] inclusive.

```python
VolumeId: uint32
```

### `Chapter`
Definition: A `Chapter` MUST contain data for ABIs that share the same starting hex character.

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

A key is an address that is 20 hex characters long (2 chars per byte = 4 bytes).
The address is stored without the "0x" prefix.

`RecordKey` definition:
```python
class RecordKey(Container):
    key: ByteVector[BYTES_PER_ADDRESS]
```

### `RecordValues`

`RecordValue` container definition:
```python
class RecordValue(Container):
    abi: List[u8, MAX_BYTES_PER_ABI]
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
      "volume": "abis_from_000_000_000",
      "chapter": "addresses_0x00",
      "CID": "Qm1234...wxyz"
    },
    {
      "volume": "abis_from_000_000_000",
      "chapter": "addresses_0x01",
      "CID": "Qm1234...wxyz"
    },
    ...
    {
      "volume": "abis_from_000_750_000",
      "chapter": "addresses_0xfe",
      "CID": "Qm1234...wxyz"
    },
    {
      "volume": "abis_from_000_750_000",
      "chapter": "addresses_0xff",
      "CID": "Qm1234...wxyz"
    },
  ]
}
```
## Interface identifier string schemas
### Database interface id

`DatabaseInterfaceId`: "/^abis$/"

```
abis
```

### Volume interface id

The number of the first ABI in the volume.
`VolumeInterfaceId`: "/^abis_from(_[0-9]{3}){3}$/"
```
abis_from_000_630_000
```


### Chapter interface id

`ChapterInterfaceId`: "/^addresses_0x[a-z0-9]{2}$/"
```
addresses_0xac
```
