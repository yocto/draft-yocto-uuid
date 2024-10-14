---
v: 3
# ipr: none
cat: info
obsoletes: '4122'
submissiontype: independent
title: Universally Unique IDentifier
abbrev: UUID
author:
- name: B.J. | van Hartingsveldt
  org: Yocto

--- abstract

TBD

--- middle

# Introduction

This specification defines the formats of UUIDs (Universally Unique IDentifier),
also known as GUIDs (Globally Unique IDentifier). The specification also
registers a Uniform Resource Name namespace for the UUID. A UUID is 128 bits
long, and requires no central registration process.

{::boilerplate bcp14-tagged}

# Motivation

One of the main reasons for using UUIDs is that no centralized authority
is required to administer them. As a result, generation on demand can be
completely automated, and used for a variety of purposes. The UUID generation
algorithm described here supports very high allocation rates of up to 10
million per second per machine if necessary, so that they could even be used
as transaction IDs.

UUIDs are of a fixed size (128 bits) which is reasonably small compared to
other alternatives. This lends itself well to sorting, ordering, and hashing
of all sorts, storing in databases, simple allocation, and ease of programming
in general.

Since UUIDs are unique and persistent, they make excellent Uniform Resource
Names. The unique ability to generate a new UUID without a registration process
allows for UUIDs to be one of the URNs with the lowest minting cost.


# Specification

This section describes the UUID format on bit level. All UUIDs are 128 bits
long.

## Variants {#variants}

The variant of an UUID is defined by one octet. At the moment, the following
variants are used:

|                       Bits | Values (hexadecimal) | Values (decimal) | Description                                               |
|                   0xxxxxxx | 0x00 - 0x7F          | 0 - 127          | Apollo Computer NCS (including Nil-UUID)                  |
|                   10xxxxxx | 0x80 - 0xBF          | 128 - 191        | Open Software Foundation DCE (including Alternating-UUID) |
|                   110xxxxx | 0xC0 - 0xDF          | 192 - 223        | Microsoft DCOM                                            |
| 111xxxxx (except 11111111) | 0xE0 - 0xFE          | 224 - 254        | Reserved                                                  |
|                   11111111 | 0xFF                 | 255              | Omni-UUID                                                 |
{: title='UUID Variant Registry'}

### Apollo Computer NCS

The UUID was first used by Apollo Computer for NCS. It had the following
format:


~~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           time_high                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           time_low            |           reserved            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|    family     |                                               |
+-+-+-+-+-+-+-+-+                     node                      |
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~
{: title='The legacy format'}


* `time_high` (32 bits): The high part of the time.

* `time_low` (16 bits): The low part of the time.

* `reserved` (16 bits): Reserved. All zeros.

* `family` (8 bits): Address family. Defines the type of address in the node field.
  See {{families}} for possible values.

* `node` (56 bits): Node. Format defined by the family field.

The `family` field and and `variant` field of {{variants}} are the same octet. The family field was specified to hold the values 0-255.
However, because only the values 0-13 were used, it was decided that the
values 0-127 were still legacy format, but the values 128-255 were used to
define other UUID variants.


### Open Software Foundation DCE

In RFC4122, a new variant was described in RFC4122 with some backward compatibility
to the older format:


~~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           time_low                            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           time_mid            |      time_hi_and_version      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|clk_seq_hi_res |  clk_seq_low  |                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+             node              |
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~
{: title='The format in RFC4122.'}


* `time_low` (32 bits): The low part of the time.

* `time_mid` (16 bits): The mid part of the time.

* `time_hi_and_version` (16 bits): The high part of the time multiplexed with the version.

* `clock_seq_hi_and_reserved` (8 bits): The high part of the clock sequence multiplexed with variant.

* `clk_seq_low` (8 bits): The low part of the clock sequence.

* `node` (48 bits): Node.

In this variant, the first two bits (10) identify the variant, the rest of `clk_seq_hi_res` is used for the clock sequence. The first 4 bits of `time_hi_and_version` define the version of the UUID. See for {{versions}} for possible versions. The remaining bits are used for time.


### Microsoft DCOM

The format of this variant isn't known at the moment.


### Omni-UUID

See {{specialTypes-omni}}.



## Address Families {#families}

The family field starts with a 0-bit if it is the legacy format and with
a 1-bit if it describes another variant. This gives 7 bits to describe family
addresses in the legacy format. The following families are described:

|    Bits | Values (hexadecimal) | Values (decimal) | Description                                       |
| 0000000 |                  0x0 |                0 | Unspecified (See `AF_UNSPEC` or `socket_$unspec`) |
| 0000001 |                  0x1 |                1 | Unix (See `AF_UNIX` or `socket_$unix`)            |
| 0000010 |                  0x2 |                2 | Internet (See `AF_INET` or `socket_$internet`)    |
| 0000011 |                  0x3 |                3 | ImpLink (See `AF_IMPLINK` or `socket_$implink`)   |
| 0000101 |                  0x4 |                4 | PUP (See `AF_PUP` or `socket_$pup`)               |
| 0000101 |                  0x5 |                5 | Chaos (See `AF_CHAOS` or `socket_$chaos`)         |
| 0000110 |                  0x6 |                6 | NS (See `AF_NS` or `socket_$ns`)                  |
| 0000111 |                  0x7 |                7 | NBS (See `AF_NBS` or `socket_$nbs`)               |
| 0001000 |                  0x8 |                8 | ECMA (See `AF_ECMA` or `socket_$ecma`)            |
| 0001001 |                  0x9 |                9 | DataKit (See `AF_DATAKIT` or `socket_$datakit`)   |
| 0001010 |                  0xA |               10 | CCITT (See `AF_CCITT` or `socket_$ccitt`)         |
| 0001011 |                  0xB |               11 | SNA (See `AF_SNA` or `socket_$sna`)               |
| 0001100 |                  0xC |               12 | Unspecified 2 (See `socket_$unspec2`)             |
| 0001101 |                  0xD |               13 | DDS (See `socket_$dds`)                           |
{: title='Address family list'}

It seems that only address family 2 (Internet) and 13 (DDS) have been used
in UUIDs.


## Versions {#versions}

The version of an UUID is defined by one nible (4 bits). At the moment, the
following versions are used:

|        Bits | Values (hexadecimal) | Values (decimal) | Description                                |
|        0000 |                  0x0 |                0 | Reserved                                   |
|        0001 |                  0x1 |                1 | Time-based version                         |
|        0010 |                  0x2 |                2 | DCE Security version                       |
|        0011 |                  0x3 |                3 | Name-based version using MD5               |
|        0100 |                  0x4 |                4 | Random version                             |
|        0101 |                  0x5 |                5 | Name-based version using SHA-1             |
|        0110 |                  0x6 |                6 | Time-based version with inverted time bits |
|        0111 |                  0x7 |                7 | Epoch time-based version                   |
|        1000 |                  0x8 |                8 | Free-form version                          |
|        1001 |                  0x9 |                9 | Reserved                                   |
|        1010 |                  0xA |               10 | Alternating-UUID                           |
| 1011 - 1111 |            0xB - 0xF |          11 - 15 | Reserved                                   |
{: title='UUID Variant Registry'}

### Version 1 - Time-based version

TODO.


### Version 2 - DCE Security version

The same as UUIDv1, but the first 32 bits of the time field (`time_low`) are used for the local identifier and `clk_seq_low` is used for the local domain. Because of this, the time field is 28 bits
instead of 60 bits and the clock sequence field is 6 bits instead of 14 bits.
At the moment there are 3 domains defined:

| Bits | Values (hexadecimal) | Values (decimal) | Stringname | Description |
| 0000000 | 0x00 | 0 | person | Defines that the local identifier is a POSIX UID. |
| 0000001 | 0x01 | 1 | group | Defines that the local identifier is a POSIX GID. |
| 0000010 | 0x02 | 2 | org | Defines that the local identifier is an organization. POSIX doesn't define organizations, so definition of this domain is up to the developer. |
| 0000011 - 1111111 | 0x03 - 0xFF | 3 - 255 | N/A | Reserved |
{: title='Local domains'}


### Version 3 - Name-based version using MD5

To generate an UUIDv3, calculate the MD5 hash of the concatenation of Namespace
ID (128 bits) and the name. The result is another 128 bit sequence. In this
hash, make the first 2 bits of `clock_seq_hi_and_reserved` to 1 and 0 to set the variant and the first 4 bits of `time_hi_and_version` to `0011` to set the version. The other 122 bits stay the same. For Namespace IDs,
see {{namespaces}}.


### Version 4 - Random version

To generate an UUIDv4, make the first 2 bits of `clock_seq_hi_and_reserved` to 1 and 0 to set the variant and the first 4 bits of `time_hi_and_version` to `0100` to set the version. The other 122 bits are randomly generated.


### Version 5 - Name-based version using SHA-1

The same as UUIDv3, but the hash is calculated using SHA-1 instead of MD5
and the version is not 3, but 5. For SHA-1, take the first 128 bits and continue.
It also uses the same Namespace IDs.


### Version 6 - Time-based version with inverted time bits

The same as UUIDv1, but all the time bits are inverted and the version is
not 1, but 6. This makes it easier to sort this UUIDs by time.


### Version 7 - Epoch time-based version

To generate an UUIDv7, make the first 2 bits of `clock_seq_hi_and_reserved` to 1 and 0 to set the variant and the first 4 bits of `time_hi_and_version` to `0111` to set the version. The first 48 bits (`time_low` and `time_mid`) are the Epoch timestamp in milliseconds (`unix_ts_ms`). The other 74 bits are pseudo-random data to provide uniqueness.


### Version 8 - Free-form version

To generate an UUIDv8, make the first 2 bits of `clock_seq_hi_and_reserved` to 1 and 0 to set the variant and the first 4 bits of `time_hi_and_version` to `1000` to set the version. The other 122 bits are custom data. It is similar with
UUIDv4, where this data MUST be "random".


### Version 10 - Alternating-UUID

See {{specialTypes-alternating}}.



## Namespaces {#namespaces}

Namespace IDs are used in UUIDv3 and UUIDv5 to define the namespace the name
belongs to. At the moment, there are 4 namespaces defined:

| UUID                                 | Description              | Example                                                                                  |
| 6ba7b810-9dad-11d1-​80b4-00c04fd430c8 | DNS                      | `example.com`                                                                            |
| 6ba7b811-9dad-11d1-​80b4-00c04fd430c8 | URL                      | `https://example.com/somepage?parameter=vale#somehash`                                   |
| 6ba7b812-9dad-11d1-​80b4-00c04fd430c8 | ISO Object ID            | `1.3.6.1.4.1.343` or `iso.identified-organization.dod.​internet.private.enterprise.intel` |
| 6ba7b814-9dad-11d1-​80b4-00c04fd430c8 | X.500 Distinguished Name | `CN=Ronald Hoffman,OU=Endicott,O=IBM,C=US`                                               |
{: title='UUID Namespace Registry'}



# Special types

There are three special types of UUIDs:

## Nil-UUID {#specialTypes-nil}

The Nil-UUID does have all the bits set to zero:

`00000000-0000-0000-0000-000000000000`


## Alternating-UUID {#specialTypes-alternating}

The Alternating-UUID has a repeating pattern of 1-bits followed by 0-bits:

`AAAAAAAA-AAAA-AAAA-AAAA-AAAAAAAAAAAA`


## Omni-UUID {#specialTypes-omni}

The Omni-UUID does have all the bits set to one:

`FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF`



# IANA Considerations

## UUID Variants

IANA maintains the list of UUID variants that can be used. See {{variants}} for the list.


## UUID Versions

IANA maintains the list of UUID versions that can be used. See {{versions}} for the list.


## UUID Namespaces

IANA maintains the list of UUID namespaces that can be used. See {{namespaces}} for the list.



--- back
