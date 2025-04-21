## Instructions

```
88 PUSH1 0x04
|  |     |     
|  |     Hex value for push.
|  Opcode.
Instruction number.
```
For now, let’s focus on the `JUMP`, `JUMPI`, `JUMPDEST`, `RETURN`, and `STOP` opcodes, and **ignore all others**.

When the EVM executes code, it does so top down with no exceptions
- here are no other entry points to the code. 
- It always starts from the top.
- It can jump around, yes, and that’s exactly what `JUMP` and `JUMPI` do.

## `JUMP`
- `JUMP` takes the topmost value from the stack and moves execution to that location.
- The target location must contain a `JUMPDEST` opcode, though, otherwise execution will fail.

## `JUMPDEST`
The sole purpose of `JUMPDEST`: to mark a location as a valid jump target.

## `JUMPI`
`JUMPI` is exactly the same as `JUMP`, but there must not be a “0” in the second position of the stack, otherwise there will be no jump.

## `STOP`
Completely halts execution of the contract.

## `RETURN`
Halts execution too, but returns data from a portion of the EVM’s memory.

# Creation Code vs. Runtime Code
## Creation Code
 It will never be a part of the contract’s code per se, but is only executed by the EVM once during the transaction that **creates the contract.**

This piece of code is in charge of:
1. setting the created contract’s initial state
2. returning a copy of its runtime code.

### How is contract deployed
The creation code gets executed in a transaction, which returns a copy of the runtime code, which is the actual code of the contract.

The contract’s constructor is part of the creation code; it will not be present in the contract’s code once it is deployed.

#### OPCODE 0-4
![[opcode-0-2.png]]
- `PUSH1` simply pushes one byte onto the top of the stack
- `MSTORE` grabs the two last items from the stack and stores one of them in memory.
- This basically stores the number `0x80` (decimal 128) into memory at position `0x40`(decimal 64)

```
mstore(0x40, 0x80)
         |     |
         |     What to store.
         Where to store.
(in memory)
```
#### OPCODE 5-15
![[Pasted image 20250415071411.png]]
* `CALLVALUE` pushes the amount of wei involved in the creation transaction
*  `DUP1` duplicates the first element on the stack
* `ISZERO` pushes 1 to the stack if the topmost value of stack is zero
* `PUSH2` is like `PUSH1` but it can push two bytes.
* `REVERT` halts the execution.

So, it seems like this solidity code:
```solidity
if(msg.value != 0) revert();
```

**This code was not actually part of our original solidity source**.  But was instead injected by the compiler because we did not declare the constructor as `payable`.

Going back to the assembly code, the `JUMPI` at instruction 11 will skip instructions 12 through 15 and jump to 16 if there is no ether involved. Otherwise, `REVERT` will execute with both parameters as 0 (meaning that no useful data will be returned).

#### OPCODE 16-34
![[Pasted image 20250415072212.png]]
- The first four instructions (17 to 20) read whatever is in memory at position `0x40` and push that to the stack.
	- that should be the number `0x80`.
- The following instructions then push `0x20` (decimal 32) to the stack (instruction 21), 
- duplicate that value (instruction 23), 
- push `0x0217` (decimal 535) (instruction 24),
- and finally duplicate the fourth value (instruction 27), which should be `0x80` again.
- On instruction 28, `CODECOPY` is executed, which takes three arguments:
	1. target memory position to copy the code to,
	2. instruction number to copy from,
	3. and number of bytes of code to copy.
	
	 it targets memory at position `0x80`, from byte position 535 in code, 32 bytes of code length. Why?
	 - there are 566 instructions entirely in the bytecode.
	 - this code tries to copy the last 32 bytes of the code.
	 - ℹ️ When deploying a contract whose constructor contains parameters, the arguments are appended to the end of the code as raw hex data.
	 - In this case, the constructor takes one `uint256` parameter, so all this code is doing is copying the argument to memory from the value appended at the end of the code.
	 - `0x0000000000000000000000000...0000000000000000000002710`. Which is the decimal value 10000 we passed to the constructor when we deployed the contract!
 - The next set of instructions (29 to 35) update the value stored in memory at position `0x40` from the number `0x80` to the number `0xa0`
	- Solidity keeps track of something called a “free memory pointer”:
		- a place in memory we can use to store stuff, with the guarantee that no one will overwrite it
	- So, since we stored the number 10000 in the old free memory position, we updated the free memory pointer by shifting it 32 bytes forward.
	- What’s in memory between `0x00` to `0x40`?
		- Nothing. It’s just a chunk of memory that Solidity reserves for calculating hashes
- instruction 34, `MLOAD` reads from memory at position `0x40` and basically downloads our 10000 value from memory into the stack
ℹ️ This is a common pattern in EVM bytecode generated by Solidity: before a function’s body is executed, the function’s parameters are loaded into the stack.

#### OPCODE 35-67

![[Pasted image 20250420164415.png]]

These instructions are nothing more and nothing less than the constructor’s body: 

```solidity
totalSupply_ = _initialSupply;
balances[msg.sender] =  _initialSupply;
```

- First, 0 is pushed to the stack,
- then the second item in the stack is duplicated (that’s our 10000 number), 
- and then the number 0 is duplicated and pushed to the stack, which is the position slot in storage of `totalSupply_`. 
- then `SSTORE` can consume the values and still keep 10000 lying around for future use

```
sstore(0x00, 0x2710) 
	   |     |
	   |     What to store.
	   Where to store.
(in storage)
```
We stored the number 10000 in the variable `totalSupply_`.

The next set of instructions deal with storing 10000 in the `balances` mapping for the key of `msg.sender`.

TODO: [[Layout of State Variables in Storage]]

- it will concatenate the slot of the mapping value (in this case the number `1`, because it’s the second variable declared in the contract) with the key used (in this case, `msg.sender`, obtained with the opcode `CALLER`),
- then hash that with the `SHA3` opcode and use that as the target position in storage.
- Storage, in the end, is just a simple dictionary or hash table.

* Moving on with instructions 43 to 45, the `msg.sender` address is stored in memory (this time at position `0x00`), 
* and then in instructions 46 to 50, the value 1 (the slot of the mapping) is stored at memory position `0x20`.
* Finally, the `SHA3` opcode calculates the Keccak256 hash of whatever is in memory from position `0x00` to position `0x40` — that is, the concatenation of the mapping’s slot/position with the key used. This is precisely where the value 10000 will be stored in our mapping

```
sstore(hash..., 0x2710)
	   |        |
	   |        What to store.
	   Where to store.
```
#### OPCODE 68-77
![[Pasted image 20250420170645.png]]
Here, we’re performing a code copy again.
we’re copying `0x01d1` (decimal 465) bytes starting from position `0x0046` (decimal 70) into memory at position 0.
- position 70 is right after our creation-time EVM code, where execution stopped.
- The runtime bytecode is contained in those 465 bytes.
- This is the part of the code that will be saved in the blockchain as the contract’s runtime code
#### OPCODE 78-81
![[Pasted image 20250420170804.png]]
- `RETURN` grabs the code copied to memory and hands it over to the EVM.
- If this creation code is executed in the context of a transaction to the `0x0` address, the EVM will execute the code and store the return value as the created contract’s runtime code.
# The Function Selector