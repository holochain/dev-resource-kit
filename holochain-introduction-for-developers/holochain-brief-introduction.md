# What is Holochain?

> Holochain is a _protocol_ and _framework_ for creating cryptographically secure, portable, and distributed applications. It provides the necessary tools to allow individuals to maintain control over their identity and data, while interacting safely with a network of untrusted peers.
>
> The purpose of this document is to help developers and technologists quickly understand:
>
> * Why we're building Holochain
> * What the project consists of
> * What advantages it has over server-based and blockchain-based architectures

## The problem

The Internet is decentralized by design. Many services, such as the web and email, have no central points. But in order to move through the Internet confidently, we need **trust.** How can you know I am who I say I am, and how can you be sure that I'm acting with integrity?

Centralized services act as proxies of trust. We may not be able to trust a million anonymous peers, but we can trust these big services to faithfully negotiate our interactions with those peers — **until we can't.** These systems are beginning to show weaknesses that are direct consequences of centralization.

* **They are more attractive to attackers.**
* **They leave users vulnerable to service outages.**
* **They concentrate power in the hands of a few.**
* **They burden the service operator with liabilities and costs.**

Blockchain opened our imaginations to the possibility of trustable, decentralized coordination with untrusted peers, but suffers from efficiency problems that are inherent to its design. The next wave of decentralized ledgers improve performance and resilience but carry over some of blockchain's assumptions. They hint at the full potential of this technology but stop before they reach the finish line.

## Holochain: fully sovereign, distributed, trustable coordination

Holochain gives you the tools to create applications that enable direct, trustable interactions among sovereign peers. It does this without needing the central coordination of a server or a group of miners. We are building:

* A **peer-to-peer protocol** that shares data among participants, giving them the power to act collectively as an 'immune system' that pushes out bad actors.
* An open-source **application development framework** that abstracts away complexity and allows you to focus on your core functionality.
* A set of **core apps and utilities**, including an identity manager, a private data vault, and a software distribution platform.
* A **distributed hosting ecosystem** that bridges Holochain to the web so people can use your apps as if they were traditional, server-based apps.

### Uncompromisable sovereignty

Each participant is in control of their own identity and data. Private data stays private, and public data is signed to prove ownership and authenticity.

### Intrinsic data integrity

Rather than ensuring trust by getting everyone to agree on a single global timeline, Holochain lets the data speak for itself. Every participant has a copy of an app's validation rules, which gives them the power to audit other participants' data.

### Resilience through interconnection

Everyone is part of a peer-to-peer network that replicates public data, audits it, and spreads warnings of bad actors. Third-party validators are chosen at random to thwart collusion.

### Circles of access

Every Holochain app enjoys its own private network. There is no global ledger. Groups can even fork an app for increased privacy. Individuals can choose to 'bridge' from one space to another to share data as needed.

### Consensus where it matters

Participants are equipped with rich information about their peers and come to their own conclusions about whom to trust. Because a coffee purchase shouldn't need the whole world's approval.

### Anti-fragility

Holochain networks work collectively to redistribute responsibility for data and traffic, which means that activity flows nimbly around outages, partitions, and attacks. Apps can even work offline.

### High performance, low cost

Distributed apps will only succeed if they're a pleasure to use. Because Holochain favors direct peer interaction over global consensus, applications are faster, more responsive, and more respectful of limited computing resources.

### Rapid application development

We're building a framework that handles the ins-and-outs of peer coordination so you can focus on building the things that matter to your users. Holochain is lightweight, batteries included, and available for your favorite operating system.

### Versatile organizational forms

Holochain doesn't impose any economic, governance, licensing, or business models on your project. You're free to use Holochain's tools to build software for a commons, a non-profit, a startup, or a huge corporation.

### Flexible interfacing

Your Holochain app is like a microservice on the user's device. Connect to it locally and securely via HTTP or WebSocket using your favorite GUI toolkit, an IoT device on the local network, or even a humble bash script.

### Peer-to-peer hosting

Holo Host offers a bridge from Holochain to the web, so you can deploy into the cloud and let a marketplace of distributed hosts serve your app as if it were a traditional web app.
