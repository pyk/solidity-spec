# Solidity Language Specification

> This specification is primarily based on Solidity version 0.8.30. It includes notes on historical changes and features from other versions for context. For a definitive list of changes between versions, please consult the official [changelog](https://github.com/ethereum/solidity/blob/develop/Changelog.md).

Solidity is an object-oriented, statically-typed, high-level language for implementing smart contracts. Smart contracts are programs that govern the behavior of accounts within the Ethereum state.

This document was derived from the official [Solidity documentation](https://docs.soliditylang.org/). It was influenced by the [Starlark specification](https://github.com/bazelbuild/starlark/blob/master/spec.md), and by extension, the Python and Go specifications.

## Overview

Solidity is a curly-braced language designed to target the Ethereum Virtual Machine (EVM). Its syntax is influenced by C++, while other features like multiple inheritance and member access are inspired by Python.

Solidity is statically typed, supports inheritance, libraries, and complex user-defined types. Its programs are designed to run in an isolated, deterministic environment (the EVM), where they can manage state and value (Ether) according to a predefined set of rules. Unlike general-purpose languages, Solidity programs have direct access to blockchain-specific concepts like account addresses, transaction data, and block properties. Memory management is manual for complex types, with explicit data locations (`storage`, `memory`, `calldata`) that determine the lifecycle and gas cost of data.

## Contents

  * [Overview](#overview)
  * [Contents](#contents)
  * [Lexical elements](#lexical-elements)
    * [Comments](#comments)
    * [Keywords](#keywords)
    * [Identifiers](#identifiers)
    * [Literals](#literals)
      * [Integer Literals](#integer-literals)
      * [Rational Number Literals](#rational-number-literals)
      * [String Literals](#string-literals)
      * [Unicode Literals](#unicode-literals)
      * [Hexadecimal Literals](#hexadecimal-literals)
      * [Address Literals](#address-literals)
  * [Data types](#data-types)
    * [Value Types](#value-types)
      * [Booleans](#booleans)
      * [Integers](#integers)
      * [Fixed-Point Numbers](#fixed-point-numbers)
      * [Address](#address)
      * [Fixed-Size Byte Arrays](#fixed-size-byte-arrays)
      * [Enums](#enums)
      * [User-Defined Value Types](#user-defined-value-types)
    * [Reference Types](#reference-types)
      * [Arrays](#arrays)
      * [Structs](#structs)
      * [Mappings](#mappings)
    * [Function Types](#function-types)
  * [Value Concepts](#value-concepts)
    * [Data Locations](#data-locations)
    * [L-values and R-values](#l-values-and-r-values)
    * [Default Values](#default-values)
  * [Expressions](#expressions)
    * [Literals](#literals-1)
    * [Identifiers](#identifiers-1)
    * [Unary Operators](#unary-operators)
    * [Binary Operators](#binary-operators)
    * [Conditional Expressions (Ternary Operator)](#conditional-expressions-ternary-operator)
    * [Function Calls](#function-calls)
      * [Internal Function Calls](#internal-function-calls)
      * [External Function Calls](#external-function-calls)
    * [Contract Creation (`new`)](#contract-creation-new)
    * [Member Access](#member-access)
    * [Index and Slice Access](#index-and-slice-access)
  * [Statements](#statements)
    * [Variable Declarations and Assignments](#variable-declarations-and-assignments)
    * [If Statements](#if-statements)
    * [For Loops](#for-loops)
    * [While and Do-While Loops](#while-and-do-while-loops)
    * [Break and Continue](#break-and-continue)
    * [Return Statements](#return-statements)
    * [Revert, Assert, and Require](#revert-assert-and-require)
    * [Try/Catch Statements](#trycatch-statements)
    * [Emit Statements](#emit-statements)
    * [Inline Assembly](#inline-assembly)
  * [Contract Structure and Execution](#contract-structure-and-execution)
    * [Layout of a Source File](#layout-of-a-source-file)
      * [SPDX License Identifier](#spdx-license-identifier)
      * [Pragma Directives](#pragma-directives)
      * [Import Directives](#import-directives)
    * [Structure of a Contract](#structure-of-a-contract)
      * [State Variables](#state-variables)
      * [Function Definitions](#function-definitions)
      * [Modifiers](#modifiers)
      * [Events](#events)
      * [Errors](#errors)
      * [Struct and Enum Types](#struct-and-enum-types)
    * [Contract Creation and Lifecycle](#contract-creation-and-lifecycle)
      * [Constructor](#constructor)
      * [Receive Ether and Fallback Functions](#receive-ether-and-fallback-functions)
      * [Destruction](#destruction)
    * [Inheritance](#inheritance)
  * [Special Variables and Functions](#special-variables-and-functions)
    * [Block and Transaction Properties](#block-and-transaction-properties)
    * [ABI Encoding and Decoding Functions](#abi-encoding-and-decoding-functions)
    * [Error Handling](#error-handling)
    * [Mathematical and Cryptographic Functions](#mathematical-and-cryptographic-functions)
    * [Contract-Related](#contract-related)
    * [Type Information](#type-information)
  * [Grammar Reference](#grammar-reference)

## Lexical elements

A Solidity source file is a sequence of Unicode characters encoded in UTF-8.

### Comments

Comments are the same as in C++ or JavaScript. Single-line comments begin with `//` and extend to the end of the line. Multi-line comments are enclosed in `/*` and `*/`.

```solidity
// This is a single-line comment.

/*
This is a
multi-line comment.
*/
```

A special form of comment is the NatSpec comment, written as `///` or `/** ... */`, which is used to generate formal documentation for contracts, functions, and variables.

### Keywords

The list of keywords evolves with the language. The following tokens are keywords and may not be used as identifiers:

```
abstract       as             assembly       break          catch
constant       constructor    continue       contract       do
else           emit           enum           event          external
fallback       for            function       if             immutable
import         interface      internal       is             library
mapping        modifier       new            override       payable
private        public         pure           receive        return
returns        revert         struct         super          this
try            using          view           virtual        while
```

*   `constructor` and `calldata` were introduced as keywords in v0.5.0.
*   `override`, `receive`, and `virtual` were introduced as keywords in v0.6.0.
*   `gwei` became a keyword in v0.7.0.

The following are reserved for future use:

```
after          alias          apply          auto           case
copyof         default        define         final          in
inline         let            macro          match          mutable
null           of             partial        promise        reference
relocatable    sealed         sizeof         static         supports
switch         typedef        typeof
```

### Identifiers

An identifier is a sequence of letters, digits, and underscores (`_`), not starting with a digit. Identifiers are used as names for contracts, variables, functions, and other user-defined items.

Examples:
```solidity
SimpleStorage   balanceOf   _internal_function   var1
```

### Literals

Literals are tokens that denote specific values. Solidity has integer, rational, string, hexadecimal, and address literals.

#### Integer Literals

Integer literals are sequences of digits 0-9. They are interpreted as decimals. Octal and binary literals are not supported. Underscores can be used as separators to aid readability (since v0.4.22).

```solidity
42
1_234_000
0
```

#### Rational Number Literals

Decimal fractional literals are formed by a `.` with at least one number on either side. Scientific notation is also supported.

```solidity
3.14
.1
1.3
2e10      // 2 * 10**10
2.5e-1    // 0.25
```

#### String Literals

String literals are written with either double or single quotes. They can only contain printable ASCII characters and support escape sequences like `\n`, `\t`, `\\`, and hexadecimal escapes `\xNN`.

```solidity
"hello"
'world'
"a\nb"
"string with \"quotes\""
```

#### Unicode Literals

Unicode literals are prefixed with the keyword `unicode` and can contain any valid UTF-8 sequence. They use the same escape sequences as regular string literals. This feature was introduced in v0.7.0.

```solidity
unicode"Hello 😃"
```

#### Hexadecimal Literals

Hexadecimal literals are prefixed with `hex` and enclosed in double or single quotes. The content must be a sequence of hexadecimal digits, optionally using underscores as separators.

```solidity
hex"001122FF"
hex'0011_22_FF'
```

#### Address Literals

Hexadecimal literals between 39 and 41 digits long that pass the EIP-55 checksum test are of `address` type.

```solidity
0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF
```

## Data types

Solidity is a statically typed language. Types are divided into two main categories: value types and reference types.

### Value Types

Variables of these types are always passed by value. They are copied when used in assignments or as function arguments.

#### Booleans

`bool`: The possible values are `true` and `false`. The logical operators `!`, `&&`, `||`, `==`, and `!=` apply.

#### Integers

`int<M>` / `uint<M>`: Signed and unsigned integers of `M` bits, where `M` is a multiple of 8 from 8 to 256. `uint` and `int` are aliases for `uint256` and `int256`.

**Note:** Integer arithmetic is checked for overflow and underflow by default since Solidity v0.8.0, causing a transaction to revert. This can be disabled using an `unchecked { ... }` block.

#### Fixed-Point Numbers

`fixed<M>x<N>` / `ufixed<M>x<N>`: Signed and unsigned fixed-point numbers. These types are not yet fully supported and cannot be assigned to or from.

#### Address

`address`: Holds a 20-byte value (the size of an Ethereum address). There are two variants (since v0.5.0):
*   `address`: A standard address.
*   `address payable`: An address that can receive Ether via the `.transfer()` and `.send()` methods.

An `address` can be explicitly converted to `address payable`, but not implicitly.

#### Fixed-Size Byte Arrays

`bytes1`, `bytes2`, ..., `bytes32`: Hold a sequence of bytes from one to 32. `.length` yields the fixed length. The alias `byte` for `bytes1` was removed in v0.8.0.

#### Enums

`enum`: A way to create a user-defined type with a finite set of constant values. Enums are convertible to and from integer types, but only explicitly.

```solidity
enum Action { GoLeft, GoRight, GoStraight, SitStill }
Action choice = Action.GoLeft;
```

#### User-Defined Value Types

`type C is V;`: Creates a zero-cost abstraction `C` over an elementary value type `V`. It provides stricter type checking than a simple alias. This feature was introduced in v0.8.8.

```solidity
type UFixed is uint256;
UFixed.wrap(100); // convert uint to UFixed
UFixed.unwrap(u); // convert UFixed to uint
```

### Reference Types

Reference types are more complex and must be managed carefully with respect to their data location: `memory`, `storage`, or `calldata`.

#### Arrays

*   Fixed-Size Arrays: `T[k]`, an array of `k` elements of type `T`.
*   Dynamically-Sized Arrays: `T[]`, an array that can change size at runtime.

`bytes` and `string` are special array types. `bytes` is a dynamic byte array, and `string` is a dynamic UTF-8 encoded string.

Arrays have a `.length` member. Dynamic storage arrays also have `.push()` and `.pop()` methods. The `bytes.concat` and `string.concat` functions were added in v0.8.12 and v0.8.15 respectively.

#### Structs

`struct`: A way to define custom types by grouping several variables.

```solidity
struct Funder {
    address addr;
    uint amount;
}
```

#### Mappings

`mapping(KeyType => ValueType)`: Hash tables that map keys to values. `KeyType` can be almost any value type, but not a mapping, a dynamically sized array, a struct, or an enum. `ValueType` can be any type, including mappings.

Mappings are only allowed for state variables (i.e., `storage` data location). They are virtually initialized such that every possible key exists and maps to a zero-byte value.

### Function Types

Function types represent functions. There are two kinds:
*   `internal` functions, which can only be called inside the current contract.
*   `external` functions, which consist of an address and a function signature and can be passed as arguments and returned from external function calls.

```solidity
function (uint) internal pure returns (uint)
function (address) external
```

## Value Concepts

### Data Locations

Every reference type (`arrays`, `structs`, `mappings`) must have an explicit data location (since v0.5.0):

*   **`storage`**: The location for state variables, which are persistent between function calls. This is the only location that can hold mappings. Reading and writing to storage is gas-intensive.
*   **`memory`**: A temporary data location whose lifetime is limited to a single external function call. Variables in `memory` are copied when passed as arguments to functions.
*   **`calldata`**: A non-modifiable, non-persistent area where function arguments are stored. It is required for parameters of `external` functions.

Assignments between different data locations (e.g., from `calldata` to `storage`) always create a copy. Assigning a `storage` variable to another local `storage` variable creates a reference (pointer).

### L-values and R-values

An expression is an l-value if it can appear on the left-hand side of an assignment. This includes state variables, local variables, and array or struct members. Other expressions are r-values and cannot be assigned to.

### Default Values

Every variable, when declared, has a default value whose byte-representation is all zeros. For `bool` this is `false`, for integers it is `0`, and for dynamic arrays and `string` it is an empty array/string.

## Expressions

An expression specifies the computation of a value.

### Literals

Integer, rational, string, hex, and address literals evaluate to their corresponding values.
See [Literals](#literals) for details.

### Identifiers

An identifier is a name that refers to a variable, function, contract, or other user-defined item.

### Unary Operators

There are five unary operators:

```solidity
!b           // logical negation (bool)
-n           // arithmetic negation (int)
++v, v++     // increment
--v, v--     // decrement
delete a     // assigns the default value to a
```

### Binary Operators

Solidity provides the following binary operators, arranged in order of increasing precedence:

```text
||
&&
==   !=
<    >   <=   >=
|
^
&
<<   >>
+    -
*    /    %
**
```

*   **Logical Operators**: `||` (logical OR) and `&&` (logical AND) use short-circuiting evaluation.
*   **Comparison Operators**: `==`, `!=`, `<`, `>`, `<=`, `>=` return a `bool`. They can be applied to integers, addresses, and fixed-size byte arrays.
*   **Bitwise Operators**: `&` (AND), `|` (OR), `^` (XOR), `<<` (left shift), `>>` (right shift).
*   **Arithmetic Operators**: `+`, `-`, `*`, `/`, `%` (modulo), `**` (exponentiation). By default (since v0.8.0), these operations are checked for overflow/underflow.
*   **Exponentiation (`**`)**: became right-associative in v0.8.0.

### Conditional Expressions (Ternary Operator)

A conditional expression has the form `condition ? trueValue : falseValue`. It evaluates `trueValue` if `condition` is true, and `falseValue` otherwise.

### Function Calls

#### Internal Function Calls

Functions of the current contract can be called directly by name (e.g., `myFunction()`). This is translated into a simple EVM jump, which is very efficient.

#### External Function Calls

Functions of other contracts must be called externally (e.g., `otherContract.myFunction()`). This creates an EVM message call. One can also call a function on the current contract externally using `this.myFunction()`. External calls can specify the amount of Ether and gas to send. The syntax for this changed in v0.7.0.

```solidity
// New syntax (since v0.6.2, mandatory since v0.7.0)
otherContract.myFunction{value: 1 ether, gas: 10000}(arg1, arg2);
```

Low-level call functions `call`, `delegatecall`, and `staticcall` are also available for direct interaction with other contracts, but they are less safe as they bypass type checking.

### Contract Creation (`new`)

A new contract can be created using the `new` keyword. The creator must know the full code of the contract being created.

```solidity
MyContract newContract = new MyContract(arg1);
MyContract newContractWithValue = new MyContract{value: 1 ether}(arg1);
```

Since version 0.6.2, a `salt` can be provided to create contracts at a predictable address using the `CREATE2` opcode:

```solidity
MyContract newContract = new MyContract{salt: mySalt}(arg1);
```

### Member Access

Members of a struct, contract, or library are accessed using the dot notation, e.g., `myStruct.myMember`.

### Index and Slice Access

Array elements are accessed by index, e.g., `myArray[i]`. Slices of `calldata` arrays are also supported (since v0.6.0), e.g., `calldataArray[start:end]`.

## Statements

### Variable Declarations and Assignments

A variable is declared by specifying its type and name. It can be initialized at declaration.

```solidity
uint a = 10;
uint b; // defaults to 0
```

Assignment uses the `=` operator. Solidity also supports compound assignments like `+=`, `-=`, etc. Destructuring assignments are possible for tuples and multiple return values.

```solidity
(uint a, uint b) = (1, 2);
(a, b) = (b, a); // Swap values
```

### If Statements

An `if` statement conditionally executes a block of code. It can be followed by an optional `else` block.

```solidity
if (x < 10) {
    // ...
} else if (x < 20) {
    // ...
} else {
    // ...
}
```

### For Loops

`for` loops are used to iterate a known number of times.

```solidity
for (uint i = 0; i < 10; i++) {
    // ...
}
```

### While and Do-While Loops

`while` loops execute a block as long as a condition is true. `do-while` loops execute the block at least once.

```solidity
while (x < 10) {
    x++;
}

do {
    x++;
} while (x < 10);
```

### Break and Continue

`break` exits a loop. `continue` skips the rest of the current iteration and proceeds to the next one.

### Return Statements

A `return` statement exits a function and returns a value (or values) to the caller. A function can return multiple values as a tuple.

```solidity
return 42;
return (a, b);
```

### Revert, Assert, and Require

These statements are used for error handling. They revert all state changes made in the current call.
*   `require(condition, "message")`: Used to validate inputs and conditions before execution. If the condition is false, it reverts.
*   `assert(condition)`: Used to check for internal errors or to check invariants. If the condition is false, it reverts with a `Panic` error (since v0.8.0).
*   `revert("message")` or `revert CustomError(args)`: Unconditionally aborts execution and reverts state changes. `revert CustomError()` was introduced in v0.8.4.

### Try/Catch Statements

A failure in an external call can be caught and handled using a `try/catch` statement (since v0.6.0).

```solidity
try otherContract.f() returns (uint v) {
    // success
} catch Error(string memory reason) {
    // handle revert with reason string
} catch Panic(uint errorCode) {
    // handle assert failure (since v0.8.1)
} catch (bytes memory lowLevelData) {
    // handle low-level failure
}
```

### Emit Statements

An `emit` statement triggers an event, logging its arguments to the blockchain where they can be accessed by off-chain clients. The `emit` keyword became mandatory in v0.5.0.

```solidity
emit MyEvent(arg1, arg2);
```

### Inline Assembly

Solidity supports inline assembly using the `assembly { ... }` block, which uses a language called Yul for low-level access to the EVM.

## Contract Structure and Execution

### Layout of a Source File

A Solidity source file can contain multiple contract definitions, import directives, pragma directives, and other top-level definitions.

#### SPDX License Identifier

It is recommended that every source file starts with a comment indicating its license.

```solidity
// SPDX-License-Identifier: MIT
```

#### Pragma Directives

Pragmas are instructions for the compiler. The most common is the version pragma.

```solidity
pragma solidity ^0.8.0;
pragma abicoder v2; // Selects ABI encoder v2 (default since v0.8.0)
```

#### Import Directives

Import statements are used to modularize code.

```solidity
import "./MyLibrary.sol";
import {symbol1 as alias, symbol2} from "filename";
import * as symbolName from "filename";
```

### Structure of a Contract

A contract is defined with the `contract` keyword and can contain state variables, functions, modifiers, events, errors, and user-defined types.

#### State Variables

Variables whose values are permanently stored in contract storage.

#### Function Definitions

Executable units of code. See [Functions](#function-types) for more.

#### Modifiers

Used to declaratively change the behavior of functions.

```solidity
modifier onlyOwner() {
    require(msg.sender == owner, "Not owner");
    _;
}
```

#### Events

A way to interface with the EVM's logging facilities. See [Emit Statements](#emit-statements).

#### Errors

Custom, named errors for more gas-efficient error reporting (since v0.8.4). See [Revert, Assert, and Require](#revert-assert-and-require).

#### Struct and Enum Types

User-defined types. See [Data Types](#data-types).

### Contract Creation and Lifecycle

#### Constructor

A special function, declared with the `constructor` keyword (since v0.5.0), that is executed only once when the contract is created. It is used for initialization. Before v0.5.0, constructors were functions with the same name as the contract.

#### Receive Ether and Fallback Functions

*   `receive() external payable`: Executed on a plain Ether transfer to the contract (no calldata). This was introduced in v0.6.0.
*   `fallback() external [payable]`: Executed when a call to the contract does not match any other function signature, or if there is no `receive` function and a plain Ether transfer occurs. Before v0.6.0, a single payable fallback function handled both cases.

#### Destruction

A contract can be removed from the blockchain with `selfdestruct(recipient)`, sending its remaining balance to the `recipient`. The behavior of `selfdestruct` was significantly restricted in the Cancun EVM upgrade (EIP-6780).

### Inheritance

Solidity supports multiple inheritance. A contract can inherit from one or more base contracts using the `is` keyword. Overridden functions in derived contracts must be marked with `override`. Base functions must be marked `virtual` to be overridable (since v0.6.0).

```solidity
contract A is B, C { ... }
```

## Special Variables and Functions

### Block and Transaction Properties

*   `block.timestamp` (uint): Current block timestamp.
*   `block.number` (uint): Current block number.
*   `block.coinbase` (address payable): Current block miner's address.
*   `block.chainid` (uint): Current chain ID (since v0.8.0).
*   `block.basefee` (uint): Current block's base fee (EIP-1559, since v0.8.7).
*   `block.prevrandao` (uint): Randomness from the Beacon Chain (since v0.8.18, for EVM versions >= Paris), replacing `block.difficulty`.
*   `msg.sender` (address): Sender of the message (current call).
*   `msg.value` (uint): Amount of wei sent with the message.
*   `msg.data` (bytes calldata): Complete calldata.
*   `tx.origin` (address): Sender of the transaction (full call chain).
*   `tx.gasprice` (uint): Gas price of the transaction.
*   `gasleft()` returns (uint256): Remaining gas.

### ABI Encoding and Decoding Functions

*   `abi.encode(...)`: ABI-encodes arguments.
*   `abi.encodePacked(...)`: Performs packed encoding (can be ambiguous).
*   `abi.decode(bytes, (...))`: Decodes ABI-encoded data (requires ABIEncoderV2).
*   `abi.encodeWithSignature(...)`: Encodes arguments with a function signature.
*   `abi.encodeCall(functionPointer, (...))`: Type-safe encoding of a function call (since v0.8.11).

### Error Handling

*   `assert(bool)`
*   `require(bool, [string])` or `require(bool, CustomError)`
*   `revert([string])` or `revert CustomError()`

### Mathematical and Cryptographic Functions

*   `keccak256(bytes memory) returns (bytes32)`
*   `sha256(bytes memory) returns (bytes32)`
*   `ripemd160(bytes memory) returns (bytes20)`
*   `ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)`
*   `addmod(uint x, uint y, uint k) returns (uint)`
*   `mulmod(uint x, uint y, uint k) returns (uint)`

### Contract-Related

*   `this`: The current contract, convertible to `address`.
*   `super`: The contract one level higher in the inheritance hierarchy.
*   `selfdestruct(address payable)`: Destroys the current contract.

### Type Information

*   `type(T).name`: The name of type `T`.
*   `type(T).creationCode`: Creation bytecode of contract `T`.
*   `type(T).runtimeCode`: Runtime bytecode of contract `T`.
*   `type(I).interfaceId`: EIP-165 interface ID of interface `I` (since v0.6.2).
*   `type(T).min` / `type(T).max`: Min/max values for integer type `T` (since v0.8.8 for enums, earlier for integers).

## Grammar Reference

TODO
