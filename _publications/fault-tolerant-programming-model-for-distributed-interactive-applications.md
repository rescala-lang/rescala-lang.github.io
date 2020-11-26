---
date: 2019-10-01 12:00 +0200
type: "publication"
name: "A Fault-Tolerant Programming Model for Distributed Interactive Applications"
abstract: "Ubiquitous connectivity of web, mobile, and IoT computing platforms has fostered a variety of distributed applications with decentralized state. These applications execute across multiple devices with varying reliability and connectivity. Unfortunately, there is no declarative fault-tolerant programming model for distributed interactive applications with an inherently decentralized system model.

We present a novel approach to automating fault tolerance using high-level programming abstractions tailored to the needs of distributed interactive applications. Specifically, we propose a calculus that enables formal reasoning about applications' dataflow within and across individual devices. Our calculus reinterprets the functional reactive programming model to seamlessly integrate its automated state change propagation with automated crash recovery of device-local dataflow and disconnection-tolerant distribution with guaranteed automated eventual consistency semantics based on conflict-free replicated datatypes. As a result, programmers are relieved of handling intricate details of distributing change propagation and coping with distribution failures in the presence of interactivity. We also provides proofs of our claims, an implementation of our calculus, and an empirical evaluation using a common interactive application."
collaborators:
  - Ragnar Mogk
  - Joscha Drechsler
  - Guido Salvaneschi
  - Mira Mezini
slides: "https://www.youtube.com/watch?v=VLi6wktoE-8"
event: "OOPSLA 2019"
link: "https://doi.org/10.1145/3360570"
---