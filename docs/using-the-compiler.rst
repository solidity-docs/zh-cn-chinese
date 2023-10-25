******************
使用编译器
******************

.. index:: ! commandline compiler, compiler;commandline, ! solc

.. _commandline-compiler:

使用命令行编译器
******************************

.. note::
    这一节并不适用于 :ref:`solcjs <solcjs>`， 即使在命令行模式下使用也不行。

基本用法
-----------

``solc`` 是 Solidity 仓库的构建目标之一, 它是 Solidity 命令行编译器。
使用 ``solc --help`` 可以为您提供所有选项的解释。编译器可以产生各种输出，
从简单的二进制文件和抽象语法树（解析树）上的汇编到gas使用量的估计。
如果您只想编译一个文件，您可以运行 ``solc --bin sourceFile.sol`` 来生成二进制文件。
如果您想通过 ``solc`` 获得一些更高级的输出信息，
可以通过 ``solc -o outputDirectory --bin --ast-compact-json --asm sourceFile.sol`` 命令
将所有的输出都保存到单独的文件中。

优化器选项
-----------------

在您部署合约之前，在编译时使用 ``solc --optimize --bin sourceFile.sol`` 激活优化器。
默认情况下，优化器将假设合约在其生命周期内被调用200次（更确切地说，它假设每个操作码被执行200次左右）。
如果您想让最初的合约部署更便宜，而后来的函数执行更昂贵，请设置为 ``--optimize-runs=1``。
如果您期望有很多交易，并且不在乎更高的部署成本和输出大小，那么把 ``--optimize-runs`` 设置成一个高的数字。
这个参数对以下方面有影响（将来可能会改变）：

- 函数调度程序中二进制搜索的大小
- 像大数字或字符串等常量的存储方式

.. index:: allowed paths, --allow-paths, base path, --base-path, include paths, --include-path

基本路径和导入重映射
------------------------------

命令行编译器将自动从文件系统中读取导入的文件，但同时，
它也支持通过如下方式，用 ``prefix=path`` 选项将 :ref:`路径重定向 <import-remapping>`：

.. code-block:: bash

    solc github.com/ethereum/dapp-bin/=/usr/local/lib/dapp-bin/ file.sol

这实质上是指示编译器在 ``/usr/local/lib/dapp-bin`` 下搜索
所有以 ``github.com/ethereum/dapp-bin/`` 开头的文件。

当访问文件系统搜索导入文件时，:ref:`不以./或../开头的路径 <direct-imports>` 被视为
相对于使用 ``--base-path`` 和 ``--include-path`` 选项指定的目录（如果没有指定基本路径，则是当前工作目录）。
此外，通过这些选项添加的路径部分将不会出现在合约元数据中。

<<<<<<< HEAD
出于安全考虑，编译器 :ref:`对它可以访问的目录有一些限制 <allowed-paths>`。
在命令行中指定的源文件的目录和重映射的目标路径被自动允许被文件阅读器访问，
但其他的都是默认为拒绝的。
通过 ``--allow-paths /sample/path,/another/sample/path`` 语句可以允许额外的路径（和它们的子目录）。
通过 ``--base-path`` 指定的路径内的所有内容都是允许的。
=======
For security reasons the compiler has :ref:`restrictions on what directories it can access <allowed-paths>`.
Directories of source files specified on the command-line and target paths of
remappings are automatically allowed to be accessed by the file reader, but everything
else is rejected by default.
Additional paths (and their subdirectories) can be allowed via the
``--allow-paths /sample/path,/another/sample/path`` switch.
Everything inside the path specified via ``--base-path`` is always allowed.
>>>>>>> english/develop

以上只是对编译器如何处理导入路径的一个简化。
关于详细的解释，包括例子和边缘情况的讨论，请参考 :ref:`路径解析 <path-resolution>` 一节。

.. index:: ! linker, ! --link, ! --libraries
.. _library-linking:

库链接
---------------

如果您的合约使用 :ref:`库合约 <libraries>`，
您会注意到字节码中含有 ``__$53aea86b7d70b31448b230b20ae141a537$__`` 形式的字符串。
这些是实际库的地址的占位符。此占位符是完全限定库名的keccak256散列的十六进制编码的34个字符前缀。
字节码文件也将包含形式为 ``// <placeholder> -> <fq library name>`` 的代码行，以帮助识别占位符代表的库。
注意，完全限定的库名是其源文件的路径和用 ``:`` 分隔的库名。
您可以使用 ``solc`` 作为链接器，意味着您将在这些地方插入库的地址：

要么在您的命令中加入
``--libraries "file.sol:Math=0x1234567890123456789012345678901234567890 file.sol:Heap=0xabCD567890123456789012345678901234567890"``，
为每个库提供一个地址（用逗号或空格作为分隔符），要么将字符串存储在一个文件中（每行一个库），
用 ``-libraries fileName`` 运行 ``solc``。

.. note::
    从Solidity 0.8.1 开始，接受 ``=`` 作为库和地址之间的分隔符，而 ``:`` 作为分隔符已被废弃。
    它将在未来被删除。目前 ``--libraries "file.sol:Math:0x1234567890123456789012345678901234567890 file.sol:Heap:0xabCD56789012345678901234567890"`` 也可以工作。

.. index:: --standard-json, --base-path

如果调用 ``solc`` 时有 ``--standard-json`` 选项，它将在标准输入中期待一个JSON输入（如下所述），
并在标准输出中返回一个JSON输出。这是对更复杂的，特别是自动化使用时的推荐接口。
该进程将始终以 “成功” 状态终止，并通过JSON输出来报告任何错误。
选项 ``--base-path`` 也以标准JSON模式处理。

如果调用 ``solc`` 时带有 ``--link`` 选项，所有输入文件都被编译成格式为 ``__$53aea86b7d70b31448b230b20ae141a537$__``
形式的未链接的二进制文件（十六进制编码），并被本地链接（如果从标准输入（stdin）读取输入，则被写到标准输出（stdout））。
在这种情况下，除了 ``--libraries`` 以外的所有选项都被忽略（包括 ``-o`` ）。

.. warning::
    不推荐在生成的字节码上手动链接库文件，因为它不会更新合约元数据。
    由于元数据包含在编译时指定的库的列表，而字节码包含元数据哈希，
    您将得到不同的二进制文件，并且这取决于何时进行链接。

    您应该在编译合约时请求编译器链接库文件，方法是使用 ``solc`` 的 ``--libraries`` 选项
    或 ``libraries`` 键（如果您使用编译器的标准JSON接口）。

.. note::
    库的占位符曾经是库本身的完全限定名称，而不是它的哈希值。
    这种格式仍然被 ``solc --link`` 支持，但编译器将不再输出它。
    这一改变是为了减少库之间发生碰撞的可能性，因为只有完全限定的库名的前36个字符可以被使用。

.. _evm-version:
.. index:: ! EVM version, compile target

将EVM版本设置为目标版本
*********************************

<<<<<<< HEAD
当您编译您的合约代码时，您可以指定以太坊虚拟机版本来编译，以避免特定的功能或行为。

.. warning::

   在错误的EVM版本进行编译会导致错误，奇怪和失败的行为。
   请确保，特别是在运行一个私有链的情况下，您使用匹配的EVM版本。

在命令行中，您可以选择EVM的版本，如下所示：
=======
When you compile your contract code you can specify the Ethereum virtual machine
version to compile for to avoid particular features or behaviors.

.. warning::

   Compiling for the wrong EVM version can result in wrong, strange and failing
   behavior. Please ensure, especially if running a private chain, that you
   use matching EVM versions.

On the command-line, you can select the EVM version as follows:
>>>>>>> english/develop

.. code-block:: shell

  solc --evm-version <VERSION> contract.sol

在 :ref:`标准 JSON 接口 <compiler-api>` 中，使用 ``"settings"`` 字段中的键 ``"evmVersion"``。

.. code-block:: javascript

    {
      "sources": {/* ... */},
      "settings": {
        "optimizer": {/* ... */},
        "evmVersion": "<VERSION>"
      }
    }

EVM版本选项
--------------

以下是一个EVM版本的列表，以及每个版本中引入的编译器相关变化。
每个版本之间不保证向后兼容。

<<<<<<< HEAD
- ``homestead``
   - （最老的版本）
- ``tangerineWhistle``
   - 访问其他账户的gas成本增加，与gas估算和优化器有关。
   - 对于外部调用，所有gas都是默认发送的，以前必须保留一定的数量。
- ``spuriousDragon``
   - ``exp`` 操作码的gas成本增加，与gas估计和优化器有关。
- ``byzantium``
   - 在汇编中可使用操作码 ``returndatacopy``， ``returndatasize`` 和 ``staticcall``。
   - ``staticcall`` 操作码在调用非库合约 view 或 pure 函数时使用，它可以防止函数在EVM级别修改状态，也就是说，甚至适用于您使用无效的类型转换时。
   - 可以访问从函数调用返回的动态数据。
   - 引入了 ``revert`` 操作码，这意味着 ``revert`` 将不会浪费gas。
=======
- ``homestead`` (*support deprecated*)
   - (oldest version)
- ``tangerineWhistle`` (*support deprecated*)
   - Gas cost for access to other accounts increased, relevant for gas estimation and the optimizer.
   - All gas sent by default for external calls, previously a certain amount had to be retained.
- ``spuriousDragon`` (*support deprecated*)
   - Gas cost for the ``exp`` opcode increased, relevant for gas estimation and the optimizer.
- ``byzantium`` (*support deprecated*)
   - Opcodes ``returndatacopy``, ``returndatasize`` and ``staticcall`` are available in assembly.
   - The ``staticcall`` opcode is used when calling non-library view or pure functions, which prevents the functions from modifying state at the EVM level, i.e., even applies when you use invalid type conversions.
   - It is possible to access dynamic data returned from function calls.
   - ``revert`` opcode introduced, which means that ``revert()`` will not waste gas.
>>>>>>> english/develop
- ``constantinople``
   - 在汇编中可使用操作码 ``create2``, ``extcodehash``, ``shl``, ``shr`` 和 ``sar``。
   - 移位运算符使用移位运算码，因此需要的gas较少。
- ``petersburg``
   - 编译器的行为与 constantinople 版本的行为相同。
- ``istanbul``
   - 在汇编中可使用操作码 ``chainid`` 和 ``selfbalance``。
- ``berlin``
<<<<<<< HEAD
   - ``SLOAD``， ``*CALL``， ``BALANCE``， ``EXT*`` 和 ``SELFDESTRUCT`` 的gas成本增加。
     编译器假设这类操作的gas成本是固定的。这与gas估计和优化器有关。
- ``london`` 
   - 区块的基本费用（ `EIP-3198 <https://eips.ethereum.org/EIPS/eip-3198>`_ 和 `EIP-1559 <https://eips.ethereum.org/EIPS/eip-1559>`_ ）
     可以通过全局的 ``block.basefee`` 或内联汇编中的 ``basefee()`` 访问。
- ``paris`` 
   - 引入了 ``prevrandao()`` 和 ``block.prevrandao``，并改变了现在已经废弃的 ``block.difficulty`` 的语义，不允许在内联汇编中使用 ``difficulty()`` （见 `EIP-4399 <https://eips.ethereum.org/EIPS/eip-4399>`_ ）。
- ``shanghai`` （ **默认项** ）
  - 由于引入了 ``push0``，代码尺寸更小，并且节省了gas（参见 `EIP-3855 <https://eips.ethereum.org/EIPS/eip-3855>`_）。
=======
   - Gas costs for ``SLOAD``, ``*CALL``, ``BALANCE``, ``EXT*`` and ``SELFDESTRUCT`` increased. The
     compiler assumes cold gas costs for such operations. This is relevant for gas estimation and
     the optimizer.
- ``london``
   - The block's base fee (`EIP-3198 <https://eips.ethereum.org/EIPS/eip-3198>`_ and `EIP-1559 <https://eips.ethereum.org/EIPS/eip-1559>`_) can be accessed via the global ``block.basefee`` or ``basefee()`` in inline assembly.
- ``paris``
   - Introduces ``prevrandao()`` and ``block.prevrandao``, and changes the semantics of the now deprecated ``block.difficulty``, disallowing ``difficulty()`` in inline assembly (see `EIP-4399 <https://eips.ethereum.org/EIPS/eip-4399>`_).
- ``shanghai`` (**default**)
   - Smaller code size and gas savings due to the introduction of ``push0`` (see `EIP-3855 <https://eips.ethereum.org/EIPS/eip-3855>`_).
>>>>>>> english/develop

.. index:: ! standard JSON, ! --standard-json
.. _compiler-api:

编译器输入和输出JSON说明
******************************************

推荐的与Solidity编译器连接的方式，特别是对于更复杂和自动化的设置，是所谓的JSON输入输出接口。
编译器的所有发行版都提供相同的接口。

这些字段一般都会有变化，有些是可选的（如前所述），但我们尽量只做向后兼容的改动。

编译器API期望JSON格式的输入，并将编译结果输出为JSON格式的输出。
不使用标准错误输出，进程将始终以 “成功” 状态终止，即使存在错误。错误总是作为JSON输出的一部分报告。

以下各小节通过一个例子来描述该格式。
当然，注释是不允许的，在此仅用于解释。

输入说明
-----------------

.. code-block:: javascript

    {
      // 必选：源代码语言。目前支持的是 “Solidity“，“Yul“ 和 “SolidityAST” （实验性的）。
      "language": "Solidity",
      // 必选
      "sources":
      {
        // 这里的键值是源文件的 “全局“ 名称，
        // 导入文件可以通过重映射使用其他文件（见下文）。
        "myFile.sol":
        {
          // 可选： 源文件的kaccak256哈希值
          // 如果通过URL导入，它用于验证检索的内容。
          "keccak256": "0x123...",
          // 必选（除非声明了 "content" 字段，参见下文）: 指向源文件的URL。
          // 应按此顺序导入URL，并根据keccak256哈希值检查结果（如果有的话）。
          // 如果哈希值不匹配，或者没有一个URL(s)的结果是成功的，就应该产生一个错误。
          // 使用命令行界面只支持文件系统路径。
          // 通过JavaScript接口，URL将被传递给用户提供的读取回调，因此可以使用回调支持的任何URL。
          "urls":
          [
            "bzzr://56ab...",
            "ipfs://Qma...",
            "/tmp/path/to/file.sol"
<<<<<<< HEAD
            // 如果使用文件，其目录应通过 `--allow-paths <path>` 添加到命令行中。
=======
            // If files are used, their directories should be added to the command-line via
            // `--allow-paths <path>`.
>>>>>>> english/develop
          ]
          // 如果语言设置为 “SolidityAST”，则需要在 “ast” 字段下提供 AST。
          // 请注意，ASTs 的导入是试验性的，尤其是：
          // - 导入无效的 ASTs 会产生未定义的结果，并且
          // - 对无效的 ASTs 不提供适当的错误报告。
          // 此外，请注意 AST 导入只消耗编译器在 “stopAfter”（停止后）模式下生成的 AST 字段： 
          // “parsing” 模式下生成的 AST 字段，然后重新执行分析，
          // 因此 AST 中任何基于分析的注释在导入时都会被忽略。
          "ast": { ... } // 格式化为 json ast 请求的 ``ast`` 输出选择。
        },
        "destructible":
        {
          // 可选：源文件的keccak256哈希值
          "keccak256": "0x234...",
          // 必选：（除非使用 “urls“）：源文件的字面内容
          "content": "contract destructible is owned { function shutdown() { if (msg.sender == owner) selfdestruct(owner); } }"
        }
      },
      // 可选
      "settings":
      {
        // 可选： 在给定的阶段后停止编译。目前这里只有 “parsing” 有效。
        "stopAfter": "parsing",
        // 可选： 经过排序的重映射列表
        "remappings": [ ":g=/dir" ],
        // 可选： 优化器设置
        "optimizer": {
          // 默认情况下是禁用的。
          // 注意：enabled=false 仍然保留了一些优化功能。见下面的注解。
          // 警告：在0.8.6版本之前，省略 “enabled“ 键并不等同于将其设置为false，
          // 实际上会禁用所有优化。
          "enabled": true,
          // 根据您打算运行代码的次数进行优化。
          // 较低的值将更多地针对初始部署成本进行优化，
          // 较高的值将更多地针对高频使用进行优化。
          "runs": 200,
          // 打开或关闭优化器组件的细节。
          // 上面的 “enabled“ 开关提供了两个默认值，
          // 可以在这里进行调整。如果给出了 “details“，“enabled“ 可以省略。
          "details": {
            // 如果没有给出 details，窥视孔优化器总是打开的，使用 details 来关闭它。
            "peephole": true,
<<<<<<< HEAD
            // 如果没有给出 details，内联器总是打开的，
            // 使用 details来关闭它。
            "inliner": true,
            // 如果没有给出 details，未使用的跳板移除器总是打开的，
            // 使用 details来关闭它。
=======
            // The inliner is always off if no details are given,
            // use details to switch it on.
            "inliner": false,
            // The unused jumpdest remover is always on if no details are given,
            // use details to switch it off.
>>>>>>> english/develop
            "jumpdestRemover": true,
            // 在换元运算中，有时会对字词重新排序。
            "orderLiterals": false,
            // 移除重复的代码块
            "deduplicate": false,
            // 常见的子表达式消除，这是最复杂的步骤，但也能提供最大的收益。
            "cse": false,
            // 优化代码中字面数字和字符串的表示。
            "constantOptimizer": false,
<<<<<<< HEAD
            // 新的Yul优化器。主要在ABI coder v2 和 内联汇编的代码上运行。
            // 它与全局优化器设置一起被激活，并且可以在这里停用。
            // 在 Solidity 0.6.0 之前，它必须通过这个开关激活。
=======
            // Use unchecked arithmetic when incrementing the counter of for loops
            // under certain circumstances. It is always on if no details are given.
            "simpleCounterForLoopUncheckedIncrement": true,
            // The new Yul optimizer. Mostly operates on the code of ABI coder v2
            // and inline assembly.
            // It is activated together with the global optimizer setting
            // and can be deactivated here.
            // Before Solidity 0.6.0 it had to be activated through this switch.
>>>>>>> english/develop
            "yul": false,
            // Yul优化器的调优选项。
            "yulDetails": {
              // 改善变量的堆栈槽的分配，可以提前释放堆栈槽。
              // 如果Yul优化器被激活，则默认激活。
              "stackAllocation": true,
<<<<<<< HEAD
              // 选择要应用的优化步骤。
              // 也可以同时修改优化序列和清理序列。
              // 每个序列的指令用 “:” 分隔，该值以 优化-序列:清理-序列 的形式提供。
              // 更多信息见 “优化器 > 选择优化”。
              // 这个字段是可选的，如果不提供，优化和清理的默认序列都会使用。
              // 如果只提供了其中一个选项，另一个将不会被运行。
              // 如果只提供分隔符 “:”，
              // 那么优化和清理序列都不会被运行。
              // 如果设置为空值，则只使用默认的清理序列，
              // 不应用任何优化步骤。
=======
              // Select optimization steps to be applied. It is also possible to modify both the
              // optimization sequence and the clean-up sequence. Instructions for each sequence
              // are separated with the ":" delimiter and the values are provided in the form of
              // optimization-sequence:clean-up-sequence. For more information see
              // "The Optimizer > Selecting Optimizations".
              // This field is optional, and if not provided, the default sequences for both
              // optimization and clean-up are used. If only one of the sequences is provided
              // the other will not be run.
              // If only the delimiter ":" is provided then neither the optimization nor the clean-up
              // sequence will be run.
              // If set to an empty value, only the default clean-up sequence is used and
              // no optimization steps are applied.
>>>>>>> english/develop
              "optimizerSteps": "dhfoDgvulfnTUtnIf..."
            }
          }
        },
<<<<<<< HEAD
        // 编译EVM的版本。
        // 影响到类型检查和代码生成。版本可以是 homestead,
        // tangerineWhistle, spuriousDragon, byzantium, constantinople, petersburg, istanbul, berlin, london or paris
        "evmVersion": "byzantium",
        // 可选：改变编译管道以通过Yul的中间表示法。
        // 这在默认情况下是假的。
=======
        // Version of the EVM to compile for.
        // Affects type checking and code generation. Can be homestead,
        // tangerineWhistle, spuriousDragon, byzantium, constantinople,
        // petersburg, istanbul, berlin, london, paris or shanghai (default)
        "evmVersion": "shanghai",
        // Optional: Change compilation pipeline to go through the Yul intermediate representation.
        // This is false by default.
>>>>>>> english/develop
        "viaIR": true,
        // 可选： 调试设置
        "debug": {
          // 如何处理 revert（和require）的原因字符串。设置是
          // "default", "strip", "debug" 和 "verboseDebug"。
          // "default" 不注入编译器生成的revert字符串，而是保留用户提供的字符串。
          // "strip" 删除所有的revert字符串（如果可能的话，即如果使用了字面意义），以保持副作用。
          // "debug" 为编译器生成的内部revert注入字符串，目前为ABI编码器V1和V2实现。
          // "verboseDebug" 甚至将进一步的信息附加到用户提供的revert字符串中（尚未实现）。
          "revertStrings": "default",
          // 可选：在产生的EVM汇编和Yul代码的注释中包括多少额外的调试信息。可用的组件是：
          // - `location`: `@src <index>:<start>:<end>` 形式的注解，
          //   表明原始 Solidity 文件中相应元素的位置，其中：
          //     - `<index>` 是与 `@us-src` 注释相匹配的文件索引。
          //     - `<start>` 是该位置的第一个字节的索引。
          //     - `<end>` 是该位置后第一个字节的索引。
          // - `snippet`: 来自 `@src` 所示位置的单行代码片断。
          //     该片段有引号，并跟随相应的 `@src` 注释。
          // - `*`: 通配符值，可用于请求所有的东西。
          "debugInfo": ["location", "snippet"]
        },
        // 元数据设置 (可选)
        "metadata": {
          // CBOR元数据默认是附加在字节码的最后。
          // 将此设置为false，会从运行时和部署时代码中省略元数据。
          "appendCBOR": true,
          // 只使用字面内容，不使用URL（默认为false）
          "useLiteralContent": true,
          // 对附加在字节码上的元数据哈希值使用给定的哈希值方法。
          // 元数据哈希可以通过选项 "none "从字节码中删除。
          // 其他选项是 "ipfs" 和 "bzzr1"。
          // 如果省略该选项，默认使用 "ipfs"。
          "bytecodeHash": "ipfs"
        },
        // 库的地址。如果这里没有给出所有的库，
        // 可能会导致未链接的对象，其输出数据是不同的。
        "libraries": {
<<<<<<< HEAD
          // 顶层键是使用该库的源文件的名称。
          // 如果使用了重映射，这个源文件应该与应用重映射后的全局路径一致。
          // 如果这个键是一个空字符串，那就是指一个全局水平。
=======
          // The top level key is the name of the source file where the library is used.
          // If remappings are used, this source file should match the global path
          // after remappings were applied.
          // If this key is an empty string, that refers to a global level.
>>>>>>> english/develop
          "myFile.sol": {
            "MyLib": "0x123123..."
          }
        },
        // 以下可用于根据文件和合约名称选择所需的输出。
        // 如果这个字段被省略，那么编译器就会加载并进行类型检查，但除了错误之外不会产生任何输出。
        // 第一层键是文件名，第二层键是合约名。
        // 一个空的合约名称用于不与合约绑定而是与整个源文件绑定的输出，如AST。
        // 以星号作为合约名称是指文件中的所有合约。
        // 同样地，以星形作为文件名可以匹配所有文件。
        // 要选择编译器可能产生的所有输出，
        // 使用 "outputSelection"。{ "*": { "*": [ "*" ], "": [ "*" ] } }"，
        // 但要注意，这可能会不必要地减慢编译过程。
        //
        // 可用的输出类型如下：
        //
        // 文件级别（需要空字符串作为合约名称）：
        //   ast - 所有源文件的AST
        //
        // 合约级别（需要合约名称或 "*"）：
        //   abi - ABI
<<<<<<< HEAD
        //   devdoc - 开发者文档（Natspec格式）
        //   userdoc - 用户文档（Natspec格式）
        //   metadata - 元数据
        //   ir - 优化代码前的Yul中间表示法
        //   irOptimized - 优化后的中间表现
        //   storageLayout - 合约的状态变量的槽位、偏移量和类型
        //   evm.assembly - 新的汇编格式
        //   evm.legacyAssembly - JSON中的旧式汇编格式
        //   evm.bytecode.functionDebugData - 在函数层面的调试信息
        //   evm.bytecode.object - 字节码对象
        //   evm.bytecode.opcodes - 操作码列表
        //   evm.bytecode.sourceMap - 源码映射（对调试有用）
        //   evm.bytecode.linkReferences - 链接引用（如果是未链接的对象）
        //   evm.bytecode.generatedSources - 由编译器生成的源码
        //   evm.deployedBytecode* - 部署的字节码（拥有evm.bytecode的所有选项）。
        //   evm.deployedBytecode.immutableReferences - 从AST id到引用不可变的字节码范围的映射
        //   evm.methodIdentifiers - 函数哈希值的列表
        //   evm.gasEstimates - 函数gas估计
        //   ewasm.wast - WebAssembly S-expressions格式的Ewasm
        //   ewasm.wasm - WebAssembly二进制格式的Ewasm
        //
        // 注意，使用 `evm`， `evm.bytecode`， `ewasm` 等将选择该输出的每个目标部分。
        // 此外，`*` 可以作为通配符来请求所有东西。
=======
        //   devdoc - Developer documentation (natspec)
        //   userdoc - User documentation (natspec)
        //   metadata - Metadata
        //   ir - Yul intermediate representation of the code before optimization
        //   irAst - AST of Yul intermediate representation of the code before optimization
        //   irOptimized - Intermediate representation after optimization
        //   irOptimizedAst - AST of intermediate representation after optimization
        //   storageLayout - Slots, offsets and types of the contract's state variables.
        //   evm.assembly - New assembly format
        //   evm.legacyAssembly - Old-style assembly format in JSON
        //   evm.bytecode.functionDebugData - Debugging information at function level
        //   evm.bytecode.object - Bytecode object
        //   evm.bytecode.opcodes - Opcodes list
        //   evm.bytecode.sourceMap - Source mapping (useful for debugging)
        //   evm.bytecode.linkReferences - Link references (if unlinked object)
        //   evm.bytecode.generatedSources - Sources generated by the compiler
        //   evm.deployedBytecode* - Deployed bytecode (has all the options that evm.bytecode has)
        //   evm.deployedBytecode.immutableReferences - Map from AST ids to bytecode ranges that reference immutables
        //   evm.methodIdentifiers - The list of function hashes
        //   evm.gasEstimates - Function gas estimates
        //
        // Note that using `evm`, `evm.bytecode`, etc. will select every
        // target part of that output. Additionally, `*` can be used as a wildcard to request everything.
>>>>>>> english/develop
        //
        "outputSelection": {
          "*": {
            "*": [
              "metadata", "evm.bytecode" // 启用每个合约的元数据和字节码输出。
              , "evm.bytecode.sourceMap" // 启用每个合约的源码映射输出。
            ],
            "": [
              "ast" // 启用每个文件的AST输出。
            ]
          },
          // 启用文件def中定义的MyContract的abi和opcodes输出。
          "def": {
            "MyContract": [ "abi", "evm.bytecode.opcodes" ]
          }
        },
        // modelChecker对象是实验性的，可能会有变化。
        "modelChecker":
        {
          // 选择哪些合约应作为部署的合约进行分析。
          "contracts":
          {
            "source1.sol": ["contract1"],
            "source2.sol": ["contract2", "contract3"]
          },
          // 选择除法和模数运算的编码方式。
          // 当使用 `false` 时，它们被替换为与松弛变量的乘法。这是默认的。
          // 如果您使用CHC引擎而不使用Spacer作为Horn求解器（例如使用Eldarica），
          // 建议在这里使用 `true`。
          // 关于这个选项的更详细的解释，请参见形式验证部分。
          "divModWithSlacks": false,
          // 选择要使用的模型检查器引擎：所有（默认）， bmc， chc， 无。
          "engine": "chc",
          // 选择在编译时可获得被调用函数代码的情况下，
          // 外部调用是否应被视为可信。
          // 详情请参阅SMT检查器部分。
          "extCalls": "trusted",
          // 选择哪些类型的不变性应该报告给用户：合约，重入。
          "invariants": ["contract", "reentrancy"],
          // 选择是否输出所有验证过的目标。默认为 `false`。
          "showProved": true,
          // 选择是否输出所有未验证的目标。默认为 `false`。
          "showUnproved": true,
          // 选择是否输出所有不支持的语言功能。默认为 `false`。
          "showUnsupported": true,
          // 如果有的话，选择应该使用哪些求解器。
          // 关于求解器的描述，见形式验证部分。
          "solvers": ["cvc4", "smtlib2", "z3"],
          // 选择哪些目标应该被检查：常数条件，下溢，溢出，除以零，余额，断言，弹出空数组，界外。
          // 如果没有给出该选项，所有目标都被默认检查，除了 Solidity >=0.8.7 的下溢/溢出。
          // 目标描述见形式化验证部分。
          "targets": ["underflow", "overflow", "assert"],
          // 每个SMT查询的超时时间，以毫秒为单位。
          // 如果没有给出这个选项，SMTChecker将默认使用确定性的资源限制。
          // 给定超时为0意味着任何查询都没有资源/时间限制。
          "timeout": 20000
        }
      }
    }


输出描述
------------------

.. code-block:: javascript

    {
      // 可选：如果没有遇到错误/警告/消息，则不存在。
      "errors": [
        {
          // 可选：在源文件中的位置。
          "sourceLocation": {
            "file": "sourceFile.sol",
            "start": 0,
            "end": 100
          },
          // 可选：更多的位置（如有冲突的声明的地方）。
          "secondarySourceLocations": [
            {
              "file": "sourceFile.sol",
              "start": 64,
              "end": 92,
              "message": "Other declaration is here:"
            }
          ],
          // 强制：错误类型，如 “TypeError“， “InternalCompilerError“， “Exception” 等等。
          // 完整的类型清单见下文。
          "type": "TypeError",
<<<<<<< HEAD
          // 强制：发生错误的组件，例如“general”，“ewasm”等
=======
          // Mandatory: Component where the error originated, such as "general" etc.
>>>>>>> english/develop
          "component": "general",
          // 强制：错误的严重级别（“error”，“warning” 或 “info”，但请注意，这可能在未来被扩展。）
          "severity": "error",
          // 可选：错误原因的唯一代码
          "errorCode": "3141",
          // 强制
          "message": "Invalid keyword",
          // 可选：带错误源位置的格式化消息
          "formattedMessage": "sourceFile.sol:100: Invalid keyword"
        }
      ],
      // 这包含文件级的输出。
      // 它可以通过outputSelection设置进行限制/过滤。
      "sources": {
        "sourceFile.sol": {
          // 标识符（用于源码映射）
          "id": 1,
          // AST对象
          "ast": {}
        }
      },
      // 这里包含了合约级别的输出。
      // 它可以通过outputSelection设置进行限制/过滤。
      "contracts": {
        "sourceFile.sol": {
          // 如果使用的语言没有合约名称，则该字段应该留空。
          "ContractName": {
            // 以太坊合约的应用二进制接口（ABI）。如果为空，则表示为空数组。
            // 请参阅 https://docs.soliditylang.org/en/develop/abi-spec.html
            "abi": [],
            // 请参阅元数据输出文档（序列化的JSON字符串）
            "metadata": "{/* ... */}",
            // 用户文档（natspec）
            "userdoc": {},
            // 开发人员文档（natspec）
            "devdoc": {},
<<<<<<< HEAD
            // 中间表示形式 (string)
            "ir": "",
            // 请参阅 "存储布局" 文档。
=======
            // Intermediate representation before optimization (string)
            "ir": "",
            // AST of intermediate representation before optimization
            "irAst":  {/* ... */},
            // Intermediate representation after optimization (string)
            "irOptimized": "",
            // AST of intermediate representation after optimization
            "irOptimizedAst": {/* ... */},
            // See the Storage Layout documentation.
>>>>>>> english/develop
            "storageLayout": {"storage": [/* ... */], "types": {/* ... */} },
            // EVM相关输出
            "evm": {
              // 汇编 (string)
              "assembly": "",
              // 旧风格的汇编 (object)
              "legacyAssembly": {},
              // 字节码和相关细节
              "bytecode": {
                // 在函数层面上调试数据。
                "functionDebugData": {
                  // 接下来是一组函数，包括编译器内部的和用户定义的函数。
                  // 这组函数不一定是完整的。
                  "@mint_13": { // 函数的内部名称
                    "entryPoint": 128, // 函数开始所在字节码的字节偏移量（可选）
                    "id": 13, // 函数定义的AST ID，或者对于编译器内部的函数为空（可选）
                    "parameterSlots": 2, // 函数参数的EVM堆栈槽的数量（可选）
                    "returnSlots": 1 // 返回值的EVM堆栈槽的数量（可选）
                  }
                },
                // 作为十六进制字符串的字节码。
                "object": "00fe",
                // 操作码列表（字符串）
                "opcodes": "",
                // 作为一个字符串的源映射。参见源映射的定义。
                "sourceMap": "",
                // 由编译器生成的源文件的数组。目前只包含一个Yul文件。
                "generatedSources": [{
                  // Yul AST
                  "ast": {/* ... */},
                  // 文本形式的源文件（可能包含注释）。
                  "contents":"{ function abi_decode(start, end) -> data { data := calldataload(start) } }",
                  // 源文件ID，用于源引用，与Solidity源文件相同的 "命名空间"。
                  "id": 2,
                  "language": "Yul",
                  "name": "#utility.yul"
                }],
                // 如果给定，这就是一个非链接的对象。
                "linkReferences": {
                  "libraryFile.sol": {
                    // 在字节码中的字节偏移量。
                    // 链接取代了位于那里的20个字节。
                    "Library1": [
                      { "start": 0, "length": 20 },
                      { "start": 200, "length": 20 }
                    ]
                  }
                }
              },
              "deployedBytecode": {
                /* ..., */ // 与上述布局相同。
                "immutableReferences": {
                  // 有两个对AST ID为3的不可变的引用，都是32字节长。
                  // 一个在字节码偏移量42，另一个在字节码偏移量80。
                  "3": [{ "start": 42, "length": 32 }, { "start": 80, "length": 32 }]
                }
              },
              // 函数哈希值的列表
              "methodIdentifiers": {
                "delegate(address)": "5c19a95c"
              },
              // 函数gas估计
              "gasEstimates": {
                "creation": {
                  "codeDepositCost": "420000",
                  "executionCost": "infinite",
                  "totalCost": "infinite"
                },
                "external": {
                  "delegate(address)": "25000"
                },
                "internal": {
                  "heavyLifting()": "infinite"
                }
              }
<<<<<<< HEAD
            },
            // Ewasm相关的输出
            "ewasm": {
              // S-expressions格式
              "wast": "",
              // 二进制格式（十六进制字符串）
              "wasm": ""
=======
>>>>>>> english/develop
            }
          }
        }
      }
    }


错误类型
~~~~~~~~~~~

<<<<<<< HEAD
1. ``JSONError``： JSON输入不符合所需格式，例如，输入不是JSON对象，不支持的语言等。
2. ``IOError``： IO和导入处理错误，例如，在提供的源里包含无法解析的URL或哈希值不匹配。
3. ``ParserError``： 源代码不符合语言规则。
4. ``DocstringParsingError``： 注释块中的NatSpec标签无法解析。
5. ``SyntaxError``： 语法错误，例如 ``continue`` 在 ``for`` 循环外部使用。
6. ``DeclarationError``： 无效的，无法解析的或冲突的标识符名称 比如 ``Identifier not found``
7. ``TypeError``： 类型系统内的错误，例如无效类型转换，无效赋值等。
8. ``UnimplementedFeatureError``： 当前编译器不支持该功能，但预计将在未来的版本中支持。
9. ``InternalCompilerError``： 在编译器中触发的内部错误 — 应将此报告为一个issue。
10. ``Exception``： 编译期间的未知失败 — 应将此报告为一个issue。
11. ``CompilerError``： 编译器堆栈的无效使用 — 应将此报告为一个issue。
12. ``FatalError``： 未正确处理致命错误 — 应将此报告为一个issue。
13. ``YulException``： 在Yul代码生成过程中出现错误 - 这应该作为一个issue报告。
14. ``Warning``： 警告，不会停止编译，但应尽可能处理。
15. ``Info``： 编译器认为用户可能会在其中发现有用的信息，并不危险，也不一定需要处理。
=======
1. ``JSONError``: JSON input doesn't conform to the required format, e.g. input is not a JSON object, the language is not supported, etc.
2. ``IOError``: IO and import processing errors, such as unresolvable URL or hash mismatch in supplied sources.
3. ``ParserError``: Source code doesn't conform to the language rules.
4. ``DocstringParsingError``: The NatSpec tags in the comment block cannot be parsed.
5. ``SyntaxError``: Syntactical error, such as ``continue`` is used outside of a ``for`` loop.
6. ``DeclarationError``: Invalid, unresolvable or clashing identifier names. e.g. ``Identifier not found``
7. ``TypeError``: Error within the type system, such as invalid type conversions, invalid assignments, etc.
8. ``UnimplementedFeatureError``: Feature is not supported by the compiler, but is expected to be supported in future versions.
9. ``InternalCompilerError``: Internal bug triggered in the compiler - this should be reported as an issue.
10. ``Exception``: Unknown failure during compilation - this should be reported as an issue.
11. ``CompilerError``: Invalid use of the compiler stack - this should be reported as an issue.
12. ``FatalError``: Fatal error not processed correctly - this should be reported as an issue.
13. ``YulException``: Error during Yul code generation - this should be reported as an issue.
14. ``Warning``: A warning, which didn't stop the compilation, but should be addressed if possible.
15. ``Info``: Information that the compiler thinks the user might find useful, but is not dangerous and does not necessarily need to be addressed.
>>>>>>> english/develop
