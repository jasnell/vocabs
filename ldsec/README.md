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

The challenge, however, is that we need to have a deterministic way of canonicalizing. Will address that problem later. For now, let's assume that magic is involved.

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

<urn:verification3> a :Verification ;
  :verifies [] ;
  :transform <http://example.org/ld-c14n> ;
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