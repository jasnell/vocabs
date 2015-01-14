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
<urn:verification> a :Verification ;
  :verifies <urn:thing> ;
  :transform <http://example.org/ld-c14n> ;
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
This ought to be simple enough. `verifies` identifies the thing to be verified. `transform` identifes the transform/c14n algorithm used (if any... in some cases it may not be necessary to transform). `alg` identifies the signature algorithm used. `key` identifies the key used for signing. `sig` contains the actual signature value. 

We can nest the thing being verified:
```
<urn:verification2> a :Verification ;
  :verifies [
    a owl:Thing ;
      rdfs:label "Test"
  ];
  :transform <http://example.org/ld-c14n> ;
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

Or include the verification inline in the thing being verified. In this case, the c14n algorithm would be required to take the nesting into consideration and omit the :verifiedBy triple from the canonicalized input. 
```
<urn:thing> a owl:Thing;
  rdfs:label "A Thing";
  :verifiedBy [
    a :Verification ;
      :transform <http://example.org/ld-c14n> ;
      :alg <http://www.w3.org/2001/04/xmldsig-more#ecdsa-sha512> ;
      :key [
        a :ECKey ;
          :crv "P-256" ;
          :x "{base64 data}"^^xsd:base64Binary ;
          :y "{base64 data}"^^xsd:base64Binary ;
          :d "{base64 data}"^^xsd:base64Binary ;
      ] ;
      :sig "{base64 signature}"^^xsd:base64Binary .
  ] .
```

We can specify the key in a number of ways, and there's no reason we can use the mechanism to verify non-RDF resources...
```
<urn:verification3> a :Verification ;
  :verifies <http://example.com/some-other-kind-of-thing>;
  :transform <:something-that-means-get-the-raw-bits> ;
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

Encrypted Content also ...

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