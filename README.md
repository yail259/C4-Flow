# C4-Flow

***A new LLM friendly, human readable, code architecting/modelling tool***

This is an informal launch of the C4 Flow specification I've been working on for the past week. I am actively seeking feedback on its usefulness and design quality, so please leave any thoughts and feedback at its [Github Issues](https://github.com/yail259/C4-Flow/issues).

The motivation of this project was to create a new type of programming. As the software paradigm shifts from humans typing code to machines typing code, a stronger emphasis should be placed on the architecture of the system and ensuring rigorous overall design. This part still requires significant human input due to:

1. LLMs lacking strong ability in design thinking (from my personal experience)
2. Limited realistic/practical [LLM context windows](https://github.com/NVIDIA/RULER)
3. Certain design decisions such as *security* require explicit instructions
4. Smaller modular generation is better than one-shot whole repository generation
5. Natural language lacks preciseness in expressing boundaries and relationships

As such, I am proposing a new type of human-readable, machine parsable domain specific language (DSL) which can express key architecture design decisions made by humans, enabling both humans and machines to produce high quality code whilst keeping the overall architecture in mind.

Furthermore, the specification will be accompanied by a visualisation tool and DSL parser inline with C4's original intention of more powerful, easy to use software system visualisers.

---

The C4 model proposed by Simon Brown under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/), which describes a 4 layer representation of software systems.

| Layer     | Description                                                                    |
| --------- | ------------------------------------------------------------------------------ |
| Context   | A high level overview of the abstract systems such as humans, product, service |
| Container | An application or data store, for instance a Postgres database                 |
| Component | A logical runtime entity, such as DocService. It can be a module or library    |
| Code      | A class, function, enum, or any other programming construct                    |

However, this specification does not explicitly specify how relationships between entities should be described, this leads to generic verbal descriptions such as "A uses B", or "C delivers D to E".

In C4-Flow, we extend upon this framework and attempt to fix this shortcoming by defining edge types at the 4 abstraction levels. This has the advantage of:

1. Preventing mixing of abstraction level when describing relationships
2. Capturing different types of relationships at different abstraction levels
3. Improving static parsability of edge labels
4. Defining key edge properties such as security and performance

As such below are the proposed new edge types under the C4-Flow specification:

| Layer     | Edge Type   | Edge Description                                                                                           |
| --------- | ----------- | ---------------------------------------------------------------------------------------------------------- |
| Context   | Interaction | An interaction is any abstract relationship between actors and systems                                     |
| Container | Protocol    | A protocol describes the technology and specifications which is used to communicate between two containers |
| Component | Contract    | A contract defines the actual message type, shape and delivery traits between components                   |
| Code      | Dataflow    | A dataflow describes the fine-grained internal data at different stages of the code                        |

As such, the above edge descriptions allow us to powerfully capture the relationships between different entities of the same abstraction level. 

More over, to satisfy our original intent of improving security of the system, as well as formalising the description of edges, I am further proposing explicit recommended edge properties at each of the abstraction levels.

## Common Essential Properties

| Property | Example                                                                                                                                                          | Motivation                                                                                                                             |
| -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| kind     | Interaction: "login", "upload file"<br>Protocol: "request", "stream", "pub/sub"<br>Contract: "comand", "query", "event"<br>Dataflow: "call", "read", "transform" | Short verbal descriptions which allow for high level understanding of category and intent of connection                                |
| source   | User                                                                                                                                                             | The entity which is initiating the data sending or action                                                                              |
| target   | System                                                                                                                                                           | The entity which is receiving data or fulfilling a request.                                                                            |
| sync     | true, false                                                                                                                                                      | Distinguishes blocking, conversational flow from non-blocking, asynchronous event systems. Can inform concurrency or thread modelling. |

These are the core properties of interactions between any two entities at any abstraction levels. They provide essential information that MUST be well defined and lead to clearer systems in most contexts.

Below are some abstraction level specific optional properties which can inform better performance engineering, devOps, or security. They can be included or omitted based on use case or needs.

## Interaction

| Property        | Examples                          | Motivation                                                            |
| --------------- | --------------------------------- | --------------------------------------------------------------------- |
| trustBoundary   | "external", "level 2"             | Indicates whether auth should be triggered                            |
| confidentiality | "public", "internal", "secret"    | GDPR / ISO 27001 handling rules                                       |
| channel         | "web", "mobile", "sms"            | Helps devOps team plan                                                |
| freq            | "interactive", "periodic", "bulk" | Capacity/load planning                                                |

## Protocol

| Property   | Examples                      | Motivation                                                                              |
| ---------- | ----------------------------- | --------------------------------------------------------------------------------------- |
| tech       | "REST", "websocket", "gRPC"   | Specifies key rules and connection characteristics                                      |
| auth       | "admin", "anon"               | Indicates what level of authorisation should have access                                |
| encryption | "tls", "mTLS", "none"         | Indicates how secure the connection is                                                  |
| endPoint   | "POST /docs", "topic:invoice" | Specifies what to connect to                                                            |
| throughput | "1000 req/s"                  | Capacity/load planning                                                                  |
| timeout    | "3000 ms"                     | Accounts for networking and latency requirements                                        |
| retries    | 0, 3                          | Specifies error handling                                                                |
| qosTier    | "bronze", "platinum"          | Lets SREs map SLIs/SLAs quickly (not too familiar with this field, AI recommended this) |

## Contract

| Property   | Examples                        | Motivation                                                                         |
| ---------- | ------------------------------- | ---------------------------------------------------------------------------------- |
| schema     | "payload.json", "binaryPayload" | A description or rule for the actual data shape; Can be JSON-Schema / Avro / Proto |
| idempotent | true, false                     | Is it safe to re-deliver?                                                          |
| version    | "v1", "v2.3"                    | Specifies whether the schema or connection properties has evolved.                 |

## Dataflow

| Property  | Examples                    | Motivation                                                 |
| --------- | --------------------------- | ---------------------------------------------------------- |
| datatype  | UserClass, string           | The type of data sent to the callee                        |
| scope     | "local", "module"           | Helps detects dependencies                                 |
| mutate    | true, false                 | Helps identify side effects and unexpected changes to data |
| returns   | PaymentData, string         | Specifies what the caller can expect from the callee.      |

With the above specifications, C4-Flow is able to capture fine-grained details about entities and their relationships when required, informing high level design decisions. 

Note that whilst all the properties are recommended, they are not compulsory in case the decision should be automatically made by LLMs.

Similarly, we can add node base properties to each of the abstraction levels but that is beyond the scope of this draft of the specification.

---
The above is the C4 Flow draft specification. However, it does not actually specify the notation or the tooling just as the original C4 specification does not. 

Below is my own custom YAML-based language called BlueprintLang. It is fully C4-flow compliant and designed to be easily readable/editable by humans and machines. Check it out here at this [Github Link](https://github.com/yail259/BlueprintLang).

