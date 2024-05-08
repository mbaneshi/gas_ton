
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
    
