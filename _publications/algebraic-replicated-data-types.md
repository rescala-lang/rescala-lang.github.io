---
date: 2023-07-17
type: "publication"
name: "Algebraic Replicated Data Types: Programming Secure Local-First Software"
abstract: "This paper is about programming support for local-first applications that manage private data
locally, but still synchronize data between multiple devices. Typical use cases are synchronizing
settings and data, and collaboration between multiple users. Such applications must preserve the
privacy and integrity of the user’s data without impeding or interrupting the user’s normal workflow
– even when the device is offline or has a flaky network connection.

From the programming perspective, availability along with privacy and security concerns pose
significant challenges, for which developers have to learn and use specialized solutions such as
conflict-free replicated data types (CRDTs) or APIs for centralized data stores. This work relieves
developers from this complexity by enabling the direct and automatic use of algebraic data types –
which developers already use to express the business logic of the application – for synchronization and
collaboration. Moreover, we use this approach to provide end-to-end encryption and authentication
between multiple replicas (using a shared secret), that is suitable for a coordination-free setting.
Overall, our approach combines all the following advantages: it (1) allows developers to design
custom data types, (2) provides data privacy and integrity when using untrusted intermediaries,
(3) is coordination free, (4) guarantees eventual consistency by construction (i.e., independent of
developer errors), (5) does not cause indefinite growth of metadata, (6) has sufficiently efficient
implementations for the local-first setting.
"
collaborators:
  - Christian Kuessner
  - Ragnar Mogk
  - Anna-Katharina Wickert
  - Mira Mezini
pdf: "assets/pdf/2023 Secure ARDTs (preprint).pdf"
event: "ECOOP 2023"
---
