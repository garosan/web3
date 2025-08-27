# Section 03. What is a Smart Contract Audit

## What is a smart contract audit?

Let's start by calling them security reviews instead of smart contract audits.

Now the big question: **What does a security review entail?**

### The Three Phases of a Security Review

The process looks like this:

```
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
```

There really isn't a "one-size-fits-all" approach to smart contract auditing. In this course, we will see 2 techniques: `"the Tincho"` and `"the Hans"`, but remember that these are just examples; there isn't a definitive _way_ to perform a security review.

### Importance of Security Reviews

**A smart contract audit is a timeboxed, security based review of your smart contract system. An auditor's goal is to find as many vulnerabilities as possible and educate the protocol on ways to improve their codebase security and coding best-practices moving forward.**

The benefits of conducting a security review go beyond just minimizing vulnerabilities. It can also help protocol developers in enhancing their understanding of the code itself, accelerating feature implementation and familiarizing them with the latest tools.

A single audit most often won't be enough. When a protocol launches, their security journey may include:

- Formal Verification
- Competitive Audits
- Mitigation Reviews
- Bug Bounty Programs

With this understanding, let's familiarize ourselves with the process of a typical audit.

➡️ Reach Out for a Review

The review process begins when a protocol reaches out, be it before or after their code is complete. After that, we can determine the cost of a review based on things like:

- Code Complexity/nSLOC
- Scope
- Duration
- Timeline

This is a very rough ballpark of the estimated duration of an audit based on LOC:

- 100: 2.5days
- 500: 1 Week
- 1000: 1-2 Weeks
- 2500: 2-3 Weeks
- 5000: 3-5 Weeks
- 5000+: 5+ weeks

The protocol then submits a commit hash and a down payment and a start date can be set!

➡️ ️️Audit Begins

The auditors get to work. Using all their tools, they strive to find as many vulnerabilities as possible.

➡️ ️️Initial Report

Once the review period is over, the auditors put together an initial report. This report includes all findings, categorized as follows:

- High
- Medium
- Low
- Informational/Non-critical
- Gas Efficiencies

- High, medium and low findings have their severity determined by the impact and likelihood of an exploit.
- Informational/Non-Critical and Gas Efficiencies aren't vulnerabilities, but ways to improve your code.

➡️ Mitigation Phase

The protocol now has a fixed period to address the vulnerabilities found in the initial report. Most often, they will just implement fixes to the vulnerabilities found.

➡️ Final Report

Upon completion of the mitigation phase, the audit team compiles a final audit report focusing exclusively on the fixes made to address the initial report's issues.

### Ensuring a Successful Audit

Try to make sure the protocol you will audit gives you these:

- Good documentation
- A solid test suite
- Clear and readable code
- Modern best practices are followed
- Clear communication channels
- An initial video walkthrough of the code

➡️ Post Audit

Even if you change **one** line of code after an audit, your code needs to be reviewed again, don't forget it.

## The Audit Process

### High-Level Overview of The Audit Process:

- Get Context
- Use Tools
- Manual Reviews
- Write a Report

Let's explain:

1. Get Context: Understand the project, its purpose and its unique aspects.
2. Use Tools: Employ relevant tools to scan and analyze the codebase.
3. Manual Reviews: Make a personal review of the code and spot out unusual or vulnerable code.
4. Write a Report: Document all findings and recommendations for the development team.

### Breakdown of the Process

➡️ Initial Review: Consists of 4 phases:

- Scoping: This is getting a sense of the protocol. In this phase, auditors go through the code to scope it. This gives an idea of how much time might be required for the audit, which can then be used to establish pricing. Key tasks include identification of all the contract's dependencies and a general overview of the code. At this stage, auditors don’t dig deep into anything yet.
- Reconnaissance: Here an auditor starts walking through the code, running tools, interacting with the protocol in an effort to break it.
- Vulnerability Identification: An auditor determines which vulnerabilities are present and how they're exploited as well as mitigation.
- Reporting: Compile a report detailing all of the identified vulnerabilities and recommendations to make the protocol more secure.

➡️ Protocol Fixes: The protocol implements suggested changes.

➡️ Mitigation Review: A new review is conducted with a focus on verifying that previously reported vulnerabilities have been resolved.

### The Smart Contract Development Lifecycle

1. Plan and Design
2. Develop and Test
3. Get an Audit
4. Deploy
5. Monitor and Maintain

Security is part of a lifecycle, it's not just about getting an audit.

## Rekt Test

Prior to getting a security review, Patrick recommends you as a protocol do these tests to your code (more importantly the rekt test):

- [nacentxyz simple-security-toolkit](https://github.com/nascentxyz/simple-security-toolkit)
- [The Rekt Test](https://blog.trailofbits.com/2023/08/14/can-you-pass-the-rekt-test/)

## Security Tools

Overview of the tools we will be learning about in this course

### Test Suites

First: Test Coverage! Using Foundry, Hardhat or whatever, **make sure the test suite is robust!**

### Static Analysis: Debugging Without Execution

This method automatically checks for issues without executing your code, hence the debugging process remains static. **Slither, 4nalyzer, Mythril, and Aderyn** are some prominent tools in the static analysis category.

### Fuzz Testing: Randomness Meets Tests

Fuzz testing comes in two flavours, `fuzz testing` and `stateful fuzz testing`.

- **Fuzz testing** also known as fuzzing involves providing random data as inputs during testing.
- **Stateful Fuzz Testing** is fuzz testing where the system remembers the state of the last fuzz test and continues with a new fuzz test.

### Formal Verification: Mathematical Proofs

**Formal verification** is a broad term for deploying formal methods to affirm the correctness of hardware or software. Often, these methods involve converting the codebase into mathematical expressions and deploying mathematical proofs to authenticate that the code does or doesn't do something specific.

A popular formal verification approach is **symbolic execution**. This method converts your Solidity function into math or a set of boolean expressions. **Manticore, Certora, Z3** stand tall in this domain.

### AI Tools and Others

There's a great GitHub repo by `ZhangZhuoSJTU` that illustrates examples of bugs that are detectable by machines and those that aren't. Check it out [here](https://github.com/ZhangZhuoSJTU/Web3Bugs).

**Other types of testing we won't be covering are differential test and chaos tests.**

### Conclusion

An important takeaway for you is that around 80% of actual bugs and competitive audit bugs are not auto-detectable by machines, including our present-day AI tools.
// TODO: Is this still the case in August 2025??

## What If Your Security Audit Fails?

From Tincho:

Auditors should provide value, regardless of whether or not they spot critical issues.

In other words, an auditor's value doesn't solely rest upon their ability to find vulnerabilities. Instead, their advice should strengthen the overall security protocol and offer pragmatic solutions for future scenarios.

Attributing the failure of a system to an auditor's incompetency is simplistic and misguided. If a vulnerability was missed, it means it slipped past numerous stages of checks and balances, of which an audit is just one.

### The Auditor’s Role in the Wake of a Breach

So, what should an auditor do if a protocol they've reviewed ends up compromised? The answer is that a responsible security partner should not abandon their client in the midst of a crisis.

As an auditor, you may be able to help mitigate the damage, restrict the scope of the attack, and possibly identify the hackers. A quality auditor must be there, lending their expertise, during the inevitable chaos that ensues after a breach.

## Top 5 Attack Vectors

- Private Keys - Stolen Private Keys are responsible for the largest loss of funds so far in 2023.
- Reward Manipulation – This vector involves the manipulation of decentralized incentive systems that could disrupt the balance and fairness within a network.
- Price Oracle Manipulation – This threat arises when a price oracle in centralized, or if a single oracle is relied upon, particularly with respect to price data.
- Insufficient Access Controls – onlyOwner modifiers, multi-sig wallets - just a couple things that could have prevented $17,000,000 in stolen funds this year.
- Re-entrancy (and Read-Only Re-entrancy) - by not adhering to proper Checks, Effects, Interactions patterns - protocols are still being rekt to the tune of $20,500,000 combined in 2023.
