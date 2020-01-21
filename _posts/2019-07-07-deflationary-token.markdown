---
title:  "Deflationary Token"
subtitle: " Burning 10% of each transaction on each trade."
image: "img/deflationaryToken.jpg"
date:   2019-07-07 12:12:12
---

<img src="../images/deflationary/deflationary.png" alt="linearly separable data">

# Resources
The Github Repository can be found here : [Deflationary Token](https://github.com/vvoluom/DeflationaryToken)

# Introduction
Fiat currencies are inflationary meaning that Governments print more money and release it into circulation without an increase in production and services which causes prices to rise. This causes the supply of the currency to go up while the demand stays the same resulting in having a lower value of the currency. Governments also do not have a clear record of how much money is currently in circulation. An experimental token on the Ethereum Blockchain was created where on every transaction a percentage of that transaction was destroyed. This means on every transaction a percentage of tokens is removed from circulation forever decreasing the total supply making it more scarce thus more valuable.


# Background
## Ethereum
[Ethereum](https://ethereum.org/) is a decentralised platform for smart contracts built using blockchain technology. It is currently run using a Proof of Work (POW) consensus mechanism where transactions are confirmed by miners solving cryptographic hashes to gain rewards. ETH is a cryptocurrency which is used to deploy, trade and run functions on the platform.

## Smart Contracts
Smart Contracts provide the ability to execute tamper-proof digital agreements, which are considered highly secure and highly reliable. Smart contracts on the Ethereum Blockchain are coded using the programming language [Solidity](https://solidity.readthedocs.io/en/v0.6.1/) which is high-level and object-oriented.

# Methodology
## Smart Contract

First information with regards to the token must be defined which can be seen in the list below and then in the code afterwards.

Token Details
* tokenName : Name of the token.
* tokenSymbol : Symbol of the token.
* tokenDecimals : Number of Decimals a token can have E.G A value of 2 would allow for 0.01 token to exist.
* _totalSupply : Initial Token Supply, amount of tokens that will be produced.
* sellPercent : What percentage of tokens to burn from the Transaction.


```java
    string constant tokenName = "Deflationary";
    string constant tokenSymbol = "DGR";
    uint8  constant tokenDecimals = 0;

    uint256 _totalSupply = 2000000;
    uint256 public sellPercent = 1000;
```

On deployment of the contract the tokens are minted and sent to the Owner(Deployer) of the contract.

```java
    constructor() public payable ERC20Detailed(tokenName, tokenSymbol, tokenDecimals) {
        contractOwner = msg.sender;
        _mint(msg.sender, _totalSupply);
    }
```

Standard ERC20 functions are used in the contract which can be seen below:


```java
    /**
    * @dev Total number of tokens in existence
    */
    function totalSupply() public view returns (uint256) {
        return _totalSupply;
    }
```
```java
    /**
    * @dev Gets the balance of the specified address.
    * @param user The address to query the balance of.
    * @return An uint256 representing the amount owned by the passed address.
    */
    function balanceOf(address user) public view returns (uint256) {
        return _balances[user];
    }
```
The function findPercent below takes in a parameter value which is the amount of tokens that is going to be transfered and calcualtes how many tokens need to be burned from the transaction, it then returns the value that will be deducted from the transaction.
```java
    //Finding the a percent of a value
    function findPercent(uint256 value) internal view returns (uint256)  {
        //Burn 10% of the sellers tokens
        uint256 sellingValue = value.ceil(1);
        uint256 tenPercent = sellingValue.mul(sellPercent).div(10000);

        return tenPercent;
    }
```

Two transfer methods have been created, the simpelTranfer() method which is used by the Owner to distribute tokens to users without burning them and the normal transfer() function which burns 10% of tokens in each transaction. The simpleTransfer() function can be seen below:

```java
    //Simple transfer Does not burn tokens when transfering only allowed by Owner
  function simpleTransfer(address to, uint256 value) public isOwner returns (bool) {
    require(value <= _balances[msg.sender]);
    require(to != address(0));

    _balances[msg.sender] = _balances[msg.sender].sub(value);
    _balances[to] = _balances[to].add(value);

    emit Transfer(msg.sender, to, value);
    return true;
  }
```
The transfer function which burns tokens on transfer which is used by all other users. This function first checks if a user initiating the transfer has the amount of tokens they want to send in their account, the address can't be the contract itself and the minimum amount of tokens has to be 2 and above. If the amount of tokens being sent is below 10 then only 1 token will be burnt but if it is above 10 then 10% of the tokens will be burned in the transaction(Calculated using the findPercent() function mentioned above). Once the percentage is calculated it is deducted from the tokens to be transfered and from the total supply.

```java
  /**
  * @dev Transfer token for a specified address
  * @param to The address to transfer to.
  * @param value The amount to be transferred.
  */
  function transfer(address to, uint256 value) public returns (bool) {
    require(value <= _balances[msg.sender],"Not Enough Tokens in Account");
    require(to != address(0));
    require(value >= 2, "Minimum tokens to be sent is 2");
    uint256 tokensToBurn;

    if(value < 10){
        tokensToBurn = 1;
    }else{
        tokensToBurn = findPercent(value);
    }

    uint256 tokensToTransfer = value.sub(tokensToBurn);

    _balances[msg.sender] = _balances[msg.sender].sub(value);
    _balances[to] = _balances[to].add(tokensToTransfer);

    _totalSupply = _totalSupply.sub(tokensToBurn);

    emit Transfer(msg.sender, to, tokensToTransfer);
    emit Transfer(msg.sender, address(0), tokensToBurn);
    return true;
  }
```
A function was created to take in multiple addresses at once together with amounts to be transfered and transfer tokens accordingly by invoking the transfer() function on each pair of receiver and amount values as seen in the code below:

```java
//Transfer tokens to multiple addresses at once
function multiTransfer(address[] memory receivers, uint256[] memory amounts) public {
    for (uint256 i = 0; i < receivers.length; i++) {
        transfer(receivers[i], amounts[i]);
    }
}
```

# Conclusion 
This token is hyper-deflationary as it burns through 10% of a transaction that is being transfered minimizing the supply rapidly and increasing the it's scarcity. This was created as a proof of concept to demonstrate how a deflationary token would work.