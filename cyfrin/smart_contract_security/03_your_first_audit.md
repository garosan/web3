# Section 03. Your First Audit | PasswordStore

## Scoping: Etherscan

So in this 1st example a protocol approaches you to review their codebase and all they provide is a link to their contracts on Etherscan. Big sign that their codebase is not mature and cannot be worked with under those circumstances. The absolute minimum is to provide a Github repo where you can also see the test suite, the deployment process, etc.

Once they've provided a Github repo you can apply the:

- [Minimal Onboarding Questions](https://github.com/Cyfrin/security-and-auditing-full-course-s23/blob/main/minimal-onboarding-questions.md)

If a project gives you pushback on providing it is a big red flag and you should either offer a different consulting contract or not work with them. Remember, you're looking for signals that the team actually cares about not being hacked.

This is the repo with the audit:

https://github.com/Cyfrin/3-passwordstore-audit

From now on, on all the repos there's gonna be an audit-data branch that will contain all the final answers for the security findings.

Make sure that you're checking out the exact commit hash that you agreed to audit.

## Tools

cloc -- counts the lines of code
https://github.com/AlDanial/cloc

Once installed, you can do cloc ./src/

## The audit process with Tincho

- Download the code
- Read the documentation
- If it's a hardhat project, Tincho creates a foundry folder where he runs his own foundry tests
- Check the options for cloc, you can actually export to CSV which you can then export to Google Sheets or smt
- Check this tool too: https://github.com/Consensys/solidity-metrics
- Normally start with the smallest contracts
- Tincho wrote fuzz tests
- Timebox yourself

## Getting Context - Solidity Metrics

Use the tool: Solidity Metrics by tintinweb

Right-click on a folder > 'Solidity Metrics'

Then command palette > Export Metrics Report in html

Sometimes the recon phase mixes with the vulnerability identification phase

## Explot: Access Control

Takes notes the way that works best for you, but always take notes:

What we've found is a fairly common vulnerability that protocols overlook. `Access Control` effectively describes a situation where inadequate or inappropriate limitations have been places on a user's ability to perform certain actions.

In our simple example - only the owner of the protocol should be able to call `setPassword()`, but in its current implementation, this function can be called by anyone.

## Protocol Tests

Check test coverage with `forge coverage`

## Phase 4: Reporting

You can copy the minimalistic file finding_layout.md:

```markdown
### [S-#] TITLE (Root Cause + Impact)

**Description:**

**Impact:**

**Proof of Concept:**

**Recommended Mitigation:**
```

## Writing an amazing private report

`fndings.md`

### Storing the password on-chain makes it visible to anyone, and no longer private

**Description:** All data stored on-chain is visible to anyone, and can be read directly from the blockchain. The `PasswordStore::s_password` variable is intended to be a private variable and only accessed through the `PasswordStore::getPassword` function, which is intended to be only called by the owner of the contract.

We show one such method of reading any data off chain below.

**Impact:** Anyone can read the private password, severely breaking the functionality of the protocol.

**Proof of Concept:** The below test case shows how anyone can read the password directly from the blockchain.

Explain step by step what we are going to do in the next section

## Writing an amazing proof of code

First lets run anvil.
Then deploy the PasswordStore contract to the local anvil chain using make deploy

Then cast storage contract_address <storage-slot-of-variable>

`cast storage 0x5FbDB2315678afecb367f032d93F642f64180aa3 1 --rpc-url https://127.0.0.1:8545`

We get back:

0x6d7950617373776f726400000000000000000000000000000000000000000014

Then, to cast we use:

`cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014`

## Recommended Mitigation

```markdown
**Recommended Mitigation:** Due to this, the overall architecture of the contract should be rethought. One could encrypt the password off-chain, and then store the encrypted password on-chain. This would require the user to remember another password off-chain to decrypt the stored password. However, you're also likely want to remove the view function as you wouldn't want the user to accidentally send a transaction with this decryption key.
```

## CodeHawks Docs

https://docs.codehawks.com/ read this, especifically this:

https://docs.codehawks.com/hawks-auditors/how-to-evaluate-a-finding-severity

## Final

Timebox yourself.

## Making a PDF of your report

https://github.com/Cyfrin/audit-report-templating
