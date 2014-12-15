This is a mapping of the draft Web Annotations model
onto the Activity Streams model, allowing Annotations
to be viewed and processed as Activities.

The oa:Annotation class is defined as a subclass of the Activity Streams as:Post. The oa:hasBody is mapped as a subpropertyof as:object; oa:hasTarget is mapped as a subpropertyof as:target; and oa:annotatedBy is mapped as a subpropertyof as:actor. This means that the following Annotation:

```json
{
  "@id": "http://example.org/anno1",
  "@type": "oa:Annotation",
  "annotatedBy": "http://example.org/agent1",
  "annotatedAt": "2013-01-28T12:00:00Z",
  "body": {"@id": "http://example.org/body1"},
  "target": "http://example.org/target1"
}
```

is generally equivalent to:

```json
{
  "@context": "http://www.w3.org/ns/activitystreams",
  "@type": ["Post", "oa:Annotation"],
  "actor": "http://example.org/agent1",
  "published": "2013-01-28T12:00:00Z",
  "object": "http://example.org/body1",
  "target": "http://example.org/target1"
}
```

Which is to say, "Actor posted object as an annotation to target".

