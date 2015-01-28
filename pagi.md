

Portable Analytics Graphical Interchange
========================================

#### J. Bunting, J. Swisher

#### 3 June 2014

##### Draft 4

Abstract {#abstract}
-------------------

The document describes a specification for the interchange of analytic
information on textual data, represented as a graph. It describes a model, a
schema format, several transfer formats, and a set of common APIs.

Table of Contents {#toc}
------------------------

1. [Introduction](#introduction)
2. [Definitions](#definitions)
3. [Graph Representation](#graph-representation)
    1. [Document Structure](#doc-structure)
    2. [Node Structure](#node-structure)
    3. [Property Types](#property-types)
    4. [Features](#features)
    5. [ID Generation](#id-generation)
    6. [Traits](#traits)
4. [Schema Syntax](#schema-syntax)
    1. [Core Syntax](#core-schema)
    2. [Extending Schemas](#extending-schemas)
    3. [Schema API](#schema-api)
5. [Common APIs](#commons-apis)
    1. [Event-Based Streaming](#event-based-streaming)
6. [Transfer Formats](#transfer-formats)
    1. [XML](#xml-format)
    2. [Binary](#binary-format)
    3. [JSON](#json-format)
7. [Graph Fragment](#graph-fragment)
8. [Graph Diff](#graph-diff)
9. [Corpus-Scoped Analytics](#corpus-scoped-analytics)
10. [Future Scope](#future-scope)
11. [Appendices](#appendices)
    * [Appendix A: XML Format XSD](#xml-format-xsd)
    * [Appendix B: PAGI Schema XSD](#pagi-schema-xsd)


Introduction {#introduction}
----------------------------

The basic data model used to perform analytics on a document is a graph. Each
document is represented as a graph, with the nodes in that graph representing
analytic results, features, and properties. Additionally, these nodes may have
relationships to other nodes in the graph.

For example, a node may represent a single word of text (or "token"). That node
may then have a child relationship to a node representing the phrase of which
the word is a part (a "chunk"). The chunk node may have other children,
representing other words in the phrase. Each of these nodes may have additional
properties, describing the analytic component that generated the node, a
confidence associated with the node, and so on.

By defining this specification in a generic manner, we gain the ability to
develop programs that work with any pagi-compliant schema. This provides us
with three powerful tools for future growth: 

1. a common language to discuss adding new types of information to our schema
2. a common language to extend our schema for situation-specific installations
   or experiments
3. the ability to write programs and tooling that work with the model in a 
   generic way, allowing those programs and tools to continue to work with
   future iterations of the schema


Definitions {#definitions}
--------------------------

node
:   A single piece of analytic information. 

nodeType
:   The type of a node -- defined in the schema. (Examples: token, chunk,
    temporal)

edge
:   A relationship between two nodes. 

property
:   A key-value pair attached to a node and specified in the schema. (Examples:
    "provenance"="NLPTokenizer", "confidence"=0.963, "startOffset"=15,
    "length"=3)

feature 
:   A key-value pair attached to a node. Features need not be specified within a
    schema. (Examples: "NW1"="dog")

id 
:   A string associated with a node that when combined with the nodeType can
    uniquely identify the node within the current document. IDs need not be
    globally unique.

document-graph 
:   The graph consisting of all nodes and edges that reference a common source
    document.

pagi
:   Portable Analytic Graphical Interchange
:   This spec and everything that it encompasses.
:   Pronounced with a hard 'G', rhymes with 'Maggie'
:   Derived acronyms typically have a suffix-consonant added, and are 
    pronounced the same except for the trailing 'i' transitioning from a long 
    "ee" sound to a short i

pagis
:   Portable Analytic Graphical Interchange Schema
:   The schema and its language as described in this spec.

pagif
:   Portable Analytic Graphical Interchange Format
:   Any of the textual or binary formats used to represent a pagi graph. 
    Typically used with the format specified as a suffix, ie. pagif-xml, 
    pagif-json


pagim
:   Portable Analytic Graphical Interchange Model
:   The general model described by this spec.

spec
:   This document.

schema
:   A user-generated document that describes the nodeTypes in use for a 
    particular application. 

source-document 
:   A single instance of text that is subject to analysis. Typically a single
    file within a filesystem.

Graph Representation {#graph-representation}
----------------------------------

### Document Structure {#doc-structure}

A document is represented as a graph. Each node in this graph represents a 
single piece of analytic information as described in the schema. The document 
also references an id and the document text.

### Node Structure {#node-structure}

Nodes are the pivotal structures in this model. 

* A node may represent any sort of information - a token, a sentence, a phrase,
  a category, a semantic role, a part of speech, etc. 
* A node has a nodeType. All nodeTypes are defined in the schema.
* A node has an ID. The ID is a string and is unique within the scope of a 
  nodeType for a single document.
* A node has relationships to other nodes, the types of which are defined in the
  schema. 
* A node has properties which are additional key-value pairs of information. The
  names of the properties that a node has and the types of those properties are 
  defined by its type.
* A node has features which are additional arbitrary key-value pairs of 
  information. Features are typically used to carry transient information along
  with the model that will be thrown away later. Features are not specified in
  the schema. APIs may provide efficient ways to handle features such as 
  generating them on demand.

### Property Types {#property-types}

* Every property has a type. The allowable types are:
    * integer
    * float
    * boolean
    * string
* A property may be single or multiple valued. Minimum and Maximum arity
  may be specified as part of the schema.

* Some property types may allow the specification of constraints:

    * integer - minimum and maximum values may be specified, with `unbounded`
      being the default
    * float - minimum and maximum values may be specified, with `unbounded`
      being the default
    * string - a list of possible values may be specified. If so, these are
      the only allowable values

### Features {#features}

* Every feature has a type. The allowable types are the same as those for a 
  property.
* Due to the lack of a schema definition, a feature may have zero or more 
  values.

### ID Generation {#id-generation}

* A nodeType is responsible for defining how IDs are to be generated for nodes
  of that type.
* The generation strategy must produce IDs that are unique within a given
  document and nodeType. If it does not, failures will occur at runtime.
* IDs are generated when a new node is created.

Available generation strategies:  

Randomly generated string
:    essentially uses an implementation-dependent random process to generated a 
     string. Length of the string may be specified in the schema.

Sequence
:    a sequence for the given type exists for the document. A new id is 
     generated by taking the current number of the sequence and incrementing 
     it. The order of ids is solely determined by the order in which nodes
     are generated. The sequence starts at 1.

Property
:    the value of one of the node's properties is used as the id. A delimiter 
     to combine mulitple values may be specified. By default the delimiter is
     a semi-colon (;).

Pattern
:    a format pattern will be used to generated the ID. This format pattern
     may contain raw strings and also may incorporate any of the other 
     generation strategies

### Traits {#traits}

A trait is a set of characteristics that may be assigned to a nodeType. Each
trait will define properties that it imports to a node type, edge types that it
imports to a node type, and semantic restrictions on those. Traits may also
define parameters that must be supplied when they are defined on a node type.
Traits are only defined in the spec.

Currently defined traits:

#### Span
* Parameters
    * *none*
* Properties
    * **start** -- the offset from the beginning of the referenced character
                   sequence within the document
    * **length** -- the length of the referenced character sequence
* Edges
    * *none*
* Semantic Description
    * Describes a nodeType that is associated with a contiguous sequence of
      characters within the source document.
    * Will have a nonnegative integer-valued property named "start" that refers
      to the character offset of the beginning of the referenced character
      sequence within the source document.
    * Will have a positive integer-valued property named "length" that refers to
      the number of consecutive characters in the referenced character sequence
      within the source document.
    * A span must reference text that exists within the source document.
      Specifically, if we take L to be the length of the source document in
      characters, then for any Span node A, A.start + A.length <= L.

#### Sequence
* Parameters
    * *none*
* Properties
    * *none*
* Edges
    * **next** -- refers to a node of the same type, has a maximum arity of 1. 
      If absent, indicates the final node of the sequence.

* Semantic Description
    * Describes a nodeType that occurs in a sequence.
    * The nodeType must have an associated ordering that permits the formation
      of a strictly well-ordered set. (The natural ordering of a set of unique
      integers satisfies this condition.) [Wikipedia: Well-
      order](https://en.wikipedia.org/wiki/Well-order)
    * Following from the above, each node may have another instance of that 
      same nodeType before and after it.
    * May have a single edge named "previous" that refers to the preceding node
      of the same type, if such a node exists.
    * May have a single edge named "next" that refers to the succeeding node of
      the same type, if such a node exists.
    * Previous and next edges, if present, must be bidirectional. If A.next=B,
      then B.previous=A, and conversely. (B.previous cannot be null and must be
      A given A.next=B, and A.next cannot be null and must be B given
      B.previous=A.)
    * The "previous" and "next" edges of a Sequential node, if present, must
      connect to the next node of the same type within the document, according
      to the associated ordering. That is, nodes connected under the 
      Sequential trait must be consecutive.
    * A Sequence is defined as a connected set of nodes with the Sequential
      trait. A document may contain multiple Sequences of nodes of the same 
      type; this implies that there are no "previous" or "next" edges 
      connecting nodes of different Sequences.

#### SpanContainer
* Parameters
    * **spanType** -- the node type that is spanned/contained by this one - 
      must be a "Sequence" and one of either "Span" or "SpanContainer"
* Properties
    * *none*
* Edges
    * **first** -- refers to the first node in the contained span
    * **last** -- refers to the last node in the contained span
* Semantic Description
    * Describes a nodeType whose existence is defined as a span over other 
      nodes that define a span. These may be direct "Span" nodes, or other 
      "SpanContainer" nodes. In both cases, the contained node type must also
      be "Sequential".
    * Traversal APIs must enable referring to the entire spanned text of a 
      "SpanContainer".

Schema Syntax {#schema-syntax}
------------------------------

### Core Syntax {#core-schema}

A PAGI Schema is defined in an XML document. Note that this XML document, which
represents a PAGI-compliant schema, is distinct from the XML transfer format
used to represent a particular instance of a PAGI document graph.

Its Internet Media Type is `application/vnd.dr.pagi.schema+xml`.

Here is the general structure:

```xml
<pagis xmlns="http://pagi.org/schema/" 
       pagis-uri="http://www.example.com/spec-example-1">
  <nodeType name="" idGenerator="">
    <span/>
    <sequence/>
    <container edgeType=""/>
    <spanContainer spanType=""/>
    <integerProperty name="" minRange="" maxRange="" minArity="" maxArity=""/>
    <floatProperty name="" minRange="" maxRange="" minArity="" maxArity=""/>
    <booleanProperty name="" minArity="" maxArity=""/>
    <stringProperty name="" minArity="" maxArity=""/>
    <enumProperty name="" minArity="" maxArity="">
      <item name=""/>
    </enum>
    <edgeType name="" targetNodeType="" minArity="" maxArity="" targetMinArity="" targetMaxArity=""/>
    <edgeType name="" minArity="" maxArity="" targetMinArity="" targetMaxArity="">
      <targetNodeType name=""/>
      <targetNodeType name=""/>
    </edgeType>
  </nodeType>
</pagis>
```

The "minArity" and "maxArity" XML attributes on property and edge XML elements
refer to the number of times a property or edge of the specified name may occur
on a single node of the specified nodeType. For instance, an optional property
may be designated by the attributes min="0", max="1". A required edge that may
occur an arbitrary number of times (for instance, designating children of a
node with the Contains-Sequence trait) would be designated by the attributes
min="1", max="unbounded". The "targetMinArity" and "targetMaxArity" attributes
on the edge element specify the same limitations, except on the node type that
the edge is targeting.

#### Id Generator

The specification of the id generator necessitates some description. The
idGenerator is specified as a pattern, consisting of some combination of one
or more of the following tokens:

* any alphanumeric string excluding `{`, and `}`
* one of the "special" tokens, enclosed in `{` and `}`
    * `seq` - indicating a sequential number
    * `random[:{len}[/{base}]]` - that is, a random string with an optional
      specification the optional spec may contain the length of the random
      string or the length of the random string and the size of the character
      set used
      * default length is 8
      * default character set size is 10
    * `prop[:{delim}]:{name}` - the value of the named property, separated by
      optional specification `delim`
      * default delim is `,`
    * `edge[:{delim}]:{name}` - the id of the node targeted by the named edge,
      separated by optional specification `delim`
      * default delim is `,`

### Extending Schemas {#extending-schemas}

Any schema can be extended.  This is done by specifying the `extends` attribute
in  the root `pagis` element. The new schema may only specify new nodeTypes, add
edges and properties to an existing node type, or add new target types to an
edge. It may not modify existing
properties.

Here is the general structure:
```xml
<pagis xmlns="http://pagi.org/schema/"
       pagis-uri="http://www.example.com/spec-extension-example-1"
       extends="http://pagi.org/spec-example-1">
  <nodeType name="" id-generator="">
    <span/>
    <sequence/>
    <spanContainer spanType=""/>
    <integerProperty name="" minRange="" maxRange="" minArity="" maxArity=""/>
    <floatProperty name="" minRange="" maxRange="" minArity="" maxArity=""/>
    <booleanProperty name="" minArity="" maxArity=""/>
    <stringProperty name="" minArity="" maxArity=""/>
    <enumProperty name="" minArity="" maxArity="">
      <item name=""/>
    </enumProperty>
    <edgeType name="" targetNodeType="" minArity="" maxArity="" targetMinArity="" targetMaxArity=""/>
  </nodeType>
  <nodeTypeExtension extends="">
    <integerProperty name="" minRange="" maxRange="" minArity="" maxArity=""/>
    <floatProperty name="" minRange="" maxRange="" minArity="" maxArity=""/>
    <booleanProperty name="" minArity="" maxArity=""/>
    <stringProperty name="" minArity="" maxArity=""/>
    <enumProperty name="" minArity="" maxArity="">
      <item name=""/>
    </enumProperty>
    <edgeType name="" targetNodeType="" min="" max="" targetMinArity="" targetMaxArity=""/>
    <edgeTypeExtension extends="">
      <targetNodeType name=""/>
    </edgeTypeExtension>
  </nodeTypeExtension>
</pagis>
```

### Schema API {#schema-api}

For the purposes of interacting with a graph within code, it is necessary to
have knowledge of the schema that applies to that graph. It is necessary that a
schema be available for introspection along with the [common apis](#common-apis)
defined in the following section.

A schema is represented as a collection of "nodeType" objects. Each NodeType
object provides all specified information for that node type, and seamlessly
merges properties and edges provided by traits as well as extensions.

Following is a general outline of the types and methods expected to be available
in implementations of the Schema API. Method signatures are intended to be
representative, not prescriptive; implementations should adhere to idiomatic
style conventions in the appropriate language.

#### Schema

* String getName()
* Map(String, NodeType) getNodeTypes()
* Schema getParentSchema()

#### NodeType

* String getName()
* boolean isSpan()
* boolean isSequence()
* boolean isContainer()
* String[] getContainerEdgeTypes()
* boolean isSpanContainer()
* String getSpanType()
* Map<String, PropertySpec> getPropertySpecs()
* Map<String, EdgeSpec> getEdgeSpecs()

#### PropertySpec

* ValueType getValueType()
* String getName()
* int getMinArity()
* int getMaxArity() // -1 represents unbounded unless a language has a better 
  representation
* IntegerRestrictions getIntegerRestrictions()
* FloatRestrictions getFloatRestrictions()
* StringRestrictions getStringRestrictions()

#### IntegerRestrictions

* int getMinRange()
* int getMaxRange()

#### FloatRestrictions

* float getMinRange()
* float getMaxRange()

#### StringRestrictions

* String[] getItems()

#### EdgeSpec

* String getName()
* String[] getTargetNodeTypes()
* int getMinArity()
* int getMaxArity()
* int getTargetMinArity()
* int getTargetMaxArity()
* Map<String, PropertySpec> getPropertySpecs()

Common APIs {#commons-apis}
---------------------------

The notion of defining Common APIs is to define a common way of working with
the data no matter the language, in much the same way that SAX and DOM do for
XML. One key difference between these APIs and the XML APIs is that writing
should be accounted for in all of these.

### Event-Based Streaming {#event-based-streaming}

This API is intended as a computationally cheap way to walk through a document
and make decisions based on the content. The Stream is the lowest level API
for reading and writing pagi structures. This API can be thought of as a stream
of events. Each event type has a set of parameters. Some of these events define
a "context" -- and as such come in START/END pairs. Some of these events simply
provide values.

DOC_START
:    Indicates the start of a document.
:    Is only valid outside of any context.
:    Has the string-typed *id* parameter indicating the document id.
:    Indicates the beginning of a "DOC" context.

DOC_END
:    Indicates the end of a document.
:    Is only valid in the "DOC" context.
:    Indicates the ending of a "DOC" context.

FRAGMENT_START
:    Indicates the start of a graph fragment.
:    Is only valid outside of any context.
:    Has the string-typed *id* parameter indicating the graph fragment id.
:    Indicates the beginning of a "FRAGMENT" context.

FRAGMENT_END
:    Indicates the end of a graph fragment.
:    Is only valid in the "FRAGMENT" context.
:    Indicates the ending of a "FRAGMENT" context.

DIFF_START
:    Indicates the start of a graph diff.
:    Is only valid outside of any context.
:    Has the string-typed *sourceId* parameter indicating the id of source graph.
:    Has the string-typed *targetId* parameter indicating the id of target graph.
:    Indicates the beginning of a "DIFF" context.

DIFF_END
:    Indicates the end of a graph diff.
:    Is only valid in the "DIFF" context.
:    Indicates the ending of a "DIFF" context.

SOURCE_TARGET_NODE
:    Indicates node id equality between source and target graphs in a diff.
:    Is only valid in the "DIFF" context and before any "NODE_*" events.
:    Has the string-typed *nodeType* parameter indicating type of node.
:    Has the string-typed *sourceId* parameter indicating the node id of source node.
:    Has the string-typed *targetId* parameter indicating the node id of target node.
:    This event can occur 0 or more times with different mappings.

USES_SCHEMA
:    Indicates that this document uses a particular pagi schema.
:    Is only valid in the "DOC" or "FRAGMENT" context and before any "NODE_START" events.
:    Has the string-typed *uri* parameter indicating the pagi schema id.
:    This event can occur 0 or more times with different schema ids.

AS_SPAN
:    Indicates that a given node type should be treated as a span.
:    Is only valid in the DOC context and before any "NODE_START" events.
:    Has the string-typed *nodeType* parameter indicating the node type.
:    This event can occur 0 or more times, each with different node types.
:    Provided as a supplement to consumers loading the graph when a schema
     is not available.

AS_SEQUENCE
:    Indicates that a given node type should be treated as a sequence.
:    Is only valid in the DOC context and before any "NODE_START" events.
:    Has the string-typed *nodeType* parameter indicating the node type.
:    This event can occur 0 or more times, each with different node types.
:    Provided as a supplement to consumers loading the graph when a schema
     is not available.

AS_SPAN_CONTAINER
:    Indicates that a given node type should be treated as a span container.
:    Is only valid in the DOC context and before any "NODE_START" events.
:    Has the string-typed *nodeType* parameter indicating the node type.
:    Has the string-typed *spanType* parameter indicating the type that this
     node type spans.
:    This event can occur 0 or more times, each with different node types.
:    Provided as a supplement to consumers loading the graph when a schema
     is not available.

CONTENT
:    Provides the content of this document.
:    Is only valid in the "DOC" context, after all "USES_SCHEMA" events, and before any "NODE_START" events.
:    Must occur exactly once.
:    Has the string-typed *contentType* parameter indicating the type of the content.
:    *contentType* should conform to the RFC-2046 rules
:    Current supported values for *contentType* are ``text/plain``.
:    Has the string-typed *content* parameter containing the content.

NODE_START
:    Represents the beginning of information about a node.
:    Is only valid in the "DOC" or "FRAGMENT" context.
:    Has the string-typed *id* parameter indicating the node id.
:    Has the string-typed *nodeType* parameter indicating the node type.
:    Indicates the beginning of a "NODE" context.

NODE_END
:    Indicates that there is no more information about the current node.
:    Is only valid in the "NODE" context.
:    Indicates the ending of a "NODE" context.

NODE_REMOVE_START
:    Represents the beginning of information about removing a node.
:    Is only valid in the "DIFF" context.
:    Has the string-typed *id* parameter indicating the node id.
:    Has the string-typed *nodeType* parameter indicating the node type.
:    Indicates the beginning of a "NODE_REMOVE" context.
:    Subsequent events before the next "NODE_REMOVE_END" are meant to capture information
     about the node being removed.

NODE_REMOVE_END
:    Indicates that there is no more information about the current node.
:    Is only valid in the "NODE_REMOVE" context.
:    Indicates the ending of a "NODE_REMOVE" context.

NODE_UPDATE_START
:    Represents the beginning of information about modifying an existing node.
:    Is only valid in the "DIFF" context.
:    Has the string-typed *id* parameter indicating the node id.
:    Has the string-typed *nodeType* parameter indicating the node type.
:    Indicates the beginning of a "NODE_UPDATE" context.

NODE_UPDATE_END
:    Indicates that there is no more information about the current node.
:    Is only valid in the "NODE_UPDATE" context.
:    Indicates the ending of a "NODE_UPDATE" context.

EDGE
:    Indicates an edge on the current node.
:    Is only valid in the "NODE" context, prior to any FEATURE events.
:    Has the string-typed *edgeType* parameter indicating the type of the edge.
:    Has the string-typed "targetNodeType* parameter indicating the type of the
     node that this edge points to.
:    Has the string-typed *targetId* parameter indicating the id of the node
     that this edge points to.

EDGE_REMOVE
:    Indicates an removal of an edge on the current node.
:    Is only valid in the "NODE_UPDATE" context.
:    Has the string-typed *edgeType* parameter indicating the type of the edge.
:    Has the string-typed "targetNodeType* parameter indicating the type of the
     node that this edge points to.
:    Has the string-typed *targetId* parameter indicating the id of the node
     that this edge points to.

EDGE_START
:    Indicates an edge on the current node.
:    Is only valid in the "NODE" context, prior to any FEATURE events.
:    Has the string-typed *edgeType* parameter indicating the type of the edge.
:    Has the string-typed "targetNodeType* parameter indicating the type of the
     node that this edge points to.
:    Has the string-typed *targetId* parameter indicating the id of the node
     that this edge points to.
:    Indicates the beginning of a "EDGE" context.

EDGE_END
:    Indicates that there is no more information about the current edge.
:    Is only valid in the "EDGE" context.
:    Indicates the ending of a "EDGE" context.

PROPERTY_START
:    Indicates the beginning of information about a property.
:    Is only valid in the "NODE" context with the exception of (#graph-fragment).
     Event must be prior to any EDGE events.
:    In a #graph-fragment, event is valid in "EDGE" context.
:    Has the string-typed *key* parameter indicating the property key.
:    Has the ValueType-typed *valueType* parameter indicating the type of the
     property values. Valid values are INTEGER, FLOAT, BOOLEAN, STRING.
:    Indicates the beginning of a "PROPERTY" context.

PROPERTY_END
:    Indicates that there are no more values for the current property.
:    Is only valid in the "PROPERTY" context.
:    Indicates the ending of a "PROPERTY" context.

PROPERTY_REMOVE_START
:    Indicates the removal of a property from node.
:    Is only valid in the "NODE_UPDATE" context.
     Event must be prior to any EDGE events.
:    Has the string-typed *key* parameter indicating the property key.
:    Has the ValueType-typed *valueType* parameter indicating the type of the
     property values. Valid values are INTEGER, FLOAT, BOOLEAN, STRING.
:    Indicates the beginning of a "PROPERTY_REMOVE" context.
:    Subsequent events before the next "PROPERTY_REMOVE_END" are meant to capture
     information about property being removed.

PROPERTY_REMOVE_END
:    Indicates that there are no information for the current property being removed.
:    Is only valid in the "PROPERTY_REMOVE" context.
:    Indicates the ending of a "PROPERTY_REMOVE" context.

FEATURE_START
:    Indicates the beginning of information about a feature.
:    Is only valid in the "NODE" context.
:    Has the string-typed *key* parameter indicating the feature key.
:    Has the ValueType-typed *valueType* parameter indicating the type of the
     feature values. Valid values are INTEGER, FLOAT, BOOLEAN, STRING.
:    Indicates the beginning of a "FEATURE" context.

FEATURE_END
:    Indicates that there are no more values for the current feature.
:    Is only valid in the "FEATURE" context.
:    Indicates the ending of a "FEATURE" context.

VALUE_INTEGER
:    Indicates an integer value on the current property or feature.
:    Is only valid in a "PROPERTY", "PROPERTY_REMOVE" or "FEATURE" context
     where *valueType* is "INTEGER".
:    Has the int-typed *value* parameter that has the value.

VALUE_FLOAT
:    Indicates a float value on the current property or feature.
:    Is only valid in a "PROPERTY", "PROPERTY_REMOVE" or "FEATURE" context
     where *valueType* is "FLOAT".
:    Has the float-typed *value* parameter that has the value.

VALUE_BOOLEAN
:    Indicates a boolean value on the current property or feature.
:    Is only valid in a "PROPERTY", "PROPERTY_REMOVE" or "FEATURE" context
     where *valueType* is "BOOLEAN".
:    Has the boolean-typed *value* parameter that has the value.

VALUE_STRING
:    Indicates a string value on the current property or feature.
:    Is only valid in a "PROPERTY", "PROPERTY_REMOVE" or "FEATURE" context
     where *valueType* is "STRING".
:    Has the string-typed *value* parameter that has the value.

There are two sides to interacting with a stream - a program may produce events
or consume events.

#### Event Generation

In order to generate events, a program will implement an EventGenerator. Typical
implementations are parsers or graph adapters. An EventGenerator is a stateful,
non-thread-safe construct that provides single method:

notify
:    Receives a listener, and notifies it of the next event in the stream.
:    Returns true if the stream contains more events, false if it is complete.

#### Event Listener -- Push Style

The first of two consumer styles, a program will implement the EventListener.
This basically serves as a stateful callback that is notified of every event
coming from the stream, in order. The EventListener has one callback method for
each event type. Each method is passed the parameters for that event. Dynamic
languages may allow any event callback to be optional, provided that the absence
of those that start contexts causes any events within that context to not be
provided as well.

handleDocStart(id)

handleDocEnd()

handleGraphFragmentStart(id, sourceGraph)

handleGraphFragmentEnd()

handleDiffStart(sourceId, targetId)

handleDiffEnd()

handleSourceTargetNode(nodeType, sourceId, targetId)

handleUsesSchema(uri)

handleAsSpan(nodeType)

handleAsSequence(nodeType)

handleAsSpanContainer(nodeType, spanType)

handleContent(contentType, content)

handleNodeStart(nodeType, id)

handleNodeEnd()

handleNodeRemoveStart(nodeType, id)

handleNodeRemoveEnd()

handleNodeUpdateStart(nodeType, id)

handleNodeUpdateEnd()

handlePropertyStart(key, valueType)

handlePropertyEnd()

handlePropertyRemoveStart(key, valueType)

handlePropertyRemoveEnd()

handleFeatureStart(key, valueType)

handleFeatureEnd()

handleFeatureRemove(key)

handleEdge(edgeType, targetType, targetId)

handleEdgeRemove(edgeType, targetType, targetId)

handleEdgeStart(edgeType, targetType, targetId)

handleEdgeEnd(edgeType, targetType, targetId)

handleValueInteger(value)

handleValueFloat(value)

handleValueBoolean(value)

handleValueString(value)

handleValueEnum(value)

#### Event Consumption -- Pull Style

The second of the two consumer styles, in this style the stream is represented
as an object and a program will consume the events by repeatedly asking the
stream for the next one.

nextEvent
:    Increments the counter in the stream to the next event.
:    Returns the event type of the next event.

getEventType
:    Returns the event type of the current event.

getId
:    Returns the ID of the document or node.
:    Valid in DOC_START or NODE_START or FRAGMENT_START.

getSourceGraphId
:    Returns the source graph identifier for a graph fragment.
:    Valid in FRAGMENT_START.

getSchemaId
:    Returns a pagi schema uri.
:    Valid in USES_SCHEMA.

getContentType
:    Returns the content type.
:    Valid in CONTENT.

getContent
:    Returns the content.
:    Valid in CONTENT.

getNodeType
:    Returns the type of the node.
:    Valid in NODE_START, AS_SPAN, AS_SEQUENCE and AS_SPAN_CONTAINER.

getSpanType
:    Returns the type of the span.
:    Valid in AS_SPAN_CONTAINER

getKey
:    Returns the key of the property or feature.
:    Valid in PROPERTY_START, PROPERTY_REMOVE, FEATURE_START, or FEATURE_REMOVE

getValueType
:    Returns the type of the property or feature.
:    Valid in PROPERTY_START or FEATURE_START.

getIntegerValue
:    Returns the integer value.
:    Valid in VALUE_INTEGER.

getFloatValue
:    Returns the float value.
:    Valid in VALUE_FLOAT.

getBooleanValue
:    Returns the boolean value.
:    Valid in VALUE_BOOLEAN.

getStringValue
:    Returns the string value.
:    Valid in VALUE_STRING.

getEdgeType
:    Returns the type of the edge.
:    Valid in EDGE or EDGE_REMOVE.

getEdgeTargetType
:    Returns the type of the target node of the edge.
:    Valid in EDGE or EDGE_REMOVE.

getEdgeTargetId
:    Returns the id of the target node of the edge.
:    Valid in EDGE or EDGE_REMOVE.

Transfer Formats {#transfer-formats}
------------------------------------

In order to store and transfer PAGIM models, we must define formats. 

### XML {#xml-format}

A PAGI document can be rendered as an xml document. This is intended as the
primary storage and transfer format. It is simply structured and human-
readable.

Its Internet Media Type is `application/vnd.dr.pagi.stream+xml`.

Here is the general structure:

```xml 
<pagif xmlns="http://pagi.org/stream/"
       id="">
  <schema uri="drs-pagis"/>
  <asSpan nt=""/>
  <asSequence nt=""/>
  <asSpanContainer nt="" st=""/>
  <content contentType="text/plain">Text content of graph.</content>
  <node type="" id="">
    <prop k="" int=""/>
    <prop k="">
      <value int=""/>
    </prop>
    <edge type="" toType="" to=""/>
    <feat k="">
      <val int=""/>
    </feat>
  </node>
</pagif>
```

The actual xsd is referenced [in the appendix](#xml-format-xsd).

### Binary {#binary-format}

In order to facilitate efficient transfer and storage between components of
analytic applications, we define a binary format.  The binary format is a
simple, byte-wise representation of the event stream.  We have taken some
steps to ensure robustness and future adaptability.

Its Internet Media Type is `application/vnd.dr.pagi.stream`.
Its extension is `pbf`.

A `pbf` file is made up of three sections. The __preamble__, the the
__header-block__, and the __event-stream__.

#### Primitive Data Types

Several primitive data types are referenced repeatedly in the rest of this spec.
All multi-byte numbers are big-endian.

short
:    a 2-byte integer-type number

string-ref
:    a short value that is a pointer into the string cache

string
:    a short value [``l``]
:    followed by ``l`` *UTF-8* encoded bytes

block_seq
:    recursive data structure
:    a string w/ length == 0xFF + a block_seq OR
:    a string w/ length < 0xFF

lrc
:    a 1-byte checksum for a short record
:    comes at the end of a record, and input is the entire preceding portion of the record
:    as defined in [ISO-1155][].

   [ISO-1155]: http://www.iso.org/iso/iso_catalogue/catalogue_tc/catalogue_detail.htm?csnumber=5723

#### Preamble

The preamble itself is composed of two pieces. The first is the __signature__,
and the second is the __version-number__. The __signature__ is a fixed literal
and does not change. The __version-number__ is a *short* that will increment as
this specification evolves. The current value is *1*. The signature is:

```
  0x87 0x50 0x41 0x47
  0x49 0x0D 0x0A 0x1A
  0x0A 0x1A
```

#### Header Block

The header block is composed of a __header-length__ followed by a series of
__headers__. The __header-length__ is a *short* indicating the total
number of bytes in the series of __headers__. Note that the __header-length__
bytes themselves are **NOT** included in this length. Each __header__ in the
series is a __header-code__ followed by a *string*, and terminated by a *lrc*.
Compliant implementations are expected to write out all headers currenlty
defined. Defined __header-codes__ are defined below:

 * __0x01__ : *DATE CREATED*
 * __0x02__ : *CREATING USER*
 * __0x03__ : *CREATING MACHINE*
 * __0x04__ : *TOOL NAME*
 * __0x05__ : *TOOL VERSION*
 * __0x06__ : *LIB NAME*
 * __0x07__ : *LIB VERSION*
 * __0x08__ : *PLATFORM DETAILS*

#### Event Stream

The __event-stream__ is a series of event records. Each event record is composed
of a 1-byte __event-code__, followed by some __event-code__-specific data, follow by
a *lrc* over both the __event-code__ and the __event-code__-specific
data. The event stream maintains a string cache. There is one *special*
__event-code__, __NEW_STRING__ that adds a string into the cache. The string cache holds
2^16 strings. Each __NEW_STRING__ event is the code ``0xFF`` followed by a *string-ref*,
followed by a *string*.  String references are typically defined in an on-demand
basis, and the references will roll over once the maximum is reached. Given this
behavior, it is not uncommon to see strings repeated in very large files. It is
presumed that this cache is large enough to prevent repetition to the point of
a severe degredation to performance. The other events are defined below:


| Code        | Name                        | Event-Code-Specific Data                                                             |
| ----------- | ----------------------------| -------------------------------------------------------------------------------------|
| ``0x01``    | __DOC_START__               | <__docId__[*string-ref*]>                                                            |
| ``0x02``    | __DOC_END__                 | <*empty*>                                                                            |
| ``0x17``    | __FRAGMENT_START__          | <__fragmentId__[*string-ref*]><__sourceGraphId__[*string-ref*]>                      |
| ``0x18``    | __FRAGMENT_END__            | <*empty*>                                                                            |
| ``0x19``    | __DIFF_START__              | <__sourceDocId__[*string-ref*]><__targetDocId__[*string-ref*]>                       |
| ``0x1A``    | __DIFF_END__                | <*empty*>                                                                            |
| ``0x1B``    | __SOURCE_TARGET_NODE__      | <__nodeType__[*string-ref*]><__sourceId__[*string-ref*]><__sourceId__[*string-ref*]> |
| ``0x03``    | __NODE_START__              | <__nodeType__[*string-ref*]><__nodeId__[*string-ref*]>                               |
| ``0x04``    | __NODE_END__                | <*empty*>                                                                            |
| ``0x1C``    | __NODE_REMOVE_START__       | <__nodeType__[*string-ref*]><__nodeId__[*string-ref*]>                               |
| ``0x1D``    | __NODE_REMOVE_END__         | <*empty*>                                                                            |
| ``0x1E``    | __NODE_UPDATE_START__       | <__nodeType__[*string-ref*]><__nodeId__[*string-ref*]>                               |
| ``0x1F``    | __NODE_UPDATE_END__         | <*empty*>                                                                            |
| ``0x05``    | __PROPERTY_START__          | <__key__[*string-ref*]><__valueType__[*value-type*]>                                 |
| ``0x06``    | __PROPERTY_END__            | <*empty*>                                                                            |
| ``0x20``    | __PROPERTY_REMOVE_START__   | <__key__[*string-ref*]><__valueType__[*value-type*]>                                 |
| ``0x21``    | __PROPERTY_REMOVE_END__     | <*empty*>                                                                            |
| ``0x07``    | __EDGE__                    | <__type__[*string-ref*]><__targetType__[*string-ref]><__targetId__[*string-ref*]>    |
| ``0x15``    | __EDGE_START__              | <__type__[*string-ref*]><__targetType__[*string-ref]><__targetId__[*string-ref*]>    |
| ``0x16``    | __EDGE_END__                | <*empty*>                                                                            |
| ``0x22``    | __EDGE_REMOVE__             | <__type__[*string-ref*]><__targetType__[*string-ref]><__targetId__[*string-ref*]>    |
| ``0x08``    | __FEATURE_START__           | <__key__[*string-ref*]><__valueType__[*value-type*]>                                 |
| ``0x09``    | __FEATURE_END__             | <*empty*>                                                                            |
| ``0x0A``    | __VALUE_INTEGER__           | 4-byte integer                                                                       |
| ``0x0B``    | __VALUE_FLOAT__             | 4-byte [IEEE-754-2008][] floating-point                                              |
| ``0x0C``    | __VALUE_BOOLEAN__           | ``0x0F`` (true) or ``0xF0`` (false)                                                  |
| ``0x0D``    | __VALUE_STRING__            | <*string-ref*>                                                                       |
| ``0x0F``    | __USES_SCHEMA__             | <__schemaId__[*string-ref*]>                                                         |
| ``0x10``    | __CONTENT__                 | <__contentType__[*string*]><__content__[*block_seq*]>                                |
| ``0x11``    | __CONTENT_CHKSUM__          | <__contentType__[*string*]><__checksum__[*4-byte integer*]>                          |
| ``0x12``    | __AS_SPAN__                 | <__nodeType__[*string-ref*]>                                                         |
| ``0x13``    | __AS_SEQUENCE__             | <__nodeType__[*string-ref*]>                                                         |
| ``0x14``    | __AS_SPAN_CONTAINER__       | <__nodeType__[*string-ref*]><__spanType__[*string-ref*]>                             |

The __valueType__ that is referenced in __PROPERTY_START__ and __PROPERTY_END__ is as follows:

| Code        | Name          |
| ----------- | ------------- |
| ``0x01``    | __INTEGER__   |
| ``0x02``    | __FLOAT__     |
| ``0x03``    | __BOOLEAN__   |
| ``0x04``    | __STRING__    |

The *block_seq* type is a mechanism to allow string content of unlimited length. It is basically
the same data structure as a string, with one exception. If the string length is equal to the
max (0xFF) then we expect another string to follow it. This content is simply intended to be 
appended to the preceding content. This pattern is repeated until a block is reached where the
length is less than the max. This is the final block in the content.

The __CONTENT_CHKSUM__ event code is used in situations where the storage mechanism wishes
to store the content externally. This is often the case when content should be accessible
both in the context of the graph and outside of that context. In this case, it is expected
that the storage mechanism will provide the content to the parser prior to parsing. When
a __CONTENT_CHKSUM__ event code is detected in the file, the checksum should be validated
with the previously provided content and then a __CONTENT__ event should be generated on the
stream, including the content itself (not the checksum). This is merely a storage mechanism,
and its existence should not be exposed in the stream API. The checksum algorithm is crc32.

   [IEEE-754-2008]: http://ieeexplore.ieee.org/xpl/mostRecentIssue.jsp?punumber=4610933

Graph Fragment {#graph-fragment}
--------------------------------

A graph fragment is an format and schema specification in extension to the core document specification.
It is a means of representing graph based information that is tied to an external source graph.
Node and edge specification for fragments largely remains identical with a few modifications for
flexibility.

The following document graph specifications do not apply to graph fragments:
 * document content
 * (#traits)
 * (#id-generation) pattern

Edge specification has been expanded to support typed properties identical to Node specification.

A schema can be specified at a graph fragment level similar to the core document schema with
adjustments to support edge property definition and exclusion of trait and node id pattern definition.

Graph Diff {#graph-diff}
--------------------------------

Graph diff is a format specification which captures differences between a source and target graph as
an event stream relative to modifications made on source graph. The diff format includes enough
information to be reversible such that a diff can be unapplied from a targetGraph to get back to
source graph. This is achieved by preserving all information that is removed in the diff and also
keeping node id equality mappings between source and target graphs.

Diff information can be stored using the standard (#transfer-formats).

Corpus-Scoped Analytics {#corpus-scoped-analytics}
--------------------------------------------------

Future Scope {#future-scope}
----------------------------

 * Representing non-textual data (Audio or Visual)
 * Stackd.io integration
 * Visualization
 * Views in CSV and GraphML

Appendices {#appendices}
------------------------

###XML Format XSD {#xml-format-xsd}
```xml
<<<<pagif-xml.xsd>>>>
```

###PAGI Schema XSD {#pagi-schema-xsd}
```xml
<<<<pagis.xsd>>>>
```
