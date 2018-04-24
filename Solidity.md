# Agenda
* answer questions on last lecture
* interoperability among Ethereum smart contracts
* show ERC20
* deploy ERC20 Cardinal Token on Rinkeby
* show balance on MetaMask
* fungible vs. non-fungible assets
* ERC721
* Cryptotrees
* Show final app
  * buy a few trees
  * Show genome explorer
  * remove tree body
  * show hue rotate
* Start fresh with only front end code
* build contract
* show concepts while building contract
* slack brainstorm on cryptotree functionality to be shown next week
  * Ideas for how the Cryptotrees game mechanics can work
  * Ideas for a cross campus Crypto-Wars game. E.g. Trees vs bears
  
---

# Questions from last lecture

* Office hours right after class

---

# Interoperability among Ethereum smart contracts

* One contract can call another
* Easier if standard funcion signatures
* [Ethereum ERCs](http://eips.ethereum.org/erc)
* ERC20 for tokens

---

# Make CardinalToken ERC 20

We will do that while learning solidity.

---
# Memory v.s. Storage variables
```solidity
contract C {
    uint[] x; // storage

    // memoryArray is in memory
    function f(uint[] memoryArray) public {
        uint memory y; // memory
        x = memoryArray; // works, copies the whole array to storage
        var z = x; // works, assigns a pointer, data location of z is storage
    }
}
```
## Arrays
* Storage arrays are dynamically resizable
* Memory arrays are not dynamically resizable

---

# Data types
* `bool`: The possible values are constants true and false.
* `int` / `uint`: Signed and unsigned integers of various sizes. Keywords `uint8` to `uint256` in steps of `8 bits` (unsigned of 8 up to 256 bits) and `int8` to `int256`. `uint` and `int` are aliases for uint256 and int256, respectively.
* `address`: Holds a 20 byte value (size of an Ethereum address). Address types also have members and serve as a base for all contracts.
* Fixed-size byte arrays: `bytes1`, `bytes2`, `bytes3`, …, `bytes32`. `byte` is an alias for `bytes1`.
* `bytes`: Dynamically-sized byte array,
* `string`: Dynamically-sized UTF-8-encoded string. 
  * As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.

For more [read the docs](http://solidity.readthedocs.io/en/v0.4.21/types.html)

---

# Functions
Can return a value
```solidity
function increment(uint x) returns (uint) {
    x += 1;
    return(x);
}
```
Can return *many* values
```solidity
function increment(uint x, uint y) returns (uint, uint) {
    x += 1;
    y += 1;
    return(x,y);
}
```
by specifying returned argument name we don't need to explicitly return

```solidity
function increment(uint x) returns (uint x) {
    x += 1;
}
```

---

# Function Declaration Modifiers
## `view`:  they promise not to modify the state.
* can read stage: storage variables
* Can not write to storage variables, emit events, send ether
* View functions execute locally on an ethereum node, not on blockchain
* Thus, they cost no gas

```solidity
uint y;

function increment(uint x) view returns (uint x) {
    x += 1;
    y += 1; // this line would fail
    // y is a state variable, and can't be changed in a view function
}
```
## `pure`: they promise not to read or modify the state.

like `view` plus cannot:
* read from state variables.
* access this.balance or `address`.balance.
* access  block, tx, msg (e.g. msg.sender)

```solidity
function f(uint a, uint b) pure returns (uint) {
    return a * (b + 42);
}
```

---

# Contract Inheritance

## abstract function
cannot be compiled, so used in base/abstract contracts that are then implemented

```solidity
contract Square {
    function square(uint x) returns(uint);
}
```

Declare inheritance with the keyword `is`.

```solidity
contract Double {
    function double(uint x) returns(uint) {
        return(x + x);
    }
}
```

A contract can inherit from multiple contracts
```solidity
contract MyContract is Square, Double {
    // Implement this function
    function square(uint x) returns(uint) {
        return(x * x);
    }

    // Override parent function
    function double(uint x) returns(uint) {
        // call immediate parent overriden function
        return(super.double(x));
    }
}
```

---

# Function visibiliy modifiers
* `public` - visible externally and internally (default)
* `external` - only visible externally
* `private` - only visible in the current contract
* `internal` - only visible in current contract, and those deriving from it

---

# Payable modifier
Can receive Ether
```solidity
    function someFunction() payable {
        ...
    }
```
---

# Fallback function
Default function if contracted called wrong or no function. If not defined an error is thrown. 
```solidity
function () public {
    ...
}
```
Frequently declared as `payable` to store any accidental Ether sent to the contract or to run ICOs.
```solidity
function () public payable {
    ...
}
```

---

# Make CardinalToken an ERC 20 token

```solidity
// code credit: https://hashnode.com/post/how-to-build-your-own-ethereum-based-erc20-token-and-launch-an-ico-in-next-20-minutes-cjbcpwzec01c93awtbij90uzn

pragma solidity ^0.4.4;

contract Token {

    /// @return total amount of tokens
    function totalSupply() constant returns (uint256 supply) {}

    /// @param _owner The address from which the balance will be retrieved
    /// @return The balance
    function balanceOf(address _owner) constant returns (uint256 balance) {}

    /// @notice send `_value` token to `_to` from `msg.sender`
    /// @param _to The address of the recipient
    /// @param _value The amount of token to be transferred
    /// @return Whether the transfer was successful or not
    function transfer(address _to, uint256 _value) returns (bool success) {}

    /// @notice send `_value` token to `_to` from `_from` on the condition it is approved by `_from`
    /// @param _from The address of the sender
    /// @param _to The address of the recipient
    /// @param _value The amount of token to be transferred
    /// @return Whether the transfer was successful or not
    function transferFrom(address _from, address _to, uint256 _value) returns (bool success) {}

    /// @notice `msg.sender` approves `_addr` to spend `_value` tokens
    /// @param _spender The address of the account able to transfer the tokens
    /// @param _value The amount of wei to be approved for transfer
    /// @return Whether the approval was successful or not
    function approve(address _spender, uint256 _value) returns (bool success) {}

    /// @param _owner The address of the account owning tokens
    /// @param _spender The address of the account able to transfer the tokens
    /// @return Amount of remaining tokens allowed to spent
    function allowance(address _owner, address _spender) constant returns (uint256 remaining) {}

    event Transfer(address indexed _from, address indexed _to, uint256 _value);
    event Approval(address indexed _owner, address indexed _spender, uint256 _value);

}

contract StandardToken is Token {

    function transfer(address _to, uint256 _value) returns (bool success) {
        //Default assumes totalSupply can't be over max (2^256 - 1).
        //If your token leaves out totalSupply and can issue more tokens as time goes on, you need to check if it doesn't wrap.
        //Replace the if with this one instead.
        //if (balances[msg.sender] >= _value && balances[_to] + _value > balances[_to]) {
        if (balances[msg.sender] >= _value && _value > 0) {
            balances[msg.sender] -= _value;
            balances[_to] += _value;
            Transfer(msg.sender, _to, _value);
            return true;
        } else { return false; }
    }

    function transferFrom(address _from, address _to, uint256 _value) returns (bool success) {
        //same as above. Replace this line with the following if you want to protect against wrapping uints.
        //if (balances[_from] >= _value && allowed[_from][msg.sender] >= _value && balances[_to] + _value > balances[_to]) {
        if (balances[_from] >= _value && allowed[_from][msg.sender] >= _value && _value > 0) {
            balances[_to] += _value;
            balances[_from] -= _value;
            allowed[_from][msg.sender] -= _value;
            Transfer(_from, _to, _value);
            return true;
        } else { return false; }
    }

    function balanceOf(address _owner) constant returns (uint256 balance) {
        return balances[_owner];
    }

    function approve(address _spender, uint256 _value) returns (bool success) {
        allowed[msg.sender][_spender] = _value;
        Approval(msg.sender, _spender, _value);
        return true;
    }

    function allowance(address _owner, address _spender) constant returns (uint256 remaining) {
      return allowed[_owner][_spender];
    }

    mapping (address => uint256) balances;
    mapping (address => mapping (address => uint256)) allowed;
    uint256 public totalSupply;
}

contract CardinalToken is StandardToken { // CHANGE THIS. Update the contract name.

    /* Public variables of the token */

    /*
    NOTE:
    The following variables are OPTIONAL vanities. One does not have to include them.
    They allow one to customise the token contract & in no way influences the core functionality.
    Some wallets/interfaces might not even bother to look at this information.
    */
    string public name;                   // Token Name
    uint8 public decimals;                // How many decimals to show. To be standard complicant keep it 18
    string public symbol;                 // An identifier: eg SBX, XPR etc..
    string public version = 'H1.0'; 
    uint256 public unitsOneEthCanBuy;     // How many units of your coin can be bought by 1 ETH?
    uint256 public totalEthInWei;         // WEI is the smallest unit of ETH (the equivalent of cent in USD or satoshi in BTC). We'll store the total ETH raised via our ICO here.  
    address public fundsWallet;           // Where should the raised ETH go?

    // This is a constructor function 
    // which means the following function name has to match the contract name declared above
    function CardinalToken() {
        balances[msg.sender] = 100000000;               // Give the creator all initial tokens. This is set to 1000 for example. If you want your initial tokens to be X and your decimal is 5, set this value to X * 100000. (CHANGE THIS)
        totalSupply = 2100000000;                        // Update total supply (1000 for example) (CHANGE THIS)
        name = "CardinalToken";                                   // Set the name for display purposes (CHANGE THIS)
        decimals = 2;                                               // Amount of decimals for display purposes (CHANGE THIS)
        symbol = "CDT";                                              // Set the symbol for display purposes (CHANGE THIS)
        unitsOneEthCanBuy = 10;                                      // Set the price of your token for the ICO (CHANGE THIS)
        fundsWallet = msg.sender;                                    // The owner of the contract gets ETH
    }

    function() payable{
        totalEthInWei = totalEthInWei + msg.value;
        uint256 amount = msg.value * unitsOneEthCanBuy;
        require(balances[fundsWallet] >= amount);

        balances[fundsWallet] = balances[fundsWallet] - amount;
        balances[msg.sender] = balances[msg.sender] + amount;

        Transfer(fundsWallet, msg.sender, amount); // Broadcast a message to the blockchain

        //Transfer ether to fundsWallet
        fundsWallet.transfer(msg.value);                               
    }

    /* Approves and then calls the receiving contract */
    function approveAndCall(address _spender, uint256 _value, bytes _extraData) returns (bool success) {
        allowed[msg.sender][_spender] = _value;
        Approval(msg.sender, _spender, _value);

        //call the receiveApproval function on the contract you want to be notified. This crafts the function signature manually so one doesn't have to include a contract in here just for this.
        //receiveApproval(address _from, uint256 _value, address _tokenContract, bytes _extraData)
        //it is assumed that when does this that the call *should* succeed, otherwise one would use vanilla approve instead.
        if(!_spender.call(bytes4(bytes32(sha3("receiveApproval(address,uint256,address,bytes)"))), msg.sender, _value, this, _extraData)) { throw; }
        return true;
    }
}
```

---

# [Demo] Cardinal ERC20 Tokens
* deploy ERC20 Cardinal Token on Rinkeby using remix
* show token balance on MetaMask

---

# Fungible vs non-fungible tokens
## [ERC721](http://erc721.org/)

---

# Cryptotrees
## [Demo]
* Show target app
  * buy a few trees
  * Show genome explorer
  * remove tree body
  * show hue rotate

---
# Grow new tree
Frontend:
```javascript
growNewTree = (address, tokenId, tree) => {
    const metadata = JSON.stringify(tree);
    return this.contract.methods.growNewTree(metadata).send({from: address});
}
```
Solidity (`Cryptotrees.sol`):
```solidity
    struct CryptoTree {
        string genome;
        uint64 hangoverEndTimestamp;
        uint64 hangoverDuration;
    }

    // Map trees ids to the CryptoTree struct containing this tree's data
    mapping (uint => CryptoTree) public trees;

    uint[] public existingIds;

    function growNewTree(string _metadata) public returns(uint tokenId) {
        tokenId = totalTokens + 1;
        _growNewTree(msg.sender, tokenId, _metadata);
    }

    function _growNewTree(address beneficiary, uint tokenId, string _metadata) internal {
        _addTokenTo(beneficiary, tokenId);
        totalTokens++;
        existingIds.push(tokenId);
        trees[tokenId].genome = _metadata;
        tokenOwner[tokenId] = msg.sender;
        emit Created(tokenId, beneficiary, _metadata);
    }
```

--- 

# 

Frontend:
```javascript
getMyTrees = async () => {
    const address = await getAccount();
    return this.contract.methods.getAllTokens(address).call({from: address});
}
```
Solidity (`BasicERC721.sol`):
```solidity
    uint public totalTokens;

    // Array of owned tokens for a user
    mapping(address => uint[]) public ownedTokens;
    mapping(address => uint) _amountOwned;
    mapping(uint => uint) _tokenIndexInOwnerArray;

    // Mapping from token ID to owner
    mapping(uint => address) public tokenOwner;

    function getAllTokens(address owner) public constant returns (uint[]) {
        uint size = _amountOwned[owner];
        uint[] memory result = new uint[](size);
        for (uint i = 0; i < size; i++) {
            result[i] = ownedTokens[owner][i];
        }
        return result;
    }
```

--- 

# Get tree genome (ERC721 calls it Metadata)

Frontend:
```javascript
getTreeMetadata = async (treeId) => {
    const jsonTree = await this.contract.methods.genome(treeId).call();
    return JSON.parse(jsonTree);
}
```
Solidity:
```solidity
function genome(uint id) public view returns (string) {
    return trees[id].genome;
}
```

--- 

# 

Frontend:
```javascript
getAllTrees = async () => {
    const ids = await this.contract.methods.allTrees().call();
    return ids;
}
```
Solidity:
```solidity
function allTrees() public returns (uint[]) {
    return existingIds;
}
```

--- 

# 

Frontend:
```javascript
getPartyingTrees = async () => {
    const address = await getAccount();
    const ids = await this.contract.methods.availableToParty().call({from: address});
    return ids.filter(i => i !== "0");
    // Excluding all ids that are === "0" (those are non-partying trees)
    // We're using "0" (string) because web3 returns string instead of numbers...
}
```
Solidity:
```solidity
    function availableToParty() public returns (uint[]) {
        uint[] memory result = new uint[](totalTokens);
        uint counter = 0;
        for (uint i = 0; i < totalTokens; i++) {
            if ((ownerOf(existingIds[i]) != msg.sender) && !isHungover(existingIds[i]) ) {
                result[counter] = existingIds[i];
                counter++;
            }
        }
        return result;
    }
```

---

# CryptoTrees Brainstorming
* game mechanics
* crypto-wars

---
