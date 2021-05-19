# did:self method specification
## Author
* Nikos Fotiou, [Mobile Multimedia Laboratory, AUEB](https://mm.aueb.gr)

## About
The did:self method enables DID document management without registries. Each
DID owner is responsible for maintaining their DID documents and the corresponding
revision history.

A Python3 [implementation](https://github.com/mmlab-aueb/did-self-py)

### Research
* Our [SCN4ND](https://mm.aueb.gr/scn4ndn/) project that uses did:self.
* Application of did:self in IPFS [N. Fotiou, V.A. Siris, G.C. Polyzos,
"Enabling self-verifiable mutable content items in IPFS using Decentralized 
Identifiers", in I2F: Decentralising the Internet with IPFS and Filecoin, IFIP Networking 2021 workshop](https://arxiv.org/abs/2105.08395)

## The did:self method 
The name of this DID method is: `self`

The method specific identifier is represented as the base64url value
of an Ed25519 public key. e.g.,

```
did:self:6varD0RjXZfW58v4DGtd7kltX6Kzn9fghX94LvrMDxo
```

Each DID is associated with a DID document and a 
list of proofs known as the `proof chain`. 

The DID document is a JSON-encoded file that must include at least
the `id` and the `controller` fields (as defined by [W3C's DID specification](https://www.w3.org/TR/did-core/)).
The controller is an Ed25519 public key and we are using the [did:key method](https://w3c-ccg.github.io/did-method-key/)
for representing them. Although the `id` and the `controller` fields are mandatory, they can use 
the same Ed25519 public key. Additionally, when an `authentication` verification method is included in the
DID document, it is used for authenticating the DID `subject`. 

Every time a DID document is created or updated a proof is calculated
and it is stored in the `proof chain`.
A proof is a compact encoded [JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515).
The payload of the proof is a JSON string that includes four 
fields: 

* `id` The DID.
* `controller` The controller of the DID document.
* `created` The string value of an ISO8601 combined date and time string
* `sha-256` The base64url encoded hash of the DID document, calculated using SHA-256.

The signature of the proof is generated using EdDSA. 

## CRUD Operation Definitions
CRUD operations are implemented by the users. 

### Create
The Create operation initializes a did:self DID and creates an initial DID document. 
The proof of the initial DID document is signed using the DID
and it is the fist element of the proof chain. The following is a valid DID document

```JSON
{
  "id": "did:self:nLyMu_3R7IKnHj_LjlLphZ1QWMp4U7Vldc0yaFI7eDU",
  "controller": "did:key:z6MKGRqQ8Pb5ZKzUpXotN1NipJYQx2edHFR6aV2tREgJJMhL",
  "authentication": [
    {
      "id": "did:self:nLyMu_3R7IKnHj_LjlLphZ1QWMp4U7Vldc0yaFI7eDU#key1",
      "type": "JsonWebKey2020",
      "publicKeyJwk": {
        "kty": "OKP",
        "crv": "Ed25519",
        "x": "nLyMu_3R7IKnHj_LjlLphZ1QWMp4U7Vldc0yaFI7eDU"
      }
    }
  ]
}
```

The following is the corresponding proof payload

```JSON
{
  "id": "did:self:nLyMu_3R7IKnHj_LjlLphZ1QWMp4U7Vldc0yaFI7eDU",
  "controller": "did:key:z6MKGRqQ8Pb5ZKzUpXotN1NipJYQx2edHFR6aV2tREgJJMhL",
  "created": "2021-03-10T22:59:54Z",
  "sha-256": "2L5DctVFPBp_po3caEvwUmg1w5-NvTskH485fV8gwek"
}
```

The following is the proof chain after the invocation of the `Create` method. The signature
of the proof can be verified using `nLyMu_3R7IKnHj_LjlLphZ1QWMp4U7Vldc0yaFI7eDU`, i.e., the key
that corresponds to the DID.

```
[
  "eyJhbGciOiAiRWREU0EifQ.eyJpZCI6ICJkaWQ6c2VsZjpuTHlNdV8zUjdJS25Ial9MamxMcGhaMVFXTXA0VTdWbGRjMHlhRkk3ZURVIiwgImNvbnRyb2xsZXIiOiAiZGlkOmtleTp6Nk1LR1JxUThQYjVaS3pVcFhvdE4xTmlwSllReDJlZEhGUjZhVjJ0UkVnSkpNaEwiLCAiY3JlYXRlZCI6ICIyMDIxLTAzLTEwVDIyOjU5OjU0WiIsICJzaGEtMjU2IjogIjJMNURjdFZGUEJwX3BvM2NhRXZ3VW1nMXc1LU52VHNrSDQ4NWZWOGd3ZWsifQ.epEVNpCsb4wP7R1xakBpkA_nMWJmC9JtPXyK4sAf5ub2ju-W-j86h3H1ISTPWJ-6zm5Ygf-yJ933hjQ_ajRsAQ"
]
```

### Update
With the Update operation an existing DID document is replaced with a new one. 
Since only controllers can update a DID document, 
the proof of the new DID document must be generated 
by the controller of the **DID document being replaced**.
If the new proof and the last element of the proof chain are generated by the same controller, 
then the last element of the chain is replaced with the new proof, otherwise the new proof 
is appended to the proof chain.

The following is an update to the previous DID document, which modifies the controller.

```JSON
{
  "id": "did:self:nLyMu_3R7IKnHj_LjlLphZ1QWMp4U7Vldc0yaFI7eDU",
  "controller": "did:key:z6MKJ8owm1HAzMRUJAUNtuBcyhkMPRaCwa7XWTi86TvShf2w",
  "authentication": [
    {
      "id": "did:self:nLyMu_3R7IKnHj_LjlLphZ1QWMp4U7Vldc0yaFI7eDU#key1",
      "type": "JsonWebKey2020",
      "publicKeyJwk": {
        "kty": "OKP",
        "crv": "Ed25519",
        "x": "nLyMu_3R7IKnHj_LjlLphZ1QWMp4U7Vldc0yaFI7eDU"
      }
    }
  ]
}
```

The following is the corresponding proof payload

```JSON
Proof payload:
{
  "id": "did:self:nLyMu_3R7IKnHj_LjlLphZ1QWMp4U7Vldc0yaFI7eDU",
  "controller": "did:key:z6MKJ8owm1HAzMRUJAUNtuBcyhkMPRaCwa7XWTi86TvShf2w",
  "created": "2021-03-10T22:59:54Z",
  "sha-256": "CjzuQPolyxr4zsZTyuP1HXVMHzVSypmBwqFzsWy1iu4"
}
```
The proof is signed with the private key that corresponds to `did:key:z6MKGRqQ8Pb5ZKzUpXotN1NipJYQx2edHFR6aV2tREgJJMhL`,
the key of the controller of
DID document that was replaced

The following is the proof chain after the invocation of the `Update` method. The signature
of the first proof can be verified using `LyMu_3R7IKnHj_LjlLphZ1QWMp4U7Vldc0yaFI7eDU`, i.e., the key
that corresponds to the DID. The signature of the second proof can be verified using the key that corresponds to
`did:key:z6MKGRqQ8Pb5ZKzUpXotN1NipJYQx2edHFR6aV2tREgJJMhL`, i.e., the key of the controller of
DID document that was replaced: this key can be found in the payload of the first 
signature.

```
[
  "eyJhbGciOiAiRWREU0EifQ.eyJpZCI6ICJkaWQ6c2VsZjpuTHlNdV8zUjdJS25Ial9MamxMcGhaMVFXTXA0VTdWbGRjMHlhRkk3ZURVIiwgImNvbnRyb2xsZXIiOiAiZGlkOmtleTp6Nk1LR1JxUThQYjVaS3pVcFhvdE4xTmlwSllReDJlZEhGUjZhVjJ0UkVnSkpNaEwiLCAiY3JlYXRlZCI6ICIyMDIxLTAzLTEwVDIyOjU5OjU0WiIsICJzaGEtMjU2IjogIjJMNURjdFZGUEJwX3BvM2NhRXZ3VW1nMXc1LU52VHNrSDQ4NWZWOGd3ZWsifQ.epEVNpCsb4wP7R1xakBpkA_nMWJmC9JtPXyK4sAf5ub2ju-W-j86h3H1ISTPWJ-6zm5Ygf-yJ933hjQ_ajRsAQ",
  "eyJhbGciOiAiRWREU0EifQ.eyJpZCI6ICJkaWQ6c2VsZjpuTHlNdV8zUjdJS25Ial9MamxMcGhaMVFXTXA0VTdWbGRjMHlhRkk3ZURVIiwgImNvbnRyb2xsZXIiOiAiZGlkOmtleTp6Nk1LSjhvd20xSEF6TVJVSkFVTnR1QmN5aGtNUFJhQ3dhN1hXVGk4NlR2U2hmMnciLCAiY3JlYXRlZCI6ICIyMDIxLTAzLTEwVDIyOjU5OjU0WiIsICJzaGEtMjU2IjogIkNqenVRUG9seXhyNHpzWlR5dVAxSFhWTUh6VlN5cG1Cd3FGenNXeTFpdTQifQ.YJG_EmOsVBWo1yp19MTYqN2tITkXjoibNFQyYghazGbDWS1Rcr1wryrSO36jmoSST6MVdf5KRaImuCNkwHt9DQ"
]
```

### Read
The Read operation simply outputs the DID document and 
the corresponding proof chain.

## DID document validation
Given a DID, a DID document, and a proof chain, any entity can attest
whether or not the DID document is a valid document for the given DID 
using the following protocol.

1. First, verify that the `sha-256` field of the payload of the last proof of the chain contains the hash of the DID document.
1. As a next step, verify that the `id` field of the payload of all proofs of the chain is equal to the DID.
1. Then, verify the first proof of the chain using the DID
1. Finally, starting from the second proof of the chain, verify all proofs using the `controller` field of the payload of their previous proof.





