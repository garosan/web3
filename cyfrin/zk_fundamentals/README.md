# Fundamentals of Zero-Knowledge Proofs (ZKPs)

- [Course Repo](https://github.com/Cyfrin/zero-knowledge-fundamentals-cu)

## ZKP Overview

**ZERO-KNOWLEDGE PROOF (ZKP)**:

A cryptographic method that allows one party (the prover) to convince another party (the verifier) that they know something without revealing that information itself.

**The Three Pillars: Fundamental Properties of ZKPs**:

- Completeness: If the prover genuinely knows the secret, and correctly follows the ZKP protocol, the verifier will always accept the proof as valid.
- Soundness: If the statement is false, no cheating prover can convince the verifier that it’s true, except with a negligible probability.
- Zero-Knowledge: The verifier learns nothing beyond the fact that the statement is true.

## Interactive vs Non-Interactive ZK Proofs

Interactive Zero-Knowledge Proofs were the pioneering form of ZKPs. Their defining feature is the necessity for a dynamic, back-and-forth conversation, or multiple rounds of interaction, between the Prover and the Verifier.

Key Characteristic: The Prover and Verifier must actively communicate, exchanging messages in a structured sequence for the proof to be established.

Kira talks about the _Ali Baba cave example_, which is a story presenting the fundamental ideas of zero-knowledge proofs, first published in 1990 by Jean-Jacques Quisquater and others in their paper _"How to Explain Zero-Knowledge Protocols to Your Children"_

[Link to the story](https://pages.cs.wisc.edu/~mkowalcz/628.pdf)

Drawbacks of Interactive ZKPs:

- Time-Consuming: Necessity for multiple rounds of interaction
- Not Practical for Blockchain: Maintaining the state of an interactive proof and storing the information from each round would be cost intensive.
- Verifier-Specific: As Kira mentioned in the example, even though she convinced Patrick that she knows the secret lockbox password, if someone else comes along, she has to convince that new person again.

**Exploring Non-Interactive Zero-Knowledge Proofs (NIZKs)**

To address the above limitations, Non-Interactive Zero-Knowledge Proofs (NIZKs) were developed.

Key characteristics:

- Single Message Proof: The Prover constructs and sends the proof in one go.
- Independent Verification: The Verifier can check the validity of this proof independently, without needing any further interaction with the Prover.
- Public Verifiability: The same proof can be verified by multiple different people. Once generated, the proof stands on its own and can be presented to anyone who needs to verify the Prover's claim.

Kira mentions the anallogy of 'Where's Wally'. To prove that she knows _where is Wally_ withouth revealing this info, she can cut out the face of Wally from the pic and show it to the verifier (she couldn't do this if she didn't know where Wally is), or she could get a big big piece of paper and just make a whole the size of Wally's face and overlay over the original picture, thus using the whole to reveal the face of Wally without revealing where he is.

**Types of NIZKs:**

The world of NIZKs is rich and evolving, with several prominent types, often referred to by acronyms:

- **SNARKs (Succinct Non-Interactive Argument of Knowledge)**: These proofs are "succinct" meaning they are very small in size and quick to verify, regardless of the complexity of the statement being proven.
- **STARKs (Scalable Transparent Argument of Knowledge)**: STARKs are "scalable" (their proof generation and verification times scale more efficiently for very complex statements) and "transparent" (they don't require a "trusted setup" phase, which is a potential vulnerability in some SNARKs).
- **Bulletproofs**: These are another type of NIZK known for not requiring a trusted setup and for being efficient for proving statements about ranges (e.g., a value is within a certain range).

These terms are often umbrella categories. For instance, SNARKs encompass various specific cryptographic schemes like Groth16 and Plonk, each with its own unique mathematical underpinnings, performance characteristics, and trade-offs.
