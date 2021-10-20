# NFTs

The Ethereum website contains an excellent breakdown of all things NFT [Non-fungible tokens (NFT) | ethereum.org](https://ethereum.org/en/nft/)

# What is an NFT?

An NFT (Non-Fungible Token) is a token that conforms to the [ERC-721 standard](https://ethereum.org/en/developers/docs/standards/tokens/erc-721/). This is very similar to the ERC-20 standard (for cryptocurrency tokens), however, each non-fungible token is **unique** and can have a different **value** (measured in another ERC-20 token, usually ETH).

This introduces a concept of NFT **ownership** , since each token is unique and can only be held by a single address at one time. Furthermore, since each NFT is unique, it can be associated with and used to **uniquely identify some external data** (art, in-game items, lottery tickets, etc). This is the key point about NFTs, they provide a way to cryptographically prove ownership in a decentralised way.

The blockchain (Ethereum) is then used to hold the transaction history (ownership records) of these NFTs. An artist can **mint (create)** a new NFT (at that time they are the owner) from which other users can bid ETH to obtain the ownership of this newly minted token. The ownership of this token is transferred to the highest bidder, and the record of the transaction immutably recorded within the blockchain. Each token comes with a private key, which can be used to sign messages in order to verify that you are the genuine token holder.

## ERC-20 VS ERC-721:

- Every minted ERC-20 token has the same intrinsic value – e.g. 1 ETH held by user A is equivalent to 1 ETH held by user B.
  - ERC-721 token can have different intrinsic values (value measured relative to some ERC-20 token)
- ERC-20 tokens are featureless, the ETH held by users A, B, C are all identical
  - ERC-721 tokens (NFTs) are unique and can thus be associated with external data – e.g. NFT A corresponds to the image A and NFT B to image B etc.
  - Each token contains a unique tokenID that consists of serialised metadata that can be read by a smart contract to reproduce the data the NFT represents – e.g. an image, mp3, etc (NFTs minted on OpenSea support up to 100mb of data for example)

# NFT uses

NFTs are most useful for cases where ownership of a particular item needs to be verifiable and/or unique. Examples include

- Unique digital artworks – the &#39;ownership&#39; is of the original signed copy minted by the artist, the artwork itself can be reproduced, but only one user can verify they hold the &#39;original&#39; minted copy (largely a status symbol).
- Digital items such as in-game loot / unique or tradeable items
- Real world items such as
  - Event tickets
  - Lottery tickets
  - House or car rental deeds
  - Signed contracts

## Ideas for potential uses:

Private data storage – only the NFT key holders can access the contents

- Develop an NFT auth API that can be used in general cases to allow only users with the correct keys to access certain objects/data

NFT 2-fa : Two factor auth platform using NFTs – user has NFT keys which are used for authentication

# NFT exchanges

NFTs can be bought and sold on many exchanges, the most popular (and first) being [OpenSea.io](https://opensea.io/)

- Most popular and easy to use marketplace – anyone with an Eth wallet can buy/sell/create NFTs on this platform.
- Largest volume of users

NFT exchange market is highly saturated – many of the traditional currency exchanges are now incorporating an NFT exchange – room for a new exchange is limited.

## In-Game Items:

A popular sub category of NFTs are in game items, that can be traded just like regular NFTs. These NTFs are linked to in-game items (usually unique, with rarities scaling with price). Some games allow players to earn these NFTs to later be traded, or simply use the NFT model to allow players to purchase unique items.

Some popular games utilising the NFT model for in-game item ownership are

- [Upland](https://www.upland.me/) – Players purchase and trade properties that are tied to real world locations, using UPX tokens
- [Decentraland](https://docs.decentraland.org/) – VR world with locations/items/in-game features created and owned by users (somewhat like a crypto second life)
- [CryptoKitties](https://www.cryptokitties.co/) – Unique kittens that can be bred to create new combinations – tradeable with ETH

There are many others that follow a similar model.

### Market:

The market for these games is new and so there is potential for a new game to succeed.

### Implementation:

Many of these games will introduce their own token (that can be swapped for ETH for example) e.g. Upland and Decentraland each have their own token. This introduces a who extra layer of complication for creating a new game.

Other games will simply mint NFTs using the traditional marketplaces, where the owner of the NFT will have a private key that they can then use to access the in-game items.

### Side Market:

Since these games must either create their new token, or utilise an existing token for transactions, a potential market exists for an in-game item currency that can be used cross platform across multiple games that implement the token.

A market also exists for an easy to use interface for creating and linking NFTs to in-game items – e.g. a unity/unreal plugin. One example that already exists is ChainSafe (open source – minimal documentation) [Docs - ChainSafe Gaming SDK](https://chainsafe.github.io/game-docs/)

## Idea : Advertising:**

A blockchain based marketplace where advertisers can buy advertisement slots and users can advertise these slots (eg an in-game advert, website advert etc). Either with a new or existing token.

Advert interactions could be tracked via the blockchain, e.g clicks/views etc – and visible to the advertisers

Already exists: [Adshares](https://adshares.net/)

## Idea : Marketplace API:**

A general and easy to use api for integrating crypto payments into any existing website / marketplace.

Already exists: Coingate [Accept Bitcoin and other Cryptocurrency Payments | CoinGate](https://coingate.com/accept)
