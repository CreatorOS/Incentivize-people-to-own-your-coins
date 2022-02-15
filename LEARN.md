# Incentivize people to own your coins
In the previous quest, we’ve seen how to launch our own ERC20 cryptocurrency.

The only way for people to get the coins was either to mine them using the mine() function or to buy some off from a decentralized exchange like Uniswap.

But why should anyone own your coins? In this quest, we will look at how you can incentivize people to own your coin. If you buy early, you can sell for a higher price later - thereby making a profit on the trade. 

This quest will also lay the foundation for what is called cryptoeconomics. Cryptoeconomics is where the application drives desired behaviour by means of financial incentives. The desired behaviour for our coin, which doesn’t really do much, is that more people buy and sell our coin. If more people start buying, the value of the coins will go up. You own 1M coins, so you might be well off if the coin’s price increases :)

We might need some basic understanding of math here.

Now, on to building…
## Buying and Selling Coins
First up we’ll write a simple function, where if someone pays us in ETH, we’ll give them a corresponding number of coins. We’ll call this function mint. 

```

function mint() public payable {

  uint coinsToBeMinted = calculateMint(msg.value);

  assert(totalMinted + coinsToBeMinted = coinsBeingReturned);

  payable(msg.sender).transfer(weiToBeReturned);

  balances[msg.sender] -= coinsBeingReturned;

  totalMinted -= coinsBeingReturned;

}

```

Pretty straightforward, right?
Whenever someone sends ETH to this contract, we’ll convert it into coins using a logic called calculateMint().

Before minting, we need to make sure that the total number of coins in circulation should never be more than 10M that we had set earlier under totalSupply()

We’ll also add another function where a user can give back the coins to the contract and take back eth.

```
function unmint(uint coinsBeingReturned) public payable {
  uint weiToBeReturned = calculateUnmint(coinsBeingReturned);
  assert(balances[msg.sender] >= coinsBeingReturned);
  payable(msg.sender).transfer(weiToBeReturned);
  balances[msg.sender] -= coinsBeingReturned;
  totalMinted -= coinsBeingReturned;
}
```


`calculateUnmint()` will return the number of wei that should be transferred given the number of coins being returned. 

We of course need to check for the fact whether the user has as many coins as they’re trying to return.

Then we will reduce their balance and also reduce the number of coins that have been minted. 

total Minted denotes the number of coins in circulation, i.e. in the wallets of users. If coins are being unminted, they’re no longer in the accounts of users, and thus not in circulation.

## What do we want to incentivize?
In this quest, we want to incentivize people to buy the coins early and hold them. Whenever people hold coins, the price in exchanges tends to go up. 

So, the best behaviour is if people buy early, and/but sell later (or never). 

We’ll devise a crypto-economic model such that this behaviour translates to a financial incentive for abiding by this behaviour. 

Here’s a simple model :

- The sooner you mint, the cheaper it is to mint
- The later you unmint, the more eth you get back 

Let’s create a simple function for this. 

The price of 1 coin is directly proportional to the number of coins in circulation. 

Price (y) = constant (k) \* number of coins in circulation (x)

y = k\*x

So we have an equation. The curve for this on a graph is a straight line up to the top right.

This curve is called the bonding curve for our minting algorithm. A bonding curve is a curve that defines the price of a given coin subject to various market signals. In our case, we use only one market signal i.e. the total number of coins in circulation.

Subquest : Now a little bit of math

You can skim through this subquest if you have limited knowledge of math. But understanding this will really help you understand cryptoeconomics.

```

y = x

```

So, this defines the cost of buying 1 coin. How much will it cost to buy n coins? It will not be n\*y because the value of y itself is changing. 

Let’s look at an example.

Let us say k = 1

So, our bonding curve is y=x 

The first coin ever bought costs 0wei, because no coin exists in circulation.

How much will the second coin cost?

y = x. The number of coins in circulation is 1 (that was minted for price 0). 

The cost of the second coin is thus 1wei.

Similarly, the 3rd coin costs 2wei. 

And so on. 

But, how much does it cost to buy 3 coins?

It should cost 0 \+ 1 \+ 2.

Those comfortable with math will quickly spot that this is nothing but the area under the binding curve. So to get the price of buying 3 coins, we need to do integration on this function with the limits: number of coins in circulation before the minting to the number of coins after minting. 

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/18f6d287-b5b5-4c58-8c6e-f3b843a41cce.jpg)

So cost of buying n coins = ((number of coins in circulation \+ n ) ^ 2 - (number of coins in circulation) ^ 2) / 2

This also assumes that you can buy fractional coins, hence the integral.

So we now know the cost of buying n coins. However, what we need to know is how many coins should be minted given a certain amount of Wei. 

So, rearranging the above formula, 

(Amount in wei \* 2 \+ (number of coins in circulation before transaction)^2) = (number of coins in circulation before transaction \+ n) ^ 2

So, n = sqrt(Amount in wei \* 2 \+ (number of coins in circulation)^2) - (number coins in circulation before the transaction).

Solidity doesn’t have an inbuilt function for sqrt, so you can use this function instead. This calculates the square root using the Babylonian method. 
```
function sqrt(uint x) returns (uint y) {
    uint z = (x + 1) / 2;
    y = x;
    while (z < y) {
        y = z;
        z = (x / z + z) / 2;
    }
}

function square(uint x) returns(uint) {
  return x*x;
}
```


```
function calculateMint(uint amountInWei) returns(uint) {
  return sqrt(amountInWei * 2 + (totalSupply * totalSupply)) - totalSupply;
}
```

Now you need to calculate the amount of Wei that will be returned if n coins are returned. 
Can you try doing the math of what that output will be?

```
  // n = number of coins returned 
  Function unmint(uint n) returns (uint) {
    return (square(totalSupply) - square(totalSupply - n)) / 2;
  }
```
We recommend removing the initial grant to yourself. If you remember we had given ourselves a million tokens when the contract is deployed. Thereby making the price of the token prohibitively high for anyone to purchase.
[https://remix.ethereum.org/#version=soljson-v0.8.4+commit.c7e474f2.js&optimize=false&runs=200&gist=7b49601268222b71e2d4114897d5df47&evmVersion=null](https://remix.ethereum.org/#version=soljson-v0.8.4+commit.c7e474f2.js&optimize=false&runs=200&gist=7b49601268222b71e2d4114897d5df47&evmVersion=null)

    
## Subquest: Get your friends to buy this!

Deploy this to Ropsten and ask your friends to buy it before it gets too expensive!

Once it is deployed, you can create a link from where your friends can start interacting with the contract you’ve just deployed - using [https://ethcontract.app](https://ethcontract.app) 

Ethcontract.app requires 2 parameters to create a link

1. The address of the contract that you just deployed
2. The ABI of the contract. You can fetch this from the Remix Compile tab. Right under compile button, you’ll see a button to copy the ABI  
![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/2694193d-02e2-44a1-a41b-38e5ad5f84aa.jpg)

Then tap on explore contract. This will create a direct link that you can share with your friends. 

Remember to deploy on ropsten. Also, remember to tell your friends to use Ropsten!

They can tap on the mint button to mint coins for themselves!

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/ddedfcbd-5b46-4200-b8d8-9d95fc91c515.jpg)
## What next?
Now you know what is a bonding curve. Different bonding curves result in slightly different incentives. You can try playing around with various bonding curves. 

Bitclout uses a similar logic but uses a different bonding curve rather than our simple `y=x` curve. They use what is called the Bancor Curve. 

Can you modify the calculateMint() and calculateUnmint() functions to reflect a bancor curve instead of a straight line?
