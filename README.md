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
The value of the `controller` field is equal to the DID. A proof is a compact encoded
JWS, using EdDSA and payload a JSON string with three fields: `did`, `controller`, and `sha-256`. 
We refer to this proof as the **self-did proof**.

The self-did proof of the create method is signed using the private key that corresponds to the DID. 


A minimal DID document and the corresponding self-did proof payload for the DID `did:self:6varD0RjXZfW58v4DGtd7kltX6Kzn9fghX94LvrMDxo` follows:
```
{
  "id": "did:self:6varD0RjXZfW58v4DGtd7kltX6Kzn9fghX94LvrMDxo", 
  "controller": "did:self:6varD0RjXZfW58v4DGtd7kltX6Kzn9fghX94LvrMDxo"
}

```
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

Proofs are stored in a list referred to as the **self-did proof chain**. If the provided
new DID-document has the same `controller` as the one being replaced, then the last element
of the  self-did proof chain is replaced with with the provided  self-did proof. Otherwise,
the provided self-did proof is appended to the self-did proof chain.

### Read
The Read method receives as input a DID, and outputs the latest version of the DID document and 
the self-did proof chain. The self-did proof chain is verified as follows:
* First, verify that the last element of chain contains a valid payload.
* Verify that the payload of all chain elements contains a `id` field with value equal to the DID.  
* Then, for each element of the chain but the first, verify the signature using the 
public key included in `controller` field of the payload of the **previous** element.
* Finally, verify the signature of the first element of the chain using the key that corresponds to the
DID.  




