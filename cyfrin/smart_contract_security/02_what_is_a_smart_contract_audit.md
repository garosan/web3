# Section 02. What is a smart contract audit

## What is a smart contract audit?

Use the term 'security review' instead of 'smart contract audit'.

2 approaches to a security review:

- The "Tincho"
- The "Hans"

The Three Phases of a Security Review:

1. Initial Review
   a. Scoping
   b. Reconnaissance
   c. Vulnerability identification
   d. Reporting
2. Protocol fixes
   a. Fixes issues
   b. Retests and adds tests
3. Mitigation Review
   a. Reconnaissance
   b. Vulnerability identification
   C. Reporting

### Ensuring a Successful Audit

For an audit to be as successful as possible, you should ensure that there's:

- Good documentation
- A solid test suite
- Clear and readable code
- Modern best practices are followed
- Clear communication channels
- An initial video walkthrough of the code

## The audit process

At a very high level:

1. Get context
2. Use tools & manual review
3. Write report

**Repetition is the mother of skill**

### OWASP

- [OWASP Top Ten](https://owasp.org/www-project-top-ten/)
- [CloudFlare - What is OWASP? What is the OWASP Top 10?](https://www.cloudflare.com/learning/security/threats/owasp-top-10/)

## Rekt Test

- [nascentxyz/simple-security-toolkit](https://github.com/nascentxyz/simple-security-toolkit)
- [TrailOfBits - The Rekt Test](https://blog.trailofbits.com/2023/08/14/can-you-pass-the-rekt-test/)

## Security Tools Overview

- Foundry, Hardhat regular test suites
- Static analysis: Slither, Aderyn, Mythril
- Fuzzing & Stateful Fuzzing: echidna, Foundry
- Differential Test, Chaos testing (not covered here)
- Symbolic Execution (One of the main types of formal verification): MAAT, Z3, certora, mantycore

There's a great GitHub repo by ZhangZhuoSJTU that illustrates examples of bugs that are detectable by machines and those that aren't. Check it out **[here](https://github.com/ZhangZhuoSJTU/Web3Bugs)**.

## What if a protocol I audit gets hacked?

Pasting from Cyfrin Updraft:

### Penetrating the Scenario: What If Your Security Audit Fails?

As the world moves towards a more digital infrastructure, the importance of security audits cannot be overstated. But who carries the blame when these audits fail? Should it always land at the feet of those responsible for conducting the audit?

While broaching upon this intricate subject, I recently had a pleasant chat with the legendary Tincho, who imparted an inspiring perspective. He offers valuable insights on the way we should perceive the role and responsibilities of auditors in these precarious scenarios. Below will be summaries based on his thoughts and perspective.

### Redefining the Role of Auditors

In the eyes of many, the fundamental purpose of a security audit is to identify and rectify the most critical vulnerabilities in a system. However, Tincho encourages us to look beyond this simplistic view.

> Auditors should provide value, regardless of whether or not they spot critical issues.

In other words, an auditor's value doesn't solely rest upon their ability to find vulnerabilities. Instead, their advice should strengthen the overall security protocol and offer pragmatic solutions for future scenarios.

Of course, it goes without saying that the fewer critical vulnerabilities that are overlooked, the better - the safer Ethereum will be. It's naive however to believe that an auditor is solely responsible for when things go wrong.

### Who Owns the Blame?

The notion of finding a scapegoat when a system is exploited is a regressive one.

> A whole chain of events leads to the successful exploitation of a vulnerability.

Attributing the failure of a system to an auditor's incompetency is simplistic and misguided. If a vulnerability was missed, it means it slipped past numerous stages of checks and balances, of which an audit is just one. When a flaw goes unnoticed for as long as four months, there are perhaps lapses in system monitoring and in many other security parameters.

### The Auditor’s Role in the Wake of a Breach

So, what should an auditor do if a protocol they've reviewed ends up compromised? The answer is that a responsible security partner should not abandon their client in the midst of a crisis.

As an auditor, you may be able to help mitigate the damage, restrict the scope of the attack, and possibly identify the hackers. A quality auditor must be there, lending their expertise, during the inevitable chaos that ensues after a breach.

> "If you are to be the trusted security partner of your clients, probably, when they are hacked, you want to be there. You want to be there supporting them." - Tincho

### Conclusion

Security is a journey.

It was great catching up with Tincho, whose outlook on security audits balances realism with the optimistic pursuit of improvement. Every party involved in a security protocol must work together as a team and learn from any failure to ensure a safer, more secure digital environment.

## Top Web3 Attacks

### Unraveling the Top Attack Vectors

Let's consider the weakest parts of Web3 and remind everyone with the **“Top Attack Vectors.”**

1. **Private Keys** - Stolen Private Keys are responsible for the largest loss of funds so far in 2023 at `$243,000,000`
2. **Reward Manipulation** – This vector involves the manipulation of decentralized incentive systems that could disrupt the balance and fairness within a network. `$200,000,000` has been rugged so far this year.
3. **Price Oracle Manipulation** – This threat arises when a price oracle in centralized, or if a single oracle is relied upon, particularly with respect to price data. These vulnerabilities are responsible for `~$146,000,000` in losses in 2023.
4. **Insufficient Access Controls** – onlyOwner modifiers, multi-sig wallets - just a couple things that could have preventing `$17,000,000` in stolen funds this year.
5. **Re-entrancy (and Read-Only Re-entrancy)** - by not adhering to proper Checks, Effects, Interactions patterns - protocols are still being rekt to the tune of `$20,500,000` combined in 2023.

## Exercise

Sign up for one security/web3 newsletter:
