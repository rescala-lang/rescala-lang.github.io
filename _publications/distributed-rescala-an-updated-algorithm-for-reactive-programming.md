---
date: 2014-10-01
type: "publication"
name: "Distributed REScala: An Update Algorithm for Distributed Reactive Programming"
abstract: "Reactive programming improves the design of reactive applications by relocating the logic for managing dependencies between dependent values away from the application logic to the language implementation. Many distributed applications are reactive. Yet, existing change propagation algorithms are not suitable in a distributed setting.

We propose Distributed REScala, a reactive language with a change propagation algorithm that works without centralized knowledge about the topology of the dependency structure among reactive values and avoids unnecessary propagation of changes, while retaining safety guarantees (glitch freedom). Distributed REScala enables distributed reactive programming, bringing the benefits of reactive programming to distributed applications. We demonstrate the enabled design improvements by a case study. We also empirically evaluate the performance of our algorithm in comparison to other algorithms in a simulated distributed setting."
collaborators:
  - Joscha Drechsler
  - Guido Salvaneschi
  - Ragnar Mogk
  - Mira Mezini
pdf: "assets/pdf/2014 Distributed REScala.pdf"
event: "OOPSLA 2014"
link: "https://dl.acm.org/citation.cfm?id=2660240"
---
