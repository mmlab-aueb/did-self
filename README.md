# did:self method specification
## Author
* Nikos Fotiou, [Mobile Multimedia Laboratory, AUEB](https://mm.aueb.gr)

## About
The did:self method enables DID document management without registries. Each
DID owner is responsible for maintaining their DID documents and the corresponding
revision history.

A Python3 [implementation](https://github.com/mmlab-aueb/did-self-py)

Our [SCN4ND](https://mm.aueb.gr/scn4ndn/) project that uses did:self.

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
  "id": "did:self:b9ewLzljpexCZFBm_lTwcRkjgmjO2gdqh8FlnfbcezI",
  "controller": "did:key:uHrHw64GaW9fVrx-QlB_1RFdJ73Jwo8a8E8ZadEQS04A",
  "authentication": [
    {
      "id": "did:self:b9ewLzljpexCZFBm_lTwcRkjgmjO2gdqh8FlnfbcezI#key1",
      "type": "JsonWebKey2020",
      "publicKeyJwk": {
        "crv": "Ed25519",
        "x": "b9ewLzljpexCZFBm_lTwcRkjgmjO2gdqh8FlnfbcezI",
        "kty": "OKP"
      }
    }
  ]
}
```

The following is the corresponding proof payload

```JSON
{
  "id": "did:self:b9ewLzljpexCZFBm_lTwcRkjgmjO2gdqh8FlnfbcezI",
  "controller": "did:key:uHrHw64GaW9fVrx-QlB_1RFdJ73Jwo8a8E8ZadEQS04A",
  "sha-256": "papnsiYT_7j2qbQ3zcHm9HHDX4UZNNApkzqm6Wj8_Ss"
}
```

The following is the proof chain after the invocation of the `Create` method. The signature
of the proof can be verified using `b9ewLzljpexCZFBm_lTwcRkjgmjO2gdqh8FlnfbcezI`, i.e., the key
that corresponds to the DID.

```
[
  "eyJhbGciOiJFZERTQSJ9.eyJpZCI6ICJkaWQ6c2VsZjpiOWV3THpsanBleENaRkJtX2xUd2NSa2pnbWpPMmdkcWg4RmxuZmJjZXpJIiwgImNvbnRyb2xsZXIiOiAiZGlkOmtleTp1SHJIdzY0R2FXOWZWcngtUWxCXzFSRmRKNzNKd284YThFOFphZEVRUzA0QSIsICJzaGEtMjU2IjogInBhcG5zaVlUXzdqMnFiUTN6Y0htOUhIRFg0VVpOTkFwa3pxbTZXajhfU3MifQ.yINGHRO7sAG2vEMmtoD1XW-QMguhH2xOGYdneWcMa-jQcfZZqksbGSZm7DZNpyk9eUcRoO420uROdPWgNew5CA"
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
  "id": "did:self:b9ewLzljpexCZFBm_lTwcRkjgmjO2gdqh8FlnfbcezI",
  "controller": "did:key:uNDBHNpMMueGdOYqgFmevrK1DFngbKZklsSKCeZzxs3g",
  "authentication": [
    {
      "id": "did:self:b9ewLzljpexCZFBm_lTwcRkjgmjO2gdqh8FlnfbcezI#key1",
      "type": "JsonWebKey2020",
      "publicKeyJwk": {
        "crv": "Ed25519",
        "x": "b9ewLzljpexCZFBm_lTwcRkjgmjO2gdqh8FlnfbcezI",
        "kty": "OKP"
      }
    }
  ]
}
```

The following is the corresponding proof payload

```JSON
Proof payload:
{
  "id": "did:self:b9ewLzljpexCZFBm_lTwcRkjgmjO2gdqh8FlnfbcezI",
  "controller": "did:key:uNDBHNpMMueGdOYqgFmevrK1DFngbKZklsSKCeZzxs3g",
  "sha-256": "AIx2JTPlXFSpgFbq54K2Mrgi3CIax-kxnG5i2oS-GzQ"
}
```

The following is the proof chain after the invocation of the `Update` method. The signature
of the first proof can be verified using `b9ewLzljpexCZFBm_lTwcRkjgmjO2gdqh8FlnfbcezI`, i.e., the key
that corresponds to the DID. The signature of the second proof can be verified using 
`HrHw64GaW9fVrx-QlB_1RFdJ73Jwo8a8E8ZadEQS04A`, i.e., the key of the controller of
DID document that was replaced: this key can be found in the payload of the first 
signature.

```
[
  "eyJhbGciOiJFZERTQSJ9.eyJpZCI6ICJkaWQ6c2VsZjpiOWV3THpsanBleENaRkJtX2xUd2NSa2pnbWpPMmdkcWg4RmxuZmJjZXpJIiwgImNvbnRyb2xsZXIiOiAiZGlkOmtleTp1SHJIdzY0R2FXOWZWcngtUWxCXzFSRmRKNzNKd284YThFOFphZEVRUzA0QSIsICJzaGEtMjU2IjogInBhcG5zaVlUXzdqMnFiUTN6Y0htOUhIRFg0VVpOTkFwa3pxbTZXajhfU3MifQ.yINGHRO7sAG2vEMmtoD1XW-QMguhH2xOGYdneWcMa-jQcfZZqksbGSZm7DZNpyk9eUcRoO420uROdPWgNew5CA",
  "eyJhbGciOiJFZERTQSJ9.eyJpZCI6ICJkaWQ6c2VsZjpiOWV3THpsanBleENaRkJtX2xUd2NSa2pnbWpPMmdkcWg4RmxuZmJjZXpJIiwgImNvbnRyb2xsZXIiOiAiZGlkOmtleTp1TkRCSE5wTU11ZUdkT1lxZ0ZtZXZySzFERm5nYktaa2xzU0tDZVp6eHMzZyIsICJzaGEtMjU2IjogIkFJeDJKVFBsWEZTcGdGYnE1NEsyTXJnaTNDSWF4LWt4bkc1aTJvUy1HelEifQ.tH3hqjpiKa6hTCYDyFJAGfssvalR5mGs8PWDPRook56iZyEnCg17St1L8b_B28yH9e-b_tlzLWTzfgFVAO05Bg"
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





