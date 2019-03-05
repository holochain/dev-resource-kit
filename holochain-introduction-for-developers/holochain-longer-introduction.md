# Holochain Essentials for Developers

> Holochain is a _protocol_ and _framework_ for creating cryptographically secure, cross-platform, distributed applications. It provides the necessary tools to allow individuals to maintain control over their identity and data, while interacting safely with a network of untrusted peers.
>
> The intention of this document is to give developers and technologists a brief overview of:
>
> * Why we're building Holochain
> * How it works
> * What core building blocks are available and how they shape your architecture
> * How our developer tooling works

## The problem

The Internet is, by design, a decentralized network. It was made to keep working through partial disruptions. This feature has allowed it to become the world's most robust communication network. But for all its strengths, it lacks a built-in way to ensure **trust**. It's hard to assess the honesty of billions of anonymous peers, most of whom aren't even human. Not only that, but it would be hard for everyone to maintain all of the services they need on their own devices.

So we turned to centralized authorities to act as proxies of trust and stewards of critical services. Domain registrars and certificate authorities manage the distribution of names and encryption keys. Search engines take responsibility for cataloguing the Internet in a useful way. Corporations build online platforms that host our online identities and data. We depend on these services to talk to our friends and family, collaborate with our colleagues, buy products and services from strangers, and share news about the world.

**But centralized systems are vulnerable.** If you're a user, you could have your data stolen by attackers, lose an entire morning of productivity due to service outages, or find yourself at the mercy of a corporation whose goals are not in your best interest. If you're a service operator, you carry a heavy responsibility to keep your users' data secure yet accessible, and to make ethical choices on their behalf. This responsibility carries high costs.

## A solution, and a few more problems

What would happen if we removed the center completely, pushing data storage and decision-making power to individuals? This is a good first question. But if it's hard to trust one central service, how much harder is it to trust a cloud of anonymous peers?

Blockchain was the first serious answer to this problem. By getting everyone to agree on one shared history of all events, and by making it very expensive to corrupt that history for everyone, it allowed anonymous agents to trust each other --- or rather, to trust the algorithm. This is a clever solution, and it works.

But that robustness comes at a cost. Global consensus demands a lot of resources, which translates to real financial and ecological costs, and eventually leads to new power concentrations. Partial solutions have been offered, but in the end, blockchain's very design prevents it from being a truly scalable, open, high-performance infrastructure for a new decentralized Internet.

## Enter agent-centric computing

We asked if it was possible to create a system that gives individuals complete control over their identity, data, and actions, while still being able to verify the trustworthiness of the actions and data of others. This required us to go back to the drawing board and throwing out assumptions about the need for central authorities or shared ledgers.

We came up with the Holochain protocol. It answers this question with two components: **intrinsic data integrity** and **peer validation**.

### Intrinsic data integrity

The first technological foundation for enabling people to trust data without needing to trust its creator is to _make the data itself carry guarantees_.

Three basic elements --- **shared rules**, **tamper-evident journals**, and **cryptographically signed proof of authorship** --- mean that anyone can make reasonable determinations about whether data is valid without relying on a central authority. Here's how it works:

1. First, we need a set of rules for our application that everyone agrees to follow. These rules govern what should be considered valid or invalid data. They're written in executable code, and you can think of them as the 'validation logic' of your application. We call a collection of rules a **DNA**, because like cells in a body, each participant has a copy.
2. When you want to communicate with others using these shared rules, you must first identify yourself. We use _public key cryptography_ to create a unique identity. Your _private key_ always stays on your machine and acts like a password. Your _public key_ is shared with others and acts like your user ID. You 'sign' all your data with your private key, and because of the way public key cryptography works, anyone with a copy of your public key can verify that your signature is correct.
3. As you create information, you publish it to your own journal of actions. This journal is stored in a _hash chain_, which is a data structure made of individual pieces of data in time sequence. Each piece of data has a header, which contains the data's hash, the hash of the previous header, and your signature as proof of authorship. (If you've ever used Git or read about Bitcoin, you'll know how hash chains work. You can't change old entries; you can only add new entries on top.) We call this structure your **source chain**. [picture of hash chain structure; maybe link to interactive demo] The first element of your source chain is always the hash of the DNA, as a proof that you've seen the rules and agree to abide by them. The second element contains your public key, which advertises your identity.
4. When you and I want to interact, we can ask for other's source chains, then audit them for invalid data. Because you have a copy of the rules, you don't need to trust whether I'm following them --- you can check for yourself.

Most of these checks --- correctness of DNA hash, presence of agent ID, integrity of source chain, and validity of signatures --- are performed at the 'subconscious' level by Holochain itself. On top of that, your DNA implements rules that are specific to your application.

But intrinsic data integrity isn't enough by itself; we still have three problems.

* How do you know that I haven't rewound my journal to a previous point? Depending on the app, I could hide a large debit transaction, or rescind my vote on a controversial issue. My source chain would pass all the rules, but what I shared with you would be different from what I've shared with others.
* My data might be a record of interactions I've made with others. Hence, the validity of my source chain might be dependent on the source chains of people I've interacted with. What if those people are offline when you need to audit them?
* Even if those people are online, my interactions with them are dependent on their interactions with others, and so forth. It could be quite expensive to traverse this tree of histories.

### The validating DHT: a shared, permanent memory for networks of peers

Holochain tackles these problems by keeping a _shared memory_ of everyone's public data. This memory lives in the devices of all the network's participants. Like a hologram which can be cut into smaller pieces and still retain a portion of the whole in each piece, every participant is responsible for storing a small amount of their peers' data. This shared memory does three things:

* It remembers participants' source chains, which acts as evidence against them if they try to rewind their history and make up a new one.
* It keeps multiple copies of the data of participants who are currently offline, for redundancy and availability.
* It acts as a third-party auditor of all public data, which speeds up auditing and allows the network to identify and blacklist bad actors.

This data, spread across all peers, forms a **distributed hash table (DHT)**. A DHT is just a key/value database distributed across many machines. Our DHT is a [_content-addressable store_](https://en.wikipedia.org/wiki/Content-addressable_storage), which means that the key is the hash of the value.

Many projects use DHTs. But in Holochain's DHT, every peer uses their copy of the rules to _validate_ the data they receive before they store it. If it passes, they produce a certificate of validity. If it fails, they produce a warrant against the data's author. Peers can then use this warrant as justification for ignoring the bad actor, effectively pushing them out of the network. That's why we call it a **validating DHT**.

Just as with a source chain, no data in a Holochain DHT can be removed or modified, for a couple reasons:

* It would break participants' ability to detect source chain modifications or revalidate old data.
* It makes it much easier to synchronize data between nodes.

In Holochain's DHT, peers and data share the same address space. Peers are responsible for any piece of data that falls into their 'neighborhood' --- that is, any data whose address is close to its own, within a certain range. It's impossible to predict what the hash of any piece of data will be, and it's impossible to control who your neighbors are. This means that a group of peers can't collude to create a 'bad neighborhood' in which they receive and validate each other's invalid data.

This is how we ensure that every bit of data is validated by a truly random collection of peers. And even one valid warrant from one honest peer is enough to alert everyone. We call it 'good enough' security --- statistically approaching 100% security for a fraction of the work of blockchain.

You can specify the number of peers who must validate and store each piece of data in your app's DHT. We call this the **resilience factor**, and apps with higher security needs may want a higher resilience factor. Neighbors in the DHT regularly reassign responsibility for data among each other as nodes join and leave, making sure the resilience factor is always met.

#### What can you store on a DHT?

Source chain entries can be private or public. Those marked public are shared into the DHT. Entries can contain anything that can be expressed as a UTF-8 string, including JSON, HTML, or Base64-encoded binary data.

#### How do you relate data to other data?

As mentioned, the key or address of a piece of data is its hash. This creates a tricky problem: you can only get a piece of data if you know its hash, and there are only two ways to get that hash: you can derive it from the content (which means you already have the content), or you can find it somewhere. So the design of a Holochain application's database is all about making it possible for agents to find the hashes of the content they need.

To this end, we have **links**. A link is a piece of metadata attached to a DHT entry, which points to the hash of another DHT entry. You can link anything that lives on the DHT: data entries, agent IDs, and even the app's DNA hash. This lets you to construct a **graph database** connecting _things that people know about_ to _things they want to find_. For instance: [graph showing blog scenario]

#### Gossip: how information gets around

Every node in a Holochain network holds a list of neighbors, and keeps track of where they fit into the network as a whole. They communicate with neighbors and others using an encrypted **gossip protocol**, which is just what it sounds like: I hear news from one of my peers, then I share it with all my other peers. At first, news travels slowly, but in a few moments, a power law take over and everyone knows about it. The network gossips about:

* Recently joined or departed nodes
* New DHT entries
* Certificates of validity or warrants against bad data
* The relative health of a neighborhood --- responsibility for sections of data, gaps in resilience, etc.

#### Node-to-node messaging

Not every communication between participants needs to be recorded forever in the DHT. For temporary and private messaging, nodes can send data directly to each other.

#### A private space for each network

Holochain does not have a single 'main chain' or DHT. Every DNA has its own private network and DHT, and each peer is responsible for their own chain. Applications can't access data that doesn't pertain to them, which also means they don't have to bear the burden of storing irrelevant data. You can also 'fork' an existing network into a private one by by changing a trivial piece of information in its DNA, such as the name or GUID. This keeps all the DNA's rules, but causes the DNA to be hashed to a different value. Those without the correct DNA hash won't be able to find your network.

### Building your distributed application

#### User identities

Holochain gives each user full control over their identity. As mentioned, each agent in the system (whether human or bot) is represented by a public key, whose private counterpart stays on the agent's device. There is no need to manage and protect passwords in a central database. An app called DeepKey manages continuity of identity across devices and allows users to retire keys belonging to lost or stolen devices. Another app called Vault allows participants to store their own personally-identifying information and share it with other apps at their discretion.

Each agent's data also remains under their control: only they can write to their source chain, because only they have the private key which can sign their entries. The canonical copy of their source chain stays on their device. No data marked private is ever shared with the DHT.

User _privileges_, on the other hand (such as being allowed to join a network or publish/edit/delete public entries on the DHT) are enforced by **validation rules**, which we'll talk about soon.

#### Application data types, API, and validation rules

DNA should contain a small set of functionality that implements only the core domain and storage logic necessary for participants to trust each other's data. Each DNA should have a tightly scoped domain of responsibility, like a microservice. Many DNAs then combine to provide functionality for a full-featured application. Most logic should live outside of DNA in the client-side portion of your app. Clients are completely decoupled from DNA, which means that users have choice when it comes to interacting with any Holochain application. It also means that you can 'remix' existing DNAs into your own application, including a number of generic DNA templates that we're building for commonly needed functionality.

The framework comes with a **Holochain development kit (HDK)** that makes it easy for you to build your DNA. Currently we have a Rust HDK, but we expect language support to grow as adoption increases. With the HDK, you can define:

* **core data types** for source chain entries, whether they're public, and the ways they can link to each other
* **genesis and migration functions**, which set up a source chain or migrate old data to a new DNA's rules
* an application-specific API made of **zome functions**, which can be called by a front end, other functions in your DNA, or even other DNAs
* **validation callbacks**, which are called when nodes commit entries and links or receive them for third-party validation
* **receive callbacks**, which respond to private messages sent from other nodes

Holochain's core takes care of connecting all these pieces together and managing the flow of activity in your own running instance and across the network.

Within your DNA's functions, you can access the Holochain core API to:

* **commit entries and links to a source chain**, which are also sent to the DHT if they're marked as public
* **query a source chain** for existing entries
* **get entries and links from the DHT**
* **send private messages** to other nodes
* **call zome functions** in other locally running DNA instances
* **sign data and check signatures**
* **output debugging messages**

#### Bridging between DNA instances and controlling access

We envision developers taking advantage of a large ecosystem of small, versatile, single-purpose DNAs, combining them together to create complex applications. This means that applications will need a way to bridge DNAs to each other, so they can access each other's functionality. This can be done on the front end, but Holochain also gives you a built-in bridging feature so DNA instances can talk directly to each other.

Access to a running DNA instance's zome functions is controlled by capability tokens, which gives users the power to grant or deny access as they like.

#### Scaffolding and testing your DNA code

Holochain comes with a development command called `hc`, which lets you quickly scaffold DNA modules (called 'zomes') and their boilerplate code, including test stubs. Tests are run through Node.JS, so you can use whatever JavaScript-based testing framework you prefer.

#### Creating a user interface

Once you've written your DNA, it needs a way of responding to the actions of a participant. Because every participant runs the DNA on their own machine, they must be able to interact with it locally. Holochain exposes a local JSON-RPC interface over WebSocket and HTTP, which gives all your DNA's zome functions. This lets users talk to their running DNA instance through a wide array of client applications, such as a GUI, a system service, a server-side web application, or even a shell script.

Holochain also comes with a small web server which serves static files to local clients. You can use this to bundle up a default web-based GUI with your DNAs.

#### Managing Holochain

The standard Holochain executable is called the 'Conductor'. It is responsible for much of the functionality we've already described:

* generating new identity keypairs for the user
* managing all the user's running DNA instances and web-based GUI bundles
* installing new DNA and web-based GUI bundles
* keeping track of capability tokens and controlling access to zome functions via JSON-RPC or bridges
* responding to JSON-RPC calls from front ends
* storing source chains and DHT data
* communicating with other nodes via gossip

Your front ends can also access some of these features through a separate JSON-RPC interface called the 'admin interface'. They can act on the user's behalf to download and install DNA and UI bundles, fork new instances of existing DNA, and generate new identities.

### Delivering your application to the world

#### As a traditional desktop or mobile app

The most basic way to make your app available to users is to bundle the Holochain runtime, the necessary DNA instances, and an appropriate GUI into an installer, and make it available for download on your website, popular download sites, and official app stores. Holochain runs on Windows, macOS, and Linux, and we're working on Android support too.

#### With Holochain

We're building an app store app and a DNA/GUI delivery app. You can expect these core services to be available on every installed Holochain instance, so you can deploy new and updated DNA and GUI bundles to any participating user.

#### With Holo Host

We recognize that not everyone will be running Holochain, especially at the beginning. So we're building a distributed hosting marketplace called Holo Host. You mark your app as Holo-ready, submit it to the app store, and allow a cloud of machines to host DNA instances and serve UI bundles to end-users over the traditional web. Users are still in control of their identities and source chains, but hosts do the heavy work of data storage, asset delivery, and DHT communication.

#### As a hybrid app

If your app has storage and performance needs that make it inappropriate for users to install Holochain on their machines --- a mobile video sharing app, for instance --- consider deploying your DNAs on Holo Host without a web-based UI bundle. Currently our Holo 'light client' library is written in JavaScript, which would make it appropriate for webview or Cordova-based applications.

### Things you'll need to keep in mind

Blockchain was a totally new idea, with its own set of advantages and pitfalls for developers. Holochain is different again from client/server and blockchain, so there are some things you'll need to chew on as you start programming in this new paradigm:

* At the core of Holochain's philosophy is the relativity of all data. Each piece of information is reported from the perspective of one agent, there is no global clock, and no one agent has the power to coerce others into accepting its own view. Shared reality is built up from overlapping perspectives that agree with each other.
* Holochain's DHT is an 'eventually consistent' database, which means that propagation delays will cause different agents to have temporarily incomplete views of shared reality. How do your validation functions accommodate this? Can data 'peg' itself to other, potentially stale data while the network is syncing?
* With no central database, there is no single source of truth -- no global timestamp, no shared timeline. What does this mean for things like username conflicts, voting deadlines, and 'canonical' copies of a document? Can data comfortably diverge, or will you need to design ways to resolve conflicts? Designated authorities are one solution, but we also encourage you to think about collective governance -- mediated conflict resolution, peer review processes, or resource auctions.
* How does the election of random validators affect your users' privacy? Given that anyone can be holding a copy of their public data, are there data mining or privacy concerns? Holochain doesn't solve the problem of unauthorised use, but it can give participants the tools they need to prove that a certain use of a piece of data was done in breach of agreement.
* Holochain is designed for facilitating networks of trust rather than anonymous, trustless interactions. For secrecy-sensitive applications, how might your users' identity be revealed by their persistent public history? If users are allowed to spin up new identities for privacy's sake, what sorts of social or technical Sybil attacks might this allow?
* The integrity of your app stands and falls on the correctness of your validation functions. This is why we encourage writing your DNA in a disciplined language like Rust, which offers compile-time guarantees of type safety. But you also need to think about the consequences of every line of code in the context of a distributed, peer-to-peer network.



## Pithy closing statement or call-to-action

---
---
---
---
---

# Missing subjects

* Onboarding process and admin GUI -- what happens when a user installs Holochain for the first time. Related to DPKI.
* Brief mention of eventual consistency, security considerations, threat models, etc -- all the weird stuff that doesn't matter until you go distributed.
