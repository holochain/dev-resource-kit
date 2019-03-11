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

The Internet is, by design, a decentralized network. This has allowed it to become the world's most robust communication network. But for all its strengths, it lacks a built-in way to ensure **trust**. It's hard to assess the honesty of billions of anonymous peers, most of whom aren't even human.

So we turned to centralized authorities to act as proxies of trust and stewards of critical services. Domain registrars and certificate authorities manage the distribution of names and encryption keys. Search engines take responsibility for cataloguing the Internet in a useful way. Corporations build online platforms that host our online identities and data. We depend on these services to talk to our friends and family, collaborate with our colleagues, buy products and services from strangers, and share news about the world.

**But centralized systems are vulnerable.** If you're a user, you could have your data stolen by attackers, lose an entire morning of productivity due to service outages, or find yourself at the mercy of a corporation whose goals are not in your best interest. If you're a service operator, you carry a heavy responsibility to keep your users' data secure yet accessible, and to make ethical choices on their behalf. This responsibility comes with high costs.

## A solution, and a few more problems

What would happen if we removed the center completely, pushing data storage and decision-making power to the edges? This is a good first question. But if it's hard to trust one central service, how much harder is it to trust a cloud of anonymous peers?

Blockchain was the first serious answer to this problem. By creating a single, shared history of all events, and by making it very expensive to corrupt that history, it allowed anonymous agents to trust each other --- or rather, to trust the design. This is a clever solution, and it works.

But that robustness comes at a cost. Global consensus demands a lot of resources, which translates to real financial and ecological costs, and eventually leads to new power concentrations. New solutions are being offered, but they often carry over assumptions from blockchain. This hampers their ability to deliver the full potential of decentralized technology.

## Enter agent-centric computing

We asked if it was possible to create a system that gives individuals complete control over their identity, data, and actions, yet assure themselves of the integrity of others' data. Our answer to that question looks a lot different from blockchain. We call it the Holochain protocol. It stands on two pillars: **intrinsic data integrity** and **peer validation**.

### Intrinsic data integrity

The first technological foundation for enabling people to trust data in a trustless system is to _make the data carry its own integrity guarantees_.

Three basic elements --- **shared rules**, **tamper-evident journals**, and **cryptographically signed proof of authorship** --- mean that anyone can make reasonable determinations about whether data is valid without relying on a central authority. Here's how it works:

1. First, we need a set of rules for our application that everyone agrees to follow. These rules govern what should be considered valid or invalid data. They're written in executable code, and you can think of them as the validation logic of your application. We call them **DNA**, because each participant has an identical copy, like cells in a body.
2. When you want to interact with others using these shared rules, you must first identify yourself. We use _public key cryptography_ to create a unique identity. Your _private key_ acts as a wax seal, proving your authorship every piece of data you create. Your _public key_ is shared with others and allows them to verify the authenticity of your seal. You can think of your private key as a password, giving you access to your account, and your public key as a user ID, identifying you to others.
3. As you create information, you publish it to your own journal. This journal is stored in a _hash chain_, which is an append-only, tamper-evident sequence of data entries. We call this journal your **source chain**. [picture of hash chain structure; maybe link to interactive demo] The first element of your source chain is always the hash of the DNA, as a proof that you've seen the rules and agree to abide by them. The second element contains your public key, which advertises your identity.
4. When you and I want to interact, we can ask for each other's source chains, then audit them for validity. Because you have a copy of the rules, you don't need to trust whether I'm following them --- you can check for yourself.

Most checks are performed at the 'subconscious' level by Holochain itself. On top of that, your DNA implements rules that are specific to your application.

But intrinsic data integrity isn't enough by itself! We still have three problems.

* How do you know that I haven't rolled my source chain back to a previous point and written a new history on top? It would pass all the rules, but it would be an 'alternate history'.
* If my data includes records of my interactions with others, its validity depends on the validity of their data. What if those people are offline when you need to audit them?
* Even if those people are online, their interactions with me depend on their interactions with others, and so forth. It could be quite time-consuming to traverse this tree of histories.

### The validating DHT: a shared, permanent memory for networks of peers

Holochain tackles these problems by keeping a _shared memory_ of everyone's public data. This memory lives in the devices of all the network's participants. Like a hologram which can be cut into smaller pieces and still retain a portion of the whole in each piece, every participant is responsible for storing a small amount of their peers' data. Just as with a source chain, no data in this shared memory can be removed or modified. This shared memory does three things:

* It remembers participants' source chains entries, which safeguards against history modification.
* It keeps multiple copies of all public data, for redundancy and availability.
* It acts as a collective third-party auditor of all public data.

This data, spread across all peers, forms a **distributed hash table (DHT)**. A DHT is just a key/value store spread across many machines. Our DHT is a [_content-addressable store_](https://en.wikipedia.org/wiki/Content-addressable_storage), which means that the key is the hash of the value.

Many projects use DHTs. But in Holochain's DHT, every peer uses their copy of the rules to _validate_ the data they receive before they store it. They can then share their results with others as a confirmation of validity or a warrant of invalidity. That's why we call it a **validating DHT**.

In Holochain's DHT, peers and data share the same address space. Peers are responsible for any piece of data that falls into their 'neighborhood' --- that is, any data whose address is close to its own. It's very difficult to both predict the hash of a piece of data or control who your neighbors are. This means that a group of peers can't collude to create a 'bad neighborhood' in which they receive and validate each other's invalid data.

This is how we ensure that every bit of data is validated by a truly random collection of peers. And even one warrant from one honest peer is enough proof to sound the alarm on a bad actor. We call it 'good enough' security --- statistically approaching 100% trust for a fraction of the work of blockchain.

You can specify the number of peers who must validate and store each piece of data in your app's DHT. We call this the **resilience factor**, and apps with higher security needs may want a higher resilience factor. Neighbors in the DHT regularly reassign responsibility for data among each other as nodes join and leave, making sure the resilience factor is always met.

#### What can you store on a DHT?

Source chain entries can be private or public. Those marked public are shared into the DHT. Entries can contain anything that can be expressed as a UTF-8 string, including JSON, HTML, or Base64-encoded binary data.

#### How do you relate data to other data?

As mentioned, the key or address of a piece of data is its hash. This creates a tricky problem: you can only get a piece of data if you know its hash, and there are only two ways to get that hash: you can derive it from the content (which means you already have the content), or someone can give it to you.

To this end, we have **links**. A link is a piece of metadata attached to a DHT entry, which points to the hash of another DHT entry. You can link anything that lives on the DHT: data entries, agent IDs, and even the app's DNA hash. This lets you construct a **graph database** connecting _things that people know about_ to _things they want to find_. For instance: [graph showing blog scenario]

#### Gossip: how information gets around

Every node in a Holochain network holds a list of neighbors. They communicate using an encrypted **gossip protocol**, which is just what it sounds like: I hear news from one of my peers, then I share it with all my other peers. At first, news travels slowly, but in a few moments, a power law take over and everyone knows about it. The network gossips about:

* Recently joined or departed nodes
* New DHT entries
* Certificates of validity or warrants of invalidity on DHT entries
* The relative health of a neighborhood and its data

#### Node-to-node messaging

Not every communication between participants needs to be recorded forever in the DHT. For temporary or private data, nodes can talk directly to each other.

#### A private space for each network

Holochain does not have a single 'main chain' or DHT. Not only is each peer responsible for their own source chains, but every DNA has its own private network and DHT. This safeguards group privacy, but it also reduces the storage burden of each app. You can also 'fork' an existing DNA, keeping the rules but creating a new, private space.

### Building your distributed application

#### User identities

Holochain gives each user full control over their identity. Because each participant holds their own private keys on their own device, there is no need to manage and protect passwords in a central database. An app called **DeepKey** manages continuity of identity across devices and allows users to retire keys belonging to lost or stolen devices. Another app called Vault allows participants to store their own personally-identifying information and share it with other apps at their discretion.

Each participant's data remains under their control: only they can write to their source chain, because only they have the private key which can sign their entries. The canonical copy of their source chain stays on their device. No data marked private is ever shared with the DHT.

User _privileges_, on the other hand (such as being allowed to join a network or publish/edit/delete public entries on the DHT) are enforced by **validation rules**, which we'll talk about soon.

#### Application data types, API, and validation rules

DNA should contain a small set of functionality that implements only the core logic necessary for participants to trust each other's data. Each DNA should have a tightly scoped domain of responsibility, like a microservice. Many DNAs then combine to create a full-featured application. Most logic can live outside of DNA in the front-end portion of your app. Front-ends are completely decoupled from DNA, which encourages user choice and 'remixing' of existing DNAs into new applications. We're also building a library of DNAs that we think will be useful for many apps.

The framework comes with a **Holochain development kit (HDK)** that makes it easy for you to build your DNA. Currently we have a Rust HDK, but we expect language support to grow as adoption increases. With the HDK, you define:

* **core data and link types** for your application
* **validation functions** for each of your data types
* an API for your application, made of **'zome' functions** ('zome' is short for 'chromosome')
* **genesis functions**, which bootstrap a participant's source chain
* **bridging functions**, which establish connections to other DNA instances running on a participant's device
* **migration functions**, which handle forks and updates to new DNAs

Holochain's core takes care of connecting all these pieces together and managing the flow of activity in a participant's own running instance and across the network.

Within your DNA's functions, you can access the Holochain core API to:

* **commit entries and links** to a source chain
* **query a source chain**
* **get entries and links** from the DHT
* **send and receive private messages** between nodes
* **call zome functions** in other running DNA instances via bridging
* **manually sign data and check signatures**
* **output debugging messages**

#### Bridging between DNA instances and controlling access

We envision developers taking advantage of a large ecosystem of small, versatile, single-purpose DNAs, combining them together to create complex applications. This means that applications will need a way to bridge DNAs to each other, so they can access each other's functionality. The front end can talk to multiple DNA instances, but Holochain also gives you a built-in bridging feature so DNA instances can talk directly to each other.

Access to a running DNA instance's zome functions is controlled by [capability tokens](https://en.wikipedia.org/wiki/Capability-based_security), which gives users the power to grant or deny access as they like.

#### Scaffolding and testing your DNA code

Holochain comes with a development command called `hc`, which lets you quickly scaffold DNA modules (called 'zomes') and their boilerplate code, including test stubs. Tests are run through Node.JS, so you can use whatever JavaScript-based testing framework you prefer.

#### Creating a user interface

Once you've written your DNA, it needs a way of responding to the actions of a user. Because everyone runs the DNA on their own machine, they must be able to interact with it locally. Holochain exposes a local JSON-RPC interface over WebSocket and HTTP, which allows front-ends to access your DNA's zome functions. This lets users talk to their running DNA instance through a wide array of client applications, such as a GUI, a system service, a server-side web application, or even a shell script.

Holochain also comes with a small web server which serves static files to local clients. You can use this to bundle up a web-based GUI with your DNAs.

#### Managing Holochain

The standard Holochain executable is called the 'Conductor'. It is responsible for much of the functionality we've already described:

* generating new identity keypairs for the user
* installing, hosting, and uninstalling DNA and web-based GUI bundles
* keeping track of capability tokens and controlling access to zome functions
* responding to JSON-RPC calls from client applications
* storing source chains and DHT data
* gossiping with other nodes

Your front ends can also access some of these features through a separate JSON-RPC interface called the 'admin interface'. They can act on the user's behalf to download and install DNA and UI bundles, fork new instances of existing DNA, and generate new identities.

### Delivering your application to the world

#### As a traditional desktop or mobile app

The most basic way to make your app available to users is to bundle the Holochain runtime, the necessary DNA instances, and a GUI into an installer. You can then make it available for download via the usual channels (app stores, package managers, GitHub, etc). Holochain runs on Windows, macOS, and Linux, and we're working on Android support too.

#### Through Holochain

We're building an app store app and a DNA/GUI delivery app. You can expect these core services to be available on every installed Holochain instance, so you can deploy new and updated DNA and GUI bundles to any participating user.

#### Through Holo Hosting

We recognize that not everyone will be running Holochain, especially at the beginning. So we're building a distributed hosting marketplace called Holo Host. You mark your app as Holo-ready, submit it to the app store, and allow a cloud of machines to host DNA instances and serve UI bundles to end-users over the traditional web. Users are still in control of their identities and source chains, but hosts do the heavy work of data storage, UI asset delivery, and DHT communication.

#### As a hybrid app

If your app works best as an installable, but your users' devices have tight resource constraints --- a video sharing app on mobile devices, for instance --- consider deploying your DNAs on Holo Hosting. Currently our Holo 'light client' library is written in JavaScript, which means you could integrate it into webview, [Electron](https://electronjs.org/), or [Cordova](https://cordova.apache.org/) applications.

### Things you'll need to keep in mind

Blockchain was a totally new idea, with its own set of advantages and pitfalls for developers. Holochain is different again from client/server and blockchain, so there are some things you'll need to chew on as you start programming in this new paradigm.

#### Agent-centricity and data relativity

* At the core of Holochain's philosophy is the relativity of all data. Each piece of information is reported from the perspective of one agent, there is no global clock, and no one agent has the power to coerce others into accepting its own view. Shared reality is built up from overlapping perspectives that agree with each other.
* Attempting to enforce global consensus on Holochain is an anti-pattern. If you're porting from blockchain, can you remodel your application in a way that removes this need?
* Holochain's DHT is an 'eventually consistent' database, which means that propagation delays will cause different agents to have temporarily incomplete views of shared reality. How can your validation functions accommodate this? Can data 'peg' itself to other, potentially stale data while the network is still receiving new data?
* With no central database, there is no single source of truth -- no global timestamp, no universal timeline. What does this mean for things like username conflicts, voting deadlines, and 'canonical' copies of a document? Can data comfortably diverge, or will you need to design ways to resolve conflicts? Are designated authorities or simple rules such as 'last writer wins' appropriate, or do you need to consider strategies such as mediation, voting, or auctions?

#### Privacy, authority, and autonomy

* How does the public nature of DHT data affect your users' privacy within a given network? Are there data mining or privacy concerns? Holochain doesn't prevent unauthorized use of data, but it gives you tools for controlling data visibility and holding recipients accountable for their usage of shared data.
* Holochain is designed for facilitating networks of trust rather than anonymous, trustless interactions. For secrecy-sensitive applications, how might your users' identity be revealed by their persistent public history? If users are allowed to spin up many anonymous identities for privacy's sake, what sorts of social or technical Sybil attacks might this allow?

#### Data integrity and validation

* The integrity of your app stands and falls on the correctness of your validation functions. This is why we encourage writing your DNA in a disciplined language like Rust, which offers compile-time guarantees of type safety. But you also need to think about the consequences of every algorithm in the context of a distributed, peer-to-peer network.
* DNAs are tightly sandboxed, and have no direct access to the host device's network or storage. This means all data comes from client applications. How can you provide proofs of authenticity such that data can be inherently trusted?
* Holochain does a lot of validation work for you at the 'subconscious' level. Leverage the DHT to speed up validation time. That is, if the validity of a new entry depends on the validity of a previous entry, that previous entry should already be considered valid if the DHT holds no warrants against it.
* Conversely, partitioned networks can take advantage of this validation technique to 'launder' bad transactions behind good ones. Consider defining heuristics that will cause a validator to perform a 'deep validation', such as the company a user keeps.
* Validation functions should deterministic: write them so that, given the same input data, they should always produce the same result. Its validity shouldn't depend on the current state of the DHT. To serve this, data should include or reference the data it depends on.
