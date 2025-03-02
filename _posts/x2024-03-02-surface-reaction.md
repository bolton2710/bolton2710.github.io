---
layout: post
title: "2. Surface catalytic reactions"
author: "Bolton Tran"
categories: tutorials
tags: [rxn]
image: "/assets/img/che1.jpg"
---
Many chemical reactions
take place on a catalyst surface.
Mechanisms of surface catalytic reactions
are often broken down
to elementary steps,
often involved adsorption, reaction, and desorption.
Setting up and solving these surface mechanisms
are the basis for 
analyzing catalytic reactions.
As usual,
the theories, model assumptions, and derivation
could be found else where;
the practical problem solving in Python
is the main focus here.

The adsorption/desorption steps
are often assumed to be in quasi-equilibria;
the reaction steps
are treated to be rate-limiting,
i.e., their kinetics control the overall catalytic rates.
Essentially,
that means
the rates of molecules adsorbing/desorbing
from/to the gas phase
are assumed to be much faster
than the rates of bond breaking/forming on the surface.

<!-- Langmuir isotherms -->
<h3> Langmuir model </h3>
The Langmuir model
is the text-book starting point
for adsorption/desorption equilibria.
This model is useful by itself
in surface sciences.
