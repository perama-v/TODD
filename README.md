---
eip: <to be assigned>
title: Time Ordered Distributable Database
description: A format for useful peer-to-peer databases.
author: perama (@perama-v)
discussions-to: <URL>
status: Draft
type: Standards Track
category (*only required for Standards Track): ERC
created: <date created on, in ISO 8601 (yyyy-mm-dd) format>
requires (*optional): <EIP number(s)>
---

> Note: This is formulated in the style of an EIP as a thought exercise for now.
It is not clear that the concept is suited for an EIP. If that changes, I'll update this
comment.

## Abstract

A format for a database comprising time-ordered dependent chunks that
facilitate distribution on content-addressable storage media. The format
makes explicit the ability to add new data without breaking identifiers for existing data.
Data is also broken into pieces that encourage users to host subsets of the whole.

## Motivation

Databases in the Ethereum ecosystem are often intended to be public goods. Yet it is
a challenge to distribute a database that changes over time and it is often
more helpful to end users if a single actor maintains a reliable, complete and up to
date database. These services are often provided generously with free and scrapeable APIs
and public repositories.

Two changes can make these databases more amenable to peer to peer distribution:
1. Break the data into time-ordered chunks with content identifiers (CIDs) that remain constant
with the addition of new data.
2. Divide the data into desirable subsets to encourage users to host retrievable portions
of the whole.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted
as described in RFC 2119 and RFC 8174.

### General Structure
The database is defined by a schema with tunable parameters.

The database consists of individually-accessible units called `Chapters`. Data within
a `Chapter` share similar properties. `Chapters` are released as part of
discrete `Volumes` that come out at some cadence. After a `Volume` is released, the data
does not change.

`Chapters` consist of data that is easily recognisable as desirable for a particular user.
When a new `Volume` is released, a user can ignore irrelevant `Chapters`. This provides
a mechanism to shard the database across users. From a user's perpsective this
is analgous to "subscribing" to a `Chapter` that is important to them.

When a new `Volume` is released, two users may obtain different `Chapters`.

A compliant database:
- MUST define a time-based definition for `Volumes` (how often new data comes out)
- MUST define a user-focused definition for `Chapters` (what the difference between chapters is)

A `Chapter` is the distributable unit of the database. It is defined by a
unique `Volume`-`Chapter` pair. A `Chapter` contains data as key-value pairs
where keys all match the `Chapter` definition and can be used by a user to obtain values.

Fundamental to this idea is that a user starts with a query. They inspect the chapters
they have obtained and obtain values that match the query. The combined result of
the values from each chapter is the complete result of that query

### Volume definition

A `Volume` is a collection of data that is constructed and published at a cadence
using a `Volume` definition. A specific `Volume` is described by a specific `VolumeId`
and consists of many individual discrete `Chapters`.

A `Volume` definition MUST provide a clear protocol for deciding when a threshold for
publication is reached. The nature of the threshold will vary between databases.

A unique `VolumeId` MUST be specified for each `Volume`. For example, using bytes
to represent a block range particular to that `Volume`. This is different to the the interface
identifier string.

It is RECOMMENDED that the definition be defined similarly to the way new data is collected.
Where there is a sequential nature to the data collection, this may be a good mechanism for
determining a threshold. Where data is stored unordered, it is RECOMMENDED
that an articifial ordering used.

For example:
- Data is exclusively derived from new sequential mainnet blocks.
    - Threshold every nth new addresses appearance detected during transaction tracing.
    - Example database: UnchainedIndex
- Data is exclusively derived from new sequential mainnet blocks.
    - Threshold every nth block of new data.
    - Example database: address-appearance-index
- Data consists of user submitted string->hash mappings that are given an incrementing index
    - Threshold every nth submitted mapping.
    - Example database: 4byte directory
- Data consists of contract metadata stored in a key-value database by address.
    - Define any ordering system for existing records (date added, lexicographically, ...)
    - Threshold every nth new addition.
    - Example database: Sourcify repository

The threshold `MAY` be subject to inter-observer disagreement. Two publishers may produce
two different volumes and users may use either or both databases. Where agreement is required
due to the nature of the data, social coordination may be required to decide a "canonical"
volume to continue building upon.

### Chapter definition

A `Chapter` is a collection of data `Records` that are constructed using a `Chapter` definition
and is published as a part of a `Volume`. A specific chapter is described by a
unique `VolumeId` and `ChapterId` pair.

The following details apply to a databases that define `Chapters`.

A `Chapter` definition MUST be identifiable by knowledge that the end user posesses. They represent
subsets of the data that they can obtain that is relevant to their needs.

A unique `ChapterId` MUST be specified for each `Chapter`. For example, using bytes
to represent a the hex characters common to all addresses particular to that `Chapter`.
This is different to the the interface identifier string.

It is RECOMMENDED that `Chapter` definitions are such that they divide the index
into shards that are small enough for a user to host, but large enough
such that the number of users required to host the complete database is achievable.

Some examples:
- User has an address `0x1234...abcd`
    - Chapters represent sections of the address namespace. Data in each chapter
    is for address that all start with the same 2 hex characters.
    - User obtains `chapter_0x12`
    - Example database: address-appearance-index
- User has a method signature `0xabcd1234`
    - Chapters represent sections of the signature namespace. Data in each chapter
    is for signature that all start with the same hex character.
    - User obtains `chapter_0xa`
    - Example database: 4byte directory
- User has a contract metadata identifier `Qm1234...wxyz`
    - Chapters represent sections of the metadata CID namespace. Data in each chapter
    is for CIDs that all start with the same base-58 character.
    - User obtains `chapter_Qm1`
    - Example database: Sourcify repostory

A `Chapter` definition MAY be broad enough to include all possible `Records`. This
mean that a `Volume` only has one `Chapter`. This may be appropriate where a
database is small enough to not warrant dividing into smaller pieces, or where a database
has an alternative method for users to determine if a Chapter is desired (such as bloom filters).

### Records

A `Chapter` consists of a collection of key-value pairs called `Records`.

A `Record` MUST consist of `RecordKey`-`RecordValue` pairs.

The format of a `Record` MUST be specified. For example a record for a specific
database may consist of a key that is a string and a value that is a
container of items conforming to some encoding.

### RecordKeys

A `Record` consists of a `RecordKey`-`RecordValue` pair.

A `RecordKey` MUST represent a user query. For example: an Ethereum address may be a `RecordKey`
if the user will use addresses to look up data.

A key MUST conform to the defintion of a `Chapter`. For example: `chapter_0x12` consists of
`RecordKey` that all start with `0x12...`.

A `RecordKey` MUST NOT appear in a `Chapter` more than once.

### RecordValues

A `Record` consists of a `RecordKey`-`RecordValue` pair. Multiple `RecordValues` are
aggregated to form the response to a user query.

A `RecordValue` MAY be a collection of data. For example: The `RecordKey` `0x1234...acbd` may map to
an array of transaction ids.

A value in a `Chapter` MUST conform to the definition of the `Volume` that the `Chapter`
belongs to. For example: If a `Volume` is defined by a block range,
then a `RecordValue` must consist of transactions within that range.

### Manifest

When a new `Volume` is released, a manifest SHOULD be published. It consists of a complete record of
the CID of every `Chapter`. By obtaining a manifest from a publisher,
a user can quickly ascertain the CID of the data that is valuable to them.

The Manifest MUST be JSON formatted document.

The Manifest MUST have a "version" field, whose value is RECOMMENDED to represent the semantic
version of the data structure.

The Manifest MUST have a "schemas" field, whose value is a string that is RECOMMENDED to
either represent the CID of the schema and/or protocols that define the data and its
preparation and use. The value MAY instead be a string representing the schema itself.
It is NOT RECOMMENDED that the value be a URL that may change.

The Manifest MUST list all chapters.

The Manifest MAY, for each unique distributable database element, included additional unique
content identifiers. For example, in addition to an IPFS CID, a swarm hash and SSZ root hash
may be included for each element.

It is RECOMMENDED that when a new manifest is published, the CID of the manifest is broadcast
to relevant parties. For example, the CID of the manifest could be announced to connected peers
in a peer to peer network, or stored in smart contract that implements
ERC-generic-attributable-manifest-broadcaster.

### Interface Identifiers

It is RECOMMENDED that the database schema specification (referred to in the "schemas" field of
the Manifest) define an identifier schema for the database, `Volumes` and `Chapters`.
These identifiers allow data to be referred to via application programming interfaces (APIs).

If defined, they SHOULD be defined in a section called: "Interface identifier string schemas",
using the following terms:

- database interface id
- volume interface id
- chapter interface id

The purpose of these identifiers is to enable interfaces to refer to the database and its components
For example, a RESTful API might define an endpoint for a
database that indexes the appearances of address inside transactions as follows:

```
Example definition:
{database_interface_id}/v1/data/by_volume_and_chapter/{volume_interface_id}/{chapter_interface_id}

Example call:
address-appearance-index-mainnet/v1/data/by_volume_and_chapter/blocks-001300000-001305432/addresses_starting_0x3f
```

An identifier SHOULD be a string specified by regular expression of following general form:
```
"/^+[a-zA-Z0-9_-]$/"
```
That is, schemas should describe regular expressions that are limited to any combination of:
letters, numbers, "_" and "-".

Example database identifier for a database that manages mappings of strings to hex signatures:
```
Database interface id:
"/^4byte-directory$/"

Example: "4byte-directory"
```
Example schema for `Volumes` that are defined by block heights:
```
Volume interface id:
"/^blocks-[0-9]{9}-[0-9]{9}$/"

Example: "blocks-001300000-001305432"
```
Example schema for `Volumes` that are defined by an incremental counter of new entries:
```
Volume interface id:
"/^[0-9]{6}$/"

Example: "091233"
```
Example schema for `Chapters` that are defined by the first two hex characters of addresses:
```
Chapter interface id:
"/^addresses_starting_0x[a-z0-9]{2}$/"

Example: "addresses_starting_0x3f"
```
Example schema for `Chapters` that are defined by the first hex character of a signature:
```
Chapter interface id:
"/^[a-z0-9]{1}$/"

Example: "d"
```

### Flat structure

All files are uniquely identified by the `Volume` and (if used) `Chapter`. As such, the
database is able to be represented as a list of elements, one per CID.

```
[volume_1_chapter_1, volume_1_chapter_2, ..., volume_2_chapter_1, volume_2_chapter_2,  ...]
```

The Manifest MUST contain all the unique elements with their CIDs. The JSON format for
these data is an implementation detail, but MUST be defined in the linked document under the
manifest "schema" field.

Implementations MAY represent and store the actual data any way including flat files nested
directories or key-value databases.

However, the manifest MUST NOT nest CIDs in a way that prevents a user from immediately
reading the individual file CIDs directly from the manifest.

## Rationale

The `Volume` cadence provides a mechanism for immutable CIDs. Publishers prepare new
data but do not release it as a `Volume` until strict criteria are met.
This discrete release model allows new data to be shared and incorporated in peer to peer
networks without disrupting CIDs for existing data.

The `Chapter` definition provides a mechanism for database partitioning. Users obtain
a functional shard of the database for personal gain. This provides an opportunity for
that person to seamlessly share the data without incurring the full burden of the whole
database.

In existing systems where data is published with content identifiers (CIDs)
such as the Interplanetary File
System (IPFS), the publisher may update records with a naming system such as then InterPlanetary
Name System (IPNS) or the Ethereum Name System (ENS). This becomes brittle if the main
provider ceases their operation. The manifest is a solution that allows different publishers to
convene and broadcast in the same place (forum, peer to peer network or smart contract).

The database format provides a path to convert a traditional (client-server architecture) database
into a distributed database. This allows benevolent actors who currently run database services
to continue publishing in a way that allows other individuals participate. User software
can act as to run software that:

- Consumer (fetch existing data for personal use and then start hosting it by default)
- Published (create new data and publish manifest for others to acquire)

The rationale for this architecture is predicated on the idea that many users are happy
to help out, and will consent to reasonable software defaults. For example, a client may
start serving data to peers automatically consuming a small disk and bandwidth footprint.
In aggregate, this would create a functional and robust network.

## Reference Implementation

The following are compliant reference implementations:

### UnchainedIndex

The UnchainedIndex comprises an database that records the transaction ids (block and index)
for all addresses that appear during transaction execution.
- `Volumes` (known as "chunks") are published every `2_000_000` address appearances.
- Each `Volume` has a single `Chapter`. Users obtain bloom filters to determine
if the `Chapters`/"chunks" are of interest for their address.

This corresponds to one ~25MB indivisible `Chapter` published every ~12 hours.

A snippet of a particular published manifest has the following form, with the required fields
as follows:
- "version" field: The version of the software used to create this instance of the database.
- "schemas" field: An IPFS CID for the specification of the UnchainedIndex
- `Chapter` are listed, each with their definition ("range" field) and CID ("indexHash" field).

```
{
  "version": "trueblocks-core@v0.40.0",
  "chain": "mainnet",
  "schemas": "QmUou7zX2g2tY58LP1A2GyP5RF9nbJsoxKTp299ah3svgb",
  "config": {
    "appsPerChunk": 2000000,
    "snapToGrid": 100000,
    "firstSnap": 2300000,
    "unripeDist": 28
  },
  "chunks": [
    {
      "range": "000000000-000000000",
      "bloomHash": "QmYhuaJu9bHAGpSsuaQug7bjnLcoE6B5PCZsg4XZGVFbKy",
      "bloomSize": 131114,
      "indexHash": "QmaKUsfH5AXqPJgjGAQFZWRheF1jtc5PqGi8cYrwmMCXdu",
      "indexSize": 320192
    },
    {
      "range": "000000001-000590502",
      "bloomHash": "QmZxFR7NmeUvyLoynX4cpN2DFKukbaFq78jmBAyGLeUkYD",
      "bloomSize": 131114,
      "indexHash": "Qmehw7ebshWsBQV4ZZLaEqETkttguRAonoQKMqVeumP9Fy",
      "indexSize": 16821060
    },
    ...
  ]
}
```
The CID of this manifest is broadcast by storing it in the smart contract
that implements ERC-generic-attributable-manifest-broadcaster, deployed on mainnet at
`0x0c316b7042b419d07d343f2f4f5bd54ff731183d`:

### address-appearance-index
The address-appearance-index comprises a database that records the transaction ids (block and
index) for all addresses that appear during transaction execution.
- `Volumes` are defined every `100_000` blocks.
- `Chapters` are defined as containing data for addresses that all share common first two
hex characters.

This corresponds to one ~500mb `Volume` published every ~2 weeks, comprising 256 ~2MB `Chapters`
that may be individually obtained.

Example manifest file demonstration. Files are nested but all CIDs are available:
```
{
  "version": "0.0.1",
  "schemas": "Qm1234...wxyz",
  "volumes": [
    {
      "volume_id": 0,
      "chapters": [
        {
          "chapter_id": "0x00"
          "cid": "Qm1234...wxyz"
        },
        {
          "chapter_id": "0x01"
          "cid": "Qm1234...wxyz"
        },
        ...
        {
          "chapter_id": "0xff"
          "cid": "Qm1234...wxyz"
        },
      ]
    },
    {
      "volume_id": 100_000,
      "chapters": [
        {
          "chapter_id": "0x00"
          "cid": "Qm1234...wxyz"
        },
        {
          "chapter_id": "0x01"
          "cid": "Qm1234...wxyz"
        },
        ...
        {
          "chapter_id": "0xff"
          "cid": "Qm1234...wxyz"
        },
      ]
    }
  ]
}
```
## Parameter Choices

### Design space
The definition of a `Chapter` determines how much the database will be split. If chosen
well, the database will naturally have a high replication number `R` amongst peers despite
having a low disk footprint for each peer.

|-|100 users|1000 users| 10000 users|
|-|-|-|-|
|0.01% of data|R0.01|R0.1|R1|
|0.1% of data|R0.1|R1|R10|
|1% of data|R1|R10|R100|
|10% of data|R10|R100|R1000|

Table 1: Replication factor `R`, where `R2` indicates that every piece of data in the database has two copies. Any `R<1` indicates that the data is not sufficiently represented in the network. A high `R` makes the data more available and resistant to loss.

It must be noted that a user only contributes to the `R` value if they are actively
participating in peer-to-peer sharing.

A consideration for some databases is that single user may want multiple `Chapters`.
Below is the effect on percentage of database obtained by the `Chapter` definition and
`Chapter count per user.

|-|1 `Chapter` per user|10 `Chapters` per user|100 `Chapters` per user|
|-|-|-|-|
|16^1 = 16 `Chapters` (0x0-0xf) |6.3% downloaded|63%|630%|
|16^2 = 256 `Chapters` (0x00-0xff) |0.39%|3.9%|39%|
|16^3 = 4096 `Chapters` (0x000-0xfff) |0.024%|0.24%|2.4%|
|16^4 = 65,536 `Chapters` (0x0000-0xffff) |0.0015%|0.015%|0.15%|

Table 2: Percentage of the entire database that a single user obtains when they get every
`Chapter` relevant to them. `Chapters` are defined by a different number of
leading hexadecimal characters (resulting in the number of `Chapters` being a function of hexadecimal orders of magnitude), but other definitions could be used.

### Example parameter selection

By starting with the known parameters for a particular database, the number of `Chapters`
can be deduced.

For example, consider a database of indexed addresseses that a wallet user will acquire
part of. `Chapters` are defined by the addresses they contain.

Supppose the following:
- An estimated 1000 users
- A target replication factor of `R10`

Using the Table 1, this results in a user needing to obtain a minimum of 1% of the data to effectively
support the network.

If a user has on average ~10 wallet addresses (10 `Chapters` per user), then Table 2 results
in a choice of 3.9% per user, equating to 256 `Chapters`. Hence `Chapters` should be defined
as having addresses that have the first two hex characters in common (0x00-0xff).

## Security Considerations

Independent parties may publish different data or data that is incorrect,
out of date or inaccurate. It is the responsibility
of the user to find a trustworthy publisher.

The verion and schema sections of the data provide a means for users to navigate breaking changes
to the structure and content of the data.

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).