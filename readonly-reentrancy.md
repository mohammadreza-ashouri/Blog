# Read-Only Reentrancy Vulnerability in the EVM

In this article I am going to talk about about the read-only reentrancy vulnerability in the Ethereum Virtual Machine (EVM), which is a nuanced type of security flaw that can affect smart contracts. Unlike classic reentrancy attacks that modify the state of a contract, read-only reentrancy doesn't change the state but manipulates reads and computations within a contract. This can occur when a contract calls another contract, such as through a view or pure function, and relies on that contract's result, without considering that the called contract might also interact with the original contract in a reentrant manner. The vulnerability can lead to unexpected behavior and erroneous computations, making it a subtle but potentially harmful issue. Okay, now let's dive into this!

## What is Reentrancy Vulnerability?
Reentrancy vulnerability is a well-known security issue in the context of smart contracts within the Ethereum Virtual Machine (EVM). This vulnerability occurs when a function of a contract calls an external contract and then allows further interaction with itself before its state is properly updated. If exploited, it can enable an attacker to repeatedly call into the vulnerable function, leading to actions such as the draining of funds or other unintended state manipulations. A classic example of this flaw is when a withdrawal function sends Ether before updating the balance, allowing a malicious contract to recursively call the withdrawal and siphon away more funds than intended.

### Example

```solidity
pragma solidity ^0.4.26;

contract Vulnerable {
    mapping(address => uint) public balances;

    function withdraw(uint amount) public {
        require(balances[msg.sender] >= amount);
        
        msg.sender.call.value(amount)("");

        balances[msg.sender] -= amount;
    }
}


In this code, Ether transfers before updating the balance, allowing a malicious contract to withdraw Ether, ignoring the balance check repeatedly.

## Read-Only Reentrancy Vulnerability
The read-only reentrancy vulnerability is a subtler form of reentrancy that doesn't change the state of the vulnerable contract but can still cause harm by manipulating its reads and computations. This vulnerability occurs when a contract calls another contract (e.g., via a view or pure function) and relies on that contract's result without considering that the called contract might also interact with the original one in a reentrant manner.


Here are some examples that read-only reentrancy can be quite harmful:

- Price Oracles and Data Manipulation: Many DeFi (Decentralized Finance) projects rely on external price oracles to fetch the current market prices of assets. If a contract is vulnerable to read-only reentrancy and relies on an external contract to get price information, a malicious contract could manipulate the returned price data. This could lead to incorrect pricing in trading functions, lending platforms, or derivatives contracts, causing users to buy or sell assets at distorted prices.

- Multi-Step Financial Transactions: In complex financial protocols involving multiple interacting contracts, read-only reentrancy could alter the logic flow. By manipulating the values read from a contract, an attacker might influence subsequent decisions in a transaction sequence, leading to financial gain in other parts of the system.

- Permission Checks and Access Control: If a contract relies on another contract to perform permission checks or access control (e.g., checking membership in a DAO or ownership of specific tokens), read-only reentrancy could manipulate these checks. An attacker could gain unauthorized access to privileged functions, which might enable them to execute trades, withdrawals, or other financial operations that should be restricted.

- Token Exchanges and Liquidity Pools: Many decentralized exchanges and liquidity pools use algorithms and functions that heavily depend on accurate data and computations. A read-only reentrancy attack manipulating these functions could lead to imbalances in the pools or unfair trading conditions, indirectly affecting users' financial positions.



### Example:

```
pragma solidity ^0.4.26;

contract Victim {
    uint256 public value;

    constructor(uint256 initialValue) {
        value = initialValue;
    }

    function getValueViaCaller(address caller) public view returns (uint256) {
        return caller.delegatecall(bytes4(keccak256("getValue()")));
    }
}

contract Attacker {
    Victim victim;

    constructor(address victimAddress) {
        victim = Victim(victimAddress);
    }

    function getValue() public view returns (uint256) {
        victim.value = 0; // Manipulate the value
        return victim.value;
    }

    function attack() public {
        victim.getValueViaCaller(this);
    }
}
```


In this code, the Victim contract relies on a call to another contract to get its value. If that contract is an Attacker contract, it can manipulate the value before returning it.

### How to mitigate the read-only reentrancy attack? 

Here are some strategies to help prevent read-only reentrancy attacks:

- Understanding Contract Interactions: Knowing how contracts interact with one another, especially when invoking view or pure functions from external contracts, is fundamental. Developers should carefully analyze the potential for reentrant behavior and the consequences of manipulated reads and computations.

- Avoiding Untrusted Calls: If possible, avoid making calls to untrusted contracts within view or pure functions. If these calls are necessary, understanding that they can potentially cause unexpected behavior is vital, and proper safeguards should be in place.

- Using Reputable Libraries and Patterns: Leveraging well-established libraries and coding patterns that have been audited and tested can help reduce the risk. Trusted libraries often include safeguards against known vulnerabilities, including read-only reentrancy.

- Implementing Access Controls: Restricting the ability to call certain functions to known, trusted addresses can mitigate the risk of an unknown attacker exploiting a vulnerability. By limiting who can interact with the sensitive parts of a contract, the surface area for potential attacks can be reduced.

- Conducting Thorough Code Reviews: Peer reviews and expert assessments can uncover subtle vulnerabilities that might escape automated analysis. Having multiple sets of eyes review the code can provide a deeper understanding of potential risks.

- Using Static Analysis Tools: Various tools are available that can analyze the code for known vulnerabilities, including read-only reentrancy. Using these tools as part of the development process can help identify potential issues early on.

- Testing with Adversarial Thinking: Simulating potential attacks and thinking like an attacker during the testing phase can uncover hidden vulnerabilities. Crafting tests that specifically target read-only reentrancy scenarios can help ensure that the code behaves as expected, even in adversarial conditions.

- Formal Verification: While more complex and time-consuming, formal verification can provide mathematical assurance that the contract behaves as intended under all possible conditions. It might be a suitable approach for particularly critical or high-value contracts.


## Conclusion
Understanding and mitigating vulnerabilities such as read-only reentrancy is crucial in the ever-evolving landscape of blockchain and smart contract security. Both classic and read-only reentrancy attacks pose significant risks, and developers must employ diligent practices to safeguard their contracts. For those seeking expert guidance in this intricate field, this is Dr. Mohammadreza Ashouri, a Ph.D. in cybersecurity with several years of experience in blockchain security audits, who offers in-depth insights and support. If you need assistance securing your smart contracts or navigating the multifaceted world of blockchain security, you can reach out to me at ashourics@gmail.com.


