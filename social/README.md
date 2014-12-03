The Social Vocabulary is an evolution of the work I initially worked on here:

  http://tools.ietf.org/html/draft-snell-social-urn-01

It is recast as a vocabulary rather than a URN. It is primarily motivated by the need for reliable and consistent audience targets when using the Activity Streams 2.0 audience targeting feature.

For instance, 

```json
{
  "@context": [
    "http://asjsonld.mybluemix.net",
    {
    "s": "http://www.w3.org/ns/social#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "s:hasDimension": {
      "@type": "@id"
    },
    "s:hasRelationship": {
      "@type": "@id"
    },
    "s:hasRole": {
      "@type": "@id"
    },
    "s:confidence": {
      "@type": "xsd:nonNegativeInteger"
    },
    "s:distance": {
      "@type": "xsd:nonNegativeInteger"
    }
    }
  ],
  "@type": "Arrive",
  "actor": "acct:sally@example.org",
  "location": {
    "@type": "Place",
    "displayName": "Work"
  },
  "to": "s:Public"
}
```

The vocabulary defines a top level Population class that is itself a subclass of owl:Class. Then it creates several Population instances:

* Everyone
* Public
* Private
* Direct
* Extended
* Common
* Interested
* Self
* EveryoneWithRole
* EveryoneRelated

Most of these can be used directly without any qualification. For instance:

```json
{
  "to": "s:Everyone"
}
```

```json
{
  "to": "s:Public"
}
```

```json
{
  "to": "s:Private"
}
```

Several, however, can be qualified with a distance, confidence or other parameters. To do so, you would essentially create a new object with it's type set appropriate. For instance:

```json
{
  "to": {
    "@type": "s:Extended",
    "s:distance": 9 
  }
}
```

Which represents: The extended network out to the 9th degree of separation.

Or

```json
{
  "to": {
    "@type": "s:Common",
    "s:hasDimension": [
      "http://www.example.org/arbitrary1",
      "http://www.example.org/arbitrary1"
    ],
    "s:confidence": 99
  }
}
'''

Which represents: The subset of the population who share common interests along dimensions "http://.../arbitrary1" and "http://.../arbitrary2" with a 99% confidence level.

