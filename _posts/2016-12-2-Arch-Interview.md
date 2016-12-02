---
layout: post
title: What I look for in a Software Design / Architecture Interview
---

When my team interviews candidates for a SWE position we often have a software design / architecture interview where we go over an application or feature that the candidate has worked on and / or ask them to come up with a design for a new system.
Some of the criteria I base my evaluation on follow.

For an existing application:
- Able to clearly express the current design of the application and how it interfaces with other systems
- Knows about the current users and how they interact with the app
- Knows about the various sources and sinks of data in the application
- Can answer what part of the application they have worked on and their role in that work with specific examples of things they have built / designed
- Have opinions about current pain points / suggestions for things they would do differently
- Understand tradeoffs and have examples of how they've made them / would make them
- Have examples of needing to optimize some (performance) bottleneck and how they went about it
- Have an understanding of how security needs are handled in their application
- Have knowledge of different parts of a web application's architecture (DB(s), web tier, load balancers etc)
- Have an understanding of deployment (plus points for talking about network topology and multiple data centers)
- Able to articulate their current testing strategy

For designing a new application:
- Able to come up with a clean, well defined, appropriate data model
- Able to come up with operations that might be needed of the system
- Think about separation of concerns / modularity if talking about different types of apps (they understand that login should probably not be a part of the core subscription mgmt application)
- Knowledge of common web protocols and how they work (e.g.: HTTP, TCP/IP, DNS, HTTPS, REST etc)
- Know how to store different types of data that your app receives (persistent stores vs caches, different types of DBs)
- Have a deployment strategy (possibly cross DC)
- Think about failure scenarios / reliability / fault tolerance / disaster recovery
- Think about security implications / secure / sensitive data and are conversant in security primitives such as encryption (of data at rest and in motion), tokens, authentication, authorization, message integrity, cookies, signing etc
- Think about audit and debugging use cases
- Consider what sort SLAs clients might need
- Able to talk about tradeoffs between the availability and consistency of information
- Able to do capacity / performance estimation
- Have an idea of how they would scale if needed
- Able to design for change / flexibility (what would you change if we now needed this new feature? how much / how many places did they have to change?)
- Design things that can be tested easily in separate modular components
- Think about monitoring / alerting for the app in production

In short everything that happens after conceiving of the application to getting it running smoothly in production and then responding to growth / feature requests / adverse incidents.
Obviously this is a lot and one probably won't get through all of it, but I try and get a feel for their breadth and depth of knowledge / experience with different things and also how they reason about design decisions.
