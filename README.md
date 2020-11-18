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
of an Ed25519 public key. 

## CRUD Operation Definitions
CRUD operations are implemented by a software library which is executed by the user itself. 

### Create
The create operation accepts as input a DID, a `minimal DID-document`, and a `proof`. 
The minimal DID document is a JSON file that MUST contain at least a `controller` field.
The value of the `controller` field is equal to the DID. Although it is possible to authenticate
a controller based only on its DID, it is recommended that the DID-document includes an
`authentication` field, which should include the controller key. Then, this field can be used by 
legacy DID systems.

A proof is a compact encoded
JWS, using EdDSA and payload a JSON string with three fields: `did`, `controller`, and `sha-256`. 
We refer to this proof as the **self-did proof**.

The self-did proof of the create method is signed using the private key that corresponds to the DID. 


A minimal DID document and the corresponding self-did proof payload for the DID `did:self:6varD0RjXZfW58v4DGtd7kltX6Kzn9fghX94LvrMDxo` follows:

DID document:
```
{
  "id": "did:self:GwJkufDcxEs9PN5-hILb1rcmD4GXCIohPvTuSdEOo9w", 
  "controller": "did:self:GwJkufDcxEs9PN5-hILb1rcmD4GXCIohPvTuSdEOo9w", 
  "authentication": [{
    "id": "did:self:GwJkufDcxEs9PN5-hILb1rcmD4GXCIohPvTuSdEOo9w#key1", 
    "type": "JsonWebKey2020", 
    "controller": "did:self:GwJkufDcxEs9PN5-hILb1rcmD4GXCIohPvTuSdEOo9w", 
    "publicKeyJwk": {
      "crv": "Ed25519", 
      "x": "GwJkufDcxEs9PN5-hILb1rcmD4GXCIohPvTuSdEOo9w", 
      "kty": "OKP"
      }
  }]
}

```

self-did proof payload
```
{
  "id": "did:self:6varD0RjXZfW58v4DGtd7kltX6Kzn9fghX94LvrMDxo", 
  "controller": "did:self:6varD0RjXZfW58v4DGtd7kltX6Kzn9fghX94LvrMDxo", 
  "sha-256": "0q6bmSX0iXXkrZxLY5QhAkrjCTuYx01kRKgDy85eOMg"
}

```
### Update
The update operation, replaces an existing DID document with a new one. It accepts as input
the `new DID-document`, and a `self-did proof`. The self-did proof is
signed using the private key that corresponds to the controller of the 
**DID-document being replaced**.

Proofs are stored in a list referred to as the **proof chain**. If the provided
self-did proof has the same signer as the last element of the proof chain, then the last element
of the chain is replaced with with the provided proof. Otherwise,
the provided proof is appended to the proof chain.

### Read
The Read method receives as input a DID, and outputs the latest version of the DID document and 
the self-did proof chain. The self-did proof chain is verified as follows:
* First, verify that the last element of the chain contains a valid payload.
* Verify that the payload of all chain elements contain a `id` field with value equal to the DID.  
* Then, for each element of the chain but the first, verify the signature using the 
public key included in `controller` field of the payload of the **previous** element.
* Finally, verify the signature of the first element of the chain using the key that corresponds to the
DID.  




