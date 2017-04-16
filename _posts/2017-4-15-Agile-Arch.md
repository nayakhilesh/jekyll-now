---
layout: post
title: "Paper Summary: How Much Up-Front? A Grounded Theory of Agile Architecture"
---

I came across the paper in the title of this post while searching for material on the phrase ‘just enough design up-front’ that I heard from a colleague. The paper can be found [here](https://ecs.victoria.ac.nz/foswiki/pub/Main/TechnicalReportSeries/ECSTR15-01.pdf).
It was published in 2015 and hence is fairly recent. I found it particularly interesting since I’ve been seeing similar factors in play at my day job and similar coping mechanisms.
I highly recommend reading the paper in it’s entirety as well since it contains many interesting anecdotal quotes from fellow engineers / managers that one may be able to relate to!

# Summary:

Software architecture is the overall structure and design of a software system / application. Since it is a system level property it may be hard to modify once in place. However ‘responding to change over following a plan’ and delivering customer value often are central tenets of [agile software development](http://agilemanifesto.org/). Hence agile software teams must balance doing:

- more up-front architecture (which takes more time but reduces risk)
- less up-front architecture (which reduces the time to first delivery - thereby tightening the feedback loop and reflects the reality that requirements may change during development, but may consequently induce accidental architecture that may make things less agile in the long term)

The paper uses [grounded theory](https://en.wikipedia.org/wiki/Grounded_theory) to formalize this domain and make recommendations on how to think about the trade-off identified above. In a nutshell grounded theory involves using interviews to gather data and then coding the responses from these interviews to extract ideas, concepts and categories which are then woven into new theories.

Dev teams make this tradeoff by doing ‘just enough’ architecture up front. The definition of ‘just enough’ varies widely and depends on a set of environmental, technical and cultural factors. These factors (referred to as ‘forces’ in the paper) are:

## F1 (REQUIREMENTS INSTABILITY)
Requirements for agile software projects can often be in flux. They could be under specified or changing as the team gets customer feedback (customer has new ideas or a better idea of what they want) and begins to understand the domain better.
If the team spends a lot of time coming up with detailed requirements up front they may find that a lot of the effort is wasted since requirements change during implementation due to the above reasons.

## F2 (TECHNICAL RISK)
Technical risk is often introduced by having a complex architecture.
Architecturally complex solutions are caused by:

- Demanding requirements: Especially non-functional ones which may necessitate trade-offs like performance vs security (referred to as Architecturally significant requirements [ASRs])
- Interfacing with many other systems
- Involving legacy systems: Particularly since [software rots](https://en.wikipedia.org/wiki/Software_rot) and good development practices are eroded over time

## F3 (EARLY VALUE)
One of the main reasons for adopting an agile software methodology is to be able to reap the benefits of the software being written incrementally - before the entire system is built. This often manifests as an [MVP](https://en.wikipedia.org/wiki/Minimum_viable_product). Early value is a prerequisite for businesses operating in today’s competitive landscape.

## F4 (TEAM CULTURE)
Team culture has a huge impact on how much architecture is done up-front.
In a culture that emphasizes collaboration and trust there may not be the need for as much formal documentation / planning and / or sign-offs. Larger teams may find it harder to communicate effectively, making responding to change quickly more difficult. Teams that do not have exposure to agile software development may struggle without a well defined up-front architecture. Inexperienced developers may also find it harder to make progress without an explicit architecture.

## F5 (CUSTOMER AGILITY)
The mindset of the customer also has a huge impact on how to approach this trade-off.
An agile team may find it difficult to serve a customer that relies on cumbersome processes for decision making, handoffs, approvals etc.
Some of the reasons for customer in-agility may be:

- need for fixed price, scope or timelines
- requiring budget approval in advance from an executive (e.g.: CFO)

Requiring more predictability around these outcomes necessitates more up-front architecture.

## F6 (EXPERIENCE)
The team’s knowledge and experience factors into this problem as well.
Experienced teams and architects have encountered a broad range of problems and developed a good instinct for what works well and what does not. They are less likely to need time to explore, PoC and research solutions.

<br>
The theory of agile architecture comprises the following 5 strategies to help determine how much architecture to design up-front:

## S1 (RESPOND TO CHANGE)
A team may use the ability to respond to change to counteract requirements changing. A team does this by:

- keeping designs simple ([YAGNI](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it))
- evaluating a design by building and refining it
- adopting widely accepted design best practices (e.g.: [separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns))
- deferring architectural decisions until the [last responsible moment](https://blog.codinghorror.com/the-last-responsible-moment/)
- architecting in a way that does not introduce unnecessary constraints

## S2 (ADDRESS RISK)
Digging deeper into the areas of the project / design that are most murky and poorly understood. This can be done upfront for overarching technical decisions.
A team must give up some of it’s agility to reduce risk. The challenge for the team is to balance S2 with S1 and do enough up-front design to be able to continue with a comfortable level of risk - all the while delaying architectural decisions that are not as risky.
Teams can reduce risk through the use of research, analysis, experiments or PoCs.

## S3 (EMERGENT ARCHITECTURE)
The team makes the bare minimum architectural decisions up-front and evolves the architecture appropriately as more features are added. The initial decisions may encompass things like technology stack and overarching architectural patterns.
S3 enables EARLY VALUE (F3). If the system requires a complex design then more upfront architecture may be needed to ADDRESS RISK (S2) - thus F2 rules out an emergent design.

## S4 (BIG DESIGN UP-FRONT)
Gather a complete set of business requirements and chart out the entire architecture in detail at the outset of the project. S4 increases time to initial feedback, the odds that architecture will need to be modified and the chance of over-engineering.
In practice S4 is chosen mainly due to lack of CUSTOMER AGILITY (F5) and not the existence of TECHNICAL RISK (F2).
The non-agile customer may be pacified by:

- proof the team could solve the problem
- knowing the approximate cost of the effort
- obviating the need for continuing communication with the customer

Not being able to defer decisions reduces the team’s ability to be agile.

## S5 (USE FRAMEWORKS AND TEMPLATE ARCHITECTURES)
This involves using standard, widely available tools and frameworks as well as following reference architectures.
Most (good) frameworks / libraries provide canned solutions to commonly encountered problems and are adopted widely enough that the team has increased confidence of succeeding if they use the framework. Thus the team does not need to design an architecture from the ground up using first principles - allowing them to focus on customer feedback and business value instead of technical gotchas.
In cases where the problem is novel and requires building entirely new components or modifying existing ones the team may use S2 (ADDRESS RISK) to address this challenge.

## Relationship

The relationship between these factors and strategies is summarized in the paper as:

![My helpful screenshot]({{ site.url }}/images/agile-arch-forces-strats.jpg)

> "Figure 3 shows the relationships between the forces and the strategies. A team’s use of S1 (RESPOND TO CHANGE) is triggered by the presence of F1 (REQUIREMENTS INSTABILITY). A team’s agility, and thus its ability to use the tactics of S1, is increased by F4 (TEAM CULTURE), F5 (CUSTOMER AGILITY) and F6 (EXPERIENCE). Agility is not directly affected by F1; rather, less stability may motivate the team to improve its agility through improving F4, F5 and F6 so that it is better able to respond to change. S1 is in tension with S2 (ADDRESS RISK), which must be used to address F2 (TECHNICAL RISK), and hence S1 and S2 must be in balance. An extremely agile team that does not need to use S2 can, when triggered by F3 (the need for EARLY VALUE), reduce its up-front effort to the point where it is using S3 (EMERGENT ARCHITECTURE). On the other hand, a team with low levels of F4, F5 and F6 may have to use S4 (BIG DESIGN UP-FRONT), which will reduce its ability to use S1. S5 (USE FRAMEWORKS AND TEMPLATE ARCHITECTURES) can be used to significantly reduce development and design effort; in particular it significantly reduces risk and up-front effort for standard problems. A team successfully using S3 would typically be using S5 to build a low risk system."

