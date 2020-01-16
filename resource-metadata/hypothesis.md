# Resource Metadata Model

## Background

*This section is non-normative*

This document introduces a mechanism for linking to metadata about resources in the Solid Ecosystem using HTTP Link headers. Examples of this linking mechanism include:

- A binary JPEG image with a Link header, pointing to a description that has an RDF representation.
- An LDP container with a Link header, pointing to an ACL resource that governs access controls for the contained resources.
- A resource whose shape is constrained by a ShEx document includes a Link header that points to that ShEx resource.
- A resource may keep an audit trail of modification events. A client could follow the URI in a Link header to access that audit trail.

A given resource might link to zero or more such related metadata representations. Having a unified model to describe how clients and Solid servers interact with and process these resources will help clarify expectations, while also providing a shared pattern for extending the features of Solid server implementations.

The metadata model described in this document makes it possible for clients to decorate resources with structured descriptive data. Those metadata may serve as supplementary descriptions or, when supported by a Solid server implementation, they may influence the behavior of the resources.

Some examples include:

- An ACL resource controls how a Solid server makes authorization decisions for a container and any child resources.
- A shape constraint resource may limit the RDF structures that can be added to a resource or container.
- A “configuration” resource may control how a resource is versioned, or which indexes are exposed for it.

Access to different categories of metadata may require different levels of privilege, which must be specified as part of the definition for a given metadata type.

## Metadata Discovery

Given the URL of a resource, a client can discover the metadata resources by issuing a GET (or HEAD) request and inspecting the Link headers in the response. The [rel= attribute](https://tools.ietf.org/html/rfc8288) will define the relationship to the target URL.

For any defined metadata type available for a given resource, all representations of that resource MUST include a Link header pointing to the location of each metadata resource. For example, as defined by the Solid [Web Access Control specification](https://github.com/solid/web-access-control-spec), a client can use this mechanism to discover the location of an ACL resource:

Link: <https://example.com/resource?acl>; rel="acl"

Metadata discovered through a Link header for a given resource is considered to be *directly associated* with that resource.

### Discovery of Annotated Resource

Certain metadata resource types MAY require the Solid server to include a link back to the annotated resource using an appropriate link relation. For example, LDP defines a bidirectional discovery mechanism for RDF descriptions of NonRDF resources, via Link headers:

Link: <https://example.com/binary?description>; rel="description"

along with:

Link: <https://example.com/binary>; rel="describedby"

For cases where link relations are not defined by IANA, a URL can be used. For example:

Link: <https://example.com/resource?meta>;
        rel="https://example.org/ns#hasMetadata"

and:

Link: <https://example.com/resource>;
        rel="https://example.org/ns#isMetadataOf"

String-based link relations, such as in the examples above, must be registered with IANA. But it is also possible to use custom relation types by using a full IRI. The [Linked Data Platform](https://www.w3.org/TR/ldp/), the [Linked Data Notifications](https://www.w3.org/TR/ldn/), and the [Web Annotation Protocol](http://www.w3.org/TR/annotation-protocol/) specifications make use of full IRIs in the `rel` attribute.

## Metadata Characteristics

A given resource MAY Link to metadata on a different server.

Metadata resources on the same Solid server MUST adhere to the same interaction model used by standard Solid resources.

## Reserved Metadata Types

### Web Access Control

ACL resources as defined by [Web Access Control](https://github.com/solid/web-access-control-spec) MUST be supported as a resource metadata type by Solid servers.

ACL metadata resources are discoverable by the client via ```rel=acl```.

A given Solid resource MUST NOT be directly associated with more than one ACL metadata resource.

An ACL metadata resource MUST be deleted by the Solid server when the resource it is directly associated with is also deleted and the Solid server is authoritative for both resources.

To access or manage an ACL metadata resource, an [acl:agent](https://github.com/solid/web-access-control-spec#describing-agents) MUST have [acl:Control](https://github.com/solid/web-access-control-spec#aclcontrol) privileges per the [ACL inheritance algorithm](https://github.com/solid/web-access-control-spec#acl-inheritance-algorithm) on the resource directly associated with it.

A Solid server SHOULD sanity check ACL metadata resources upon creation or update to restrict invalid changes.

### Resource Description

Resource description is a general mechanism to provide descriptive metadata for a given resource. It MUST be supported as a resource metadata type by Solid servers.

The Descriptive metadata resource for a given resource is discovered by the client via ```rel=describedby```. Conversely, the resource being described by a Descriptive metadata resource is discovered by the client via ```rel=http://www.w3.org/ns/solid/terms#resource```.

A given Solid resource MUST NOT be directly associated with more than one Descriptive metadata resource.

A Descriptive metadata resource MUST be deleted by the Solid server when the resource it is directly associated with is also deleted and the Solid server is authoritative for both resources.

Access or management of a Descriptive metadata resource by a given [acl:agent](https://github.com/solid/web-access-control-spec#describing-agents) is subject to the [modes of access](https://github.com/solid/web-access-control-spec#modes-of-access) granted per the [ACL inheritance algorithm](https://github.com/solid/web-access-control-spec#acl-inheritance-algorithm) on the resource directly associated with it.

### Shape Validation

Shape Validation ensures that any data changes in a given resource conform to an associated [SHACL](https://www.w3.org/TR/shacl/) or [ShEx](https://shex.io/shex-semantics/index.html) data shape. It MUST be supported as a resource metadata type by Solid servers.

The Shape validation metadata resource for a given resource is discovered by the client via ```rel=http://www.w3.org/ns/solid/terms#shape```. Conversely, the resource being described by a Shape validation metadata resource is discovered by the client via ```rel=http://www.w3.org/ns/solid/terms#resource```.

A given Solid resource MUST NOT be directly associated with more than one Descriptive metadata resource.

A Shape validation metadata resource MUST be deleted by the Solid server when the resource it is directly associated with is also deleted and the Solid server is authoritative for both resources.

To access or manage a Shape validation metadata resource, an [acl:agent](https://github.com/solid/web-access-control-spec#describing-agents) MUST have [acl:Control](https://github.com/solid/web-access-control-spec#aclcontrol) privileges per the [ACL inheritance algorithm](https://github.com/solid/web-access-control-spec#acl-inheritance-algorithm) on the resource directly associated with it.

A Solid server SHOULD sanity check Shape validation metadata resources upon creation or update to restrict invalid changes.

### Server Managed

A Solid server stores information about a resource that clients can read but not change in Server Managed metadata. It MUST be supported as a resource metadata type by Solid servers.

A Server Managed metadata resource is discovered by the client via ```rel=http://www.w3.org/ns/solid/terms#servermanaged```. Conversely, the resource being described by a Server Managed metadata resource is discovered by the client via ```rel=http://www.w3.org/ns/solid/terms#resource```.

A given Solid resource MUST NOT be directly associated with more than one Server Managed metadata resource.

A Server Managed metadata resource MUST be deleted by the Solid server when the resource it is directly associated with is also deleted and the Solid server is authoritative for both resources.

To access a Server Managed metadata resource, an [acl:agent](https://github.com/solid/web-access-control-spec#describing-agents) MUST have [acl:Read](https://github.com/solid/web-access-control-spec#aclread) privileges per the [ACL inheritance algorithm](https://github.com/solid/web-access-control-spec#acl-inheritance-algorithm) on the resource directly associated with it.

### Configuration

Configuration metadata is used to store configurable parameters for a given resource. It MUST be supported as a resource metadata type by Solid servers.

A configuration metadata resource is discovered by the client via ```rel=http://www.w3.org/ns/solid/terms#configuration```. Conversely, the resource being described by a Configuration metadata resource is discovered by the client via ```rel=http://www.w3.org/ns/solid/terms#resource```.

A given Solid resource MUST NOT be directly associated with more than one Configuration metadata resource.

A Configuration metadata resource MUST be deleted by the Solid server when the resource it is directly associated with is also deleted and the Solid server is authoritative for both resources.

To access or manage a Configuration metadata resource, an [acl:agent](https://github.com/solid/web-access-control-spec#describing-agents) MUST have [acl:Control](https://github.com/solid/web-access-control-spec#aclcontrol) privileges per the [ACL inheritance algorithm](https://github.com/solid/web-access-control-spec#acl-inheritance-algorithm) on the resource directly associated with it.

## Non-Reserved Types

A Solid server may support other metadata types.

## Implementation Patterns

*This section is non-normative.*

There are many ways a Solid server could implement these features. A file-based Solid server could have a special naming scheme reserved for these metadata resources. Alternatively, a Solid server could represent every resource internally as a dataset, storing each separate type of metadata in its own named graph.

A Solid server needs to maintain a working knowledge of which resources are metadata, because it tells the clients where to find them. This means that it can similarly apply this knowledge to know when someone is writing to a known metadata resource, such as an ACL, and can apply the appropriate validation and sanity checks to ensure the changes are valid.