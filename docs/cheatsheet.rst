**********
速查表
**********

.. index:: operator;precedence

操作符的优先顺序
================================

.. include:: types/operator-precedence-table.rst

.. index:: abi;decode, abi;encode, abi;encodePacked, abi;encodeWithSelector, abi;encodeCall, abi;encodeWithSignature

<<<<<<< HEAD
全局变量
================

- ``abi.decode(bytes memory encodedData, (...)) returns (...)``： :ref:`ABI <ABI>` -对提供的数据进行解码。类型在括号中作为第二个参数给出。
  示例： ``(uint a, uint[2] memory b, bytes memory c) = abi.decode(data, (uint, uint[2], bytes))``
- ``abi.encode(...) returns (bytes memory)``： :ref:`ABI <ABI>`-对给定的参数进行编码。
- ``abi.encodePacked(...) returns (bytes memory)``： 对给定的参数执行 :ref:`紧密打包 <abi_packed_mode>`。
  请注意，这种编码可能是不明确的!
- ``abi.encodeWithSelector(bytes4 selector, ...) returns (bytes memory)``： :ref:`ABI <ABI>`-对给定参数进行编码，
  并以给定的函数选择器作为起始的 4 字节数据一起返回
- ``abi.encodeCall(function functionPointer, (...)) returns (bytes memory)``： 对 ``functionPointer`` 的调用进行ABI编码，
  参数在元组中找到。执行全面的类型检查，确保类型与函数签名相符。结果等于 ``abi.encodeWithSelector(functionPointer.selector(..))``。
- ``abi.encodeWithSignature(string memory signature, ...) returns (bytes memory)``： 等价于
  ``abi.encodeWithSelector(bytes4(keccak256(bytes(signature))), ...)``
- ``bytes.concat(...) returns (bytes memory)``： :ref:`将可变数量的参数连接成一个字节数组 <bytes-concat>`。
- ``string.concat(...) returns (string memory)``： :ref:`将可变数量的参数连接成一个字符串数组 <string-concat>`。
- ``block.basefee`` (``uint``)： 当前区块的基本费用 （ `EIP-3198 <https://eips.ethereum.org/EIPS/eip-3198>`_ 和 `EIP-1559 <https://eips.ethereum.org/EIPS/eip-1559>`_ ）
- ``block.chainid`` (``uint``)： 当前链的ID
- ``block.coinbase`` (``address payable``)： 当前区块矿工的地址
- ``block.difficulty`` (``uint``)： 当前区块的难度值（ ``EVM < Paris`` ）。对于其他EVM版本，它是 ``block.prevrandao`` 的一个废弃的别名，将在下一个重大改变版本中被删除。
- ``block.gaslimit`` (``uint``)： 当前区块的 gas 上限
- ``block.number`` (``uint``)： 当前区块的区块号
- ``block.prevrandao`` (``uint``)： 由信标链提供的随机数（ ``EVM >= Paris`` ）（见 `EIP-4399 <https://eips.ethereum.org/EIPS/eip-4399>`_ ）。
- ``block.timestamp`` (``uint``)： 当前区块的时间戳，自Unix epoch以来的秒数
- ``gasleft() returns (uint256)``： 剩余gas
- ``msg.data`` (``bytes``)： 完整的调用数据
- ``msg.sender`` (``address``)： 消息发送方（当前调用）
- ``msg.sig`` (``bytes4``)： Calldata的前四个字节（即函数标识符）。
- ``msg.value`` (``uint``)： 随消息发送的 wei 的数量
- ``tx.gasprice`` (``uint``)： 交易的 gas 价格
- ``tx.origin`` (``address``)： 交易发送方（完整调用链上的原始发送方）
- ``assert(bool condition)``： 如果条件为 ``false``，则中止执行并恢复状态变化（用于内部错误）。
- ``require(bool condition)``： 如果条件为 ``false``，则中止执行并恢复状态变化（用于错误的输入或外部组件的错误）。
- ``require(bool condition, string memory message)``： 如果条件为 ``false``，则中止执行并恢复状态变化（用于错误的输入或外部组件的错误）。同时提供错误信息。
- ``revert()``： 中止执行并恢复状态变化
- ``revert(string memory message)``： 中止执行并恢复状态变化，提供一个解释性的字符串
- ``blockhash(uint blockNumber) returns (bytes32)``： 给定区块的哈希值 - 只对最近的256个区块有效
- ``keccak256(bytes memory) returns (bytes32)``： 计算输入的Keccak-256哈希值
- ``sha256(bytes memory) returns (bytes32)``： 计算输入的SHA-256哈希值
- ``ripemd160(bytes memory) returns (bytes20)``： 计算输入的RIPEMD-160的哈希值
- ``ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)``： 从椭圆曲线签名中恢复与公钥相关的地址，错误时返回0
- ``addmod(uint x, uint y, uint k) returns (uint)``： 计算 ``(x + y) % k`` 的值，其中加法的结果即使超过 ``2**256`` 也不会被截取。从 0.5.0 版本开始会加入对 ``k != 0`` 的 assert（即会在此函数开头执行 ``assert(k != 0);`` 作为参数检查，译者注）。
- ``mulmod(uint x, uint y, uint k) returns (uint)``： 计算 ``(x * y) % k`` 的值，其中乘法的结果即使超过 ``2**256`` 也不会被截取。从 0.5.0 版本开始会加入对 ``k != 0`` 的 assert（即会在此函数开头执行 ``assert(k != 0);`` 作为参数检查，译者注）。
- ``this`` （当前合约的类型）： 当前合约，可明确转换为 ``address`` 或 ``address payable``。
- ``super``： 继承层次中高一级的合约
- ``selfdestruct(address payable recipient)``： 销毁当前合约，将其资金发送到给定的地址。
- ``<address>.balance`` (``uint256``)： :ref:`address` 的余额，以 Wei 为单位
- ``<address>.code`` (``bytes memory``)： 在 :ref:`address` 的代码（可以是空的）。
- ``<address>.codehash`` (``bytes32``)： :ref:`address` 的代码哈希值。
- ``<address payable>.send(uint256 amount) returns (bool)``： 向 :ref:`address` 发送给定数量的 Wei，失败时返回 ``false``
- ``<address payable>.transfer(uint256 amount)``： 向 :ref:`address` 发送给定数量的 Wei，失败时会把错误抛出（throw）
- ``type(C).name`` (``string``)： 合约的名称
- ``type(C).creationCode`` (``bytes memory``)： 给定合约的创建字节码，参见 :ref:`类型信息 <meta-type>`。
- ``type(C).runtimeCode`` (``bytes memory``)： 给定合约的运行时字节码，参见 :ref:`类型信息 <meta-type>`。
- ``type(I).interfaceId`` (``bytes4``)： 包含给定接口的EIP-165接口标识符的值，参见 :ref:`类型信息 <meta-type>`。
- ``type(T).min`` (``T``)： 整数类型 ``T`` 所能代表的最小值，参见 :ref:`类型信息 <meta-type>`。
- ``type(T).max`` (``T``)： 整数类型 ``T`` 所能代表的最大值，参见 :ref:`类型信息 <meta-type>`。
=======
ABI Encoding and Decoding Functions
===================================

- ``abi.decode(bytes memory encodedData, (...)) returns (...)``: :ref:`ABI <ABI>`-decodes
  the provided data. The types are given in parentheses as second argument.
  Example: ``(uint a, uint[2] memory b, bytes memory c) = abi.decode(data, (uint, uint[2], bytes))``
- ``abi.encode(...) returns (bytes memory)``: :ref:`ABI <ABI>`-encodes the given arguments
- ``abi.encodePacked(...) returns (bytes memory)``: Performs :ref:`packed encoding <abi_packed_mode>` of
  the given arguments. Note that this encoding can be ambiguous!
- ``abi.encodeWithSelector(bytes4 selector, ...) returns (bytes memory)``: :ref:`ABI <ABI>`-encodes
  the given arguments starting from the second and prepends the given four-byte selector
- ``abi.encodeCall(function functionPointer, (...)) returns (bytes memory)``: ABI-encodes a call to ``functionPointer`` with the arguments found in the
  tuple. Performs a full type-check, ensuring the types match the function signature. Result equals ``abi.encodeWithSelector(functionPointer.selector, (...))``
- ``abi.encodeWithSignature(string memory signature, ...) returns (bytes memory)``: Equivalent
  to ``abi.encodeWithSelector(bytes4(keccak256(bytes(signature))), ...)``

.. index:: bytes;concat, string;concat

Members of ``bytes`` and  ``string``
====================================

- ``bytes.concat(...) returns (bytes memory)``: :ref:`Concatenates variable number of
  arguments to one byte array<bytes-concat>`

- ``string.concat(...) returns (string memory)``: :ref:`Concatenates variable number of
  arguments to one string array<string-concat>`

.. index:: address;balance, address;codehash, address;send, address;code, address;transfer

Members of ``address``
======================

- ``<address>.balance`` (``uint256``): balance of the :ref:`address` in Wei
- ``<address>.code`` (``bytes memory``): code at the :ref:`address` (can be empty)
- ``<address>.codehash`` (``bytes32``): the codehash of the :ref:`address`
- ``<address payable>.send(uint256 amount) returns (bool)``: send given amount of Wei to :ref:`address`,
  returns ``false`` on failure
- ``<address payable>.transfer(uint256 amount)``: send given amount of Wei to :ref:`address`, throws on failure

.. index:: blockhash, block, block;basefree, block;chainid, block;coinbase, block;difficulty, block;gaslimit, block;number, block;prevrandao, block;timestamp
.. index:: gasleft, msg;data, msg;sender, msg;sig, msg;value, tx;gasprice, tx;origin

Block and Transaction Properties
================================

- ``blockhash(uint blockNumber) returns (bytes32)``: hash of the given block - only works for 256 most recent blocks
- ``block.basefee`` (``uint``): current block's base fee (`EIP-3198 <https://eips.ethereum.org/EIPS/eip-3198>`_ and `EIP-1559 <https://eips.ethereum.org/EIPS/eip-1559>`_)
- ``block.chainid`` (``uint``): current chain id
- ``block.coinbase`` (``address payable``): current block miner's address
- ``block.difficulty`` (``uint``): current block difficulty (``EVM < Paris``). For other EVM versions it behaves as a deprecated alias for ``block.prevrandao`` that will be removed in the next breaking release
- ``block.gaslimit`` (``uint``): current block gaslimit
- ``block.number`` (``uint``): current block number
- ``block.prevrandao`` (``uint``): random number provided by the beacon chain (``EVM >= Paris``) (see `EIP-4399 <https://eips.ethereum.org/EIPS/eip-4399>`_ )
- ``block.timestamp`` (``uint``): current block timestamp in seconds since Unix epoch
- ``gasleft() returns (uint256)``: remaining gas
- ``msg.data`` (``bytes``): complete calldata
- ``msg.sender`` (``address``): sender of the message (current call)
- ``msg.sig`` (``bytes4``): first four bytes of the calldata (i.e. function identifier)
- ``msg.value`` (``uint``): number of wei sent with the message
- ``tx.gasprice`` (``uint``): gas price of the transaction
- ``tx.origin`` (``address``): sender of the transaction (full call chain)

.. index:: assert, require, revert

Validations and Assertions
==========================

- ``assert(bool condition)``: abort execution and revert state changes if condition is ``false`` (use for internal error)
- ``require(bool condition)``: abort execution and revert state changes if condition is ``false`` (use
  for malformed input or error in external component)
- ``require(bool condition, string memory message)``: abort execution and revert state changes if
  condition is ``false`` (use for malformed input or error in external component). Also provide error message.
- ``revert()``: abort execution and revert state changes
- ``revert(string memory message)``: abort execution and revert state changes providing an explanatory string

.. index:: cryptography, keccak256, sha256, ripemd160, ecrecover, addmod, mulmod

Mathematical and Cryptographic Functions
========================================

- ``keccak256(bytes memory) returns (bytes32)``: compute the Keccak-256 hash of the input
- ``sha256(bytes memory) returns (bytes32)``: compute the SHA-256 hash of the input
- ``ripemd160(bytes memory) returns (bytes20)``: compute the RIPEMD-160 hash of the input
- ``ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)``: recover address associated with
  the public key from elliptic curve signature, return zero on error
- ``addmod(uint x, uint y, uint k) returns (uint)``: compute ``(x + y) % k`` where the addition is performed with
  arbitrary precision and does not wrap around at ``2**256``. Assert that ``k != 0`` starting from version 0.5.0.
- ``mulmod(uint x, uint y, uint k) returns (uint)``: compute ``(x * y) % k`` where the multiplication is performed
  with arbitrary precision and does not wrap around at ``2**256``. Assert that ``k != 0`` starting from version 0.5.0.

.. index:: this, super, selfdestruct

Contract-related
================

- ``this`` (current contract's type): the current contract, explicitly convertible to ``address`` or ``address payable``
- ``super``: a contract one level higher in the inheritance hierarchy
- ``selfdestruct(address payable recipient)``: destroy the current contract, sending its funds to the given address

.. index:: type;name, type;creationCode, type;runtimeCode, type;interfaceId, type;min, type;max

Type Information
================

- ``type(C).name`` (``string``): the name of the contract
- ``type(C).creationCode`` (``bytes memory``): creation bytecode of the given contract, see :ref:`Type Information<meta-type>`.
- ``type(C).runtimeCode`` (``bytes memory``): runtime bytecode of the given contract, see :ref:`Type Information<meta-type>`.
- ``type(I).interfaceId`` (``bytes4``): value containing the EIP-165 interface identifier of the given interface, see :ref:`Type Information<meta-type>`.
- ``type(T).min`` (``T``): the minimum value representable by the integer type ``T``, see :ref:`Type Information<meta-type>`.
- ``type(T).max`` (``T``): the maximum value representable by the integer type ``T``, see :ref:`Type Information<meta-type>`.
>>>>>>> english/develop


.. index:: visibility, public, private, external, internal

函数可见性说明符
==============================

.. code-block:: solidity
    :force:

    function myFunction() <visibility specifier> returns (bool) {
        return true;
    }

- ``public``： 内部、外部均可见（参考为存储/状态变量创建 :ref:`getter function<getter-functions>` 函数）
- ``private``： 仅在当前合约内可见
- ``external``： 仅在外部可见（仅可修饰函数）——就是说，仅可用于消息调用（即使在合约内调用，也只能通过 ``this.func`` 的方式）
- ``internal``： 仅在内部可见（也就是在当前 Solidity 源代码文件内均可见，不仅限于当前合约内，译者注）


.. index:: modifiers, pure, view, payable, constant, anonymous, indexed

修改器
=========

<<<<<<< HEAD
- ``pure`` 修饰函数时：不允许修改或访问状态。
- ``view`` 修饰函数时：不允许修改状态。
- ``payable`` 修饰函数时：允许从调用中接收以太币。
- ``constant`` 修饰状态变量时：不允许赋值（除初始化以外），不会占据 存储插槽（storage slot）。
- ``immutable`` 修饰状态变量时：在构造时允许有一个确切的赋值，之后是恒定的。被存储在代码中。
- ``anonymous`` 修饰事件时：不把事件签名作为 topic 存储。
- ``indexed`` 修饰事件参数时：将参数作为 topic 存储。
- ``virtual`` 修饰函数和修改时：允许在派生合约中改变函数或修改器的行为。
- ``override`` 表示该函数、修改器或公共状态变量改变了基类合约中的函数或修改器的行为。
=======
- ``pure`` for functions: Disallows modification or access of state.
- ``view`` for functions: Disallows modification of state.
- ``payable`` for functions: Allows them to receive Ether together with a call.
- ``constant`` for state variables: Disallows assignment (except initialisation), does not occupy storage slot.
- ``immutable`` for state variables: Allows assignment at construction time and is constant when deployed. Is stored in code.
- ``anonymous`` for events: Does not store event signature as topic.
- ``indexed`` for event parameters: Stores the parameter as topic.
- ``virtual`` for functions and modifiers: Allows the function's or modifier's
  behavior to be changed in derived contracts.
- ``override``: States that this function, modifier or public state variable changes
  the behavior of a function or modifier in a base contract.
>>>>>>> english/develop

