# C4-Flow
***A new LLM friendly, human readable, code architecting/modelling tool***

This is an informal launch of the C4 Flow specification I've been working on for the past week. I am actively seeking feedback on its usefulness and design quality, so please leave any thoughts and feedback at TODO.

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

## Contract

| Property        | Examples                          | Motivation                                                            |
| --------------- | --------------------------------- | --------------------------------------------------------------------- |
| kind            | "login", "upload file"            | Human-readable purpose; drives threat-modelling & UX docs             |
| direction       | ->, <-                            | Clarifies who initiates action                                        |
| sync            | true, false                       | Distinguishes blocking, conversational flow from notification systems |
| trustBoundary   | "external", "level 2"             | Indicates whether auth should be triggered                            |
| confidentiality | "public", "internal", "secret"    | GDPR / ISO 27001 handling rules                                       |
| channel         | "web", "mobile", "sms"            | Helps devOps team plan                                                |
| freq            | "interactive", "periodic", "bulk" | Capacity/load planning                                                |

## Protocol

| Property   | Examples                       | Motivation                                                                              |
| ---------- | ------------------------------ | --------------------------------------------------------------------------------------- |
| kind       | "request", "stream", "pub/sub" | Describes the utility of the connection                                                 |
| tech       | "REST", "websocket", "gRPC"    | Specifies key rules and connection characteristics                                      |
| direction  | ->, <-                         | Clarifies who initiates action                                                          |
| sync       | true, false                    | Distinguishes blocking, conversational flow from notification systems                   |
| auth       | "admin", "anon"                | Indicates what level of authorisation should have access                                |
| encryption | "tls", "mTLS", "none"          | Indicates how secure the connection is                                                  |
| endPoint   | "POST /docs", "topic:invoice"  | Specifies what to connect to                                                            |
| throughput | "1000 req/s"                   | Capacity/load planning                                                                  |
| timeout    | "3000 ms"                      | Accounts for networking and latency requirements                                        |
| retries    | 0, 3                           | Specifies error handling                                                                |
| qosTier    | "bronze", "platinum"           | Lets SREs map SLIs/SLAs quickly (not too familiar with this field, AI recommended this) |

## Contract

| Property   | Examples                        | Motivation                                                                         |
| ---------- | ------------------------------- | ---------------------------------------------------------------------------------- |
| kind       | "comand", "query", "event"      | Specifies command vs query and support CQRS patterns                               |
| direction  | ->, <-                          | Clarifies who initiates action                                                     |
| sync       | true, false                     | Guides thread model or concurrency design                                          |
| schema     | "payload.json", "binaryPayload" | A description or rule for the actual data shape; Can be JSON-Schema / Avro / Proto |
| idempotent | true, false                     | Is it safe to re-deliver?                                                          |
| version    | "v1", "v2.3"                    | Specifies whether the schema or connection properties has evolved.                 |

## Dataflow

| Property  | Examples                    | Motivation                                                 |
| --------- | --------------------------- | ---------------------------------------------------------- |
| kind      | "call", "read", "transform" | Describes the action type                                  |
| direction | ->, <-                      | Clarifies who initiates action                             |
| sync      | true, false                 | Guides thread model or concurrency design                  |
| datatype  | UserClass, string           | The type of data sent to the callee                        |
| scope     | "local", "module"           | Helps detects dependencies                                 |
| mutate    | true, false                 | Helps identify side effects and unexpected changes to data |
| returns   | PaymentData, string         | Specifies what the caller can expect from the callee.      |

With the above specifications, C4-Flow is able to capture fine-grained details about entities and their relationships when required, informing high level design decisions. 

Note that whilst all the properties are recommended, they are not compulsory in case the decision should be automatically made by LLMs.

Similarly, we can add node base properties to each of the abstraction levels but that is beyond the scope of this draft of the specification.

# DSL: BlueprintLang

The above is the C4 Flow draft specification. However, it does not actually specify the notation or the tooling just as the original C4 specification does not. Below is my own custom YAML-based language called BlueprintLang. It is fully C4-flow compliant and designed to be easily readable/editable by humans and machines.

```
# ─────────── NODES ───────────────────────────────────────────
# ──────────────── NODES ─────────────────────────────────────
nodes:
  # Context layer
  actor.user:
    label: User
    c4Type: context
    role: actor

  system.collab:
    label: Collab-Docs
    c4Type: context
    role: internalSystem

  # Container layer
  ctr.frontend:
    parent: system.collab
    label: Web Frontend
    c4Type: container
    role: frontend

  ctr.api:
    parent: system.collab
    label: API Gateway
    c4Type: container
    role: api

  ctr.storage:
    parent: system.collab
    label: Doc Storage
    c4Type: container
    role: infra

  # Component layer
  cmp.editorUI:
    parent: ctr.frontend
    label: Editor UI
    c4Type: component
    role: controller

  cmp.docService:
    parent: ctr.api
    label: Doc Service
    c4Type: component
    role: service

  cmp.docRepo:
    parent: ctr.storage
    label: Doc Repo
    c4Type: component
    role: repository

  # Code layer (single illustrative function)
  func.save:
    parent: cmp.docService
    label: save()
    c4Type: code
    role: function

# ──────────────── EDGES ─────────────────────────────────────
edges:
  # 1 ▸ Interaction  (Context → Context)
  ctx_login:
    source: actor.user
    target: system.collab
    c4FlowType: interaction
    kind: login                 # human-readable purpose
    direction: "->"
    sync: true
    trustBoundary: external
    confidentiality: confidential
    channel: web
    freq: interactive

  # 2 ▸ Protocol  (Container ↔ Container)
  prot_front_to_api:
    source: ctr.frontend
    target: ctr.api
    c4FlowType: protocol
    kind: request
    tech: REST
    direction: "->"
    sync: true
    auth: userToken
    encryption: TLS
    endPoint: "POST /docs"
    throughput: "5 req/s"
    timeout: "3000 ms"
    retries: 0
    qosTier: bronze

  # 3 ▸ Contract  (Component ↔ Component)
  contract_ui_to_service:
    source: cmp.editorUI
    target: cmp.docService
    c4FlowType: contract
    kind: command
    direction: "->"
    sync: true
    schema: SaveDoc
    idempotent: false
    version: v1

  # 4 ▸ Data-flow  (Code ↔ Component)
  data_save_to_repo:
    source: func.save
    target: cmp.docRepo
    c4FlowType: dataflow
    kind: call
    direction: "->"
    sync: true
    datatype: DocEntity
    scope: module
    mutate: true
    returns: boolean
```

One can then parse this language, merging it for example with a json with visual rendering information, to be visualised with the accompanying visualisation tool. 

For instance:

```
# ────────── Canonical semantics (tiny test graph) ──────────
nodes:
  actor.user:
    label: User
    c4Type: context
    role: actor

  system.collab:
    label: Collab-Docs
    c4Type: context
    role: internalSystem

edges:
  ctx_login:
    source: actor.user
    target: system.collab
    c4FlowType: interaction
    kind: login
    direction: "->"
    sync: true
    channel: web
```

```
{
  "nodes": [
    { "id": "actor.user",    "position": { "x": 40,  "y": 60 }, "width": 120, "height": 40 },
    { "id": "system.collab", "position": { "x": 280, "y": 60 }, "width": 160, "height": 60 }
  ],
  "edges": [
    {
      "id": "ctx_login",
      "source": "actor.user",
      "target": "system.collab",
      "markerEnd": { "type": "arrow" },
      "zIndex": 1
    }
  ]
}
```

Combines to make

```
{
  nodes: [
    {
      id: "actor.user",
      data: { label: "User", c4Type: "context", role: "actor" },
      position: { x: 40, y: 60 },
      width: 120,
      height: 40,
      zIndex: 0
    },
    {
      id: "system.collab",
      data: {
        label: "Collab-Docs",
        c4Type: "context",
        role: "internalSystem"
      },
      position: { x: 280, y: 60 },
      width: 160,
      height: 60,
      zIndex: 0
    }
  ],
  edges: [
    {
      id: "ctx_login",
      data: {
        source: "actor.user",
        target: "system.collab",
        c4FlowType: "interaction",
        kind: "login",
        direction: "->",
        sync: true,
        channel: "web"
      },
      zIndex: 1,
      markerEnd: { type: "arrow" },
      points: undefined
    }
  ]
}
```

Which can be rendered to the following using the accompanying web app Blueprint.

---
You can try to model some problems using the accompanying Blueprint app. It has quite a few rough edges right now and will be incrementally improved along with this spec in the coming days. Hope you find this model, language, and tool useful!
