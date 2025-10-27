# Section 02. Wallets

## Wallet and Private Key Important Tips

### Private Key Management Tips

Managing your private key securely is crucial for protecting your cryptocurrency assets. Here are some essential tips to ensure your private key remains safe:

- **Keep Your Balance Private:** Avoid disclosing the amount of money you have to others.
- **Purchase from Official Sources:** Always buy hardware wallets directly from the vendor, company, or official resellers. Verify the authenticity of the resellers.
- **Regular Key Changes:** While you can use a single private key for years, it's recommended to change private keys periodically (e.g., every six months, year, or two). Multi-sig wallets are ideal for this as they allow you to keep the same wallet address and change the signers.
- **Recurring Reviews:** Establish a recurring review of your private key every six months or at an interval that you find suitable.
- **No Digital Copies:** Never take a photo of your private key, upload it to the cloud, text it, email it, or share it with others.
- **Respond to Key Exposure:** If your private key is lost, exposed, or leaked, consider it compromised. Promptly move your assets to a new wallet or swap out the key (in the case of a multi-sig wallet).
- **Avoid Plain Text Storage:** Do not store your private key in plain text in a password manager, especially a cloud-based one. Past incidents, like the LastPass breach, have shown that private keys stored in password managers can be stolen, leading to significant losses. If you must store your private key in a cloud provider, ensure it is encrypted to add an extra layer of security.
- **Prioritize Security:** It's better to be safe than sorry when it comes to your private keys. Always take precautions and prioritize their security.

### Further reading

- [Referenced Cyfrin Article](https://www.cyfrin.io/blog/best-web3-wallets-safest-way-to-store-crypto)
- [Experts Fear Crooks are Cracking Keys Stolen in LastPass Breach](https://krebsonsecurity.com/2023/09/experts-fear-crooks-are-cracking-keys-stolen-in-lastpass-breach/)
- [Safe: Create a wallet](https://safe.global/)

## Where Is My Private Key

Popular format to encrypt private keys, the one most browser wallet use, also Foundry uses this when you use cast.

- [ERC-2335](https://eips.ethereum.org/EIPS/eip-2335)

It's good to read the Metamask docs to learn more and know that your private key (for Windows) is stored in a path like this:

`C:\Users\{user}\AppData\Local\Google\Chrome\User Data\Default\Local Extension Settings\nkbihfbeogaeaoehlefnkodbefgpgknn`

- [User Guide: Secret Recovery Phrase, password, and private keys](https://support.metamask.io/start/user-guide-secret-recovery-phrase-password-and-private-keys/)

## Multi Sig Wallets

A multi-sig wallet is a smart contract that governs transactions. These can be setup so that they require several people to sign a transaction for it to go through.

These can be customized such that any X number of people are required to sign out of Y before a transaction will process.

For example, if 3 people are needed to sign out of 5 authorized signers, this is known as a 3-of-5 wallet.

If you don't have a multi-sig wallet, we are going to walk you through setting one up by the end of this course. Why? Because multi-sig is absolutely my most preferable method of storing crypto assets for a number of reasons that we will get into shortly.

Social recovery is another method of storing crypto assets, and it is pretty similar to multi-sig, but we will talk about both of them shortly.

A multi-sig wallet, unlike browser wallets and hardware wallets, is a contract deployed on-chain where the money is stored, and you have defined parameters to interact with the wallet. Probably the best multi-sig wallet out there is Safe.global, which we will also walk through. There are other fantastic wallets too; for example, Aragon has a multi-sig feature for DAOs specifically.

A multi-sig wallet is a wallet that requires multiple signers to send out a transaction. For example, in a 3 of 5 multi-sig wallet, three out of the five wallets registered on the multi-sig wallet must sign a transaction before it is sent out.

## Social Recovery Wallet

Method recommended by Vitalik.
