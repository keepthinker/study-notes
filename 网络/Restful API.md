# Representational state transfer

Representational state transfer (REST) is a software architectural style which uses a subset of HTTP. It is commonly used to create interactive applications that use Web services. A Web service that follows these guidelines is called RESTful. Such a Web service must provide its Web resources in a textual representation and allow them to be read and modified with **a stateless protocol** and a predefined set of operations. This approach allows interoperability between the computer systems on the Internet that provide these services. REST is an alternative to, for example, SOAP as way to access a Web service.

## Architectural constraints

Six guiding constraints define a RESTful system.These constraints restrict the ways that the server can process and respond to client requests so that, by operating within these constraints, the system gains desirable [non-functional properties](https://en.wikipedia.org/wiki/Non-functional_requirement), such as performance, scalability, simplicity, modifiability, visibility, portability, and reliability.[[1\]](https://en.wikipedia.org/wiki/Representational_state_transfer#cite_note-Fielding-Ch5-1) If a system violates any of the required constraints, it cannot be considered RESTful.

The formal REST constraints are as follows:

### Client–server architecture

See also: [Client–server model](https://en.wikipedia.org/wiki/Client–server_model)

The principle behind the client–server constraints is the separation of concerns. Separating the user interface concerns from the data storage concerns improves the portability of the user interfaces across multiple platforms. It also improves scalability by simplifying the server components. Perhaps most significant to the Web is that the separation allows the components to evolve independently, thus supporting the Internet-scale requirement of multiple organizational domains.[[1\]](https://en.wikipedia.org/wiki/Representational_state_transfer#cite_note-Fielding-Ch5-1)

### Statelessness

See also: [Stateless protocol](https://en.wikipedia.org/wiki/Stateless_protocol)

In computing, a stateless protocol is a [communications protocol](https://en.wikipedia.org/wiki/Communications_protocol) in which no session information is retained by the receiver, usually a server. Relevant session data is sent to the receiver by the client in such a way that every packet of information transferred can be understood in isolation, without context information from previous packets in the session. This property of stateless protocols makes them ideal in high volume applications, increasing performance by removing server load caused by retention of session information.

### Cacheability

See also: [Web cache](https://en.wikipedia.org/wiki/Web_cache)

As on the World Wide Web, clients and intermediaries can cache responses. Responses must, implicitly or explicitly, define themselves as either cacheable or non-cacheable to prevent clients from providing stale or inappropriate data in response to further requests. Well-managed caching partially or completely eliminates some client–server interactions, further improving scalability and performance.

### Layered system

See also: [Layered system](https://en.wikipedia.org/wiki/Layered_system)

A client cannot ordinarily tell whether it is connected directly to the end server or to an intermediary along the way. If a [proxy](https://en.wikipedia.org/wiki/Proxy_server) or [load balancer](https://en.wikipedia.org/wiki/Load_balancing_(computing)) is placed between the client and server, it won't affect their communications, and there won't be a need to update the client or server code. Intermediary servers can improve system [scalability](https://en.wikipedia.org/wiki/Scalability) by enabling load balancing and by providing shared caches. Also, security can be added as a layer on top of the web services, separating business logic from security logic.[[11\]](https://en.wikipedia.org/wiki/Representational_state_transfer#cite_note-11) Adding security as a separate layer enforces [security policies](https://en.wikipedia.org/wiki/Security_policy). Finally, intermediary servers can call multiple other servers to generate a response to the client.

### Code on demand (optional)

See also: [Client-side scripting](https://en.wikipedia.org/wiki/Client-side_scripting)

Servers can temporarily extend or customize the functionality of a client by transferring executable code: for example, compiled components such as [Java applets](https://en.wikipedia.org/wiki/Java_applet), or client-side scripts such as JavaScript.

### Uniform interface

The uniform interface constraint is fundamental to the design of any RESTful system.[[1\]](https://en.wikipedia.org/wiki/Representational_state_transfer#cite_note-Fielding-Ch5-1) It simplifies and decouples the architecture, which enables each part to evolve independently. The four constraints for this uniform interface are:

- Resource identification in requests

  Individual resources are identified in requests, for example using [URIs](https://en.wikipedia.org/wiki/Uniform_resource_identifier) in RESTful Web services. The resources themselves are conceptually separate from the representations that are returned to the client. For example, the server could send data from its database as [HTML](https://en.wikipedia.org/wiki/HTML), [XML](https://en.wikipedia.org/wiki/XML) or as [JSON](https://en.wikipedia.org/wiki/JSON)—none of which are the server's internal representation.

- Resource manipulation through representations

  When a client holds a representation of a resource, including any [metadata](https://en.wikipedia.org/wiki/Metadata) attached, it has enough information to modify or delete the resource's state.

- Self-descriptive messages

  Each message includes enough information to describe how to process the message. For example, which parser to invoke can be specified by a [media type](https://en.wikipedia.org/wiki/Media_type).[[1\]](https://en.wikipedia.org/wiki/Representational_state_transfer#cite_note-Fielding-Ch5-1)

- Hypermedia as the engine of application state ([HATEOAS](https://en.wikipedia.org/wiki/HATEOAS))

  Having accessed an initial URI for the REST application—analogous to a human Web user accessing the [home page](https://en.wikipedia.org/wiki/Home_page) of a website—a REST client should then be able to use server-provided links dynamically to discover all the available resources it needs. As access proceeds, the server responds with text that includes [hyperlinks](https://en.wikipedia.org/wiki/Hyperlink) to other resources that are currently available. There is no need for the client to be hard-coded with information regarding the structure or dynamics of the application.[[12\]](https://en.wikipedia.org/wiki/Representational_state_transfer#cite_note-RESTfulAPI.net-12)



## Classification models

Several models have been developed to help classify REST APIs according to their adherence to various principles of REST design, such as the [Richardson Maturity Model](https://en.wikipedia.org/wiki/Richardson_Maturity_Model).[[13\]](https://en.wikipedia.org/wiki/Representational_state_transfer#cite_note-13)

## Applied to web services

Web service APIs that adhere to the REST architectural constraints are called RESTful APIs. HTTP-based RESTful APIs are defined with the following aspects:

- a base URI, such as `http://api.example.com/`;
- standard HTTP methods(e.g., GET, POST, PUT, and DELETE);
- a media type that defines state transition data elements (e.g., Atom, microformats, application/vnd.collection+json). The current representation tells the client how to compose requests for transitions to all the next available application states. This could be as simple as a URI or as complex as a Java applet.

### Semantics of HTTP methods

The following table shows how HTTP methods are intended to be used in HTTP APIs, including RESTful ones.

| HTTP method | CRUD equivalent |                         Description                          |
| :---------: | :-------------: | :----------------------------------------------------------: |
|     GET     |      Read       |     Get a representation of the target resource’s state.     |
|    POST     |     Create      | Let the target resource process the representation enclosed in the request. |
|     PUT     |     Update      | Set the target resource’s state to the state defined by the representation enclosed in the request. |
|   DELETE    |     Delete      |             Delete the target resource’s state.              |

The GET method is safe meaning that applying it to a resource does not result in a state change of the resource (read-only semantics). The GET, PUT, and DELETE methods are idempotent, meaning that applying them multiple times to a resource results in the same state change of the resource as applying them once, though the response might differ. The GET and POST methods are [cacheable](https://en.wikipedia.org/wiki/Web_cache), meaning that responses to them are allowed to be stored for future reuse.

The GET (read), PUT (create and update), and DELETE (delete) methods are [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) operations as they have [storage management](https://en.wikipedia.org/wiki/Memory_management) semantics, meaning that they let [user agents](https://en.wikipedia.org/wiki/User_agent) directly manipulate the states of target resources. The POST method is not a CRUD operation but a process operation that has target-resource-specific semantics excluding storage management semantics, so it does not let user agents directly manipulate the states of target resources.



## References

[Representational state transfer](https://en.wikipedia.org/wiki/Representational_state_transfer)