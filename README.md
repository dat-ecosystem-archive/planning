
# Dat Planning

This repository will help us organize and communicate planned features for the Dat ecosystem. Through the [Dat working groups](https://github.com/datproject/governance), decisions around the protocol and project are coordinated.

Dat is used by various organizations which loosely coordinate through IRC and the working groups. However, because the ecosystem is still young we still need to work together on adding new features and making sure we move forward together. This repository is a place to help communicate plans for projects using Dat and planned features. Because of the nature of the projects, these documents are not meant to be firm roadmaps but more so a starting place for individual projects, managed by separate teams, to create and align those roadmaps (e.g. the Dat CLI, Beaker Browser, Dat Desktop, etc.).

---

### Protocol performance and features

In April of 2017, we shipped a new version of Dat and hyperdrive. Since then, we've worked with users to identify performance issues and other needed features. Through this feedback, the community identified three priorities for improvement:

*   Performance on many small files
*   Ability for multiple people to collaborate on a single Dat
*   Networking improvements for improved peer-to-peer connectivity and security.

These features are detailed below. Please note that the terminology and specifics are not settled, and we more than welcome any feedback or questions.

**Performance Improvements**

We have multiple improvements underway for the core Dat protocol. They stem from a new internal data structure which integrates a ["Trie"](https://en.wikipedia.org/wiki/Trie) to improve lookup performance and a [CRDT](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) to support multiple writers. The improvements will be deployed incrementally in order to simplify the process and give us time to test changes.

The first improvement will be performance. We will deploy the new trie structure and add code for handling the multiple data-structure versions on the Dat network. Updates to the applications should be largely invisible to the users – though the apps will go faster!

**Multiple Writers**

In adding support for multiple writings, we face both technical and design hurdles. As git has shown, the design needs for managing conflicts and branching of data can be as complex as the technical requirements. To simplify this process, we've identified a few steps we can take to incrementally work on this

The second improvement will be a new feature called "one-way mounts." Mounts merge dats together by causing one dat to act as a folder within another dat. A "one-way" mount sets the folder as a read-only subfolder of a parent. Mounts may also have their versions pinned, and so are a good way to handle upstream dependencies (as in the case of code dependencies). Finally, mounts give the Dat networking stack a way to multiplex data transfer over existing connections, reducing the number of DHT lookups necessary. (For instance, a dat with 5 mounts will only require one DHT lookup for all 6 dats.)

The third and final improvement will be "union mounts." Like one-way mounts, the union mount sets a dat as a subfolder of another dat. However, union mounts are able to overlap each other. This is accomplished using a CRDT. As a result, union mounts can be used to create multiwriter dats (dats with multiple authors).



1.  **Performance improvements**
    1.  [ ] New trie data-structure
        1.  [x] Integrate Hypertrie into Hyperdrive [issue](https://github.com/mafintosh/hyperdrive/pull/233)
        1.  [ ] Add header-detection and incompatibility handling
        1.  [ ] Backwards compatibility (correctly use the loaded version's code)
        1.  [ ] Update the protocol specs
    1.  Implement in applications and downstream modules
        1.  [ ] Dat CLI (TBD)
        1.  [ ] Dat SDK module & deprecate dat-node (RangerMauve)
        1.  [ ] Beaker browser (Paul)
1.  **"One-way" mounts**
    1.  [x] Data structure updates
        1.  [x] Implement one-way mounts in Hyperdrive [issue](https://github.com/mafintosh/hyperdrive/pull/233)
        1.  [x] Update the protocol specs
    1.  [ ] Implement in applications and downstream modules
        1.  [ ] Dat CLI
        1.  [ ] Beaker browser
1.  **"Union" mounts (Multiwriter)**
    1.  [ ] Data structure updates
        1.  [ ] Implement union mounts in Hyperdrive
        1.  [ ] Update the protocol specs
    1.  Implement in applications and downstream modules
        1.  [ ] Dat CLI
        1.  [ ] Beaker browser
1. **Community**
    1.  [ ] Dat 1 to Dat 2 Upgrade guide
    1.  [ ] Replace dat-node with Dat SDK in docs.datproject.org

### Networking improvements

The networking layer of Dat is getting two sets of upgrades.

The first upgrade is [Hyperswarm](https://github.com/hyperswarm/), a new [DHT](https://en.wikipedia.org/wiki/Distributed_hash_table) designed to arrange connections between users' devices. Hyperswarm will improve connection reliability using NAT Hole Punching and power new APIs in the Beaker browser such as PeerSockets.

Hyperswarm will be deployed in a backwards-compatible fashion so that old peers can still reach the new peers.

The second upgrade is a connection security system using the [NOISE protocol](http://noiseprotocol.org/). Noise will provide forward-secret connections with the option of authenticating connections. In the Dat network, this will improve overall privacy and open the possibility to implement network access-control.

## Tooling / Usability

In addition to the work on the core protocol, we are working on features to help with the ease of development of Dat and to address some of it's pain points.

### A high level SDK for building applications with Dat

![A diagram of the various pieces of the Dat ecosystem](https://user-images.githubusercontent.com/911495/59621049-627c9180-90fc-11e9-9acc-1f6deea4932d.png)

Getting Dat to work in different environments has meant combining different libraries together for each application.

The [SDK](https://github.com/datproject/sdk) will abstract some of the interaction with the networking and storage portions of Dat in order to simplify application development.

The first version of the SDK will reflect the current state of the Dat stack in order to get developers to start playing with it before we migrate to the new hyperdrive / networking layer.

### A Daemon for unifying Beaker and the CLI's method of keeping Dat's online

Beaker / the CLI / Other applications all have to create some sort of method for keeping Dats online and sharing data between processes. This makes application development harder, and makes it more confusing to use the Dat ecosystem.

The Daemon will either be run from a CLI, or spawned by Beaker or some other application and will enable applications to share Dats and keep them online.

It will also bundle up support for FUSE filesystems so that you can mount Dats as folders on your computer rather than syncing the folder to Dat.

### Third-party services to make it safe to transfer a hypercore across devices

A hiccup that a lot of Dat users face is the inability of Dat to enable updates from multiple devices. This is a result of the way that Dat uses append-only logs. If you try to append to the log from two devices at once, you might have a Conflict which will branch the log's history and mess up all the replication logic.

A potential solution is to have a trusted third party to sign the latest block in the chain so that there's an "official" branch if branching ever happens.

This will enable people to bring their Dats with them on different devices without needing to wait for union mounts to land.
