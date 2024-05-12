# Manage and develop a strategy for gas
<details>
  <summary>

  ## Preface
  </summary>

**Target audience** :

Experienced FunC developers who already know FunC basics

**Purpose:**

The TON smart contract gas model is unique and quite different from the EVM model. Contract developers must design a gas strategy. If they don't do this well, the contract can run out of TON balance for rent and be removed. Messages that the contract sends might not have enough gas and be rejected. So the purpose of this article is to help FunC developers manage gas and develop a strategy for gas usage to prevent pitfall.

</details>
<details>
  <summary>
  
## Introduction :
</summary>

TON is almost a new platform, It needs to devote time and energy to understand it. One key concept in Blockchain part that is major difference in contrast to other 
known blockchain is the role of Gas. 

Gas in TON is both an indicator of efficiency and a guideline and guard line of development processes. It may be times you think about a solution that may result in high gas consumption. This is a symptom of revising your solution and studying more to figure out how this novel system works as a whole.
You may conduct a solution that has a problem with the principle of who will be in charge of payment. If you plan to have a million users
but one smart contract is in charge of all payments, this may be a fault.
In TON, as smart contract developers, we must pay for both data and instruction smart contract has, it is obvious
that putting less data and utilizing smooth simple instruction is saving money. it is intentionally designed for light
overhead, resulting in speed and scalability.
So it is not a surprise if you hear smart contract contests awarded with low gas consumption.

To demonstrate why gas consumption in TON is an indicator of efficiency, we are going to inspect two versions of one famous
smart contract.
Version 5 of Tonkeeper Wallet in contrast to version 4. This upgrade has 93% lower storage fees, just by utilizing offloading
the code into a shared library on Masterchain.

Gas price, rent price, and forwarding fees are necessary to protect the network against spam and reward validators for keeping consensus
running. This is of the utmost importance because TON is a public resource shared by everyone. It is our duty as developers to make
efficient use of this resource and produce the most optimal software possible.

Till now, in the TON document, there is some content addressing gas-related issues.

For example, [here](https://docs.ton.org/develop/smart-contracts/fees) concept of gas was introduced and transaction fees were discussed. By reading it we
can figure out what elements of transaction fees are, what formula they utilize for calculation, and how storage fees will be calculated.

And [here](https://docs.ton.org/develop/howto/fees-low-level), provided a comprehensive overview of the low-level fees associated with transactions on the TON blockchain. The information covers transactions and phases, computation fees, gas costs, TVM instructions costs, and fee calculation formulas for storage, forwarding, and actions.

This detailed documentation is valuable for developers and users who want to understand the inner workings of fee structures on the TON blockchain, especially when dealing with FunC code and low-level interactions.

In addition to two previous resources, there is a dedicated directory with some useful information [here](https://docs.ton.org/develop/smart-contracts/guidelines).
It is recommended you first read those resources and then come back here since this article writes upon those concepts.

### How is this text organized? :

How we can address one indicator of efficiency(gas) in such a complex system (TON), without knowing how this system works?
To formulate an effective strategy for managing gas within our smart contract, it is imperative to comprehend the workings of the TON blockchain, with a specific focus on the role of gas in the overall system.
The blockchain industry has suffered from one significant problem for years. 
TON, Solve this issue, known as scalability.
There is a new and novel perspective on this solution that should be considered.  

We have some years of experience from the past, for example, we have lessons from bleeding hard forks, so we design systems around reconfigurability parameters.

TON is not just blockchain.TON is the platform.
We have had some concepts in computer science, for example, the actor model in functional programming like Erlang, we have stack machine concept in Forth, and sharding concept in database management as well.
We have async programming as well. The admiration comes from the fact that putting everything from the past together and developing paradigm-changing tools.

We use gas in this text as a pivotal engineering concept, using it, we explore parts of the system to reach overal understanding of the whole system.

</details>

<details>
  <summary>

## Origin of Gas :

</summary>

## General Idea

The TON Virtual Machine (TVM) serves as the environment where smart contract gas fees are incurred. To facilitate a clearer understanding of the subject, we have categorized gas-related aspects into three segments.

![Gas Partition Diagram](assets/gas-partition-diagram.png)

- **Gas Storage Fee**
  This part deals with spaces that smart contracts occupy in the network and is applicable to every smart contract. We also call it a rent fee. In simple words, No one can put everything they want on the network and system without payment. This fee has an important effect, in preventing system abuse.
  We measure space as bit, and time as delta to calculate this fee. We have one simple data structure known as a cell.
  
- **Gas Computation Fee**
When we talk about Gas in general, we talk about this part. In fact the complex and most important gas fee is here. And deserve more attention.
Every calculation, and commputation in our code has a fee. We call them instructions, and we have big column explaining them in TON document website, here. 
This part deals with business logic, data structure, and algorithms.
 
- **Gas Communication Fee**
  This part as its name indicates, is related to communication and message passing. Messaging concept in TON pillar of scalability. We have no way but sending message to do any operation, and it has a fee.

 For every part, we try to address some common interesting aspects: 

 - Theory and Idea,
 - What is the formula, What is involved
 - Who is in charge
 - When the fee is deducted or what is the process
 - Preventing Pitfalls
 


 </details>


<details>
   
<summary>
 
 ## Storage
 </summary>
 Theory :
 Every Smart contract must pay rent based on the space it occupies as bit and time that it exists on Blockchain. In simple terms we can reduce the storage fee concept as follows :
 
 **used storage**:
```cpp
bytes * second
```
 If we are eager to know more details, here it is :
```cpp
  storage_fee = (cells_count * cell_price + bits_count * bit_price)
  / 2^16 * time_delta
```

Let's examine each value more closely:

- `price`—price for storage for `time_delta` seconds
- `cells_count`—count of cells used by smart contract
- `bits_count`—count of bits used by smart contract
- `cell_price`—price of single cell
- `bit_price`—price of the single bit
In TON we have a data structure known as a Cell, everything is composed of this entity. 

Who is in charge of rent payments:

Every smart contract is responsible for its storage fee.   

Process and billing payment:

When a smart contract receives a message for processing, the first thing is, that TVM looks at some data that by design is available in every contract,
for example, what is the last payment, and how much fee does this smart contract owe?
according to this information, the amount of rent will be calculated.

Every smart contract may have one of three statuses, uninitialized, active, and frozen. 



This is one pitfall that may be we trapped into. if our smart contract has no appropriate balance for payment, the message is discarded.   


This characteristic aspect of smart contracts that is responsible for their storage payment is a fact that we should keep in mind in the designing of our system. For example, we developed a program that, every user can have their own smart contract in case the user wants to make use of our system. In this way, it is reasonable that the user is in charge of storage payment. This simple technique may prevent the population of our system in the bottom line from scams as well.

</details>
<details>
  
<summary>


## Computation

</summary>
All computations take place in a sandbox environment known as a TON virtual machine.

<details>
  <summary>
    
  #### TON Virtual Machine (TVM) Overview    
  </summary>


The TON Smart Contracts operate on the TON Virtual Machine (TVM), utilizing the stack principle for efficiency and ease of implementation.

This virtual machine is crafted specially for smart contract processing. All data structure used in the environment is made of Cells. We can consider all information presented in TON Blockchain as a database compromised as Cell. So every record in this database has a unique index known as a hash of Cell. Why knowing this fact matters, because we have access to this context in **TVM** .
Another aspect of this place where all processing takes place (TVM) is the stack-oriented characteristics. We can consider This place as a function that received the previous state and messages as input and, the new state and outgoing message as output.
 It means it is impossible to have two states identical.
knowing this fact is crucial because this characteristic has two important consequences. First, we can consider even one smart contract as one separate Blockchain. Second This aspect by itself prevents double spending.


As all we know hash function that is pillar of the blockchain industry works as follows.
we can not find two identical series as bits that result in the same hash. So as we all know one of the data presented in each smart contract is the public key of the owner. This public key itself is a hash of random bits that are resilient enough to be considered by others.
So we always have distinct series of bits, that result in a completely different hash.

When the message arrives TVM is initialized, and all data and instruction that was saved in this smart contract, are available for processing.
The first thing smart contract should pay for rent, by its balance. If here balance is insufficient message processing is discarded.
the second phase is credit. It means the balance accompaniment with a received message will added to the balance of the smart contract.
after this phase computation by watching credit starts, will result in an error or new state and maybe an outgoing message.

from this point on, every instruction has its own gas fee. It means dealing with a large array of data has more expense than dealing simple data structure.

#### Transactions and Phases

When events occur on an account in a TON chain, it triggers a transaction. Transactions consist of up to five phases:

1. **Storage Phase**: Calculates storage fees.
2. **Credit Phase**: Calculates the contract balance considering incoming message value and storage fees.
3. **Compute Phase**: Executes the contract, yielding results like `exit_code`, `actions`, etc.
4. **Action Phase**: Processes actions if the compute phase is successful.
5. **Bounce Phase**: Forms a bounce message if the compute phase fails.

#### Compute Phase

The compute phase is where TVM execution occurs, and the TVM state is crucial in this process.

#### TVM State

The TVM state comprises:
- **Stack**: Last-input-first-output stack machine.
- **Control Registers**: Up to 16 variables are directly set and read during execution.
- **Current Continuation**: Describes the currently executed instruction sequence.
- **Current Codepage**: The version of TVM in use.
- **Gas Limits**: Four integers representing current, maximal, remaining gas, and gas credit.
- **Library Context**: HashMap of libraries callable by TVM.

#### TVM as a Stack Machine

TVM operates as a stack machine with variable types like Integer, Tuple, and Null, and distinct cell flavors like Cell, Slice, Builder, and Continuation.

#### Control Registers

Notable control registers include `c0` (next continuation), `c4` (root of persistent data), `c5` (output actions), etc.


</details>

Now let's look at TVM initialization more closely:
<details>
  

<summary>

#### Initialization of TVM

</summary>

## TVM Initialization Process

### Control Registers Initialization

1. **Current Continuation (cc):** Initialized using the cell slice from the `code` section of the smart contract. If the account is frozen or uninitialized, the code must be supplied in the `init` field of the incoming message.

2. **Current TVM Codepage (cp):** Set to the default value, 0. Can be switched using `SETCODEPAGE` if needed.

3. **Gas Values:** Initialized according to Credit phase results.

4. **Libraries (Library Context):** Computed based on the global library environment, local library environment of the smart contract, and the `library` field of the incoming message.

5. **Stack Initialization:** Depends on the event causing the transaction (internal message, external message, tick-tock, split prepare, merge install).

6. **Control Registers c0 to c5:** Initialized with specific continuations and data related to the smart contract's code, data, and actions.

### Library Context

- The library context is a hashmap mapping 256-bit cell hashes to the corresponding cells. It is computed by combining the global library environment, the local library environment of the smart contract, and the `library` field of the incoming message.

### Stack Initialization

- Stack initialization varies based on the transaction event:
  - Internal Message: Initializes stack with smart contract balance, inbound message details, and function selector.
  - External Message: Similar to internal message with modifications for external messages.
  - Tick and Tock: Initializes stack with account balance, address, transaction type, and function selector.
  - Split Prepare: Initializes stack with account balances, split information, addresses, and function selector.
  - Merge Install: Initializes stack with balances, message, state, split information, addresses, and function selector.

### Control Register c5 (Output Actions)

- Accumulates output actions in a linked list structure.
- Possible actions include sending messages, setting opcode, reserving currency, and changing the library.

### Control Register c7 (Temporary Data)

- Contains the root of temporary data as a Tuple, including blockchain context data such as time, global config, actions, messages sent, logical times, balance, address, and global config.


</details>
  <details>
  
<summary>
  

#### TVM Instructions

</summary>
Explore the [list of TVM instructions](https://docs.ton.org/learn/tvm-instructions/tvm-overview) for a comprehensive understanding.

### Result of TVM Execution

In addition to `exit_code` and consumed gas data, TVM indirectly outputs:
- `c4` register: The cell is stored as new `data` of the smart contract.
- `c5` register: List of output actions, recursively referencing the last action.

This overview provides a foundational understanding of TON Smart Contracts' execution on TVM.



</details>

<details>
  
<summary>
  
#### Accept Message Effect
</summary> 
   The `ACCEPT` instruction in the context of smart contracts is a fundamental operation related to gas management. Gas is a unit that represents the computational resources consumed by the execution of instructions in a smart contract on the TON blockchain.
   
   Here's a breakdown of the `ACCEPT` instruction:
   
   - **Purpose**: The primary purpose of the `ACCEPT` instruction is to signal the agreement of the smart contract to allocate additional gas for the continuation of the current transaction. External messages, which may not bring any value or gas with themselves, often require the smart contract to allocate gas for their processing.
   
   - **Gas Limit Adjustment**: The instruction sets the current gas limit (`g_l`) to its maximal allowed value (`g_m`). Additionally, it resets the gas credit (`g_c`) to zero. The gas credit represents the accumulated unused gas from previous computations. The gas reserve (`g_r`) is then decreased by the gas credit. In other words, the smart contract agrees to buy some gas to complete the current transaction.
   
   - **Gas Credit Reset**: By resetting the gas credit to zero, the smart contract ensures that only the gas allocated explicitly using `ACCEPT` will be considered for the current transaction. This is crucial for managing gas consumption accurately.
   
   - **Exception Handling**: If the gas consumed so far (including the present instruction) exceeds the resulting value of `g_l`, an unhandled out-of-gas exception is thrown before setting new gas limits. This ensures that the execution is stopped if the allocated gas is not sufficient to complete the transaction.
   
   - **External Message Processing**: External messages, which are often used for communication between smart contracts, may not provide gas or value. The `ACCEPT` instruction allows the smart contract to allocate the necessary gas to process these messages effectively.
   
</details>

</details>


<details>
  <summary>


## Communication

</summary>
  Message from outside of Blockchain can not bear value. So it can not pay any fee. Hence when a message arrives from the outside world, it is the duty of smart contracts to pay.
If a smart contract does not have enough balance to pay for this transaction, the received message will be discarded.
So as developers, we should take care if our code is meant to process external messages.

Any smart contract by its nature is responsible for its storage rent.
Suppose a smart contract that is compiled, has been deployed, and also has a balance for payment but suffers from
business logic that results in an infinitive loop. In this case, we should have a mechanism to prevent breakdowns. This is a role of the gas_limit configurable parameter. These days gas_limit is 10k gas or In other words 10k \* 10k naton(nano ton) (the second 10k is currently the gas price in nano, also configurable).
So gas_limit as its name indicates is the most upbound instruction fee that smart contract can pay in just one round transaction. In fact, this is a safety guard.
At the beginning of processing external messages, gas_limit is set to zero, and the balance of the smart contract acts as gas_credit, in case the balance is zero or not equal to the processing transaction fee, the message will be discarded. This is the place we should care about our smart contract balance.

---

### External Messages:

The gas*limit for external messages is initially set to gas_credit (ConfigParam 20 and ConfigParam 21), which is 10k gas.
To process the message, a contract should use accept_message to set the gas limit. Failure to do so may result in the message being discarded if gas_credit is reached or computation is finished without calling accept_message.
After the transaction ends, full computation fees are deducted from the contract balance based on the new gas limit.
If an error occurs after accept_message, the transaction is written to the blockchain, fees are deducted, but storage is not updated, and actions are not applied. This can lead to repeated acceptance of the same message until the contract balance is depleted.
So what should we do ? \_TODO*

---

### Internal Messages:

By default, the gas limit for internal messages is set to message*balance/gas_price, with the message paying for its processing.
The contract can use accept_message/set_gas_limit to change the gas limit during execution.
Manual gas limit settings do not interfere with bouncing behavior. Bounceable messages will bounce if sent in bounceable mode and contain sufficient funds for processing and creating bounce messages.\_TODO*
why ? What ? How ?

bounceable and non-bounceable internal messages
Bounceable and non-bounceable internal messages are concepts in smart contract development that relate to how messages are processed and handled between contracts on a blockchain.

1. **Bounceable Messages:**

   - Bounceable messages have their "bounce" bit set. If the destination smart contract does not exist or encounters an unhandled exception while processing the message, the message is "bounced" back.
   - The bounced message contains a 32-bit `0xffffffff` followed by the original 256 bits, with the "bounce" flag cleared and the "bounced" flag set.
   - Smart contracts should check the "bounced" flag of inbound messages. They can either silently accept the message (by terminating with a zero exit code) or perform special processing to identify the failed outbound query.
   - The query in the body of a bounced message should not be executed.

2. **Handling Bounced Messages:**

   - Smart contracts need to check the "bounced" flag of inbound messages to handle bounced messages appropriately.
   - Silently accepting bounced messages with a zero exit code is a common practice, or special processing can be implemented based on the contract's requirements.

3. **Non-Bounceable Messages:**

   - On certain occasions, non-bounceable internal messages are necessary. For example, new accounts cannot be created without at least one non-bounceable internal message.
   - Unless this message contains a `StateInit` with the code and data of the new smart contract, having a non-empty body in a non-bounceable internal message does not make sense.

4. **Best Practices and Considerations:**
   - It is advisable not to allow end users to send non-bounceable messages containing large amounts of value. A better practice is to send a small amount first, initialize the new smart contract, and then send a larger amount.
   - This precaution helps mitigate potential risks associated with large transactions and ensures a smoother initialization process for new smart contracts.

The smart contract relies on messages to interact, and these messages trigger transactions that modify an account's state. There are three types of messages: inbound external, internal, and outbound external messages.

- **Inbound External Message:** Initiated from outside the blockchain, these messages don't have a 'from' address and can declare intent to transfer value to another account.

- **Internal Message:** Sent from one contract to another, and it can be a value-bearing message, updating the state of the smart contract.

- **Outbound External Message:** Emitted by a smart contract, it can be subscribed to by off-chain participants, known as "messages to nowhere."

The structure of a message includes a 'header' with sender and receiver information and a 'body' containing virtual machine instructions for smart contract execution.

A transaction is a result of processing an inbound message. It involves multiple phases:

1. **Credit Phase:** Adds the value of the received internal message to the account's balance.

2. **Storage Phase:** Collects storage payments for the account state, freezing the smart contract if the balance is insufficient.

3. **Computing Phase:** Executes the smart contract code, leading to a new state, gas payment, and an action list of outbound messages.

4. **Action Phase:** Performs actions from the list if the smart contract terminates successfully.

5. **Bounce Phase:** Triggered when a transaction is aborted, involving generating an outbound message and sending it back to the original sender.

Various fees are incurred during these phases.
~~like incentivizing validators, maintaining network operation, and preventing spam.~~

Note: Only internal messages can transfer value, and fees are deducted from the account balance after the credit phase.
Because external messages can not bear value, so we should start with an internal message, a message within Blockchain, from one smart contract to another.
A smart contract that starts sending a message, is in charge of paying gas, so to accurately calculate the amount of value to attach to a message in a TON Blockchain smart contract, we should Understand Gas Costs:

~~- Gas price in TON is constant, and additional threads are added with increased load.
Calculate the gas cost for each action in the contract. -~~
gas costs are fixed, but if the contract involves storage operations with dynamic-sized types, costs may increase logarithmically with storage size.

Dynamic-sized types (arrays, mappings, strings) increase gas costs logarithmically with storage size.
Avoid using mappings, arrays, and mutable strings if possible.
If dynamic-sized types are used, calculate the cost of operations with a large margin for a worst-case scenario (O(log n)).
Calculate Storage Fees:

Before the compute phase, a storage fee for code + data is charged from the account balance.

Reserve enough TON to ensure the contract lives as long as needed.
Calculate Value for Message:

Reserve additional TON with a margin for potential errors, especially in the action phase.

Consider Action Phase Errors:

In case of errors in the action phase.
Ensure that enough value is attached for rawReserve or send a message to avoid potential issues.
External Messages and Events:


External messages (e.g., Event(some_data)) are sent from the contract to nowhere.
Creation of external messages is paid from the contract's balance.
Use tvm.rawReserve before creating external messages to account for potential costs.
By following these steps, you can accurately calculate how much value needs to be attached to a message in a TON Blockchain smart contract, considering various factors such as gas costs, storage fees, and potential errors in the action phase.


</details>
<details>
  <summary>
    
  ## Design Pattern, Who is in charge?
  
  </summary>

Every smart contract, can send a message to the validator, and ask for initializing another smart contract. Smart contract address is based on initial data and initial code. It means before initialization its address is known and available for us, and we can save it on our smart contract registry. What does it mean and how it can help us develop our system?
well, suppose a smart contract which meant to serve for million users, it means there is no need to do every calculation and every communication by itself. The address of the other contract itself can act like a unique entity or in simple words index and identity of the user who wants to use our service. So master contract can extract every account public key which sends a message for it and initialize another smart contract as outbound internal messages to network. Newly created smart contracts with predefined codes and data base on our service, now reside in blockchain and can do consequence transaction base on the private key of the user.
In this case we do not populated our main smart contract with unnecessary data and also prevent heavy calculation in  future , simply with this simple trick we save significance gas fee.More than this we measure security as well.Newly created smart contract rent fee in deducted from new user, this simple fact can prevent us from bad actor who want to abuse our system.
In overall big picture storage fee in most fewer than computational fee in long run.So we should consider this fact that every work chain can have to up to 2 powered by 60 number of account. This means we have one hundred thousand number of possible account address for population of nowadays world.
So be generous for creation.

</details>

<details>
  <summary>
    
  ## Conclusion
    
  </summary>
## Optimizing Gas Usage in TON Smart Contracts


Gas usage optimization is crucial for efficient and cost-effective TON smart contract development. Developers can follow these best practices to optimize gas usage, considering the role of the `ACCEPT` instruction:

1. **Fine-Tune Gas Limits:**
   - Developers should carefully analyze the computational requirements of their smart contracts and set appropriate gas limits. Adjusting gas limits can prevent unnecessary gas consumption and ensure optimal resource allocation.

2. **Strategic Use of `ACCEPT` Instruction:**
   - Utilize the `ACCEPT` instruction judiciously, especially in scenarios where additional gas is required for external message processing. Strategic use ensures that gas is allocated dynamically, avoiding unnecessary overhead.

3. **Gas Credit Management:**
   - Efficient management of gas credits is vital. Developers should understand the relationship between gas credit (`g_c`), gas limit (`g_l`), and gas reserve (`g_r`). Proper handling ensures that unused gas from previous computations is appropriately considered.

4. **Exception Handling Strategies:**
   - Implement robust exception handling to address out-of-gas scenarios. This involves monitoring gas consumption and triggering appropriate actions when gas limits are exceeded. Handling exceptions ensures the integrity of transactions.

5. **External Message Optimization:**
   - Since external messages may lack attached gas or value, use the `ACCEPT` instruction to dynamically allocate gas for their processing. This optimizes communication between smart contracts and ensures smooth execution of operations triggered by external inputs.

6. **Cryptographic Operations Efficiency:**
   - Optimize cryptographic operations to enhance overall contract efficiency. This includes utilizing efficient algorithms, minimizing redundant cryptographic processes, and adopting secure coding practices.

7. **Security Measures with Cryptography:**
   - Strengthen smart contract security by incorporating cryptographic measures. Secure key management, proper usage of digital signatures, and robust hash functions contribute to a resilient security framework.

</details>
