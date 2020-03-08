# ethereum-lab1

Developing Blockchain Use Cases                      Spring 2020 Lab 1
Due: Monday, March 30, 2020                               10 Points

Learning Objective: In this lab the student will set up an Ethereum
development environment and deploy two smart contracts to an Ethereum
test blockchain - Ganache.

We will review the contracts in detail at a later date. For now, we
want to get some experience deploying and interacting with the code.


Part 1. Installations
=====================
1) Download and install Google Chrome.
2) Install the Metamask extension for Google Chrome.
   See: https://metamask.io/
3) Install git.
   See https://git-scm.com/downloads
4) Download and install node.js and npm.
   The node package manager (npm) is included with the download of
   node.js. See https://nodejs.org/en/download/
5) Download and install truffle.
   See: https://truffleframework.com/docs/truffle/getting-started/installation
   npm install -g truffle
6) Download and install Ganache.
   See: https://truffleframework.com/ganache
7) If you want to use Atom as your editor, download and install atom.
   See: https://atom.io
8) If you are using Atom, you may want to download and install
   language-solidity for Atom.
   See: https://atom.io/packages/language-solidity
9) If you are using Atom, you may want to download and install atom-solidity-linter
   for Atom.
   See: https://atom.io/packages/atom-solidity-linter

Part 2  (Modified from "Mastering Ethereum" by Antonopoulos and Wood)
======
1) Run ganache quick start and leave it running for the remainder of this part.
2) Create a new project directory named DBUC_Lab1_Part2 and cd into it.
3) Execute the command:
   truffle init
4) Examine the directory structure
     -- contracts holds solidity source code
           Migrations.sol   a deployment contract
     -- migrations
           1_initial_migrations.js
           This first migration deploys the Migrations.sol contract.
           In complex deployments, you only want to deploy changed
           contracts. truffle helps with that using
           migrations. Only code that has changed will be redeployed.
     -- test
           This directory is for writing tests in Javascript.
           Typically uses the mocha framework and chai assertions library.
           In more complex deployments, this directory structure will mirror
           the directory structure required by the application.
     truffle-config.js
           This file contains configuration parameters for truffle.

5) Create a new contract in the contracts directory. This file will be named
   Faucet.sol. The content of Faucet.sol is:
   // Solidity code Faucet.sol
   pragma solidity ^0.5.0;
   contract Faucet {
       function withdraw(uint withdraw_amount) public {
           require(withdraw_amount <= 100000000000000000000);
           msg.sender.transfer(withdraw_amount);
       }
       function() external payable {}
   }
6) Create a new migration file in the migrations directory. Do
   not remove the initial migration 1_initial_migration.js.
   This new file will be named 2_deploy_migration.js.
   This file will contain a migration script to deploy Faucet.sol.
   The content of 2_deploy_migrations.js is:

   // Javascript to deploy Faucet.sol
   var Faucet = artifacts.require("./Faucet.sol");
   module.exports = function(deployer) {
     deployer.deploy(Faucet);
   };

7) To create a package.json file, run the following command
   from within the project directory:
   npm init

   Take the suggested defaults.

8) From the project directory, run the command
   npm install dotenv truffle-wallet-provider ethereumjs-wallet
   This creates a node_modules directory.

9) To compile and deploy the two contracts (Migrations.sol and Faucet.sol)
   to the blockchain (represented by Ganache). Make sure that Ganache is running
   and execute the command:
   truffle migrate --reset

10)To interact with the deployed contract and accounts, use the truffle console.
   Execute the following commands:
   a. truffle console
   b. Access the contract with an asynchronous request. Use a
      callback function and a promise. The callback
      function is defined within the "then" clause.
      Execute the following command within the truffle console.
      Faucet.deployed().then(function(x){ myApp = x; });
      The response should be 'undefined'.
   c. To view the response enter the name myApp.
      myApp
   d. To get access to a web3 object, enter three lines of Javascript.
      The first two will return 'undefined'.
      var Web3 = require('web3');
      var web3 = new Web3(new Web3.providers.HttpProvider('http://127.0.0.1:7545'));
      web3.isConnected() // should return true if all three lines worked.
   e. Get the balance on the contract.
      contractBalance = web3.eth.getBalance(Faucet.address).toNumber()
   f. View the account addresses available on Ganache
      web3.eth.accounts
   g. View the first address. This address is our default address provided
      by Ganache. We are in possession of its private key. Ganache has
      provided this account with 100 eth (but only usable on Ganache).
      web3.eth.accounts[0]
   h. View the contract address.
      Faucet.address
   i. Using web3, transfer eth from the first account to the contract.
      The value returned is the transaction hash. Check the logs in Ganache.
      web3.eth.sendTransaction({from:web3.eth.accounts[0],to:Faucet.address,value:web3.toWei(0.5, 'ether')})
   j. Check that the balance on the contract is higher than before.
      web3.eth.getBalance(Faucet.address).toNumber();
   k. Withdraw some ether from the contract and deposit to account[0]. This returns undefined.
      Faucet.deployed().then(instance => {receipt = instance.withdraw(web3.toWei(0.1,'ether'))});
   l. But we can examine the returned receipt.
      receipt
   m. Check the balance on account[0]. Should be 99591514500000000000.
      web3.eth.getBalance(web3.eth.accounts[0]).toNumber();

11) At this point, take a screen shot of your Ganache Accounts, Blocks,
    and Transactions. Place these in a single Word or PDF document named
    Lab1Part2.doc or Lab1Part2.pdf and submit to Canvas.

Part 3  (Modified from "Mastering Ethereum" by Antonopoulos and Wood)
======
1) Run a new instance of ganache and leave it running for the remainder of this part.
2) Create a new project directory named DBUC_Lab1_Part3 and cd into it.
3) Execute the command:
   truffle init

4) Create a new contract named Faucet.sol.

5) Use this code for your improved Faucet:

// Solidity code Faucet8.sol from Pg. 150 Mastering Ethereum
// Modified for 5.0 and with comments.
   pragma solidity ^0.5.0;

   contract owned {
      address payable owner;
      // This constructor will be inherited.
      // It takes no arguments and so the
      // migration will be simple.
      constructor() public {
        owner = msg.sender;
      }
      // A modifier is a precondition. If the precondition is not satisfied then
      // the transaction will revert to its prior state.
      modifier onlyOwner {
        require (msg.sender == owner, "Only the creator of this contract may call this function");
        _;
      }
   }
   // Mortal inherits all features of the contract owned.
   // Mortals may die.
   contract mortal is owned {
     // only the creator may destroy this contract
     function destroy() public onlyOwner {
       selfdestruct(address(uint160(owner)));
     }
   }
   // Single inheritance
   contract Faucet is mortal {
       // Events end up in the receipt of the transaction.
       // The events may be examined by the contracts caller.

       event Withdrawal(address indexed to, uint amount);
       event Deposit(address indexed from, uint amount);

       // Withdraw by anyone as long as it's not greater
       // than 0.1 ether and as long as the contract has ether
       // to spare.
       function withdraw(uint withdraw_amount) public {
           require(withdraw_amount <= 0.1 ether);
           require(address(this).balance >= withdraw_amount, "Balance too small for this withdrawal");
           msg.sender.transfer(withdraw_amount);
           emit Withdrawal(msg.sender, withdraw_amount);
       }
       // Pay ether to the contract. Generate an event
       // upon deposit.
       function() external payable {
         emit Deposit(msg.sender, msg.value);
       }
   }

   6) Use Part 2 as a guide to deploy this new contract to Ganache.
      That is, repeat the following steps from Part 2: Step 6, 7,
      8, 9 and Step 10a to 10d - while replacing myApp with myFaucet
      in Step 10.

   7) Using the console and web3, view the generated events. Again, see part 2 for help with web3
      and promises.
      a. myFaucet.send(web3.toWei(1,"ether")).then(res => { console.log(res.logs[0].event)})
      b. myFaucet.send(web3.toWei(1,"ether")).then(res => { console.log(res.logs[0].event, res.logs[0].args)})

   8) At this point, take a screen shot of your Ganache Accounts, Blocks,
      and Transactions. Place these in a single Word or PDF document named
      Lab1Part3.doc or Lab1Part3.pdf and submit to Canvas.

Grading rubric
==============
    4 points for completion of Part 2
    1 point for correct submission of Part 2 (required Word or PDF document)
    4 points for completion of Part 3
    1 point for correct submission of Part 3 (required Word or PDF document)

Penalty for any late  work
==========================
    -1 point per day late
