# The StepFlow diagram spec

A spec describes a diagram without coordinates. StepFlow lays it out, sizes the
boxes, matches the icons and works out the animation. This file is read both by
the `stepflow-diagram` skill and by StepFlow's own AI, so it describes the spec
and nothing about either.

**Never put coordinates in a spec.** There is no field for them. Working out
where things go is StepFlow's job, and it is better at it than you are.

## Which shape of diagram

**A sequence diagram** when the thing being explained happens over time: a
request on its way through, a retry, a handshake, an ordering of calls. Most changes to
behaviour are this.

**A flow diagram** when the thing being explained is a structure: which services
exist, what talks to what, a decision tree, the steps of a business process.
Reach for this for architecture and for anything with cloud icons.

## Animation and step numbers

Both kinds animate by default: a dot travels each connector in turn, in the order
the diagram happens. An animated diagram is also **numbered**: each connector's
label starts with its place in that order, `1. POST /orders`, `2. enqueue`, so
the order reads in a still frame and to somebody who has not watched it play.
Connectors that fire together share a number, and one with no label shows the
number alone. Write labels without numbers; they are added for you.

- `"animate": false` leaves the diagram still: no dot, and no numbers. Set it
  when somebody asks for no animation, or when the diagram shows
  what exists rather than something travelling through it: a topology, a set of
  components, anything with no direction of flow.
- `"numbered": false` keeps the animation and leaves the numbers off. Set it
  only when somebody asks for that.

Both go at the top level of either kind of spec, beside `name`.

## Paths

A diagram with a decision has more than one way through it: an order approved,
an order rejected. Each way is a **path**, a named route the animation plays,
which StepFlow lists in its Path tab and can highlight while the rest of the
diagram steps back. A path is not a line; never show one by colouring lines.

Leave `paths` out and the diagram has one path, in its own order. Give them,
at the top level beside `name`, and they are the whole of the animation, the
first the main one:

```json
"paths": [
  { "name": "Approved", "steps": [1, 2, 3, 4], "highlight": true },
  { "name": "Rejected", "steps": [1, 2, 5, 6], "highlight": true }
]
```

- `steps` is what the path fires, in order: positions in `edges` for a flow, or
  rows of `messages` for a sequence diagram (notes included in the count),
  counting from 1. With paths given, `step` is not read.
- Each path takes **one** branch at each decision (a `diamond`), and one
  section of an `alt` block. A path that takes both is refused: give each
  outcome its own path, all of them starting the same way and parting at the
  decision.
- `highlight: true` steps the rest of the diagram back while that path plays.
  Worth it for paths that share most of a diagram, as outcomes do.
- Changing a diagram you were given, keep each path's `id` as it came, also when
  you rename it or change its route; a new path has no `id`. A path left out is
  removed. To split one path into outcomes, change the existing path to one
  outcome, rename it after that outcome, and add the others.

## A sequence spec

```json
{
  "kind": "sequence",
  "name": "Order request",
  "participants": ["Client", "Orders API", "Order queue", "Fulfilment"],
  "messages": [
    { "from": "Client", "to": "Orders API", "label": "POST /orders" },
    { "from": "Orders API", "to": "Order queue", "label": "enqueue" },
    { "from": "Orders API", "to": "Client", "label": "202 Accepted", "reply": true },
    { "from": "Order queue", "to": "Fulfilment", "label": "consume" }
  ]
}
```

- `participants` go across the top in the order you list them. Put them in the
  order the work flows, so the arrows mostly point one way.
- `messages` happen in the order you list them, and that **is** the animation
  order and the numbering. There is nothing to ask for, unless the diagram has
  outcomes that are alternatives: then give it `paths` (see "Paths" above).
- `reply: true` draws it dashed, which is what a return is.
- `from` and `to` have to match a participant exactly, and must differ.
- Two messages given the same `"step"` fire together. Leave `step` off
  everywhere else; if you give it once, give it to every message.
- `"note": "right"` (or `"left"`, or `"over"` with `to` as the second
  participant) makes a row a note instead of a message, with `label` as its text.
- `"blocks": [{ "kind": "loop", "label": "Every minute", "start": 2, "end": 4 }]`
  draws a frame around `messages[2]` and `messages[3]`. `sections: [{ "start": 3,
  "label": "failed" }]` divides one, the way an `alt` has an `else`.
- Colours are CSS colours, and only worth setting when somebody asks for them.
  A participant can be an object instead of a name, `{ "name": "Orders API",
  "fill": "#fee2e2", "activation": "#fca5a5" }`: `fill`, `stroke` and
  `textColour` style its box across the top, and `activation` fills the narrow
  box on its lifeline. A message takes `stroke` for its line and `textColour`
  for its label; a note takes `fill`, `stroke` and `textColour`; a block takes
  `stroke` and `textColour`.

## A flow spec

```json
{
  "name": "Order fulfilment",
  "direction": "LR",
  "nodes": [
    { "id": "client", "label": "Client", "sub": "browser" },
    { "id": "orders-api", "label": "Orders API", "icon": "API Gateway" },
    { "id": "order-queue", "label": "Order queue", "icon": "SQS" },
    { "id": "fulfilment", "label": "Fulfilment", "icon": "Lambda" },
    { "id": "orders-db", "label": "Orders", "icon": "DynamoDB" }
  ],
  "edges": [
    { "from": "client", "to": "orders-api", "label": "POST /orders" },
    { "from": "orders-api", "to": "order-queue", "label": "enqueue" },
    { "from": "order-queue", "to": "fulfilment", "label": "consume" },
    { "from": "fulfilment", "to": "orders-db", "label": "INSERT" }
  ]
}
```

- `direction` is `LR` for a request on its way through, `TB` for a decision tree. `LR` reads
  better in a README, which is a wide, short space.
- `animate: false` leaves the diagram still, and `numbered: false` drops the
  step numbers: see "Animation and step numbers" above.
- `kind` is `rounded` by default. The others, and what each is for:

  | `kind` | Draws | Use it for |
  | --- | --- | --- |
  | `rect` | a plain box | a step, a component |
  | `ellipse` | an oval | a state, an event |
  | `diamond` | a diamond | a decision |
  | `triangle` | a triangle | a warning, a merge |
  | `cylinder` | a drum | a datastore |
  | `hexagon` | a stretched hexagon | a service |
  | `pill` | a box with round ends | the start and end of a flowchart |
  | `parallelogram` | a slanted box | input or output: "user enters details" |
  | `document` | a page with a wavy bottom | a document, report, invoice or form |
  | `subroutine` | a box with double sides | a process defined elsewhere |
  | `chevron` | an arrow-shaped block | a stage of a process: Lead, Trial, Paid |
  | `note` | a page with a folded corner | a comment on the diagram |
  | `cloud` | a cloud | the internet, or a third party with no icon |
- `sub` is a second, quieter line: a URL, a port, a qualifier.
- Edges animate in the order the graph is walked. Give an edge a `step` only to
  change that order; two edges with the same `step` fire together.

### Ids have to be stable and mean something

Write `orders-api`, never `n3`. Regenerating a diagram keeps the position of
every node whose id survives, so an id that changes between runs throws away
whatever anyone moved by hand. Name nodes after the thing they are.

### Frames: accounts, VPCs, subnets

A boundary drawn around a group. An AWS diagram without one is a set of boxes
with AWS logos on them, so reach for these whenever the answer involves an
account, a VPC, a subnet, a region, a cluster, a namespace, or any "this part is
inside that part".

```json
{
  "name": "Shopping order system",
  "direction": "LR",
  "nodes": [
    { "id": "shopper",      "label": "Shopper", "sub": "browser" },
    { "id": "api",          "label": "Public API",     "icon": "API Gateway" },
    { "id": "orders-svc",   "label": "Orders service", "icon": "Fargate" },
    { "id": "inventory-db", "label": "Inventory",      "icon": "Aurora" },
    { "id": "order-queue",  "label": "Order queue",    "icon": "SQS" }
  ],
  "edges": [
    { "from": "shopper", "to": "api", "label": "checkout" },
    { "from": "api", "to": "orders-svc", "label": "place order" },
    { "from": "orders-svc", "to": "inventory-db", "label": "reserve" },
    { "from": "orders-svc", "to": "order-queue", "label": "enqueue" }
  ],
  "frames": [
    { "id": "account", "label": "AWS account 4021-8837-1155", "group": "AWS Account",
      "contains": ["vpc", "api", "order-queue"] },
    { "id": "vpc", "label": "VPC 10.0.0.0/16", "group": "VPC",
      "contains": ["orders-svc", "inventory-db"] }
  ]
}
```

* **`group` draws it as the provider's own boundary**, with its badge on the
  corner and its colour and dash on the outline, instead of a plain grey box.
  Always set it on a cloud diagram: it is what makes the picture look like the
  ones the providers publish. The names to write:

  | `group` | Looks like |
  | --- | --- |
  | `"AWS Cloud"` | dark navy, dashed |
  | `"AWS Account"` | pink, dashed |
  | `"Region"` | teal, dashed |
  | `"VPC"` | purple, solid |
  | `"Public subnet"` | green, solid |
  | `"Private subnet"` | teal, solid |
  | `"Auto Scaling group"` | orange, dashed |
  | `"Corporate data center"` | grey, solid |
  | `"EC2 instance contents"` | orange, dashed |
  | `"Server contents"` | grey, dashed |
  | `"Spot Fleet"` | orange, dashed |
  | `"IoT Greengrass deployment"` | green, dashed |
  | `"Cluster"` | Kubernetes blue, solid |
  | `"Namespace"` | Kubernetes blue, dashed |
  | `"Node"` | grey, solid |

  A name nothing matches comes out as a plain outline and is reported. Leave
  `group` off for a boundary that is not a cloud container, such as one drawn
  round a few steps of a process to label them.
* **Nest by naming a frame inside another frame's `contains`.** The account holds
  the VPC; the VPC holds the service and the database. Any depth works.
* The grouping goes into the layout, so members of a frame are placed together
  rather than having a box drawn around wherever they landed.
* **Leave out what is not in there.** The shopper is a browser, so it belongs to
  no frame and is drawn outside the account. That distinction is most of what
  makes the diagram true.
* Frame ids follow the same rule as node ids: `vpc`, not `f1`.
* A thing can be in **one** frame only. Put it in the innermost one that holds
  it and let the nesting do the rest: something in the VPC is in the account
  already, by virtue of the VPC being.

A spec is refused, with the reason, if two frames claim the same thing, a frame
holds something that does not exist, or the nesting loops.

### Icons

Put in `icon` whatever the vendor calls the service: `"EC2"`, `"Lambda"`,
`"SQS"`, `"DynamoDB"`, `"API Gateway"`, `"Cloud Run"`, `"Storage Accounts"`.
It is matched against the short name, the official name and the id across the
AWS, Azure, Google Cloud and Kubernetes sets, so you do not need to know the
internal id. An id in the form `provider/id`, such as `aws/AmazonEC2`, is taken
exactly.

Three more sets sit beside the clouds:

- **Kubernetes**: write `"Kubernetes Pod"`, `"Kubernetes Deployment"`,
  `"Kubernetes Service"` and so on, with the word Kubernetes, since `"Service"`
  on its own means something in other sets too.
- **Everyday things**, for a business process rather than a system: `"User"`,
  `"Users"`, `"Mail"`, `"Credit card"`, `"Shopping cart"`, `"File text"`,
  `"Calendar"`, `"Circle check"`. They are line drawings that follow the theme.
- **Apps**, by brand name: `"Stripe"`, `"Shopify"`, `"HubSpot"`, `"Notion"`,
  `"GitHub"`, `"PostgreSQL"`, `"Redis"`. Some brands are not available, Slack,
  Salesforce and OpenAI among them; for those, use an everyday icon and name
  the app in the label.

A name nothing matches is reported and that node is drawn as a plain box. If
that happens, try the vendor's own wording: it is `"Storage Accounts"`, not
`"Blob Storage"`.

An `icon` node draws the artwork with its label underneath, so leave `kind` off
when you set one.
