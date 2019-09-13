# dDatabase Whitepaper
**A Distributed Web Protocol For Building Decentralized Database Management Systems**
***Author: Jared Rice Sr.***
***Date Created: 09/12/2019***

## Abstract
------------
dDatabase is an append-only database that is identified by a public key and can only be written to by the holder of the corresponding private key. Due to the append-only nature of dDatabase, old values cannot be deleted nor modified and due to it's cryptographically secure nature, a dDatabase can be downloaded from untrustworthy peers and still be verified for its authenticity. From a computer science perspective, dDatabases are binary append-only streams, the contents of which are cryptographically hashed and signed. Therefore, any dDatabase can be verified by anyone with access to the public key of the database's creator. dDatabase is considered to be a core component of the dWeb and has been used to create dDBMS's (decentralized database management systems) like dAppDB.

## Background 
Over the HTTP protocol, datasets are shared everyday, although, there is no built-in support for version control or the content addressability of data. dWeb became the solution for this, ultimately forming a distributed and decentralized versioned file-sharing network, that enables multiple untrusted devices, to act as a single virtual host. For dWeb to work, it requires a data structure which authenticates the content's integrity and one that keeps a historical log of revisions.

## dDatabase Use-Case: dWeb
For dWeb to work, it requires a data structure which authenticates the content's integrity and one that keeps a historical log of revisions. A dWeb-based data structure must be able to retain a historical log of revisions for a particular dataset, prevent dataset creators from mutating revision logs, the capability of verifying individual file-based integrity by a dataset's identifier and the ability for a dataset to be randomly or partially replicated over the dWeb network. dDatabase provides the ability for dWeb to randomly or partially distribute versioned and immutable datasets, where the files within them are verified for their integrity on a per-file basis. 

## dDatabase Use-Case: dAppDB
dAppDB is a scalable decentralized, peer-to-peer, database management system for the distributed web protocol. A value can be written at locations like /presidents/45/dtrump and the API supports querying of tracking values. For example, when changes are made to /presidents/45/dtrump, changes will be reported to /presidents/45/dtrump/news and /presidents/45/dtrump/photos. 

dAppDB is fundamentally a set of dDatabase feeds and because it is append-only, old values cannot be deleted individually or modified. Due to its cryptographically-secure nature, a dAppDB can be downloaded from untrustworthy peers as well and still verified for its authenticity and originality. Since a dAppDB can only be appended to by the creator of the dAppDB (like dDatabase), if a decentralized social network were created using dAppDB, each user's page would have its own dWeb key and dWeb-based repository, that stored a dAppDB containing their profile data, along with any media files associated with their profile. This would allow users to control their page and their data, rather than data being in one central location. While this presents a challenge when trying to search for users, dApps like a social network could utilize specific kinds of metadata within each profile's dWeb repository to filter a dWeb-based network for repositories only related to the social networking dApp itself, just like a search engines over HTTP, use metadata within HTML files to filter through millions of websites for the particular website(s) a user is trying to locate. A social network known as dStatus, a Twitter-like social networking dApp on the dWeb, currently uses dAppDB and dSiteDB as its dDBMSs.

## Append-Only Data Feed
A dDatabase could be seen as an append-only data feed, where the data within a data feed is an abstract "block" of data. 

**dDatabase feeds can:**
- be distributed partially over a dWeb-based network.
- be distributed fully over a dWeb-based network.
- be distributed to/received by multiple dWeb-based peers, simultaneously.

dDatabases are identified internally by signed Merkle trees and identified over the public network by a public key. A dDatabase's public key is used to verify the signature that's related to the received data. A dDatabase's internal Merkle tree is outputted as a "Flat In-Order Tree" or "FIO Tree".

## FIO Trees
FIO trees, per PPSP RFC 7574, are defined as "bin numbers". These FIO trees, allow for the numerical-based identification of every leaf node of a binary-based Merkle tree and creates a simplistic way of representing a binary tree as a list . These properties of FIO trees are used in the simplification of "wire protocols" that use Merkle tree structures, within distributed and/or decentralized applications. 

Below is an example of a FIO tree, that is sized at 4 blocks of data:
```
0L
1P
2L
3P
4L
5P
6L 
```
In the above FIO tree example, even numbers (0,2,4,6) represent leaf nodes on the tree and odd numbers represent parent-nodes that each contain two children. 

Using "binary notation", we can count the total number of "trailing 1s", to calculate the depth of a tree's node. For example, the following numbers below are converted to binary:
```
5=101
3=011
4=100
```
The number 5 has one trailing one, the number 3 has two trailing ones and the number 4 has zero trailing ones. In the FIO tree example, 1 is the parent node of (0,2), 5 is the parent node of (4,6) and 3 is the parent of (1,5). The FIO tree would only have a single root if the leaf node count is a power of 2, otherwise a FIO tree will always have more than one root. 

Below is another pseudo-representation of a FIO tree with a total of 6 leaf-nodes:
```
0
1
2
3
4
5
6
7
8
9
10
```

**NOTE:** ***In the above example, the roots are 3 and 9***

## Merkle Trees 
A Merkle tree is best defined as a tree of binary-based data, where every "leaf" (an even-numbered tree node that has no children) is a hash of a data block and every "parent" (an odd-numbered tree node that has two children) is the hash of both of its children. dDatabases are feeds, represented by Merkle trees that are ultimately encoded with "bin numbers". 

For example, a dDatabase that consists of 4 values , would always map to the 0,2,4 and 6 leaf nodes. Below is a pseudo-representation of this:

```
fragment 0 -> 0 
fragment 1 -> 2
fragment 2 -> 4
fragment 3 -> 6
```

When converting to FIO tree-style notation, a Merkle tree spanning these data blocks looks like:

```
0 = hash(fragment0)
1 = hash(0+2)
2 = hash(fragment1)
3 = hash(1+5) 
4 = hash(fragment2)
5 = hash(4+6)
6 = hash(fragment3)
```

The even and odd nodes store different types of information as follows:
**Even Numbers** - List of data hashes ***[frag0, frag1, frag2...]***
**Odd Numbers** - List of Merkle hashes (hashes of child even nodes) ***[hash0, hash1, hash2 ...]***

The root node within a Merkle tree hashes the entire dataset. Although, the root node will change every time data is added to a database. In the example below, of 4 fragments, node 3 hashes the entire dataset and node 3 is used to verify the rest of the dataset. 

``` 
0
1
2
3 (root node)
4
5
6
```

FIO-based Merkle trees can have multiple roots. If we expand to six fragments, from four, this will ultimately result in two root nodes.

```
0
1
2
3 (root node)
4
5
6
7
8
9 (root node)
10
```

**The Merkle tree's nodes are calculated as follows:**

```
0 = hash(fragment0)
1 = hash(0+2)
2 = hash(fragment1) 
3 = hash(1+5)
4 = hash(fragment2)
5 = hash(4+6)
6 = hash(fragment3)
8 = hash(fragment4)
9 = hash(8+10)
10 = hash(fragment5)
```

When there are multiple root hashes, it is convenient to capture the entire state of a dDatabase as a fixed-sized hash, by hashing all of the root hashes into one single hash, where in the example above:

```root = hash(9+3)``` or ```tr=h(r1+r2)``` 
***Where, tr = top root, h = hash, r1 = root 1 and r2 = root 2***

## Root Hash Signatures
Merkle trees are utilized by dDatabase to create a way of identifying the content of a dataset through hashes. The concept is simple, if the underlying content of a dataset changes, the hash changes. For example, a dDatabase acts as a list that calls the append() mutation, when an entry is added to the database feed, thereby adding a new leaf to the tree, which ultimately generates a new root hash. 

When a dDatabase is created, a keypair is generated as well. The public key is used as a public identifier for data addressability, whereas the private key is used to sign a root hash, every time a new one is generated. The signature is always distributed with the root hash, to verify its integrity. 

## dDatabase Verification
Data belonging to a dDatabase that is received in a streaming update, goes through the following process:
1. The root hash's signature is verified 
2. The received data is hashed with "ancestor" hashes, in order to reproduce the root hash
3. If the root hash that was calculated is an exact match of the signed root hash that was received, then the data has been verified.

**A dDatabase containing four values, represented as a tree of hashes, would look like:**

```
0
1
2
3 (root node)
4
5
6
```

If we want to verify the data for 0 (f0), we first need 2, which is the sibling hash, 5 which is the uncle hash and 3 which is the signed root hash.

```
0 = hash(fragment0)
2 = (hash received)
1 = hash(0+2)
5 = (hash received)
3 = hash(1+5)
```

If what we calculate for 3 is equal to the signed root hash we received for 3, then fragment0 is valid. It's important to note that all new signatures, verify the entire database, since the signature spans all data in the Merkle tree. Also, a dDatabase is considered corrupt, if a signed mutation results in a conflict against previous-verified trees. When a database is considered corrupt, the dWeb protocol is designed to stop distribution of the dDatabase feed.

## Specifications
- The hash function uses BLAKE2B-256 encryption, while signatures are ed25519 with the sha-512 hash function.
- Hash function inputs are prefixed with different "constants" based on the type of data being hashed. 

**These constants include:**
```
0x00 - Leaf 
0x01 - Parent
0x02 - Root
```
This protects against a "second preimage attack"
- Hashes a lot of the time will include the sizes and indexes of their content, so that the structure of a tree can be described along with the tree's data. 

## Conclusion
While IPNS is more user-friendly when compared to dDatabase, since data isn't deemed "corrupted" when an author revises its history, dDatabases are meant to be immutable, decentralized and safe to distribute amongst peers. Users of IPNS freely exchange private keys between devices as well, while dDatabase conceals the private keys of users. dDatabase is also far better at the real-time streaming of data, considering consumers can subscribe to broadcasts of new database updates. dDatabase and solutions like dAppDB and dSiteDB present an infinitely scalable database solution for building decentralized applications on the decentralized web. This enables databases to be distributed amongst peers, where peers themselves control their own data while keeping data immutable and trusted at the same time. This also gives dApp developers a toolset for placing a majority of their data off-chain, so blockchains remain free to focus their resources on application logic via smart contracts and the payment-based functionality they're known for.
