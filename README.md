[![](.maintain/media/kilt-header.png)](https://kilt.io)

# KiltTransferAssetRecipientV1

### Editors

- **Antonio Antonino** - KILT Protocol [antonio@kilt.io](mailto:antonio@kilt.io)
- **Albrecht Weiche** - KILT Protocol [albrecht@kilt.io](mailto:albrecht@kilt.io)

### Version History

<!-- TODO: Update before releasing -->
- **v1.0 - Feb.09 2023**: Initial spec publishing

---

## Abstract

This document defines an extension to the service types supported in the [DID Core W3C spec][did-core-spec] by defining the `KiltTransferAssetRecipientV1` service type.
The goal of the endpoints of this class is to expose a collection (i.e., a list) of addresses to which assets of some class can be sent to.
For more information about the KILT DID method, please visit our [official specification][kilt-did-spec].

## Data structure

A service endpoint of type `KiltTransferAssetRecipientV1` does not include any additional properties compared to what is defined within the [relative section of the official DID Core spec][did-core-spec-services].
Furthermore, endpoints of such type MUST include at least *one* URI for the `serviceEndpoint` property.
Each of the URIs in `serviceEndpoint`, when dereferenced, MUST return **a JSON containing an object** with a mapping from each type of asset to the list of accounts the DID subject controls for that asset, optionally with a description of each account.
An example of the object described is given below.

```json
{
  "polkadot:411f057b9107718c9624d6aa4a3f23c1/slip44:2086": [
    {
      "account": "4qBSZdEoUxPVnUqbX8fjXovgtQXcHK7ZvSf56527XcDZUukq",
      "description": "Treasury proposals transfers"
    },
    {
      "account": "4oHvgA54py7SWFPpBCoubAajYrxj6xyc8yzHiAVryeAq574G",
      "description": "Regular transfers"
    },
    {
      "account": "4taHgf8x9U5b8oJaiYoNEh61jaHpKs9caUdattxfBRkJMHvm"
    }
  ],
  "eip:1/slip44:60": [
    {
      "account": "0x8f8221AFBB33998D8584A2B05749BA73C37A938A",
      "description": "NFT sales"
    },
    {
      "account": "0x6b175474e89094c44da98b954eedeac495271d0f"
    }
  ]
}
```

Each asset is identified by its [CAIP-19 identifier][caip-19-spec].
The value of the property for each asset MUST be a set of accounts encoded according to the chain rules.
For example, for Spiritnet accounts, the account is the base58-prefixed encoding of the Spiritnet chain ID + the account public key.
For Ethereum accounts, it's the 20-byte HEX representation of the account public key, prefixed with `0x`.
Other chains have different encoding rules for accounts, and each chain defines the format and encoding logic for public keys representing accounts on those chains.

Hence, the example above shows a `KiltTransferAssetRecipientV1` endpoint indicating other parties that the DID subject can accept transfers of the following two assets:

- *KILT Spiritnet tokens* sent to either of the addresses `4qBSZdEoUxPVnUqbX8fjXovgtQXcHK7ZvSf56527XcDZUukq`, `4oHvgA54py7SWFPpBCoubAajYrxj6xyc8yzHiAVryeAq574G`, or `4taHgf8x9U5b8oJaiYoNEh61jaHpKs9caUdattxfBRkJMHvm` on the Spiritnet parachain.
- *Ether tokens* sent to either of the addresses `0x8f8221AFBB33998D8584A2B05749BA73C37A938A`, or `0x6b175474e89094c44da98b954eedeac495271d0f` on the Ethereum mainnet.

The `id` property of the endpoint MUST be the [multibase][multibase] representation of the Blake256 output calculated from the Base64 encoding of the resource dereferenced by the URIs in `serviceEndpoint`.
The multibase chosen must yield values that do not contain invalid characters as per the definition of the `id` property in the DID specification, i.e., the resulting `id` must still be a valid URI conforming to [RFC3986][rfc3986].

Hence, calling `M` the multibase encoding operation of some data, `H` the Blake256 hashing, and `B` the binary representation of some information, the service `id` for a given object `O` is `M(H(B(O)))`.

For example, with the object `O` being the example `serviceEndpoint` shown above, and the multibase `M` being `base64urlpad`, the resulting service endpoint looks like the following:

```json
{
  "id": "did:kilt:4pqDzaWi3w7TzYzGnQDyrasK6UnyNnW6JQvWRrq6r8HzNNGy#Ug10blEje4DrZsqVV7AfyUKoJ93Jgk8xkvxJkehTmO8o=",
  "type": [
    "KiltTransferAssetRecipientV1"
  ],
  "serviceEndpoint": [
    "https://ipfs.io/ipfs/QmNUAwg7JPK9nnuZiUri5nDaqLHqUFtNoZYtfD22Q6w3c8"
  ]
}
```

## Security considerations

The list of addresses where the DID owner wants to receive funds must always be under the subject's control even if stored off-chain.
This ensures the authenticity and integrity of the list.
Implementations must verify that the list of addresses retrieved from the service URI can be hashed and encoded to the same value as the service `id`.
Failure to verify this condition MUST be treated as an attack either towards the DID subject or the entity willing to initiate the asset transfer, and the operation must be aborted.

[did-core-spec]: https://www.w3.org/TR/did-core
[kilt-did-spec]: https://github.com/KILTprotocol/spec-kilt-did
[multibase]: https://github.com/multiformats/multibase#multibase-by-example
[did-core-spec-services]: https://www.w3.org/TR/did-core/#services=
[caip-19-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-19.md
[caip-2-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-2.md
[caip-13-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-13.md
[rfc3986]: https://www.w3.org/TR/did-core/#bib-rfc3986