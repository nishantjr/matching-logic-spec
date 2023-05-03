There are currently multiple formalizations of matching logic,
in [Coq], [Dedukti], [Lean], [MM0] and [Metamath].
To build a cohesive matching logic ecosystem, it is in our best interest
to have these formalization cooperate, rather than compete with each other.

To enable this, in this project we build a high-level representation of matching
logic specifications and use this as a medium of exchange. We will have
translations of specifications from this high-level representation to Metamath,
and each of the formalizations. This will enable us to use each of the
formalizations to handle the reasoning that they are best at, while handing off
other reasoning to more suitable formalizations.

At the same time, having a higher-level formalization enables us to make changes
to the Metamath formalization without forcing each of immediately change their
representation. This will let us take a more agile approach and not have each of
the formalizations tighly bound to the Metamath formalization.

## How is this different from `kore` and `kompile`?

`kore` and `kompile` bring K closer to Matching Logic,
by reducing K Specifications to simpler ones.
Our goal here, is to move in the opposite direction,
building more complex notions, from matching logic, to eventually meet Kore
in the middle.

