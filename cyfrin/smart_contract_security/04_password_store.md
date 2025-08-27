# Section 04. Your First Audit | PasswordStore

## Our First Security Review

Start here:

- [Section 3: Your first audit (security review) | PasswordStore Audit](https://github.com/Cyfrin/security-and-auditing-full-course-s23?tab=readme-ov-file#%EF%B8%8F-section-3-your-first-audit-security-review--passwordstore-audit)

Check the audit-data of [this repo](https://github.com/Cyfrin/3-passwordstore-audit/tree/audit-data). This is what we're going to achieve.

A very well crafted report of all our findings, but let's pretend in this scenario this protocol has no idea how to prepare for a security review...

## Scoping: Etherscan

First approach, they want you to review their codebase...they send you an [Etherscan link](https://sepolia.etherscan.io/address/0x2ecf6ad327776bf966893c96efb24c9747f6694b)... 😐😐😐

So if they give you this...where is the test suite?? Make them write and provide a test suite!

🤔 Question: When a protocol gives you only an Etherscan link, what should be your next step?

Apply the checks for audit readiness, for example:

- [Simple Security Checklist](https://github.com/nascentxyz/simple-security-toolkit/blob/main/audit-readiness-checklist.md)
- Test suite?
- Natspec?
- The Rekt Test!!

If you try to apply the Rekt Test, you'll see most of the questions will be answered No / I don't know. So next natural step would be to ask the client for a documented GitHub repo.

## Scoping: Audit Details

Client now sends [this GH repo](https://github.com/Cyfrin/3-passwordstore-audit).

It's a great start but still it doesn't contain a test folder and we have some other questions.

These are the [minimum onboarding questions](https://github.com/Cyfrin/security-and-auditing-full-course-s23/blob/main/minimal-onboarding-questions.md) so that we can begin to audit a protocol.

Really go through the above list and understand it.

In this scenario, after browsing to what they have in their GH repo, you ask the protocol for additional documentation like:

- Exact commit hash they want audited
- Deployment process
- What roles exist in the protocol, etc.

So the protocol now sends you [this version](https://github.com/Cyfrin/3-passwordstore-audit/tree/onboarded)

You can see now they've [answered our questions](https://github.com/Cyfrin/3-passwordstore-audit/blob/onboarded/minimal-onboarding-filled.md)

When you clone the repo, **make sure that you're auditing the exact commit hash that is in scope**.

So let's go to the README.md of their provided repo and start by cloning it:

```bash
git clone https://github.com/Cyfrin/3-passwordstore-audit
cd 3-passwordstore-audit
git checkout 7d55682ddc4301a7b13ae9413095feffd9924566
git switch -c passworestore-audit
forge build
```

## Scoping: cloc

- [Instructions to install cloc](https://github.com/AlDanial/cloc)

`cloc --help` to confirm its installed correctly.

Then:

`cloc ./src`

## The 'Tincho'

His audit process:

- Download the code
- Read the documentation
- Use cloc and then export the output to a CSV, google sheet, etc.
- Use Solidity Metrics too, good extension

## Recon: Context

So let's go back to our PasswordStore audit and get to work.

We start by reading the README, we read the app's purpose:

`A smart contract application for storing a password. Users should be able to store a password and then retrieve it later. Others should not be able to access the password.`

From here, we can start doing some questions:

- How does the retrieving later works? Is there a way we can break that?
- How might others access the password?
- Is there a way users might be blocked from storing a password?
