The Social Vocabulary is an evolution of the work I initially worked on here:

  http://tools.ietf.org/html/draft-snell-social-urn-01

It is recast as a vocabulary rather than a URN. It is primarily motivated by the need for reliable and consistent audience targets when using the Activity Streams 2.0 audience targeting feature. This is purely experimental at this point.

For instance,

Assuming that "http://www.w3.org/ns/social" points to the JSON-LD context document:

```json
{
  "@context": {
    "s": "http://www.w3.org/ns/social#",
    "xsd": "http://www.w3.org/2001/XMLSchema#",
    "s:havingDimension": {
      "@type": "@id"
    },
    "s:havingRelationship": {
      "@type": "@id"
    },
    "s:havingRole": {
      "@type": "@id"
    },
    "s:confidence": {
      "@type": "xsd:nonNegativeInteger"
    },
    "s:distance": {
      "@type": "xsd:nonNegativeInteger"
    },
    "s:a": {
      "@type": "@id"
    },
    "s:b": {
      "@type": "@id"
    },
    "s:relationship": {
      "@type": "@id"
    }
  }
}
```

```json
{
  "@context": [
    "http://www.w3.org/ns/activitystreams",
    "http://www.w3.org/ns/social"
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

* Everyone (Everyone visible to the context )
* Public (Subset visible to the context and that have a publicly visible relationship with the context)
* Private (Subset visible to the context but have publicly invisible relationship with the context)
* Direct (Subset visible to the context and directly connected to the context)
* Common (Subset visible to the context and that shares one or more common interests with the context)
* Interested (Subset visible to the context and that has expressed an interest in the context, e.g. followers)
* Self (the context itself)

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

They can, however, be qualified with a distance, confidence or other parameters. To do so, you would essentially create a new object with it's type set appropriate. For instance:

Everyone who is a sibling of the context and connected within 10 degrees of separation (which doesn't make much practical sense but it is just an example...):

```json
{
  "to": {
    "@type": "s:Everyone",
    "s:havingRelationship": "http://purl.org/vocab/relationship/siblingOf",
    "s:distance": 10
  }
}
```

Everyone who has the role of 'http://example.org/Moderator':

```json
{
  "to": {
    "@type": "s:Everyone",
    "s:havingRole": "http://example.org/Moderator"
  }
}
```

The extended network out to the 9th degree of separation:
```json
{
  "to": {
    "@type": "s:Everyone",
    "s:distance": 9
  }
}
```

The subset of the population who share common interests along dimensions "http://.../arbitrary1" and "http://.../arbitrary2" with a 99% confidence level:

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
```

Typically, the "context" will be determined implicitly by use. For instance, in an authenticated API request the "context" will be the authenticated user. In other cases tho, we might need to specify the context explicitly. If we're using the proposed AS 2.0 vocabulary, that would be done like:

```json
{
  "to": {
    "@type": "s:Common",
    "s:hasDimension": [
      "http://www.example.org/arbitrary1",
      "http://www.example.org/arbitrary1"
    ],
    "s:confidence": 99,
    "context": {
      "@type": "Person",
      "@id": "acct:sally@example.org"
    }
  }
}
```

Which essentially says, "The subset of the population sharing common interests with Sally along dimensions ..."

Note that these can also be used with the AS2 "scope" property. "scope" differs from the audience targeting properties in that the "scope" simply identifies the total population for whom an object might be relevant while audience targeting is intended more towards driving specific notifications. Implementations can use "scope" more or less as a way of controlling the overall visibility of an object to any particular population.

```json
{
  "displayName": "A publicly visible note",
  "scope": "s:Public"
}
```

```
{
  "displayName": "A private note",
  "scope": "s:Private"
}
```

The key relationship between "scope" and audience targeting is that "scope" defines the population to which the object is visible while to/bto/cc/bcc define specific subsets within that scoped population to direct targeted notifications or special attention.

For instance:

```json
{
  "displayName": "A public note but only within the context of my employer, notify my direct connections about it.",
  "scope": {
    "@type": "s:Public",
    "context": {
      "@type": "Organization",
      "displayName": "IBM"
    }
  },
  "to": "s:Direct"
}
```

Not yet certain how common this will be, but we can support n-ary populations... like so:

```json
{
  "to": {
    "@type": "s:All",
    "s:member": [
      "s:Private", "s:Direct"
    ]
  }
}
```

Which says "to the subset that is both private and directly connected to the context"

Or

```json
{
  "to": {
    "@type": "s:Any",
    "s:member": [
      "s:Private", "s:Direct"
    ]
  }
}
```

"to the subset that is either private or directly connected to the context"

Or

```json
{
  "to": {
    "@type": "s:None",
    "s:member": [
      "s:Private", "s:Direct"
    ]
  }
}
```

"to the subset that is not private and not directly connected to the context"

## Modeling Connections

Connections between Individuals are essentially Reified statements. For instance, to say that "John knows Sally" we can simply use:

```json
{
  "@type": "s:Connection",
  "s:a": "http://john.example.org",
  "s:relationship": "http://xmlns.com/foaf/0.1/knows",
  "s:b": "http://sally.example.org"
}
```

This allows us to model Activities that occur against the Connection itself

```json
{
  "actor": "acct:mary@example.org",
  "@type": "Like",
  "object": {
    "@type": "s:Connection",
    "s:a": "http://john.example.org",
    "s:relationship": "http://xmlns.com/foaf/0.1/knows",
    "s:b": "http://sally.example.org"
  }
}
```

Here, "Mary" indicate that she "likes" the connection itself, as opposed to liking either John or Sally.
