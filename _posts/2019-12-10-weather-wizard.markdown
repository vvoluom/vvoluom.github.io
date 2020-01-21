---
title:  "Weather Wizard"
subtitle: "Weather Prediction Markets Powered by Chainlink"
image: "img/weatherwizard.jpg"
date:   2019-12-10 12:12:12
---

<img src="../images/weatherwizard/meteogram.png" alt="linearly separable data">

# Resources
The Github Repository can be found here : [Weather Wizard Repo](https://github.com/vvoluom/Weather-Wizard)  

# Inspiration

Prediction markets are common nowadays even in the world of crypto-currencies but even in this world of decentralized applications there is some sort of authority that users have to rely on to validate the correct outcome of a Question. What Chainlink and honeycomb solve is that exact problem. There is no longer a need to trust one singular source of data with the truth, just by using Chainlink's decentralized network of oracles and Honeycomb's extensive service line of API's. I've created a prototype to showcase the possibilities of what this technology is capable of and it's main focus is on solving humanities longest prediction problem which has affected our day to day plans throughout the ages, the Weather.

# Background
## Ethereum
[Ethereum](https://ethereum.org/) is a decentralised platform for smart contracts built using blockchain technology. It is currently run using a Proof of Work (POW) consensus mechanism where transactions are confirmed by miners solving cryptographic hashes to gain rewards. ETH is a cryptocurrency which is used to deploy, trade and run functions on the platform.

## Smart Contracts
Smart Contracts provide the ability to execute tamper-proof digital agreements, which are considered highly secure and highly reliable. Smart contracts on the Ethereum Blockchain are coded using the programming language [Solidity](https://solidity.readthedocs.io/en/v0.6.1/) which is high-level and object-oriented.

## Chainlink
[Chainlink](https://chain.link/) facilitates the retrieval of reliable and tamper-proof information from and to smart contracts. It does this in a decentralised way by sending and retrieving information from multiple nodes. The data that is received by the majority of the nodes is what the smart contract agrees on as the correct data. Chainlink uses LINK tokens as their cryptocurrency to execute data transactions on their nodes.

## Metamask
[Metamask](https://metamask.io/) is a bridge that allows you to visit the distributed web of tomorrow in your browser today. It allows you to run Ethereum dApps right in your browser without running a full Ethereum node.

## Honeycomb
[Honeycomb](https://honeycomb.market/) is a venture of CLC Group - a network of professional services and technological solutions focused on facilitating the 4th industrial revolution powered by smart contracts.

# How it works

<img src="../images/weatherwizard/coverphoto.png" alt="linearly separable data">

This is a prediction market where users can post a question about the future weather conditions of a city such as temperature, sunlight, snowfall etc. Once this question is posted other users can bet on the outcome of the prediction with regards to whether or not it will come true. For Example : "Will the average Degrees Celsius temperature in Malta on 12/12/19 be 14 Degrees Celsius or not?", once this question is posed users can now vote Yes/No on it by betting Ethereum thereby putting money where their mouths are. When the date passes 12/12/2019 the voting is stopped and then the answer can be calculated. The answer is calculated by retrieving data from the World Weather Online API through the use of Chainlinks decentralized oracle network and averaging all the retrieved results out. For example : 3 API's are pinged and the results are [13,14,15] -> the average of these will be 14 therefore the question is true. All the users who voted correctly can now withdraw their Ethereum plus winnings and the losers are left with nothing. This makes it an incentive for people to accurately predict the weather and improve the technology that does so, by creating a market to value such information.

## Contract

The Weather Wizard is built using the Factory Design Pattern where the Weather Wizard Factory produces Weather Markets containing a future question about the weather. The function below is invoked inside a Factory contract to produces a new market. The market produced needs the parameters below to make a prediction on the weather.

Details needed to determine the outcome of a Weather Market:
* _forecast : Question being asked with regards to weather E.G  "Will the average Degrees Celsius temperature in Malta on 12/12/19 be 14 Degrees Celsius or not?"
* _date : The Specific Date when the prediction is being asked for E.G "12/12/19"
* _location : The Location where the weather is measured E.G "Malta"
* _timePeriod : The period over which the weather is measured for E.G "A Day"
* _infoType : The Measurement used to quantify weather depending on question E.G fahrenheit,celcius avg,max,min,uvindex,sunHour
* _jobId : The Specified Job that will be run on the Oracle
* _oracle : The Specific Oracle that will verify the data being requested.
* _timeDays : Deadline till betting stops on the market and verification begins.
* _apis : A list of different data sources that will verify the information.

```java
   //Function used to create a new market and deploy the contract
    function createMarket(
        
        string _forecast,
        string _date,
        string _location,
        uint8 _timePeriod,
        string _infoType, //fahrenheit or celcius avg,max,min,uvindex,sunHour, 
        string _jobId,
        address _oracle,
        string _timeDays,
        string[] _apis

    )public returns(uint256){

        //Creates new market object here
        WeatherMarket newMarket = new WeatherMarket(
            _forecast,
            msg.sender,
            _date,
            _location,
            _timePeriod,
            _infoType, //fahrenheit or celcius avg,max,min,uvindex,sunHour, 
            _jobId,
            _oracle,
            _timeDays, 
            _apis
        );
        weathermarkets.push(newMarket);
        submissions[newMarket] = true;
		uint256 currentIndex = weathermarkets.length;
        
        emit newSubmission(address(newMarket));
        return currentIndex;
    }
```
Once created the address of the newely deployed contract is stored in an array containing all the markets and an event with the details of the market is emitted.
The code below shows the constructor of the newely created market. The data specified in the Factory contract is stored in two Structures a Market Information Sturct and a RequestInformation Struct.
```java
    constructor(

        string _forecast,
        string _date,
        uint _afterDays,
        string _location,
        uint8 _timePeriod,
        string _infoType, //fahrenheit or celcius avg,max,min,uvindex,sunHour, 
        string _jobId,
        address _oracle,
        string _timeDays,
        string[] _apis

    )public{

        mI.deploymentTime = block.timestamp;
        mI.afterDays = _afterDays;
        mI.Owner = msg.sender;
        mI.remainder = 0;
        mI.yesBetTotal = 0;
        mI.noBetTotal = 0;
        rI.forecast = _forecast;
        rI.location = _location;
        rI.date = _date;
        rI.infoType = _infoType;
        rI.jobId = _jobId;
        rI.oracle = _oracle;
        rI.timeDays =_timeDays;
        rI.released = false;
        rI.apis = _apis;
        bettingIsOpen = true;
        setPublicChainlinkToken();

    }
```

The Market Information Struct seen in the code below specifies the information pretaining to the betting market.

```c
    struct MarketInformation{
        address Owner;
        bool actualOutcome;
        uint afterDays;
        uint256 remainder;
        uint256 deploymentTime;
        uint256 yesBetTotal;
        uint256 noBetTotal;
        uint256 average;
        address[] betters;
    }
```

The Request Information Struct seen in the code below specifies the information with regards to the prediction being bet on.

```c
    struct RequestInformation {
        string forecast;
        uint256 prediction;
        string date;
        string location;
        address oracle;
        string jobId;
        string infoType;
        string timeDays;
        string[] apis;
        uint256[] results;
        bool released;
        uint8 trueCount;
        uint8 falseCount;
    }
```

Once a market is created the bettingIsOpen boolean is set to true which allows for the running of the placeBet() function below. This function takes in a boolean prediction signifying it a user believes that the question will result to a True or False. Apart from the boolean parameter, Ethereum is sent with the invocation of the function which is the bet that is being placed on that outcome. Two counters are kept check off the Yes Bet total and the No Bet Total which are incremented by how much Ethereum is sent.

```java
function placeBet(bool _prediction)
    public
    payable
    assertBettingIsOpen
{
    require(msg.value > 0);
    require(msg.value < 10 ether);
    require(totalContributions() + msg.value < 1000 ether);
    require(bets[msg.sender].amount == 0);

    bets[msg.sender].prediction = _prediction;
    bets[msg.sender].amount = msg.value;
    mI.betters.push(msg.sender);

    if (_prediction) {
        SafeMath.add(mI.yesBetTotal,msg.value);
    } else {
        SafeMath.add(mI.noBetTotal,msg.value);
    }

    LogBet(msg.sender, _prediction, msg.value);
}
```

Once the time limit is up no more bets are allowed to be processed. Therefore the contract can now retrieve information with regards to the actual real life outcome of the forecast question by using the requestConfirmations() function that can be seen below:

```java
//Depending on how the API needs to work.
//Information should be preset from before and uneditable
function requestConfirmations(string memory tx_hash)
public
assertBettingIsOpen
{
    //apiAddress = strConcat("https://blockchain.info/q/txresult/", tx_hash, "/", sellerTargetCryptoAddress, "?confirmations=3");
    //Reset them to 0 to be able to safely re-run the oracles
    rI.trueCount = 0;
    rI.falseCount = 0;

    //Loop to iterate through all the responses from different nodes
    for(uint i = 0; i < rI.apis.length; i++){
        Chainlink.Request memory req = buildChainlinkRequest(stringToBytes32(rI.jobId), this, this.fulfillNodeRequest.selector);
        string memory path = strConcat("data.weather.", rI.apis[i],".", rI.infoType, "");
        req.add("q", rI.location);
        req.add("date", rI.date);
        req.add("tp", rI.timeDays);
        req.add("copyPath", path);
        sendChainlinkRequestTo(rI.oracle, req, ORACLE_PAYMENT);
    }
}
```

This function builds a Chainlink Request with the specific details of the forecast which had been previously mentioned and sends those requests to the Oracle with a Specific Job and API. The response of the oracle is then received by the function fullfillnodeRequest() which accepts the result and stores it in the results array.

```java
    //This should fullfill the node request
    function fullfillnodeRequest(bytes32 _requestId, uint256 result)
    public
    recordChainlinkFulfillment(_requestId){
        rI.results.push(result);
    }
```

After all the requests have been sent and results received the outcome can be calculated using the function settleOutcome which iterates through all the results and returns an average, so if the results received are [13,14,15] -> the average of these will be 14 therefore the outcome of the forecast is True and the winners can withdraw their Ethereum.

```java
    //This will be replaced with the oracle confirmation
    function settleOutcome()
        public
        assertBettingIsOpen
        returns (bool success)
    {
        require(block.timestamp >= mI.deploymentTime + mI.afterDays);
        uint256 sum = 0;
        for(uint i = 0; i < rI.results.length; i++){
            sum += rI.results[i];
        }

        mI.average = sum / rI.results.length;
        if(mI.average == rI.prediction){
            mI.actualOutcome = true;
        }else{
            mI.actualOutcome = false;
        }
        bettingIsOpen = false;
        return true;
    }
```

The function below allows the winners to withdraw their earnings on the correctly places bets. The Requirements of the function check if the invoker has any bets placed and if their bet was correct, their winnings are then calculated and sent to their address.

```java
    function withdrawWinnings()
        public
        assertBettingIsClosed
        returns (bool success)
    {
        // Require a bet was made
        require(bets[msg.sender].amount > 0);
        // Require bet was correct
        require(bets[msg.sender].prediction == mI.actualOutcome);

        // Calculate winnings
        uint256 contributions = totalContributions();
        uint256 numerator = bets[msg.sender].amount * contributions;
        assert(numerator / contributions == bets[msg.sender].amount);
        uint256 denominator = mI.actualOutcome ? mI.yesBetTotal : mI.noBetTotal;
        uint256 winnings = numerator / denominator;
        // Move funds
        mI.remainder += numerator % denominator;
        msg.sender.transfer(winnings);
        return true;
    }
```

There are special functions for the Owner of the contract to withdraw the remainder of the winnings if they weren't exactly divisible between the Winners. This function can be seen below:

```java
    function withdrawRemainder() public assertBettingIsClosed returns (bool success){
        require(msg.sender == mI.Owner);
        require(mI.remainder > 0);
        uint256 amount = mI.remainder;
        mI.remainder = 0;
        msg.sender.transfer(amount);
        return true;
    }
```

# Conclusion
This was a small smart contract experiment to show how full decentralized and tamper proof prediction markets could be built using Chainlink Oracles and Ethereum Smart Contracts.