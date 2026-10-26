# 2FA SeedPlus Wallet
The project turns 12 or 24 standard recovery words plus up to three add-on phrases into one Ethereum wallet, and a working browser demo of that already exists.

## Built and tested

•	Compatible mode: the add-ons are joined as P1+P2+P3 into one standard BIP-39 passphrase, then BIP-32 and BIP-44 give the account at path m/44'/60'/0'/0/n.

•	A single-file demo, wallet-demo.html, with Create wallet and Open wallet screens that show the public address, public key and private key.

## Planned, not built

•	A Chrome extension (Manifest V3) with encrypted storage.

•	Vault mode: add-ons stretched with Argon2id, recoverable only with this software.

•	An external security audit before any real funds are held.

## Architecture and data flow
Image
The add-ons only change the passphrase that enters the seed step. From the seed onward the steps are standard BIP-32 and BIP-44, so the published test vectors apply there. Vault mode swaps the seed step for the Argon2id chain and leaves everything below it unchanged.

## Install and run

The demo needs no install: unzip the folder and open wallet-demo.html in Chrome, Edge, Firefox or Safari. It works offline and makes no network requests.
### Use the demo

1.	Open wallet-demo.html.
2.	On Create wallet, pick 12 or 24 words and 0 to 3 add-ons. Random add-ons are filled in for you, and you can edit them.
3.	Press Create wallet. Write the words and add-ons on paper, in order, and note the check value.
4.	Tick the backup box and press Open this wallet, or switch to Open wallet and type everything back in.
5.	Open wallet shows the public address, public key and private key (hidden until you press Show). Press Lock wallet when done.
6.	Open the Self-test panel and press Run self-test. All 14 checks should pass.
If the page says the browser can't run the wallet, open the file from your computer in a current browser; the wallet needs the browser's built-in WebCrypto.

## Next build 2: Argon2id vault mode
Vault mode replaces PBKDF2's 2048 cheap rounds with memory-hard Argon2id for the add-ons, and it can only be recovered with software that implements this spec.

```S0 = BIP39 seed of the words with an empty passphrase      (64 bytes)

for each non-empty add-on i = 1..k, in order:
    ctx    = "TFSW-VAULT-v1" || 0x00 || byte(i) || S(i-1)
    salt   = SHA-256(ctx)
    K(i)   = Argon2id(NFKD(add-on i), salt, m = 65536 KiB, t = 3, p = 1, tag = 32 bytes)
    S(i)   = HMAC-SHA512(key = K(i), data = ctx)
seed = S(k)     then BIP-32 and BIP-44 exactly as in compatible mode
```

**•	Parameters:** 64 MiB, 3 passes, 1 lane, 32-byte output. The target is about one second per add-on on a mid-range laptop, which has to be measured before the numbers are frozen.

•	Compatible mode does not change. The wallet shows a mode label, and the recovery sheet prints "Mode: Vault v1" so the right software and version can be found later.

•	Hidden wallets come for free. Words plus add-on 1 only is one wallet, plus add-on 2 is another. A small balance in the first one works as a decoy.

•	Chaining does not multiply the attacker's cost. There is no checkpoint between stages, so a guesser pays about one Argon2id per full guess, the same as hashing the joined add-ons once. The strength gain comes from memory-hard hashing, and the chain adds the nested wallets.

**•	Libraries:** Argon2id from hash-wasm or argon2-browser, checked against the RFC 9106 test vectors before use.

**•	Screens:** a mode switch at creation with a clear "only this software can restore it" warning, and a progress bar while Argon2id runs.

**Done when:** _the spec is frozen as v1, vault vectors come from two independent Argon2id implementations that agree, derivation takes under two seconds on the reference devices, and the compatible vectors are untouched._

