---
eip: <to be assigned>
title: Interoperable Names
description: A human-parsable string representation of an Interoperable Address.
author: Teddy (@0xteddybear), Thomas Clowes (@clowestab), Mono (@0xMonoAx), Orca (0xrcinus), premm.eth (nxt3d)
discussions-to: <to-do>
status: Draft
type: Standards Track
category: ERC
created: 2025-12-17
requires: 7930
---

## Abstract
This proposal defines an Interoperable Name as a human-parsable string representation of an Interoperable Address as defined in [ERC-7930], providing a baseline for improved end-user experience.

## Motivation

Interoperable Addresses as defined in ERC-7930 are compact and extensible chain-specific addresses in a binary format. As such they are not straightforward for humans to parse and validate.

This proposal introduces an _Interoperable Name_ as a string representation of an _Interoperable Address_ which:

- Combines chain identification and a target address with clear visual separation.
- Is compact for ease of use ("copy pasting").
- Includes short, easily verifiable checksums to protect users.
- Extends beyond EVM blockchains.
- Can be easily extended by future ERCs for enhanced usability (for example with name registries).

### Comparisons with other standards

#### CAIP-10

[CAIP-10] proposes a standard text format to represent an address on a specific chain (referenced by its [CAIP-2] identifier).

An _Interoperable Name_ contains all of the components of the [CAIP-10] identifier. It is therefore easy to convert between the two in cases where the [CAIP-10] standard is still in use.

## Specification
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### Terminology
**Target Address**
: The address itself, independent of chain context. Serialized per the [CAIP-350] rules for the applicable namespace. In the context of the _Interoperable Name_ and _Interoperable Address_ examples below, the target address is `0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`.

**Chain Specific Address**
: An address representation that includes both the _target address_ **and** the chain being targeted. The following are examples of chain specific addresses:

- The _Interoperable Address_ definition outlined in [ERC-7930]
- The _Interoperable Name_ definition outlined in this specification
- The addressing format outlined in [ERC-3770], e.g. `arb:0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`

**Interoperable Address**
: A binary payload which unambiguously identifies a target address on a target chain as defined in [ERC-7930]. e.g. `0x00010000010114D8DA6BF26964AF9D7EED9E03E53415D37AA96045`

**Interoperable Name**
: A string representation of an _Interoperable Address_, meant to be used by humans. e.g. `0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045@eip155:1#4CA88C9C`

### Interoperable Name Definition

An _Interoperable Name_ is a human-parsable representation of an underlying _Interoperable Address_, as defined in [ERC-7930].

Its format is `<address>@<chain>#<checksum>` where the components match the following regular expressions:

#### Syntax
```bnf
<interoperable-name>  ::= <address> "@" <chain> [ "#" <checksum> ]
<address>             ::= [.-:_%a-zA-Z0-9]*
<chain>               ::= [.-:_a-zA-Z0-9]+
<checksum>            ::= [0-9A-F]{8}
```

The `@` and `#` characters are ASCII literals and MUST appear without surrounding whitespace. Clients MUST treat any leading or trailing whitespace as invalid or trim it prior to parsing.

These components have the following meanings:

`<address>` is the _target address_. `<address>` MAY be empty (i.e., the Interoperable Name MAY start with `@`) to represent "no specific address" when targeting all addresses within a CASA namespace (i.e., when the _Interoperable Address_ `AddressLength` is zero).

`<chain>` is the string representation of a specific blockchain as defined in [CAIP-350]. `<chain>` MUST NOT be empty; at minimum the namespace (e.g., `eip155`) MUST be provided. For example `eip155:1` for Ethereum Mainnet, or `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdpKuc147dw2N9d` for Solana Mainnet.

If you are representing a specific address namespace without intending to refer to a particular chain (i.e., when `ChainReferenceLength` in the _Interoperable Address_ is zero), use the CASA namespace alone, with no trailing colon - for example, `eip155` or `solana`. It is not possible to specify a reference without a namespace.

The 4-byte `<checksum>` is the first 4 bytes of the hexadecimal ([RFC 4648]) string representation of a keccak256 hash. This hash is computed over the concatenation of the _Interoperable Address_'s binary fields: `ChainType`, `ChainReferenceLength`, `ChainReference`, `AddressLength`, and `Address`. The `Version` field MUST NOT be included in the hashed data.

#### Checksums

A 4-byte checksum MUST be computed and included when sharing an _Interoperable Name_. 

If a user-provided _Interoperable Name_ includes a checksum, clients MUST derive the underlying _Interoperable Address_, recalculate the checksum, and compare it to the provided value. In case of a mismatch, clients MUST surface a prominent warning and MUST require an explicit confirmation step (e.g., a modal, explicit "proceed anyway" button, or equivalent) before proceeding with any action that depends on the interpreted address.

If the Interoperable Name omits the `#<checksum>` suffix, clients MUST treat it as having no checksum and MUST NOT attempt to infer or silently inject one.

Clients MAY include the checksum when displaying an _Interoperable Name_ within their interface.
Clients MAY accept _Interoperable Name_ inputs without a checksum.

When displayed the checksum MUST be displayed using the 'Base 16 Alphabet', `[0-9A-Z]` as defined in [RFC 4648].

#### Parsing and Serialization:

When parsing an _Interoperable Name_ to generate its binary _Interoperable Address_, clients MUST follow the normalization and serialization rules defined in the relevant [CAIP-350] profile for the given `<chain>` and `<address>`. This ensures that different valid text representations (e.g., case variations in an address) resolve to a single, canonical binary form, which is essential for consistent checksum calculation and data integrity.

Clients MUST compute the checksum over the canonical binary form derived from CAIP-350 serialization, not the raw textual form supplied by the user.

### Examples

These examples extend the examples in [ERC-7930], adding the _Interoperable Name_ representation.

#### Example 1: Ethereum mainnet address

**Components**

| Key | Value |
| :--- | :--- |
| **Version** | `1` |
| **ChainType** | `0x0000` |
| **ChainReferenceLength** | `1` |
| **ChainReference** | `1` |
| **AddressLength** | `20` |
| **Address** | `0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045` |

**Interoperable Address**

These components produce the following _Interoperable Address_:

```
0x00010000010114d8da6bf26964af9d7eed9e03e53415d37aa96045
  ^^^^-------------------------------------------------- Version              
      ^^^^---------------------------------------------- ChainType            
          ^^-------------------------------------------- ChainReferenceLength
            ^^------------------------------------------ ChainReference       
              ^^---------------------------------------- AddressLength       
                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ Address             
```

**Interoperable Name**

The _Interoperable Name_ representation is:

| Key | Value |
| :--- | :--- |
| **Checksum Input** | `0x0000010114d8da6bf26964af9d7eed9e03e53415d37aa96045` |
| **Checksum** | `4CA88C9C` |
| **Interoperable Name** | `0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045@eip155:1#4CA88C9C` |

---  

#### Example 2: Solana mainnet address

**Components**

| Key | Value |
| :--- | :--- |
| **Version** | `1` |
| **ChainType** | `0x0002` |
| **ChainReferenceLength** | `32` |
| **ChainReference** | `5eykt4UsFv8P8NJdTREpY1vzqKqZKvdpKuc147dw2N9d` |
| **AddressLength** | `32` |
| **Address** | `MJKqp326RZCHnAAbew9MDdui3iCKWco7fsK9sVuZTX2` |


**Interoperable Address**

These components produce the following _Interoperable Address_:

```
0x000100022045296998a6f8e2a784db5d9f95e18fc23f70441a1039446801089879b08c7ef02005333498d5aea4ae009585c43f7b8c30df8e70187d4a713d134f977fc8dfe0b5
  ^^^^---------------------------------------------------------------------------------------------------------------------------------------- Version
      ^^^^------------------------------------------------------------------------------------------------------------------------------------ ChainType
          ^^---------------------------------------------------------------------------------------------------------------------------------- ChainReferenceLength
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^------------------------------------------------------------------ ChainReference
                                                                            ^^---------------------------------------------------------------- AddressLength
                                                                              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^--- Address
```

**Interoperable Name**

The _Interoperable Name_ representation is:

| Key | Value |
| :--- | :--- |
| **Checksum Input** | `0x00022045296998a6f8e2a784db5d9f95e18fc23f70441a1039446801089879b08c7ef02005333498d5aea4ae009585c43f7b8c30df8e70187d4a713d134f977fc8dfe0b5` |
| **Checksum** | `88835C11` |
| **Interoperable Name** | `MJKqp326RZCHnAAbew9MDdui3iCKWco7fsK9sVuZTX2@solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdpKuc147dw2N9d#88835C11` |

---  

#### Example 3: EVM address without ChainReference

**Components**

| Key | Value |
| :--- | :--- |
| **Version** | `1` |
| **ChainType** | `0x0000` |
| **ChainReferenceLength** | `` |
| **ChainReference** | N/A |
| **AddressLength** | `20` |
| **Address** | `0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045` |

**Interoperable Address**

These components produce the following _Interoperable Address_:

```
0x000100000014d8da6bf26964af9d7eed9e03e53415d37aa96045
  ^^^^------------------------------------------------ Version
      ^^^^-------------------------------------------- ChainType
          ^^------------------------------------------ ChainReferenceLength
            ^^---------------------------------------- AddressLength
              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ Address
```

**Interoperable Name**

The _Interoperable Name_ representation is:

| Key | Value |
| :--- | :--- |
| **Checksum Input** | `0x00000014d8da6bf26964af9d7eed9e03e53415d37aa96045` |
| **Checksum** | `B26DB7CB` |
| **Interoperable Name** | `0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045@eip155#B26DB7CB` |

---  

#### Example 4: Solana mainnet network, no address

**Components**

| Key | Value |
| :--- | :--- |
| **Version** | `1` |
| **ChainType** | `0x0002` |
| **ChainReferenceLength** | `32` |
| **ChainReference** | `5eykt4UsFv8P8NJdTREpY1vzqKqZKvdpKuc147dw2N9d` |
| **AddressLength** | `0` |
| **Address** | N/A |

**Interoperable Address**

These components produce the following _Interoperable Address_:

```
0x000100022045296998a6f8e2a784db5d9f95e18fc23f70441a1039446801089879b08c7ef000
  ^^^^------------------------------------------------------------------------ Version
      ^^^^-------------------------------------------------------------------- ChainType
          ^^------------------------------------------------------------------ ChainReferenceLength
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^-- ChainReference
                                                                            ^^ AddressLength
```

**Interoperable Name**

The _Interoperable Name_ representation is:

| Key | Value |
| :--- | :--- |
| **Checksum Input** | `0x00022045296998a6f8e2a784db5d9f95e18fc23f70441a1039446801089879b08c7ef000` |
| **Checksum** | `2EB18670` |
| **Interoperable Name** | `@solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdpKuc147dw2N9d#2EB18670` |

### Versioning

This format does not have a concept of versioning, so any subsequent changes would require a new standard.

Future standards that extend Interoperable Names:

- MUST remain trivially derivable from, or convertible to, an underlying _Interoperable Address_ as defined in [ERC-7930].
- MUST NOT alter the mapping between a given _Interoperable Address_ and its canonical _Interoperable Name_ without changing version semantics in a way that prevents silent misinterpretation.
- MAY introduce additional components (e.g., human-friendly aliases or labels), but such components MUST NOT affect the checksum or the binary _Interoperable Address_.

## Rationale

The rationale for some of the low level specification decisions are outlined below:

- The `@` symbol is used as a separator as it provides visual clarity to humans, is easy for software to parse, and avoids confusion with the colon (`:`) symbol utilized in [CAIP-2], and [CAIP-10] identifiers. 
- The address field includes `%` as a valid character to allow for URL-encoding of any other characters that are not explicitly allowed. This allows backward compatibility with [CAIP-10].
- Similarly to ERC-7930, we chose to support zero-length addresses and chain IDs to make this standard flexible and to allow developers to use a single, uniform standard for many different jobs. For example if a user wants to represent an address on any compatible chain, or if the user simply wants to represent the chain itself.
- We chose *not* to use alternate encoding formats (e.g., `base58` or `base64`) in order to make it easier for wallets and dApps to work with, and convert between, addresses that both use and do not use this addressing standard. Whilst utilizing alternate formats could have reduced the size of the _Interoperable Name_ we decided that user and developer experience should be prioritized.
- Checksums are short enough to be visually comparable by the human eye, allowing for easy differentiation.
- This does not make specific decisions regarding name registries which might make chains and addresses more user-friendly, leaving such extensions to further ERCs.
- It is straightforward to map between ERC-3770, CAIP-10, and Interoperable Names by parsing the chain identifier and address components, then constructing the appropriate format. For example, `arb:0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045` (ERC-3770) can be converted to `0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045@eip155:42161#<checksum>` by identifying the Arbitrum One chain ID (42161) and computing the appropriate checksum.

## Security Considerations

This ERC inherits the security considerations from ERC-7930.

## Copyright
Copyright and related rights waived via [CC0](../LICENSE.md).

[ERC-55]: ./eip-55.md
[ERC-3770]: ./eip-3770.md
[ERC-7930]: ./erc-7930.md
[CAIP-2]: https://github.com/ChainAgnostic/CAIPs/blob/2a7d42aebaffa42d1017c702974395ff5c1b3636/CAIPs/caip-2.md
[CAIP-10]: https://github.com/ChainAgnostic/CAIPs/blob/2a7d42aebaffa42d1017c702974395ff5c1b3636/CAIPs/caip-10.md
[CAIP-50]: https://github.com/ChainAgnostic/CAIPs/blob/2a7d42aebaffa42d1017c702974395ff5c1b3636/CAIPs/caip-50.md
[CAIP-350]: https://github.com/ChainAgnostic/CAIPs/blob/29762ef99a6ffea1e07e3f796c0d1a5a95e89b88/CAIPs/caip-350.md
[RFC 4648]: https://www.rfc-editor.org/rfc/rfc4648