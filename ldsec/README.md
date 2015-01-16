First stab at a vocabulary for signatures (and eventually encryption) in linked data...

This is still definitely a work in progress. One of the major unanswered questions is how the transform / c14n algorithm would work. The signature is based on the triples signed. Given something like...

```
<urn:thing> a owl:Thing;
  rdfs:label "Test" .
```

The normalized output would be:
```
<urn:thing> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://www.w3.org/2002/07/owl#Thing> .
<urn:thing> <http://www.w3.org/2000/01/rdf-schema#label> "Test" .
```

The challenge, however, is that we need to have a deterministic way of canonicalizing. Will address that problem later. For now, let's assume that magic is involved (or perhaps something like this: http://json-ld.org/spec/latest/rdf-graph-normalization/ ). There is some overlap here with https://web-payments.org/specs/source/secure-messaging/ that would need to be worked out.

```
<urn:Signature> a :Signature ;
  :resource [
    a :Transform ;
      :resource <urn:thing> ;
      :alg <http://example.org/ld-c14n> .
  ]  ;
  :alg <http://www.w3.org/2001/04/xmldsig-more#rsa-sha256> ;
  :key [ 
    a :RSAKey ;
      :n "{base64 data}"^^xsd:base64Binary ;
      :e "{base64 data}"^^xsd:base64Binary ;
      :d "{base64 data}"^^xsd:base64Binary ;
      :p "{base64 data}"^^xsd:base64Binary ;
      :q "{base64 data}"^^xsd:base64Binary ;
      :dp "{base64 data}"^^xsd:base64Binary ;
      :dq "{base64 data}"^^xsd:base64Binary ;
      :qi "{base64 data}"^^xsd:base64Binary .
  ];
  :sig "{base64 signature}"^^xsd:base64Binary .
```
This ought to be simple enough. `resource` identifies the thing to be verified. In this case, we're 
signing a transformed view of `urn:thing`. In the transform, `alg` identifies the canonicalization algorithm being applied. We can have several types of transformations (see below). In the signature object, `alg` identifies the signature algorithm used. `key` identifies the key used for signing. `sig` contains the actual signature value. 

We can nest the thing being verified:
```
<urn:Signature2> a :Signature ;
  :resource [
    a :Transform ;
      :resource [
        a owl:Thing ;
          rdfs:label "Test" .
      ] ;
      :alg <http://example.org/ld-c14n> .
  ]  ;
  :alg <http://www.w3.org/2001/04/xmldsig-more#ecdsa-sha512> ;
  :key [
    a :ECKey ;
      :crv "P-256" ;
      :x "{base64 data}"^^xsd:base64Binary ;
      :y "{base64 data}"^^xsd:base64Binary ;
      :d "{base64 data}"^^xsd:base64Binary ;
  ] ;
  :sig "{base64 signature}"^^xsd:base64Binary .
```

We can specify the key in a number of ways, and there's no reason we can't use the mechanism to verify non-RDF resources...
```
<urn:Signature3> a :Signature ;
  :resource <http://example.com/some-other-kind-of-thing>;
  :alg <http://www...> ;
  :x509 "{base64-encoded cert}"^^xsd:base64Binary
  :sig "{base64 signature}"^^xsd:base64Binary .
```

We can represent Keys too

```
<urn:myeckey> a :ECPublicKey ;
  :crv "P-256" ;
  :x "{base64 data}"^^xsd:base64Binary ;
  :y "{base64 data}"^^xsd:base64Binary ;
  :use :Signature ;
  :kid "1" .

<urn:myrsakey> a :RSAPublicKey ;
  :n "{base64 data}"^^xsd:base64Binary ;
  :e "{base64 data}"^^xsd:base64Binary ;
  :alg <http://www.w3.org/2001/04/xmldsig-more#rsa-sha256> ;
  :use :Signature ;
  :kid "2" .

<urn:mykeyset> a :KeySet ;
  :hasKey <urn:myeckey> ;
  :hasKey <urn:myrsakey> .
```

It's possible that we'd only want to sign a subset of the properties available for a resource. We'd need a way of selecting which bits to sign, or which bits to exclude.

To cherry pick which properties to sign, we'd use an InclusionTransform.

```
<urn:Signature> a :Signature ;
  :resource [
    a :InclusionTransform ;
      :resource <urn:thing> ;
      :alg <http://example.org/ld-c14n> :
      :select (
        <http://www.w3.org/2000/01/rdf-schema#label>
      ).
  ]  ;
  :alg <http://www.w3.org/2001/04/xmldsig-more#rsa-sha256> ;
  :key [ 
    a :RSAKey ;
      :n "{base64 data}"^^xsd:base64Binary ;
      :e "{base64 data}"^^xsd:base64Binary ;
      :d "{base64 data}"^^xsd:base64Binary ;
      :p "{base64 data}"^^xsd:base64Binary ;
      :q "{base64 data}"^^xsd:base64Binary ;
      :dp "{base64 data}"^^xsd:base64Binary ;
      :dq "{base64 data}"^^xsd:base64Binary ;
      :qi "{base64 data}"^^xsd:base64Binary .
  ];
  :sig "{base64 signature}"^^xsd:base64Binary .
```

The InclusionTransform is the best way to ensure that a signature does not become invalidated by the introduction of new triples but it's not foolproof. For instance, if a new triple adds a new value for an existing property already covered by the signature, the InclusionTransform will not filter out the new value, causing the signature to become invalidated.

To pick which properties to exclude, we'd use an ExclusionTransform.

```
<urn:Signature> a :Signature ;
  :resource [
    a :ExclusionTransform ;
      :resource <urn:thing> ;
      :alg <http://example.org/ld-c14n> :
      :select (
        <http://www.w3.org/2000/01/rdf-schema#label>
      ).
  ]  ;
  :alg <http://www.w3.org/2001/04/xmldsig-more#rsa-sha256> ;
  :key [ 
    a :RSAKey ;
      :n "{base64 data}"^^xsd:base64Binary ;
      :e "{base64 data}"^^xsd:base64Binary ;
      :d "{base64 data}"^^xsd:base64Binary ;
      :p "{base64 data}"^^xsd:base64Binary ;
      :q "{base64 data}"^^xsd:base64Binary ;
      :dp "{base64 data}"^^xsd:base64Binary ;
      :dq "{base64 data}"^^xsd:base64Binary ;
      :qi "{base64 data}"^^xsd:base64Binary .
  ];
  :sig "{base64 signature}"^^xsd:base64Binary .
```

Suppose we have a more complex type of object, with nested objects, etc and we want to make sure we only sign a particular subset. We can use a Selection.

```
<urn:Signature> a :Signature ;
  :resource [
    a :ExclusionTransform ;
      :resource <urn:thing> ;
      :alg <http://example.org/ld-c14n> :
      :select (
        <http://www.w3.org/2000/01/rdf-schema#label>
        [
          a :Selection ;
            :property <http://example.org/foo> ;
            :select <http://www.w3.org/2000/01/rdf-schema#label>
        ]
      ).
  ]  ;
  :alg <http://www.w3.org/2001/04/xmldsig-more#rsa-sha256> ;
  :key [ 
    a :RSAKey ;
      :n "{base64 data}"^^xsd:base64Binary ;
      :e "{base64 data}"^^xsd:base64Binary ;
      :d "{base64 data}"^^xsd:base64Binary ;
      :p "{base64 data}"^^xsd:base64Binary ;
      :q "{base64 data}"^^xsd:base64Binary ;
      :dp "{base64 data}"^^xsd:base64Binary ;
      :dq "{base64 data}"^^xsd:base64Binary ;
      :qi "{base64 data}"^^xsd:base64Binary .
  ];
  :sig "{base64 signature}"^^xsd:base64Binary .
```

The above basically states, Transform the resource `urn:thing` by first excluding the objects `http://www.w3.org/2000/01/rdf-schema#label` property, then selecting all values of the `http://example.org/foo` object property and excluding the selected objects `http://www.w3.org/2000/01/rdf-schema#label` property. By omitting the `:property` on the selection, you can select all object properties on the selection. We 
can then recursively exclude all rdf:labels from our signature by using circular references:

```
_:c14n0 a :Selection ;
  :select (
    <http://www.w3.org/2000/01/rdf-schema#label>,
    _:c14n0
  ) .

<urn:Signature> a :Signature ;
  :resource [
    a :ExclusionTransform ;
      :resource <urn:thing> ;
      :alg <http://example.org/ld-c14n> :
      :select _:c14n0 .
  ]  ;
  :alg <http://www.w3.org/2001/04/xmldsig-more#rsa-sha256> ;
  :key [ 
    a :RSAKey ;
      :n "{base64 data}"^^xsd:base64Binary ;
      :e "{base64 data}"^^xsd:base64Binary ;
      :d "{base64 data}"^^xsd:base64Binary ;
      :p "{base64 data}"^^xsd:base64Binary ;
      :q "{base64 data}"^^xsd:base64Binary ;
      :dp "{base64 data}"^^xsd:base64Binary ;
      :dq "{base64 data}"^^xsd:base64Binary ;
      :qi "{base64 data}"^^xsd:base64Binary .
  ];
  :sig "{base64 signature}"^^xsd:base64Binary .
```

Suppose we have multiple separate objects that we want sign all at once. We can accomplish that using the `MergeTransform`

```
<urn:Signature> a :Signature ;
  :resource [
    a :MergeTransform ;
      :resource <urn:thing> ;
      :alg <http://example.org/ld-c14n> :
      :select (
        <urn:thing2>
        <urn:thing3>
      ).
  ]  ;
  :alg <http://www.w3.org/2001/04/xmldsig-more#rsa-sha256> ;
  :key [ 
    a :RSAKey ;
      :n "{base64 data}"^^xsd:base64Binary ;
      :e "{base64 data}"^^xsd:base64Binary ;
      :d "{base64 data}"^^xsd:base64Binary ;
      :p "{base64 data}"^^xsd:base64Binary ;
      :q "{base64 data}"^^xsd:base64Binary ;
      :dp "{base64 data}"^^xsd:base64Binary ;
      :dq "{base64 data}"^^xsd:base64Binary ;
      :qi "{base64 data}"^^xsd:base64Binary .
  ];
  :sig "{base64 signature}"^^xsd:base64Binary .
```

This will merge the triples from `urn:thing2` and `urn:thing3` in with the triples of `urn:thing` so that they are all signed together.

Suppose you have an object such as:

```
<urn:thing> a owl:Thing ;
  <http://example.org/property> [
    a owl:Thing ;
      rdf:label "A random object"
  ] .
```

And you want to generate the signature on the blank node object value of `http://example.org/property` rather than on the `urn:thing` itself. You can use a :Selection to do so:

```
<urn:Signature> a :Signature ;
  :resource [
    a :Selection ;
      :resource <urn:thing> ;
      :alg <http://example.org/ld-c14n> :
      :property <http://example.org/property> .
  ]  ;
  :alg <http://www.w3.org/2001/04/xmldsig-more#rsa-sha256> ;
  :key [ 
    a :RSAKey ;
      :n "{base64 data}"^^xsd:base64Binary ;
      :e "{base64 data}"^^xsd:base64Binary ;
      :d "{base64 data}"^^xsd:base64Binary ;
      :p "{base64 data}"^^xsd:base64Binary ;
      :q "{base64 data}"^^xsd:base64Binary ;
      :dp "{base64 data}"^^xsd:base64Binary ;
      :dq "{base64 data}"^^xsd:base64Binary ;
      :qi "{base64 data}"^^xsd:base64Binary .
  ];
  :sig "{base64 signature}"^^xsd:base64Binary .
```

Note, however, that if `url:thing` happened to have multiple values for `http://example.org/property`, all of the values would be included in the selection and the signature would be generated based on complete graph. 

--------

We can handle Encrypted Content also ...

```
<urn:myencrypteddata> a :CipherObject ;
  :alg <http://www.w3.org/2001/04/xmlenc#aes256-cbc> ;
  :key "{the key}"^^:JSONEncryptedKey ;
  :iv "{the initialization vector}"^^xsd:base64Binary ;
  :mediaType "text/plain" ;
  :value "{the encrypted data}"^^xsd:base64Binary .
```

The primary goal here is to have a Verification and Encryption mechanism that works well with the Linked Data model while still allowing things like JSON Web Encryption, JSON Web Signature and JSON Web Key to work. The JOSE mechanisms work well when transmitting data over the wire, they are not so good when dealing with Linked Data. The vocabulary declares rdfs:Datatypes for JSON Web Encryption, JSON Web Signatures and JSON Encrypted Web Keys all in compacted form.

The other possibility here, of course, is to declare a JSON-LD context for JSON Web Signatures and Encryption. Such as:

```json
{
  "@context": {
    "jose": "http://www.ietf.org/id/draft-ietf-jose-json-web-signature-39#",
    "payload": {
      "@id": "jose:payload",
      "@type": "xsd:base64Binary"
    },
    "protected": {
      "@id": "jose:protected",
      "@type": "xsd:base64Binary"
    },
    "header": {
      "@id": "jose:header"
    },
    "kid": {
      "@id": "jose:kid"
    },
    "signature": {
      "@id": "jose:signature",
      "@type": "xsd:base64Binary"
    }
  },
  "payload": "ABABAB",
  "protected":"ABCABC",
  "header": {"kid":"2010-12-29"},
  "signature": "ABCABC"
 }
 ```

 The challenge here, of course, is with the payload, and the header details. With JWS, the "payload" is base64 encoded within the object itself. JWS would not give us any method of signing an RDF graph unless we snapshot a set of triples at a given point in time, normalized it and embedded it as the payload. Which can be problematic.

 The Secure Messaging (https://web-payments.org/specs/source/secure-messaging/) proposal gives another possible approach. I've still yet to look into that one.