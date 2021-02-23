# DID self method specification
## Author
* Nikos Fotiou, [Mobile Multimedia Laboratory, AUEB](https://mm.aueb.gr)

## Abstract
The did self method enables DID document management without registries. Each
DID owner is responsible for maintaining their DID documents and the corresponding
revision history.

## DID Method 
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
* `version` A version number used for revoking deprecated documents.
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
  "id": "did:self:0ViQW-YuT0zwHAcCc0KfHFRu7dc7rXUMEg7w-lJQhkQ",
  "controller": "did:key:ukSk6CMdyKW55KPuE4JFu2gsMJkD2G_zQpM11bvvF3Pg",
  "authentication": [
    {
      "id": "did:self:0ViQW-YuT0zwHAcCc0KfHFRu7dc7rXUMEg7w-lJQhkQ#key1",
      "type": "JsonWebKey2020",
      "controller": "did:self:0ViQW-YuT0zwHAcCc0KfHFRu7dc7rXUMEg7w-lJQhkQ",
      "publicKeyJwk": {
        "crv": "Ed25519",
        "x": "0ViQW-YuT0zwHAcCc0KfHFRu7dc7rXUMEg7w-lJQhkQ",
        "kty": "OKP"
      }
    }
  ]
}
```

The following is the corresponding proof payload

```JSON
{
  "id": "did:self:0ViQW-YuT0zwHAcCc0KfHFRu7dc7rXUMEg7w-lJQhkQ",
  "controller": "did:key:ukSk6CMdyKW55KPuE4JFu2gsMJkD2G_zQpM11bvvF3Pg",
  "version": 1,
  "sha-256": "DsChaHOed0dtcu8Y-X0DivqZWuTM-hRKHP90lJYRFSY"
}
```

The following is the proof chain after the invocation of the `Create` method. The signature
of the proof can be verified using `0ViQW-YuT0zwHAcCc0KfHFRu7dc7rXUMEg7w-lJQhkQ`, i.e., the key
that corresponds to the DID.

```
[
  "eyJhbGciOiJFZERTQSJ9.eyJpZCI6ICJkaWQ6c2VsZjowVmlRVy1ZdVQwendIQWNDYzBLZkhGUnU3ZGM3clhVTUVnN3ctbEpRaGtRIiwgImNvbnRyb2xsZXIiOiAiZGlkOmtleTp1a1NrNkNNZHlLVzU1S1B1RTRKRnUyZ3NNSmtEMkdfelFwTTExYnZ2RjNQZyIsICJ2ZXJzaW9uIjogMSwgInNoYS0yNTYiOiAiRHNDaGFIT2VkMGR0Y3U4WS1YMERpdnFaV3VUTS1oUktIUDkwbEpZUkZTWSJ9.xTVzzeEcgyhIz-X76FvsqZx9i57LEV6_jydsAYPtxoeLZO5i10aP6KfB35bq_Q57Rm5vPqD2_clNdqp7Xs5eDg"
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

The following is an update to the previous DID document, which modified the controller.

```JSON
{
  "id": "did:self:0ViQW-YuT0zwHAcCc0KfHFRu7dc7rXUMEg7w-lJQhkQ",
  "controller": "did:key:uDvl4lkV0CdbDC7C5lltDqPTjTIbAoJqtoyVJG8My7P0",
  "authentication": [
    {
      "id": "did:self:0ViQW-YuT0zwHAcCc0KfHFRu7dc7rXUMEg7w-lJQhkQ#key1",
      "type": "JsonWebKey2020",
      "controller": "did:self:0ViQW-YuT0zwHAcCc0KfHFRu7dc7rXUMEg7w-lJQhkQ",
      "publicKeyJwk": {
        "crv": "Ed25519",
        "x": "Dvl4lkV0CdbDC7C5lltDqPTjTIbAoJqtoyVJG8My7P0",
        "kty": "OKP"
      }
    }
  ]
}
```

The following is the corresponding proof payload

```JSON
{
  "id": "did:self:0ViQW-YuT0zwHAcCc0KfHFRu7dc7rXUMEg7w-lJQhkQ",
  "controller": "did:key:uDvl4lkV0CdbDC7C5lltDqPTjTIbAoJqtoyVJG8My7P0",
  "version": 1,
  "sha-256": "fBgPfcvvMVcJqY_-DFYXX7b-3Yzv-11IKRr12yvu8c8"
}
```

The following is the proof chain after the invocation of the `Update` method. The signature
of the first proof can be verified using `0ViQW-YuT0zwHAcCc0KfHFRu7dc7rXUMEg7w-lJQhkQ`, i.e., the key
that corresponds to the DID. The signature of the second proof can be verified using 
`kSk6CMdyKW55KPuE4JFu2gsMJkD2G_zQpM11bvvF3Pg`, i.e., the key of the controller of
DID document that was replaced: this key can be found in the payload of the first 
signature.

```
[
  "eyJhbGciOiJFZERTQSJ9.eyJpZCI6ICJkaWQ6c2VsZjowVmlRVy1ZdVQwendIQWNDYzBLZkhGUnU3ZGM3clhVTUVnN3ctbEpRaGtRIiwgImNvbnRyb2xsZXIiOiAiZGlkOmtleTp1a1NrNkNNZHlLVzU1S1B1RTRKRnUyZ3NNSmtEMkdfelFwTTExYnZ2RjNQZyIsICJ2ZXJzaW9uIjogMSwgInNoYS0yNTYiOiAiRHNDaGFIT2VkMGR0Y3U4WS1YMERpdnFaV3VUTS1oUktIUDkwbEpZUkZTWSJ9.xTVzzeEcgyhIz-X76FvsqZx9i57LEV6_jydsAYPtxoeLZO5i10aP6KfB35bq_Q57Rm5vPqD2_clNdqp7Xs5eDg",
  "eyJhbGciOiJFZERTQSJ9.eyJpZCI6ICJkaWQ6c2VsZjowVmlRVy1ZdVQwendIQWNDYzBLZkhGUnU3ZGM3clhVTUVnN3ctbEpRaGtRIiwgImNvbnRyb2xsZXIiOiAiZGlkOmtleTp1RHZsNGxrVjBDZGJEQzdDNWxsdERxUFRqVEliQW9KcXRveVZKRzhNeTdQMCIsICJ2ZXJzaW9uIjogMSwgInNoYS0yNTYiOiAiZkJnUGZjdnZNVmNKcVlfLURGWVhYN2ItM1l6di0xMUlLUnIxMnl2dThjOCJ9.d7fm20dCr2bEOUAtxcG5oD4ZJ6lF6L06ym84kHtNgykeB4xX9LGnpkN3FAqfjBAJDsFpUGFOLexSnpKjQDu6CA"
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





