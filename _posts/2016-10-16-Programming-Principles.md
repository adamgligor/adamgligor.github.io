---
layout: post
title:  Programming principles worth knowing
date: '2016-10-16'
tags: programming
---

Programming principles, got to memorize a few. They do contain some useful advice.


## KISS - Keep It Simple Stupid


“The KISS principle states that most systems work best if they are kept simple rather than made complex; therefore simplicity should be a key goal in design and unnecessary complexity should be avoided.”

Source: [wikipedia](https://en.wikipedia.org/wiki/KISS_principle)

Variations & quotes: 

"Perfection is achieved, not when there is nothing more to add, but when there is nothing left to take away." - Antoine de Saint-Exupery

"Everything should be made as simple as possible, but not simpler." This means that one should simplify the design of a product and success is achieved when a design is at its maximum simplicity." - Albert Einstein

"Debugging is twice as hard as writing the code in the first place. Therefore, if you write the code as cleverly as possible, you are, by definition, not smart enough to debug it." - Brian Kernighan


## YAGNI - You Aint Gonna Need It


Is a principle of extreme programming (XP) that states a programmer should not add functionality until deemed necessary.

Source: [wikipedia](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it)


## SOLID


Acronym for five basic principles of object-oriented programming and design. The intention is that these principles, when applied together, will make it more likely that a programmer will create a system that is easy to maintain and extend over time

List of principles: 

- **Single responsibility principle** A class should have only a single responsibility (i.e. only one potential change in the software's specification should be able to affect the specification of the class)

- **Open/closed principle** Software entities … should be open for extension, but closed for modification.

- **Liskov substitution principle** Objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program.

- **Interface segregation principle** No client should be forced to depend on methods it does not use. 

- **Dependency Inversion Principle** High-level modules should not depend on low-level modules. Both should depend on abstractions.

Source: [wikipedia](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design))


## GRASP


General responsibility assignment software patterns (or principles), abbreviated GRASP, consist of guidelines for assigning responsibility to classes and objects in object-oriented design.

List of patterns: 

 - Controller
 - Creator
 - High cohesion
 - Indirection
 - Information expert
 - Low coupling
 - Polymorphism
 - Protected variations
 - Pure fabrication


 Source: [wikipedia](https://en.wikipedia.org/wiki/GRASP_(object-oriented_design))


## DRY Don't repeat yourself


The DRY principle is stated as "Every piece of knowledge must have a single, unambiguous, authoritative representation within a system". The principle has been formulated by Andy Hunt and Dave Thomas in their book The Pragmatic Programmer. 

Source: [wikipedia](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)


## General OOP principles


- Encapsulation
- Abstraction
- Inheritance 
- plimorphism

Source: various books


## Boyscout rule


Always check a module in cleaner than when you checked it out, No matter who the original author was, what if we always made some effort, no matter how small, to improve the module. 

Source: various books. 


## POLA - Principle of least astonishment 


If a necessary feature has a high astonishment factor, it may be necessary to redesign the feature.

Source: [wikipedia](https://en.wikipedia.org/wiki/Principle_of_least_astonishment)


## Broken window principle/theory


The theory: an abandoned building with a single broken window attracts additional broken windows, graffiti, litter, and a sense that no one cares about this building. 
However this can be avoided if the authorities repair the window straight away.


Don’t leave “broken windows” (bad designs, wrong decisions, or poor code) unrepaired


Source: [wikipedia](https://en.wikipedia.org/wiki/Broken_windows_theory)


## Avoid premature optimization


We should forget about small efficiencies, say about 97% of the time; premature optimization is the root of all evil” – Donald E. Knuth, Structured Programming with go to Statements


##  Conway's law


Organizations which design systems ... are constrained to produce designs which are copies of the communication structures of these organizations — M. Conway


## Law of Demeter


 - Each unit should have only limited knowledge about other units: only units "closely" related to the current unit.
 - Each unit should only talk to its friends; don't talk to strangers.
 - Only talk to your immediate friends.

Source: [wikipedia](https://en.wikipedia.org/wiki/Law_of_Demeter)


## Ninety ninety rule 


The first 90 percent of the code accounts for the first 90 percent of the development time. The remaining 10 percent of the code accounts for the other 90 percent of the development time. —  Tom Cargill, Bell Labs

Source: [wikipedi](https://en.wikipedia.org/wiki/Ninety-ninety_rule)


## Inversion of control / Hollywood principle 


Don’t call us, we’ll call you.

Source: [wikipedia](https://en.wikipedia.org/wiki/Inversion_of_control)

