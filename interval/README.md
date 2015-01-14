The Interval vocabulary is primarily motivated by Activity Streams Collection uses cases and Object validity cases. The vocabulary allows the description of various kinds of intervals. 

For instance, to represent a Closed Interval (lower and upper inclusive bounds), we can use:

```json
{
  "@type": "i:ClosedInterval",
  "i:lower": {
    "@value": "2012-12-12T12:34:56Z",
    "@type": "xsd:dateTime"
  },
  "i:upper": {
    "@value": "2014-12-12T12:34:56Z",
    "@type": "xsd:dateTime"
  }
}
```

The vocabulary is able to express the following kinds of intervals:

* ClosedInterval
* OpenInterval
* OpenClosedInterval
* ClosedOpenInterval
* LeftOpenInterval
* LeftClosedInterval
* RightOpenInterval
* RightClosedInterval

The lower and upper properties are DatatypeProperties so Intervals can only express intervals of literal values.

One of the use cases for Interval is to allow us to express the data range interval for items contained in an Activity Streams 2.0 collection. For instance:

```json
{
  "@type": "Collection",
  "x:itemRange": {
    "@type": "i:ClosedInterval",
    "i:lower": {
      "@value": "2012-12-12T12:34:56Z",
      "@type": "xsd:dateTime"
    },
    "i:upper": {
      "@value": "2014-12-12T12:34:56Z",
      "@type": "xsd:dateTime"
    }
  },
  items [ ... ]
}
```

Another possible use is for expressing validity ranges for objects...

```json
{
  "@type": "Offer",
  "x:validity": {
    "@type": "i:ClosedInterval",
    "i:lower": {
      "@value": "2012-12-12T12:34:56Z",
      "@type": "xsd:dateTime"
    },
    "i:upper": {
      "@value": "2014-12-12T12:34:56Z",
      "@type": "xsd:dateTime"
    }
  },
  items [ ... ]
}
```

Or a price range ...

```json
{
  "@type": "Offer",
  "x:priceRange": {
    "@type": "i:OpenInterval",
    "i:lower": {
      "@value": "10.00",
      "@type": "xsd:double"
    },
    "i:upper": {
      "@value": "20.00",
      "@type": "xsd:double"
    }
  },
  items [ ... ]
}
```

```html
<div itemscope itemtype="http://www.w3.org/ns/interval#ClosedInterval">
  <time itemprop="lower" datetime="2015-01-14T12:12:12Z">12:12:12Z on January 14, 2015</time> - 
  <time itemprop="upper" datetime="2015-01-15T12:12:12Z">12:12:12Z on January 15, 2015</time>
</div>
```

```html
<div vocab="http://www.w3.org/ns/interval#" typeof="ClosedInterval">
  <time property="lower" datetime="2015-01-14T12:12:12Z">12:12:12Z on January 14, 2015</time> - 
  <time property="upper" datetime="2015-01-15T12:12:12Z">12:12:12Z on January 15, 2015</time>
</div>
```
The Microformats mapping would be:

* ClosedInterval --> h-interval-closed
* OpenInterval --> h-interval-open
* OpenClosedInterval --> h-interval-open-closed
* ClosedOpenInterval --> h-interval-closed-open
* LeftOpenInterval --> h-interval-left-open
* LeftClosedInterval --> h-interval-left-closed
* RightOpenInterval --> h-interval-right-open
* RightClosedInterval --> h-interval-right-closed

* lower -->
** dt-lower
** p-lower
* upper -->
** dt-lower
** p-lower 

```html
<div class="h-interval-closed">
  <time class="dt-lower" datetime="2015-01-14T12:12:12Z">12:12:12Z on January 14, 2015</time> - 
  <time class="dt-upper" datetime="2015-01-15T12:12:12Z">12:12:12Z on January 15, 2015</time>
</div>
```