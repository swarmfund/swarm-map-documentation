FORMAT: 1A

# Market Access Protocol

Market Access Protocol (MAP) solves existing inefficiencies and difficulties encountered by token issuers, investors, exchanges, and qualification providers. 

Until now, there has been no ‘one’ globally compliant protocol to provide a framework on which the industry can build to integrate and transfer security tokens in a uniform way. This is because each centralized, for-profit entity is incentivized to do what is best for themselves, not to think about interoperability or broader access beyond their scope. To that end, MAP is an open source infrastructure, designed to be the industry standard protocol empowering the entire security token market.

Market Access Protocol allows:

- Qualification Providers
- Token Issuers
- Investors
- Wallets
- Exchanges

to easily add a compliance layer to any cryptocurrency based digital asset.

# API structure

## URL and REST structure

MAP API uses HTTPS only. All HTTP requests will be completely ignored.

The API relies on REST HTTP verbs to interact with the objects in the system. 

---

Resources are created/registered using the POST action.

        `POST /token-issuers`
        
If the resource already exists, it is returned with a `409 Conflicts` instead of a `201 Created` response code.

---

Resource data is retrieved using the GET action.

        `GET /token-issuers/<pubKey>`
        
--- 

Relationships are created between resources using the PUT action. The exact resource on which the relationship will be attached is implicitly recognized from the successfuly authenticated user.

        `PUT /token-issuer/<pubKey>`


---

`DELETE /qualification-providers/<pub>/certs/<uid>`


## Response structure


A standard response structure will include the following header:

    `Content-Type: application/json`
    

## Data types

The response body will include a **JSON** encoded object of the resource requested.
The JSON structure returned is limited to primitives (boolean, number, string) and compounds (array, objects, null).

Date and DateTime elements are explicitly expected to be formatted in **ISO8601**.

DateTime elements accept a timezone in input but will always return a **UTC** formatted **ISO8601** string.

### Response codes and errors


**200**: OK - the action was executed and completed

**201**: Created - standard response to a POST action which also indicates that resource creation is also completed

Errors are indicated by http response code along with a corresponding human readable message. Additional details will be attached as a JSON encoded resource structure.

Standard errors are:

**400**: Parameter validation failed
            
**401**: Authentication headers required or failed
    
**403**: Requested resource is not available for the currently authenticated user

**404**: Resource not found

**405**: Method not allowed

**409**: Conflict

**429**: Too many requests, slow down and try again later

**422**: Unprocessable Entity due to payload being too large in size.

**500**: Resource action failed due to internal error

**503**: Resource is currently unavailable due to maintenance or overloading, please try again later

## API versioning

The MAP API currently does not support versioning.

Versioning will be supported once braking changes are inevitable due to software improvement.

# Authentication

All users using the MAP API are authenticated using their public keys. Details of the authentication mechanism will be determined.

For now, we will assume the use of provided headers.

Whenever authentication is needed, these HTTP headers are required:

| Header name      | Description                                                                                   | Example                            |
|------------------|-----------------------------------------------------------------------------------------------|------------------------------------|
| X-MAP-Blockchain | The underlying blockchain of the wallet, needed to determine the exact signing algorithm used | Bitcoin                            |
| X-MAP-Address    | The public key of the wallet                                                                  | 1HZwkjkeaoZfTSaJxDw6aKkxp45agDiEzN |
| X-MAP-Signature  | The wallet address signed by the wallet's private key                                         |  NG8tFK3dNwQaiYA0VOu...            |


> ***WARNING:*** We **strongly** recommend generating a new wallet when first registering with MAP to prevent use of a signed public key that is compromised from a third-party service which uses a similar authentication scheme.
***NOTE:*** Sucessfully authenticated `PUT` requests will attach a relationship to the resource owned by the authenticated user. 

# Getting started

## Qualification Provider

The role of a qualification provider is to issue certificates representing verification checks against any information about a subject. In the context of MAP, certificates usually verify the authenticity or truth of some data about an investor.

**Steps to register as a qualification provider:**

1. [Register](#reference/qualification-provider/account/register)
2. [Get qualification provider](#reference/qualification-provider/api-calls/get-qualification-provider)
2. [Certificate registration](#reference/qualification-provider/certificates/create-certificate)
3. [List of all certificates](#reference/qualification-provider/certificates/get-certificate-list)
4. [Get certificate](#reference/qualification-provider/certificates/get-certificate-by-uid)
5. [Delete certificate](#reference/qualification-provider/certificates/delete-certificate)
5. [Register claim about investor](#reference/qualification-provider/claims/register-claim-about-investor)

<!--![Qualification Provider Flow](http://swarm-map-public-images.s3-website.us-east-2.amazonaws.com/Qualification-provider-light.svg)-->

## Token Issuer

The token issuer is the user which registers an asset on the MAP blockchain.

1. [Register](#reference/token-issuer/api-calls/register)
2. [Get token issuer](#reference/token-issuer/api-calls/get-token-issuer)
3. [Register a new asset](#reference/token-issuer/api-calls/register-asset)
4. [List of wallet connection requests](#reference/token-issuer/api-calls/list-of-wallet-connection-requests)
5. [Get wallet connection request](#reference/token-issuer/api-calls/get-wallet-connection-request)
6. [Update wallet connection request](#reference/token-issuer/api-calls/update-wallet-connection-request)
7. [Update Token Fundamentals](#reference/token-issuer/api-calls/update-token-fundamentals)

    <!--![Token Issuer Flow](http://swarm-map-public-images.s3-website.us-east-2.amazonaws.com/Token-issuer-light.svg)-->

## Investor

Investors may require to hold certain certificates pertaining to the token they wish to buy, verifying the authenticity of some necessary data or that they meet certain criteria. These certificates are specified by the Token Issuer in order to comply with their specific regulatory requirements.

1. [Send wallet connection request](#reference/investor/send-wallet-connection-request/send-wallet-connection-request)
2. [GET Authorization signature](#reference/investor/get-authorization-signature/get-authorization-signature)

<!--![Investor Flow](http://swarm-map-public-images.s3-website.us-east-2.amazonaws.com/Investor-light.svg)-->


## API integration

For using the SDK and MAP API please consult the [MAP API Docs](/docs/api).

## MAP blockchain

MAP is a registry of tokens, qualification providers, certificates and associations. It is architected as a decentralized network of nodes with a foundational blockchain layer. This permits:
- an open market for compliance;
- a fully auditable trail of compliance transactions;
- permissionless integration and participation;
- reusability of qualifications;
- accountability through cryptographic proofs.

Privacy of personal investor information is preserved by decoupling the process of *requesting certificates* from the process of *associating* a wallet with a certificate.

All actions triggered via the API are in fact blockchain transactions and are associated with each address (e.g. Token Issuer's).

The MAP blockchain is an immutable store of records that has two basic types of transactions:
1. Qualification - The action of issuing a certificate to a requester
2. Association - The action of associating a certificate with a wallet


## Transfer Restriction rules


Transfer restrictions and rules are written by the token issuer into a document called Token Fundamentals which is referenced in the original token contract. Restrictions and rules are required to be checked in the process of approving a transfer. Since rules can include properties that are only known by the token contract itself at the time of transfer, they are defined in two groups:
1. rules to be checked natively on-chain where the token contract is deployed
2. rules to be checked outside the native chain by MAP nodes.


### Transfer authorization flow


Transfer rules which are required to be checked outside of the original chain—by MAP nodes—are written in the token fundamentals. Transfer rules which happen on-chain are referenced from the token contract.

Transfers that require checks outside the original chain must include an authorization signature from a MAP node with proof that these checks have been performed.
An example token contract should have the transfer method with at least the following signature:

```
transfer(address _to, uint256 _value, uint256 _nonce, uint256 _expiration, bytes32 msgHash, uint8 v, bytes32 r, bytes32 s) public returns (bool success)
```

where msgHash consists of `(asset, from, to, data, nonce, expiration)` values. The **nonce** is used to prevent replay attacks while **expiration** represents the authorization's validity period. 

If token fundamentals include rules pertinent to the native token contract’s blockchain, the transfer method implementation should account for additional checks to be performed on-chain before allowing a transfer to be executed.

Transfers that require MAP authorization contain the following properties which are defined by the rules declaratively stated in token fundamentals. 
1. Asset
2. Transfer
3. Transfer context

####Asset properties `(<token> prefix)`

Stable properties acquired from token fundamentals or the token contract.
1. Creation time
2. Supply
3. Decimals
4. Blockchain
5. Network

####Transfer properties `(<transfer> prefix)`

Properties taken from transfer request.
1. Asset
2. From
3. To
4. Value


####Transfer context `(<ctx> prefix)`

Delivered to MAP nodes accepting authorization request.
1. Nonce
2. Expiration
3. IP
4. Timestamp
5. GEOIP

#### MAP properties `(<map> prefix)`

Properties MAP holds about objects such as wallets, assets, token issuers, qualification providers, etc.

### Transfer restriction rules

As stated earlier, two sets of transfer restriction rules are defined in token fundamentals:

1. Rules enforced natively on-chain where the token contract is deployed
2. Rules enforced by MAP which are not possible or economically infeasible to be performed natively on-chain

For now, the first set of rules are defined by a single address of the contract enforcing the rules, and MAP enforced rules are listed in token fundamentals. 

Example of the rules section of token fundamentals:

```
transferRules: {
    contract: {
        address: "0x..."
    },
    map: {
        type: "cert-list",
        version: 1,
        description: "https://example/assets/XYZ/rules.html"
        rules: [{
            ...
        }]
    }
}
```

### Properties of transfer

Transfer rules have the following properties:

1. `type`: an identifier for the types of rules stated in the *rules* property, allowing for various implementations on how to check them. For example, “type: cert-list” would list all certification types that a wallet requires to be checked by a simple rule-engine. “Type: transfer-rules” could include a much more complex set of rules.
2. `version`: maybe not needed as type can handle versioning issues.
3. `description`: URL of a document describing token transfer rules to investors. 
4. `rules`: JSON document with a definition of rules for the specific type.

MAP nodes need to have an implementation of rule checks for transfer_rules.map.type, or transfer restriction authorizations will return an error indicating unknown rule type in Token Fundamentals.

## Token fundamentals document

The token fundamentals document has, at least, the following top level properties:

1. `token`: token properties listing which can be used on MAP node.
2. `properties`: various token properties, like an icon, description, etc.
3. `transferRules`: properties defining native on-chain and MAP transfer rules.

Example of token fundamentals document follows.

```
{
    token: {
        blockchain: Ethereum,
        network: 1,
        address: "0x...",
        name: "Security token",
        symbol: "XYZ",
        decimals: 18,
        supply: 10000000,
        created: "2019-02-19T12:04:50+00:00"  
    },
    properties: {
        ...
    },
    transferRules: {
    contract: {
        address: "0x..."
    },
    map: {
        type: "cert-list",
        version: 1,
        description: "https://example/assets/XYZ/rules.html"
        rules: [{
            ...
        }]
    }
}
```




# Group Qualification Provider

Qualification providers are entities which can verify information about identities and issue different types of trusted certificates. Qualification providers define details about their certificates and the required data the subject needs to provide in order to obtain one. One example of such certificates would be a KYC certificate that can be issued to prospective investors. Qualification providers must register themselves and update a list of their supported certificates 
on MAP, with certificate types identified by a unique id.

Instances of certificates issued to subject identities are claims. Information about the existence of a claim is
registered on MAP, along with the basic data required to check the availability of registered Claim. The claim is identified by a certificate's unique id and subject's public key.

## API calls [/qualification-providers]

### Register [POST]

Registers new qualification provider.

+ Request (application/json)

    + Headers
        
            X-MAP-Signature: 1HZwkjkeaoZfTSaJxDw6aKkxp45agDiEzNG8tFK3dNwQaiYA0VOuhMBOjcdydR5gyKh/IAr9rSqDXy93cVqKDZkYv/LjfLpJ8PfJKGrUovjisT9rhMWg8Z5wM=
    
    + Body
    
            {
                name: "The user-provided arbitrary name of the qualification provider",
                publicKey: "{publicKey}",
            }
    
    + Attributes

        + name: `The user-provided arbitrary name of the qualification provider.` (required)
        + publicKey: `public key (ie. ED25519) QP will use to sign objects.` (required)

+ Response 201 (application/json)

    + Attributes (Qualification provider)            

## Get qualification provider [GET /qualification-providers/{pubKey}]

Retrieve qualification provider object.

+ Request

    + Parameters
    
        + pubKey: Ed25519 public key QP will use to sign objects.

+ Response 200 (application/json)

    + Attributes (Qualification provider)
    
## Certificates [/certificates]

A certificate is a cryptographically proven verification of the authenticity or truth of some data that is issued by anyone taking the role of a qualification provider.

Examples of a certificate may be:
- Proof of identity
- Proof of residence
- Sanctions list clearance

### Create certificate [POST /qualification-providers/{pubKey}/certificates]

Register possible certificate by qualification provider on MAP.

+ Request (application/json)
    
    + Headers

            X-MAP-Signature: 1HZwkjkeaoZfTSaJxDw6aKkxp45agDiEzNG8tFK3dNwQaiYA0VOuhMBOjcdydR5gyKh/IAr9rSqDXy93cVqKDZkYv/LjfLpJ8PfJKGrUovjisT9rhMWg8Z5wM=
    
    + Body
    
            {
                description: "Description of certificate",
                constraints: {
                    ...
                }
            }
            
    + Attributes
            
        + description: Holds specific details of this qualification. (required)
        + constraints: Additional constrainst that may apply to the certificate. (optional)

+ Response 201

    + Attributes (Certificate)
    

### Get certificate list [GET /qualification-providers/{pubKey}/certificates/]

List certificates for requested qualification provider.

+ Request (application/json)

+ Response 200

    + Attributes (array[Certificate])
    

### Get certificate by UID [GET /qualification-providers/{pubKey}/certificates/{uid}]

Get details about specific certificate.

+ Request (application/json)

+ Response 200

    + Attributes (Certificate)
    

### Delete certificate [DELETE /qualification-providers/{pubKey}/certificates/{uid}]

Remove certificate registration. Further claim registration on this certificate is not possible, but issued claims using this certificate remain valid until specific invalidation.

+ Request (application/json)

+ Response 200

## Claims [/claims]

### Register claim about investor [POST /qualification-providers/{pubKey}/claims]

Register issuance of claim for specific investor and certificate.

+ Request (application/json)
    + Headers

            X-MAP-Signature: 1HZwkjkeaoZfTSaJxDw6aKkxp45agDiEzNG8tFK3dNwQaiYA0VOuhMBOjcdydR5gyKh/IAr9rSqDXy93cVqKDZkYv/LjfLpJ8PfJKGrUovjisT9rhMWg8Z5wM=
    
    + Body
    
            {
                issuer: "",
                expirationDate: "",
                investor: "",
                certificate: ""
            }
            
    + Attributes
            
        + issuer: Qualification provider public key. (required)
        + expirationDate: Expiration datetime. (required)
        + investor: Investor public key. (required)
        + certificate: Certificate UID. (required)

+ Response 201

    + Attributes
        
        + uid: Generated claim uid.
        + issuer: Qualification provider public key.
        + expirationDate: Expiration datetime.
        + investor: Investor public key.
        + certificate: Certificate UID.


# Group Token Issuer

Token issuers are entities who control token operations, and on MAP are primarily responsible for token and investor's wallet registration. They need to register themselves on the MAP for other parties to authenticate their actions. 

Apart from simple registration of themselves and their tokens, the most important task token issuers are handling
is responding to investor's wallet registration requests - separating investor's Wallet from investor's claims. 
Token issuers are the only entities in MAP in possession of investor-Wallet connection information, handling registration tasks and saving connection information off-site.

## API calls [/token-issuers]

### Register [POST]

Registers the token issuer.

+ Request (application/json)

    + Headers
        
            X-MAP-Signature: 1HZwkjkeaoZfTSaJxDw6aKkxp45agDiEzNG8tFK3dNwQaiYA0VOuhMBOjcdydR5gyKh/IAr9rSqDXy93cVqKDZkYv/LjfLpJ8PfJKGrUovjisT9rhMWg8Z5wM=
    
    + Attributes
    
        + name: `Token issuer name` (string, required)
        + publicKey: `Public key` (string, optional)

+ Response 201 (application/json)

    + Attributes (Token issuer)
    
### Get token issuer [GET /token-issuers/{pubKey}]

Gets the information about token issuer.

+ Response 200 (application/json)

    + Attributes (Token issuer)
    
### Register asset [POST /token-issuers/{pubKey}/assets]

Adds a new asset to the MAP blockchain.

+ Request (application/json)

    + Headers
        
            X-MAP-Signature: 1HZwkjkeaoZfTSaJxDw6aKkxp45agDiEzNG8tFK3dNwQaiYA0VOuhMBOjcdydR5gyKh/IAr9rSqDXy93cVqKDZkYv/LjfLpJ8PfJKGrUovjisT9rhMWg8Z5wM=
    
    + Body
    
            {
                address: ""
                name: "",
                description: "",
                symbol: "",
                decimals: 0,
                supply: 0,
                fundamentals: {
                    hash: "",
                    url: ""
                }
            }
    
    + Attributes
    
        + address: `Token contract address` (string, required)
        + name: `The asset name` (string, required)
        + description: `The description of asset` (string, required)
        + symbol: `Token symbol` (string, required)
        + decimals: `Token decimals` (number, required)
        + supply: `Initial token supply` (number, required)
        + properties: `Some other fields that will be inserted` (string, optional)
        + fundamentals: `Asset description document` (fundamentals, required)

+ Response 201 (application/json)

    + Attributes (Asset)
    
    
### List of wallet connection requests [GET /token-issuers/{pubKey}/assets/{pubKey}/requests?status={status}]

List wallet connection requests.

+ Request

    + Parameters
    
        + status: `type of status` (string, optional)

+ Response 200 (application/json)

    + Attributes (array[string])
    
    
### Get wallet connection request [GET /token-issuers/{pubKey}/assets/{pubKey}/requests/{uid}]

Retrieve specific wallet connection request.

+ Response 200 (application/json)

    + Attributes
        + request: `Encrypted investor request.` (string, required)

### Update wallet connection request [PUT /token-issuers/{pubKey}/assets/{pubKey}/requests/{uid}]

Update specific wallet connection request. This call is used to reassign identity claims to a Wallet. 

+ Request 
    + Body 
        {
            status: "",
            claims: []
        }
    
    + Attributes
        + status: `Status of request` (string, required)
        + claims: `Array of claims for wallet` (array[string], required)

+ Response 200 (application/json)

### Update token fundamentals [PUT /token-issuers/{pubKey}/assets/{pubKey}/fundamentals]

Update token fundamentals document properties. 

+ Request 
    + Body 
        {
            hash: "",
            url: ""
        }
    
    + Attributes (fundamentals)

+ Response 200 (application/json)


# Group Investor

Investors, identified by their MAP public keys, are entities expected to obtain certificates from qualification
providers in form of MAP registered claims and participate in assets trading by registering wallets with token issuers. 


## Investor wallet registration
Protection of investor's privacy is accomplished by separating wallet and investor objects, and keeping that
connection information off-site from MAP in the token issuer's records. Wallet registration flow facilitates that
separation:

1. An investor sends a wallet registration request for a selected asset to MAP, encrypting the whole request object in such a manner that only the destination token issuer can read the information about the investor and the connecting Wallet.

2. The token issuer handles the request by recording investor-wallet connections for possible future regulatory queries, reissuing investor claims with the same certificates and properties, and connecting a new wallet's public key.


Any entity wishing to check a wallet's certificates for a particular token can use the token issuer claims about that wallet, extending trust from Qualification Provider to the Issuer of the particular token.

### Send wallet connection request [POST /token-issuers/{pubKey}/assets/{pubKey}/requests]

Send register wallet request with asset to token issuer. Encrypted request data object consists of:

1. Investor public key
2. Details of wallet (blockchain, address)
3. Investor signature
4. Wallet signature
5. Decrypted Investor claim properties

Request data object is encrypted with the designated asset's token issuer key.

+ Request (application/json)

    + Body
    
            {
                request: "encrypted request data",
            }
    
    + Attributes

        + request: `Encrypted request data.` (string, optional)

+ Response 200 (application/json)
+ Response 404 (application/json)
+ Response 500 (application/json)


### Get authorization signature [GET /authorizations]

An Authorization signature must be obtained for transfer rules which require checks made by MAP. 
This authorization needs to be obtained before the transfer method is called in the token contract and is given after all rules are checked by MAP nodes.

Authorization returned needs to carry a relatively short expiration time, to prevent a user from using it too far into the future.


+ Request (application/json)

    + Body
        {
            asset: "",
            from: "",
            to: "",
            value: ""
        }
        
    + Attributes
        + assets: `UID of asset` (string, required)
        + from: `Asset's wallet` (string, required)
        + to: `Asset's wallet` (string, required)
        + value: `Transfer value` (string, required)

+ Response 200
    + Attributes
        + assets: `UID of asset` (string, required)
        + from: `Asset's wallet` (string, required)
        + to: `Asset's wallet` (string, required)
        + value: `Transfer value` (string, required)
        + authorization: `signed authorization data <asset/from/to/value/nonce/expiration>` (string, required)
        + nonce: `Incremented nonce` (string, required)
        + expiration: `Expiration of authorization` (string, required)
        

      

# Group Exchange

Exchanges use MAP to get claims for specified Investors.

### Get investor claim [GET /claims/{pubKey}:{uid}]

Retrieve certificate claim for investor.

+ Request (application/json)
    + Attributes

        + pubKey: `Investor MAP public key` (string, required)
        + uid: `Certificate uid` (string, required)

+ Response 200 (application/json)
+ Response 404 (application/json)
+ Response 500 (application/json)
            

# Data structure


## Token issuer (object)

+ name: `The token issuer name` (string, required)
+ publicKey: `The public key of the token issuer.` (string, optional)


## Asset (object)

+ uid: `The asset uid` (string, required)
+ name: `The user-provided arbitrary name of the asset` (string, required)
+ description: `The user-provided arbitrary description of the asset` (string, required)
+ fundamentals: `The Token Fundamentals` (fundamentals, required)


## Qualification provider (object)

+ name: `The user-provided arbitrary name of the qualification provider.` (string, required)
+ publicKey: `Ed25519 public key QP will use to sign objects.` (string, required)


## Certificate (object)

+ uid: `Generated certificate uid` (string, required)
+ description: `Holds specific details of this qualification.` (string, required)
+ constraints: `Additional constrainst that may apply to the certificate.` (optional)

## fundamentals (object)

+ hash: `sha256 hash of document data.` (string, required)
+ url: `location of documents.` (string, required)


## Investor (object)

+ name: `The user-provided arbitrary name of the token issuer` (string, required)
+ ipfsMetadata: `The IPFS hash which contains arbitrary user-provided metadata.` (string, optional)
+ domain: `The fully qualified domain name which will be used for Qualification provider verification purposes.`
