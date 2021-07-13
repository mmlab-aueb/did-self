# did:self method specification
## Author
* Nikos Fotiou, [Mobile Multimedia Laboratory, AUEB](https://mm.aueb.gr)

## About
did:self is a DID method that enables DID document management without registries. 
A did:self identifier is an Ed25519 public key.
The corresponding 
DID document is protected by a "proof", which is a JSON Web Signature generated
by the private key that corresponds to the did:self identifier.

A Python3 [implementation](https://github.com/mmlab-aueb/did-self-py)

### Research
* Our [SCN4ND](https://mm.aueb.gr/scn4ndn/) project that uses did:self.
* Application of did:self in IPFS [N. Fotiou, V.A. Siris, G.C. Polyzos,
"Enabling self-verifiable mutable content items in IPFS using Decentralized 
Identifiers", in DI2F: Decentralising the Internet with IPFS and Filecoin, IFIP Networking 2021 workshop](https://arxiv.org/abs/2105.08395)
* Application of did:self in securing routing in Named Data Networking
[N.Fotiou, Y. Thomas, V.A. Siris, G. Xylomenos and G.C. Polyzos, "Securing Named Data Networking routing using Decentralized Identifiers," in Proc. SARNET-21 workshop, IEEE International Conference on High Performance Switching and Routing (HPSR), Paris, France, June 2021](https://mm.aueb.gr/publications/12279f1a-8166-4560-aead-56dfe90df93f.pdf)

## The did:self method 
The name of this DID method is: `self`

The method specific identifier is represented as the base64url value
of an Ed25519 public key. e.g.,

```
did:self:6varD0RjXZfW58v4DGtd7kltX6Kzn9fghX94LvrMDxo
```

The DID document is a JSON-encoded file that must include at least
the `id` property.  

The integrity of a DID document is verified using a 
`document proof`. A `document proof` is a compact encoded 
[JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515).
The payload of the proof is a JSON string that includes at least the following 
fields: 

* `id` The DID.
* `created` The string value of an ISO8601 combined date and time string
* `sha-256` The base64url encoded hash of the DID document, calculated using SHA-256.

The signature of the `document proof` is an EdDSA signature which is generated 
by the  DID `controller`. 

If the `controller` is the DID `owner`, the signature is generated using the 
private key that corresponds to the did:self DID. 

If the `controller` is another entity, the signature is generated using a private key
owned by that entity. In this key an additional `delegation proof` is required 
to prove that the controller key is authorized by the DID owner to sign a DID 
document proof. 

A `delegation proof` is also a JWS, which is signed by the DID owner, and 
its payload is a JSON string that includes at least the following fields:

* `id` The DID.
* `created` The string value of an ISO8601 combined date and time string
* `controller` The public key of the controller encoded as a JSON Web Key. 


## CRUD Operation Definitions
CRUD operations are implemented by the users. 

### Create
The Create operation initializes a did:self DID and creates a DID document. 
The proof of the DID document is signed using the controller key.
The following is a valid DID document

```JSON
{
  "id": "did:self:0Ytyg58WeBIq9awMyxvo8WPqikfKI8L6MIe_Ju5eRY0",
  "authentication": [
    {
      "id": "did:self:0Ytyg58WeBIq9awMyxvo8WPqikfKI8L6MIe_Ju5eRY0#key1",
      "type": "JsonWebKey2020",
      "publicKeyJwk": {
        "kty": "OKP",
        "crv": "Ed25519",
        "x": "0Ytyg58WeBIq9awMyxvo8WPqikfKI8L6MIe_Ju5eRY0"
      }
    }
  ]
}
```

The following is the corresponding proof payload

```JSON
{
  "id": "did:self:0Ytyg58WeBIq9awMyxvo8WPqikfKI8L6MIe_Ju5eRY0",
  "created": "2021-07-13T13:26:38Z",
  "sha-256": "J0MX-F2FErMx4pYJVF9XYPW78nHC0ZOuSERkkLRCzKE"
}
```

The proof signature is
```
580647ff0592ed0fa32a94ec8582b5eb47b57db06c8f4bbad75c42352d84a6927c8e9b7c48ea56bf9a529ebc4524c2a6f06807fb8fb849ad28cadcc505283a07
```


### Read
The Read operation simply outputs the DID document, 
the corresponding `document proof`, and if needed the `delegation proof`.

### Update
With the update operation, the DID document and the `document proof` are replaced
with new ones

### Delegate
With the delegate operation, the DID `owner`, delegates DID management
to a 3rd party entity. An already delegated DID cannot be further delegated. 
The operation outputs a `delegation proof` which must be transmitted to the 
new controller. The following is an example of the payload of a `delegation proof` 

```JSON
{
  "id": "did:self:0Ytyg58WeBIq9awMyxvo8WPqikfKI8L6MIe_Ju5eRY0",
  "created": "2021-07-13T13:26:38Z",
  "controller": {
    "kty": "OKP",
    "crv": "Ed25519",
    "x": "1eDZExzypVtXSpSKFrWFUQ8fTuhFOZYMloFmVk26z7M"
  }
}
```

## DID document validation
Given a did:self DID, a DID document, a `document proof`, and optionally a 
`delegation proof`, any entity can attest whether or not the DID document is a 
valid document for the given DID using the following protocol.

1. If there is a `delegation proof` verify its signature using the did:self `DID`:
if it is valid, the public key of the `controller` is the public key included
in the delegation proof`.
1. If there is not a `delegation proof` the public key of the `controller` is the
did:self DID
1. Verify that the `sha-256` field of the payload of the `document proof`contains 
the hash of the DID document.
1. Verify that the `id` field of the payload of `document proof` is equal to the DID.
1. Verify the signature of the `document proof` using the public key of the
`controller`





