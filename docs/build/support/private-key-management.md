---
head:
  - - meta
    - name: "twitter:title"
      content: Private Key Management | zkSync Docs
---

# Working with Private Keys

In all our tutorials and guides, we use private keys to interact with zkSync. Most tutorials use `.env` files to store private keys in "plaintext", which [is considered bad practice](https://github.com/Cyfrin/foundry-full-course-f23/discussions/5). In this guide, we'll teach you a few safer ways to manage your keys.

You can take a look at the [.env pledge](https://github.com/Cyfrin/foundry-full-course-f23/discussions/5) which is an oath developers should take before working with private keys in their development files.

The main rule of thumb, is that you should never have a private key associated with real funds in plaintext. By "in plaintext" we mean unencrypted. If you are sure that a private key has no funds and will never have funds associated with it, feel free to put that key in a `.env` file. Otherwise, head the suggestions below.

## Hardhat

For the hardhat framework, there is no built-in encryption methods for private keys, and for this reason alone foundry is preferred. They have [configuration variables](https://hardhat.org/hardhat-runner/docs/guides/configuration-variables), however, these keep your private key in plain text and just move the file location to somewhere else in your directory.

Instead, we can use javascript (or typescript) to encrypt and decrypt our keys ourselves according to [EIP-2335](https://eips.ethereum.org/EIPS/eip-2335).

You can see the full example with [deployment in Ethers here.](https://github.com/PatrickAlphaC/ethers-simple-storage-fcc/blob/main/encryptKey.js)

`encryptKey.js`:

```javascript
const ethers = require("ethers"); // ^6.2.3 of ethers
const fs = require("fs-extra");
require("dotenv").config();

async function main() {
  const wallet = new ethers.Wallet(process.env.PRIVATE_KEY);

  const encryptedJsonKey = await wallet.encrypt(process.env.PRIVATE_KEY_PASSWORD);
  console.log(encryptedJsonKey);
  fs.writeFileSync("./.encryptedKey.json", encryptedJsonKey);
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

This is a `nodejs` script you can run to encrypt your key and save it to `.encryptedKey.json`. Once you have your key encrypted, you can then decrypt it in your scripts like so:

```javascript
const ethers = require("ethers");
const fs = require("fs-extra");
require("dotenv").config();

async function main() {
  const encryptedJson = fs.readFileSync("./.encryptedKey.json", "utf8");
  let wallet = new ethers.Wallet.fromEncryptedJsonSync(encryptedJson, process.env.PRIVATE_KEY_PASSWORD);
}
```

::: warning
If you do this, remember to add `.encryptedKey.json` to your `.gitignore` file!
:::

## Foundry

Foundry supports [encrypting keys by default](https://www.youtube.com/watch?v=VQe7cIpaE54) by using the `cast wallet import` command. Normally, to run a foundry script, you'd run a command like so to deploy a smart contract:

```bash
forge script script/Deploy.sol --private-key $PRIVATE_KEY --rpc-url $RPC_URL
```

Where `PRIVATE_KEY` is your private key from a `.env` file. This is of course not good because once again our private key is in plaintext.

However, we can encrypt our key by running:

```
cast wallet import account_name --interactive
```

And it will drop you into a shell to import your private key. Once it's imported, you'll be able to see it with `cast wallet list`.

Then, you can run your script like so:

```bash
forge script script/Deploy.sol --account account_name --sender $WALLET_ADDRESS --rpc-url $RPC_URL
```

The [foundry documentation](https://book.getfoundry.sh/tutorials/best-practices?highlight=patrick#private-key-management) has additional notes on secure private key management.

## Your own

And of course, you can always roll your own private key solution if you choose to do so.

# Summary

1. Never have a private key associated with real funds in plaintext.
2. Ideally, use a separate wallet for testing and development.
3. If you accidentally expose a private key, consider the whole thing compromised and try to move funds to a new wallet.
