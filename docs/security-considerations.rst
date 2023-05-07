.. _security_considerations:

#######################
安全考虑
#######################

虽然通常很容易建立起按预期工作的软件，但要检查没有人能够以 **非** 预期的方式使用它，就难得多了。

<<<<<<< HEAD
在 Solidity 中，这一点更加重要，因为您可以使用智能合约来处理代币，
甚至可能是更有价值的东西。此外，
智能合约的每一次执行都是公开的，而且源代码也通常是容易获得的。

当然，您总是要考虑有多大的风险：您可以将智能合约与一个对公众开放（因此也对恶意行为者开放），
甚至可能是开源的网络服务进行比较。如果您只在该网络服务上存储您的杂货清单，
您可能不必太过小心，但如果您使用该网络服务管理您的银行账户，您就应该更加小心。

本节将列出一些陷阱和一般安全建议，但当然不可能是完整的。
此外，请记住，即使您的智能合约代码没有错误，编译器或平台本身也可能有一个错误。
编译器的一些公开的，与安全有关的bug列表可以在 :ref:`已知错误列表 <known_bugs>` 中找到，
它也是机器可读的。请注意，有一个涵盖 Solidity 编译器代码生成器的错误赏金计划。

像往常一样，对于开源文档，请帮助我们扩展这部分内容（尤其是，一些例子不会有什么影响）！
=======
In Solidity, this is even more important because you can use smart contracts to handle tokens or,
possibly, even more valuable things.
Furthermore, every execution of a smart contract happens in public and,
in addition to that, the source code is often available.

Of course, you always have to consider how much is at stake:
You can compare a smart contract with a web service that is open to the public
(and thus, also to malicious actors) and perhaps even open-source.
If you only store your grocery list on that web service, you might not have to take too much care,
but if you manage your bank account using that web service, you should be more careful.

This section will list some pitfalls and general security recommendations
but can, of course, never be complete.
Also, keep in mind that even if your smart contract code is bug-free,
the compiler or the platform itself might have a bug.
A list of some publicly known security-relevant bugs of the compiler can be found
in the :ref:`list of known bugs<known_bugs>`, which is also machine-readable.
Note that there is a `Bug Bounty Program <https://ethereum.org/en/bug-bounty/>`_
that covers the code generator of the Solidity compiler.

As always, with open-source documentation,
please help us extend this section (especially, some examples would not hurt)!
>>>>>>> english/develop

注意：除了下面的列表，您也可以在
`Guy Lando 的知识列表 <https://github.com/guylando/KnowledgeLists/blob/master/EthereumSmartContracts.md>`_
和 `Consensys GitHub 代码仓库 <https://consensys.github.io/smart-contract-best-practices/>`_
中找到更多的安全建议和最佳实践。

********
陷阱
********

隐私信息和随机性
==================================

<<<<<<< HEAD
您在智能合约中使用的所有东西都是公开可见的，即使是标记为 ``private`` 的局部变量和状态变量。

如果你不希望区块构造者能够作弊，在智能合约中使用随机数是相当棘手的。

重入
===========

一个合约（A）与另一个合约（B）的任何交互和任何以太币的转移都会将控制权交给该合约（B）。
这使得 B 有可能在这个交互完成之前回调回 A。举个例子，
下面的代码包含了一个漏洞（这只是一个片段，而不是一个完整的合约）：
=======
Everything you use in a smart contract is publicly visible,
even local variables and state variables marked ``private``.

Using random numbers in smart contracts is quite tricky if you do not want block builders to be able to cheat.

Reentrancy
==========

Any interaction from a contract (A) with another contract (B)
and any transfer of Ether hands over control to that contract (B).
This makes it possible for B to call back into A before this interaction is completed.
To give an example, the following code contains a bug (it is just a snippet and not a complete contract):
>>>>>>> english/develop

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;

    // 此合约包含一个漏洞 - 请勿使用
    contract Fund {
        /// @dev 合约的以太币份额的映射。
        mapping(address => uint) shares;
        /// 提取您的份额。
        function withdraw() public {
            if (payable(msg.sender).send(shares[msg.sender]))
                shares[msg.sender] = 0;
        }
    }

<<<<<<< HEAD
这里的问题不是太严重，因为作为 ``send`` 的一部分，gas 有限，
但它仍然暴露了一个弱点: 以太币的转移总是可以包括代码的执行，
所以接收者可以是一个回调到 ``withdraw`` 的合约。
这将让它获得多次退款，并基本上取回合约中的所有以太。
特别的是，下面的合约将允许攻击者多次退款，因为它使用了 ``call``，
它会默认转发所有剩余 gas。
=======
The problem is not too serious here because of the limited gas as part of ``send``,
but it still exposes a weakness:
Ether transfer can always include code execution,
so the recipient could be a contract that calls back into ``withdraw``.
This would let it get multiple refunds and, basically, retrieve all the Ether in the contract.
In particular, the following contract will allow an attacker to refund multiple times
as it uses ``call`` which forwards all remaining gas by default:
>>>>>>> english/develop

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.2 <0.9.0;

    //此合约包含一个漏洞 - 请勿使用
    contract Fund {
        /// @dev 合约的以太币份额的映射。
        mapping(address => uint) shares;
        /// 提取您的份额。
        function withdraw() public {
            (bool success,) = msg.sender.call{value: shares[msg.sender]}("");
            if (success)
                shares[msg.sender] = 0;
        }
    }

<<<<<<< HEAD
为了避免重入，您可以使用如下所示的 检查-生效-交互（Checks-Effects-Interactions）模式：
=======
To avoid reentrancy, you can use the Checks-Effects-Interactions pattern as demonstrated below:
>>>>>>> english/develop

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;

    contract Fund {
        /// @dev 合约的以太币份额的映射。
        mapping(address => uint) shares;
        /// 提取您的份额。
        function withdraw() public {
            uint share = shares[msg.sender];
            shares[msg.sender] = 0;
            payable(msg.sender).transfer(share);
        }
    }

<<<<<<< HEAD
检查-生效-交互模式确保所有通过合约的代码路径
在修改合约的状态（检查）之前完成对所提供的参数的所有必要检查；只有这样它才会对状态进行任何改变（生效）；
在所有计划的状态改变被写入存储 *之后* ，它可能会对其他合约中的函数进行调用（交互）。
这是一种常见的防止 *重入性攻击* 的万无一失的方法，在这种情况下，
外部调用的恶意合约能够通过使用在原始合约最终完成交易之前回调原始合约的逻辑来重复花费津贴，重复提取余额，以及其他事情。

请注意，重入不仅是对以太币转移的影响，
也是对另一个合约的任何函数调用的影响。
此外，您还必须考虑到多合约的情况。
一个被调用的合约可以修改您所依赖的另一个合约的状态。
=======
The Checks-Effects-Interactions pattern ensures that all code paths through a contract
complete all required checks of the supplied parameters before modifying the contract's state (Checks);
only then it makes any changes to the state (Effects);
it may make calls to functions in other contracts
*after* all planned state changes have been written to storage (Interactions).
This is a common foolproof way to prevent *reentrancy attacks*,
where an externally called malicious contract can double-spend an allowance,
double-withdraw a balance, among other things,
by using logic that calls back into the original contract before it has finalized its transaction.

Note that reentrancy is not only an effect of Ether transfer
but of any function call on another contract.
Furthermore, you also have to take multi-contract situations into account.
A called contract could modify the state of another contract you depend on.
>>>>>>> english/develop

gas 限制和循环
===================

<<<<<<< HEAD
对于没有固定迭代次数的循环，例如，依赖于存储值的循环，必须谨慎使用：
由于块 gas 的限制，事务只能消耗一定量的 gas。
无论是明确的还是仅仅由于正常的操作，循环中的迭代次数可以增长到超过块 gas 限制，
这可能导致完整的合约在某一点上停滞。这可能不适用于只为从区块链上读取数据而执行的 ``view`` 函数。
但是，这样的函数可能会被其他合约调用，作为链上操作的一部分，并使其停滞。请在您的合约文档中明确说明这种情况。
=======
Loops that do not have a fixed number of iterations, for example,
loops that depend on storage values, have to be used carefully:
Due to the block gas limit, transactions can only consume a certain amount of gas.
Either explicitly or just due to normal operation,
the number of iterations in a loop can grow beyond the block gas limit
which can cause the complete contract to be stalled at a certain point.
This may not apply to ``view`` functions that are only executed to read data from the blockchain.
Still, such functions may be called by other contracts as part of on-chain operations and stall those.
Please be explicit about such cases in the documentation of your contracts.
>>>>>>> english/develop

发送和接收以太币
===========================

<<<<<<< HEAD
- 无论是合约还是 “外部账户”，目前都无法阻止有人向他们发送以太币。
  合约可以对普通的转账做出反应并拒绝，但有一些方法可以在不创建消息调用的情况下转移以太币。
  一种方法是简单地向合约地址“挖矿”，第二种方法是使用 ``selfdestruct(x)``。

- 如果一个合约收到了以太（没有函数被调用），要么是执行 :ref:`receive 方法 <receive-ether-function>`，
  要么执行 :ref:`fallback <fallback-function>` 函数。如果它没有 receive 也没有 fallback 函数，
  那么该以太将被拒绝（抛出一个异常）。在这些函数的执行过程中，
  合约只能依靠此时它所传递的 “gas津贴”（2300 gas）可用。但这个津贴不足以修改存储
  （但不要认为这是理所当然的，这个津贴可能会随着未来的硬分叉而改变）。
  为了确保您的合约能够以这种方式接收以太，请检查 receive 和 fallback 函数的 gas 要求
  （在 Remix 的“详细”章节会举例说明）。

- 有一种方法可以使用 ``addr.call{value: x}("")`` 将更多的 gas 转发给接收合约。
  这与 ``addr.transfer(x)`` 本质上是一样的，只是它转发了所有剩余的 gas，
  并为接收方提供了执行更昂贵的操作的能力（而且它返回一个失败代码，而不是自动传播错误）。
  这可能包括回调到发送合约或其他您可能没有想到的状态变化。
  因此，这种方法无论是给诚实用户还是恶意行为者都提供了极大的灵活性。

- 尽可能使用最精确的单位来表示 wei 的数量，因为您会因为缺乏精确性而失去任何四舍五入的结果。
=======
- Neither contracts nor "external accounts" are currently able to prevent someone from sending them Ether.
  Contracts can react on and reject a regular transfer, but there are ways to move Ether without creating a message call.
  One way is to simply "mine to" the contract address and the second way is using ``selfdestruct(x)``.

- If a contract receives Ether (without a function being called), either the :ref:`receive Ether <receive-ether-function>`
  or the :ref:`fallback <fallback-function>` function is executed.
  If it does not have a ``receive`` nor a ``fallback`` function, the Ether will be rejected (by throwing an exception).
  During the execution of one of these functions, the contract can only rely on the "gas stipend" it is passed (2300 gas)
  being available to it at that time.
  This stipend is not enough to modify storage (do not take this for granted though, the stipend might change with future hard forks).
  To be sure that your contract can receive Ether in that way, check the gas requirements of the receive and fallback functions
  (for example in the "details" section in Remix).

- There is a way to forward more gas to the receiving contract using ``addr.call{value: x}("")``.
  This is essentially the same as ``addr.transfer(x)``, only that it forwards all remaining gas
  and opens up the ability for the recipient to perform more expensive actions
  (and it returns a failure code instead of automatically propagating the error).
  This might include calling back into the sending contract or other state changes you might not have thought of.
  So it allows for great flexibility for honest users but also for malicious actors.

- Use the most precise units to represent the Wei amount as possible, as you lose any that is rounded due to a lack of precision.
>>>>>>> english/develop

- 如果您想用 ``address.transfer`` 来发送以太，有一些细节需要注意：

  1. 如果接收者是一个合约，它会导致其 receive 或 fallback 函数被执行，
     而该函数又可以回调发送以太的合约。
  2. 发送以太可能由于调用深度超过1024而失败。由于调用者完全控制着调用深度，他们可以迫使传输失败；
     考虑到这种可能性，或者使用 ``send``，并确保总是检查其返回值。
     更好的办法是，使用接收者可以提取以太币的模式来编写您的合约。
  3. 发送以太也可能失败，因为接收合约的执行需要超过分配的 gas 值
     （确切地说，是使用了 :ref:`require <assert-and-require>`， :ref:`assert <assert-and-require>`，
     :ref:`revert <assert-and-require>` 或者因为操作太昂贵）- 它 “耗尽了 gas“（OOG）。
     如果您使用 ``transfer`` 或 ``send``，并带有返回值检查，这可能为接收者提供一种手段来阻止发送合约的进展。
     同样，这里的最佳做法是使用 :ref:`"提款" 模式而不是 "发送"模式 <withdrawal_pattern>`。

调用栈深度
================

<<<<<<< HEAD
外部函数调用随时都可能失败，因为它们超过了最大调用堆栈大小1024的限制。
在这种情况下，Solidity 会抛出一个异常。恶意的行为者可能会在与您的合约交互之前，
将调用堆栈逼到一个高值。请注意，由于 `桔子哨子（Tangerine Whistle） <https://eips.ethereum.org/EIPS/eip-608>`_
硬分叉， `63/64规则 <https://eips.ethereum.org/EIPS/eip-150>`_ 使得调用栈深度攻击不切实际。
还要注意的是，调用栈和表达式栈是不相关的，尽管两者都有1024个栈槽的大小限制。

注意 ``.send()`` 在调用栈被耗尽的情况下 **不会** 抛出异常，
而是会返回 ``false``。低级函数 ``.call()``， ``.delegatecall()`` 和 ``.staticcall()``
也都是这样的。
=======
External function calls can fail at any time
because they exceed the maximum call stack size limit of 1024.
In such situations, Solidity throws an exception.
Malicious actors might be able to force the call stack to a high value
before they interact with your contract.
Note that, since `Tangerine Whistle <https://eips.ethereum.org/EIPS/eip-608>`_ hardfork,
the `63/64 rule <https://eips.ethereum.org/EIPS/eip-150>`_ makes call stack depth attack impractical.
Also note that the call stack and the expression stack are unrelated,
even though both have a size limit of 1024 stack slots.

Note that ``.send()`` does **not** throw an exception if the call stack is depleted
but rather returns ``false`` in that case.
The low-level functions ``.call()``, ``.delegatecall()`` and ``.staticcall()`` behave in the same way.
>>>>>>> english/develop

授权的代理
==================

<<<<<<< HEAD
如果您的合约可以作为一个代理，也就是说，如果它可以用用户提供的数据调用任意的合约，
那么用户基本上可以承担代理合约的身份。即使您有其他的保护措施，
最好是建立您的合约系统，使代理没有任何权限（甚至对自己也没有）。
如果需要，您可以使用第二个代理来完成：
=======
If your contract can act as a proxy, i.e. if it can call arbitrary contracts with user-supplied data,
then the user can essentially assume the identity of the proxy contract.
Even if you have other protective measures in place, it is best to build your contract system such
that the proxy does not have any permissions (not even for itself).
If needed, you can accomplish that using a second proxy:
>>>>>>> english/develop

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.0;
    contract ProxyWithMoreFunctionality {
        PermissionlessProxy proxy;

        function callOther(address addr, bytes memory payload) public
                returns (bool, bytes memory) {
            return proxy.callOther(addr, payload);
        }
        // 其他函数和其他功能
    }

    // 这是完整的合约，它没有其他功能，不需要任何权限就可以工作。
    contract PermissionlessProxy {
        function callOther(address addr, bytes memory payload) public
                returns (bool, bytes memory) {
            return addr.call(payload);
        }
    }

tx.origin
=========

<<<<<<< HEAD
永远不要使用 tx.origin 做身份认证。假设您有一个这样的钱包合约：
=======
Never use ``tx.origin`` for authorization.
Let's say you have a wallet contract like this:
>>>>>>> english/develop

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
    // 本合约包含一个漏洞 - 请勿使用
    contract TxUserWallet {
        address owner;

        constructor() {
            owner = msg.sender;
        }

        function transferTo(address payable dest, uint amount) public {
            // 漏洞就在这里，您必须使用 msg.sender 而不是 tx.origin。
            require(tx.origin == owner);
            dest.transfer(amount);
        }
    }

现在有人欺骗您，让您向这个攻击钱包的地址发送以太币：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.0 <0.9.0;
    interface TxUserWallet {
        function transferTo(address payable dest, uint amount) external;
    }

    contract TxAttackWallet {
        address payable owner;

        constructor() {
            owner = payable(msg.sender);
        }

        receive() external payable {
            TxUserWallet(msg.sender).transferTo(owner, msg.sender.balance);
        }
    }

<<<<<<< HEAD
如果您的钱包检查了 ``msg.sender`` 的授权，它将得到攻击钱包的地址，而不是所有者地址。
但是通过检查 ``tx.origin``，它得到的是启动交易的原始地址，这仍然是所有者地址。
攻击钱包会立即耗尽您的所有资金。
=======
If your wallet had checked ``msg.sender`` for authorization, it would get the address of the attack wallet,
instead of the owner's address.
But by checking ``tx.origin``, it gets the original address that kicked off the transaction,
which is still the owner's address.
The attack wallet instantly drains all your funds.
>>>>>>> english/develop

.. _underflow-overflow:

二进制补码 / 下溢 / 上溢
=========================================

正如在许多编程语言中，Solidity 的整数类型实际上不是整数。
当数值较小时，它们类似于整数，但也不能表示任意大的数字。

下面的代码会导致溢出，因为加法的结果太大，不能存储在 ``uint8`` 类型中：

.. code-block:: solidity

  uint8 x = 255;
  uint8 y = 1;
  return x + y;

Solidity 有两种模式来处理这些溢出。检查和不检查或 “包装” 模式。

默认的检查模式将检测到溢出并导致一个失败的断言。
您可以使用 ``unchecked { ... }``，使溢出被无声地忽略。
上面的代码如果用 ``unchecked { ... }`` 包装，将返回 ``0``。

即使在检查模式下，也不要认为您受到了保护，不会出现溢出错误。
在这种模式下，溢出总是会被还原。如果无法避免溢出，这可能导致智能合约被卡在某个状态。

一般来说，请阅读关于二进制补码表示法的限制，它甚至对有符号的数字有一些更特殊的边缘情况。

尝试使用 ``require`` 将输入的大小限制在一个合理的范围内，
并使用:ref:`SMT 检查器 <smt_checker>` 来发现潜在的溢出。

.. _clearing-mappings:

清除映射
=================

<<<<<<< HEAD
Solidity ``mapping`` 类型（见 :ref:`mapping-types`）是一个仅有存储空间的键值数据结构，
它不跟踪被分配非零值的键。正因为如此，清理映射时不可能有关于写入键的额外信息。
如果 ``mapping`` 被用作动态存储数组的基本类型，删除或弹出数组将不会对 ``mapping`` 元素产生影响。
例如，如果一个 ``mapping`` 被用作一个 ``struct`` 的成员字段的类型，
而该结构是一个动态存储阵列的基本类型，同样的情况也会发生。
``mapping`` 在包含 ``mapping`` 的结构或数组的分配中也会被忽略。

=======
The Solidity type ``mapping`` (see :ref:`mapping-types`) is a storage-only key-value data structure
that does not keep track of the keys that were assigned a non-zero value.
Because of that, cleaning a mapping without extra information about the written keys is not possible.
If a ``mapping`` is used as the base type of a dynamic storage array,
deleting or popping the array will have no effect over the ``mapping`` elements.
The same happens, for example, if a ``mapping`` is used as the type of a member field of a ``struct``
that is the base type of a dynamic storage array.
The ``mapping`` is also ignored in assignments of structs or arrays containing a ``mapping``.
>>>>>>> english/develop

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;

    contract Map {
        mapping(uint => uint)[] array;

        function allocate(uint newMaps) public {
            for (uint i = 0; i < newMaps; i++)
                array.push();
        }

        function writeMap(uint map, uint key, uint value) public {
            array[map][key] = value;
        }

        function readMap(uint map, uint key) public view returns (uint) {
            return array[map][key];
        }

        function eraseMaps() public {
            delete array;
        }
    }

<<<<<<< HEAD
考虑一下上面的例子和下面的调用序列： ``allocate(10)``， ``writeMap(4, 128, 256)``。
此时，调用 ``readMap(4, 128)`` 返回256。如果我们调用 ``eraseMaps``，
状态变量 ``array`` 的长度被清零，但由于它的 ``mapping`` 元素不能被清零，
它们的信息在合约的存储中仍然存在。
删除 ``array`` 后，调用 ``allocate(5)`` 允许我们再次访问 ``array[4]``，
调用 ``readMap(4, 128)`` 则返回256，即使没有再次调用 ``writeMap``。
=======
Consider the example above and the following sequence of calls: ``allocate(10)``, ``writeMap(4, 128, 256)``.
At this point, calling ``readMap(4, 128)`` returns 256.
If we call ``eraseMaps``, the length of the state variable ``array`` is zeroed,
but since its ``mapping`` elements cannot be zeroed, their information stays alive in the contract's storage.
After deleting ``array``, calling ``allocate(5)`` allows us to access ``array[4]`` again,
and calling ``readMap(4, 128)`` returns 256 even without another call to ``writeMap``.
>>>>>>> english/develop

如果您的 ``mapping`` 信息必须被删除，可以考虑使用类似于
`可迭代的映射 <https://github.com/ethereum/dapp bin/blob/master/library/iterable_mapping.sol>`_ 的库，
它允许您在适当的 ``mapping`` 中遍历键并删除其值。

细枝末节
=============

<<<<<<< HEAD
- 没有占满32字节的类型可能包含 “脏高位”。
  这在当您访问 ``msg.data`` 的时候尤为重要 —— 它带来了延展性风险：
  您既可以用原始字节 ``0xff000001``，也可以用 ``0x00000001`` 作为参数来调用
  函数 ``f(uint8 x)``  以构造交易。
  您可以制作一些交易，调用一个函数 ``f(uint8 x)`` ，这两个参数都会被正常提供给合约，
  就 ``x``  而言，两者看起来都是数字 ``1``， 但 ``msg.data`` 将是不同的，
  所以如果您无论怎么使用 ``keccak256(msg.data)``，您都会得到不同的结果。

=======
- Types that do not occupy the full 32 bytes might contain "dirty higher order bits".
  This is especially important if you access ``msg.data`` - it poses a malleability risk:
  You can craft transactions that call a function ``f(uint8 x)``
  with a raw byte argument of ``0xff000001`` and with ``0x00000001``.
  Both are fed to the contract and both will look like the number ``1`` as far as ``x`` is concerned,
  but ``msg.data`` will be different, so if you use ``keccak256(msg.data)`` for anything,
  you will get different results.
>>>>>>> english/develop

***************
推荐做法
***************

认真对待警告
=======================

<<<<<<< HEAD
如果编译器警告您一些事情，您应该改变它。
即使您不认为这个特定的警告有安全问题，但也可能在它下面埋藏着另一个问题。
我们发出的任何编译器警告都可以通过对代码的轻微修改来消除。

始终使用最新版本的编译器，以获知所有最近引入的警告。

编译器发出的 ``info`` 类型的信息并不危险，只是代表编译器认为可能对用户有用的额外建议和可选信息。
=======
If the compiler warns you about something, you should change it.
Even if you do not think that this particular warning has security implications,
there might be another issue buried beneath it.
Any compiler warning we issue can be silenced by slight changes to the code.

Always use the latest version of the compiler to be notified about all recently introduced warnings.

Messages of type ``info``, issued by the compiler, are not dangerous
and simply represent extra suggestions and optional information
that the compiler thinks might be useful to the user.
>>>>>>> english/develop

限制以太币的数量
============================

<<<<<<< HEAD
限制智能合约中可存储的以太币（或其他代币）的数量。
如果您的源代码，编译器或平台有错误，这些资金可能会丢失。
如果您想限制您的损失，就限制以太币的数量。
=======
Restrict the amount of Ether (or other tokens) that can be stored in a smart contract.
If your source code, the compiler or the platform has a bug, these funds may be lost.
If you want to limit your loss, limit the amount of Ether.
>>>>>>> english/develop

保持合约简练且模块化
=========================

<<<<<<< HEAD
保持您的合约短小而容易理解。把不相关的功能单独放在其他合约中或放在库中。
关于源代码质量的一般建议当然也适用：限制局部变量的数量和函数的长度，等等。
给您的函数添加注释，这样别人就可以看到您的意图是什么，
并判断代码是否按照正确的意图实现。
=======
Keep your contracts small and easily understandable.
Single out unrelated functionality in other contracts or into libraries.
General recommendations about the source code quality of course apply:
Limit the amount of local variables, the length of functions and so on.
Document your functions so that others can see what your intention was
and whether it is different than what the code does.
>>>>>>> english/develop

使用“检查-生效-交互”（Checks-Effects-Interactions）模式
=======================================================

<<<<<<< HEAD
大多数函数会首先进行一些检查（谁调用了这个函数，参数是否在范围内，
他们是否发送了足够的以太，这个人是否有代币，等等）。这些检查应该首先完成。

第二步，如果所有的检查都通过了，就应该对当前合约的状态变量进行影响。
与其他合约的交互应该是任何函数的最后一步。

早期的合约延迟了一些效果，等待外部函数调用在非错误状态下返回。
这往往是一个严重的错误，因为上面解释了重入问题。
=======
Most functions will first perform some checks and they should be done first
(who called the function, are the arguments in range, did they send enough Ether,
does the person have tokens, etc.).

As the second step, if all checks passed, effects to the state variables of the current contract should be made.
Interaction with other contracts should be the very last step in any function.

Early contracts delayed some effects and waited for external function calls to return in a non-error state.
This is often a serious mistake because of the reentrancy problem explained above.
>>>>>>> english/develop

请注意，对已知合约的调用也可能反过来导致对未知合约的调用，因此，最好总是应用这种模式。

包含故障-安全（Fail-Safe）模式
==============================

<<<<<<< HEAD
尽管将系统完全去中心化可以省去许多中间环节，但包含某种故障-安全模式仍然是好的做法，
尤其是对于新的代码来说：

您可以在您的智能合约中添加一个功能，执行一些自我检查，如 “是否有任何以太币泄漏？”，
“代币的总和是否等于合约的余额？” 或类似的事情。
请记住，您不能为此使用太多的 gas，所以可能需要通过链外计算的帮助。

如果自我检查失败，合约会自动切换到某种 “故障安全” 模式，
例如，禁用大部分功能，将控制权移交给一个固定的，可信赖的第三方，
或者只是将合约转换为一个简单的 “退回我的钱” 的合约。
=======
While making your system fully decentralized will remove any intermediary,
it might be a good idea, especially for new code, to include some kind of fail-safe mechanism:

You can add a function in your smart contract that performs some self-checks like "Has any Ether leaked?",
"Is the sum of the tokens equal to the balance of the contract?" or similar things.
Keep in mind that you cannot use too much gas for that,
so help through off-chain computations might be needed there.

If the self-check fails, the contract automatically switches into some kind of "failsafe" mode,
which, for example, disables most of the features,
hands over control to a fixed and trusted third party
or just converts the contract into a simple "give me back my money" contract.
>>>>>>> english/develop

请求同行评审
===================

<<<<<<< HEAD
检查一段代码的人越多，发现的问题就越多。
要求其他人审查您的代码也有助于作为交叉检查，
找出您的代码是否容易理解 - 这是好的智能合约的一个非常重要的标准。
=======
The more people examine a piece of code, the more issues are found.
Asking people to review your code also helps as a cross-check to find out
whether your code is easy to understand -
a very important criterion for good smart contracts.
>>>>>>> english/develop
