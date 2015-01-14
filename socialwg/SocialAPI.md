Social API
----------

1. Needs to focus on a minimal core set of Artifacts

   a. Identity      Identity
   b. Individual    Person
   c. Profiles      Profile
   d. Personas      x
   e. Roles         Role
   f. Relationships x
   g. Groups        Group
   h. Organizations Organization
   i. Communities   Community
   j. Permissions   x
   k. Activities    Activity
   l. Assets        Object, Content, Document, etc
   m. Collections   Collection

An Identity is a uniquely identifiable actor.
Every Identity has one or more Personas. A Persona represents some distinct view on the Identity.
A Role is a specialization of Persona with a specific definition. While Personas can be adopted by an Identity, Roles are typically assigned to an Identity.
A Profile is a description of an Identity, Role or Persona.

An Individual is a specialization of Identity that represents a single rational entity (like a person)

An Group is a specialization of Identity that represents multiple Identities.

An Organization is a specialization of Group containing multiple Identities that share a common purpose.

A Community is a specialization of Group containing multiple Identities that share a common characteristic. 

Relationships are qualified directed or undirected edges between Identities. 

An Asset is something that an Identity can own, use or interact with in any way.

A Permission describes what kind of Activity an Identity can perform on an Asset.

Assets are organized into Collections. A Collection is a specialization of Asset that contains multiple Assets.

An Activity is a specialization of Asset that provides a description of something an Identity may do, is doing, or has done.

A Profile consists of zero or more Attributes. Attributes are essentially RDF Statements. The Statements can be Verified or Unverified. An Unverified statement is the usual kind of Subject+Predicate+Object triple. Verified statements are undefined right now.

@prefix : <http://www.w3.org/ns/social#> .
@prefix social: <http://www.w3.org/ns/social#> .
@prefix as: <http://www.w3.org/ns/activitystreams#> .

:Identity a owl:Class                                 .
:Persona a owl:Class      ; rdfs:subClassOf :Identity .
:Role a owl:Class         ; rdfs:subClassOf :Persona  .
:Individual a owl:Class   ; rdfs:subClassOf :Identity .
:Group a owl:Class        ; rdfs:subClassOf :Identity .
:Organization a owl:Class ; rdfs:subClassOf :Group    .
:Community a owl:Class    ; rdfs:subClassOf :Group    .
:Relationship a owl:Class                             .
:Asset a owl:Class                                    .
:Permission a owl:Class                               .
:Profile a owl:Class                                  .
as:Activity rdfs:subClassOf :Asset                    .

:hasPersona a owl:ObjectProperty ;
  rdfs:range :Persona ;
  rdfs:domain :Identity .

:describes a owl:ObjectProperty ;
  rdfs:range :Identity ;
  rdfs:domain :Profile ;
  owl:inverseOf :hasProfile .

:hasProfile a owl:ObjectProperty ;
  rdfs:range :Profile ;
  rdfs:domain :Identity ;
  owl:inverseOf :describes .

:memberOf a owl:ObjectProperty ;
  rdfs:range :Group ;
  rdfs:domain :Identity .

:hasRelationship a owl:ObjectProperty ;
  rdfs:range :Relationship ;
  rdfs:domain :Identity .

:hasAsset a owl:ObjectProperty ;
  rdfs:range :Asset ;
  rdfs:domain :Identity .

:hasPermission a owl:ObjectProperty ;
  rdfs:range :Permission ;
  rdfs:domain :Identity .

:forIdentity a owl:ObjectProperty ;
  rdfs:range :Identity ;
  rdfs:domain :Permission ;

:onResource a owl:ObjectProperty ;
  rdfs:range owl:Thing ;
  rdfs:domain :Permission .

:allow a owl:ObjectProperty ;
  rdfs:range as:Activity ;
  rdfs:domain :Permission .

:deny a owl:ObjectProperty ;
  rdfs:range as:Activity ;
  rdfs:domain :Permission .

2. Key Functional Requirements for the Social API

   a. The API must, at a minimum, allow for scoped and protected discovery of information about individual artifacts.
      - Determine what information, if any, is available for any given artifact
      - Determine ability to request information about the artifact

   b. The API should allow owners of Identities and Assets to modify details and control access to those details.

   c. The API should allow owners of Identities to manage their relationships with other Identities, regardless of 
      whether those exist in the same system or not.

The Social API largely becomes a mix of things pulled from other apis...

Basically following a Linked Data Platform model, a Social API Provider would expose one or more Containers.

  Identity Container - A repository of all Identities managed by the Provider. 

    Identities are typed. They can be :Individual, :Group, :Persona, :Role, :Organization, :Community

  Profile Container - A repository of all Profiles describing an Identity.

  Relationship Container - A repository of Relationships between Identities. 

    <urn:foo> a :Relationship;
      rdf:Subject <acct:joe@example.org> ;
      rdf:Predicate <:Knows> ;
      rdf:Object <acct:sally@example.org> .

    <urn:foo> a :Relationship;
      rdf:Subject <acct:joe@example.org> ;
      rdf:Predicate <:Follows> ;
      rdf:Object <acct:sally@example.org> .

    <urn:foo> a :Relationship;
      rdf:Subject <acct:joe@example.org> ;
      rdf:Predicate <:MemberOf> ;
      rdf:Object <http://example.org/groups/1> ;

  Asset Container - A repository of Assets

    Assets are typed. Assets have owners.

    <urn:foo> a as:Album ;
      social:hasOwner <acct:joe@example.org> .

    <urn:foo:1> a as:Image ;
      social:hasOwner <acct:joe@example.org> ;
      as:memberOf <urn:foo> .

    <urn:device:a> a as:Device ;
      social:hasOwner <acct:joe@example.org> ;
      as:displayName "iOS Device" .

  Permissions Container - A repository of Permissions

    <urn:perm:1> a :Permission ;
      :forIdentity <acct:joe@example.org> ;
      :onResource <urn:device:a> ;
      :allow <as:Post> .

Identity Container

  GET    /identities --> Return list of all identities, optionally filtered
  GET    /identities/{subject} --> Return a specific identity ... todo: figure out aliases
  POST   /identities --> Create a new Identity
  PUT    /identities/{subject} --> Modify/Create a new Identity
  PATCH  /identities/{subject} --> Modify an existing Identity
  DELETE /identities/{subject} --> Delete an Identity
  SEARCH /identities --> Search the container

Profile Container

  GET    /profiles --> Return list of all profiles, optionally filtered
  GET    /profiles/{subject} --> Return a specific profile
  POST   /profiles --> Create a new Profile
  PUT    /profiles/{subject} --> Modify/Create a new Profile
  PATCH  /profiles/{subject} --> Modify an existing Profile
  DELETE /profiles/{subject} --> Delete a Profile
  SEARCH /profiles --> Search the container

Relationship Container

  GET    /relationships --> Return list of all relationships, optionally filtered
  GET    /relationships/{subject} --> Return a specific relationship
  POST   /relationships --> Create a new Relationship
  PUT    /relationships/{subject} --> Modify/Create a new Relationship
  PATCH  /relationships/{subject} --> Modify an existing Relationship
  DELETE /relationships/{subject} --> Delete a Relationship
  SEARCH /relationships --> Search the container

Asset Container

  GET    /assets --> Return a list of all assets, optionally filtered
  GET    /assets/{subject} --> Return a specific asset
  POST   /assets --> Create a new Asset
  PUT    /assets/{subject} --> Modify/Create a new Asset
  PATCH  /assets/{subject} --> Modify and existing Asset
  DELETE /assets/{subject} --> Delete an Asset
  SEARCH /assets --> Search the container

  If the Asset is itself a container Asset (like an as:Album, as:Folder, as:Story, etc)... then 
  the Asset automatically becomes it's own sub-container...

  POST   /assets/{subject} --> Create a new sub-asset for the container asset
  PUT    /assets/{subject}/{subject} --> Modify/Createa a sub-asset
  PATCH  /assets/{subject}/{subject} --> Modify an existing sub-asset
  DELETE /assets/{subject}/{subject} --> Delte a sub-asset
  SEARCH /assets/{subject}/{subject} --> Search the container

  If the Asset is a container asset, deleting it will automatically delete all of it's subassets.

  Scenario: If we wanted to create a Photo Album for a specific user.. we would first

    POST /assets
    {"@type":"Album", "social:owner": "acct:joe@example.org"}

    HTTP/1.1 201 Created
    Location: /assets/abc123

    Then we would post images to that...

    POST /assets/abc123
    {"@type": "Image"} ...

    HTTP/1.1 201 Created
    Location: /assets/abc123/xyz456

  Scenario: Suppose we want to create an Article asset with a collection of Replies ... we need to create two separate assets, linked together:

    POST /assets
    {"@type": "Collection", displayName: "Articles"}

    HTTP/1.1 201 Created
    Location: /assets/articles

    POST /assets/articles
    {"@type": "Article", displayName: "Article"};

    HTTP/1.1 201 Created
    Location: /assets/articles/123

    POST /assets
    {"@type": "OrderedCollection", displayName: "Replies", inReplyTo: "/assets/articles/123"}

    HTTP/1.1 201 Created
    Location: /assets/abc123

  Replies to the article would then be posted to the separate Replies collection asset. 


<urn:service> a social:SocialService ;
  hasContainer (
    [
      a ldp:Container,
        social:forArtifacts ( social:Identity ),
        as:displayName "Identities"
    ]
    [
      a ldp:Container,
        social.forArtifacts ( social:Profiles ),
        as:displayName "Profiles"
    ]
    ...
  ) .

SEARCH http://example.org/r/identities
Content-Type: application/sparl-query
Accept: application/ld+json

{sparql query here}

There needs to be a higher-level interface here as well... to support use from more limited browser clients. Essentially, form-post support for more basic operations.