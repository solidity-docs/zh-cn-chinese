.. _metadata:

#################
合约的元数据
#################

.. index:: metadata, contract verification

Solidity 编译器自动生成一个 JSON 文件。
该文件包含两种有关已编译合约的信息：

- 如何与合约交互：ABI 和 NatSpec 文档。
- 如何重现编译并验证已部署的合约：
  编译器版本、编译器设置和使用的源文件。

默认情况下，编译器会将元数据文件的 IPFS 哈希值附加到
每个合约的运行时字节码（不一定是创建字节码）末尾，
这样，如果发布了该文件，就可以通过验证的方式检索该文件，而无需求助于集中式数据提供者。
其他可用选项包括 Swarm 哈希值和不在字节码中附加元数据哈希值。
这些选项可通过 :ref:`标准 JSON 接口 <compiler-api>` 进行配置。

您必须将元数据文件发布到IPFS，Swarm或其他服务，
以便其他人可以访问它。您可以通过使用 ``solc --metadata`` 命令
和 ``--output-dir`` 参数来创建该文件。如果没有这个参数，
元数据将被写到标准输出。
元数据包含 IPFS 和 Swarm 对源代码的引用，
所以除了元数据文件外，您还必须上传所有的源文件。
对于IPFS， ``ipfs add`` 返回的 CID 中包含的哈希值（不是文件的直接sha2-256哈希值）
应与字节码中包含的哈希值相匹配。

元数据文件的格式如下。下面的示例是以人类可读的方式呈现的。
正确格式化的元数据应正确使用引号，
尽量减少空白，并按字母顺序对所有对象的键值进行排序，以形成规范格式。
不允许使用注释，此处注释仅用于解释目的。

.. code-block:: javascript

    {
      // 必选：编译器的详情，内容视语言而定。
      "compiler": {
        // 可选：生成此输出的编译器二进制文件的哈希值
        "keccak256": "0x123...",
        // 对 Solidity 来说是必选的：编译器的版本
        "version": "0.8.2+commit.661d1103"
      },
      // 必选：源代码的编程语言，一般会选择规范的“子版本”
      "language": "Solidity",
      // 必选：合约的生成信息
      "output": {
        // 必选：合约的 ABI 定义，见 “合约 ABI 规范”
        "abi": [/* ... */],
        // 必选：合约的开发者 NatSpec 文档，详见 https://docs.soliditylang.org/en/latest/natspec-format.html
        "devdoc": {
          // 合约 @author NatSpec字段的内容
          "author": "John Doe",
          // 合约中 @dev NatSpec 字段的内容
          "details": "Interface of the ERC20 standard as defined in the EIP. See https://eips.ethereum.org/EIPS/eip-20 for details",
          "errors": {
            "MintToZeroAddress()" : {
              "details": "Cannot mint to zero address"
            }
          },
          "events": {
            "Transfer(address,address,uint256)": {
              "details": "Emitted when `value` tokens are moved from one account (`from`) toanother (`to`).",
              "params": {
                "from": "The sender address",
                "to": "The receiver address",
                "value": "The token amount"
              }
            }
          },
          "kind": "dev",
          "methods": {
            "transfer(address,uint256)": {
              // 方法的 @dev NatSpec 字段的内容
              "details": "Returns a boolean value indicating whether the operation succeeded. Must be called by the token holder address",
              // 方法的 @param NatSpec 字段的内容
              "params": {
                "_value": "The amount tokens to be transferred",
                "_to": "The receiver address"
              },
              // @return NatSpec 字段的内容。
              "returns": {
                // 如果存在，返回var名称（这里是 “success”）。如果返回的var是未命名的，“_0” 作为键。
                "success": "a boolean value indicating whether the operation succeeded"
              }
            }
          },
          "stateVariables": {
            "owner": {
              // 状态变量的 @dev NatSpec 字段的内容
              "details": "Must be set during contract creation. Can then only be changed by the owner"
            }
          },
          // 合约中 @title NatSpec 字段的内容
          "title": "MyERC20: an example ERC20",
          "version": 1 // NatSpec 版本
        },
        // 必选：合约的用户 NatSpec 文档。请参阅“NatSpec 格式”
        "userdoc": {
          "errors": {
            "ApprovalCallerNotOwnerNorApproved()": [
              {
                "notice": "The caller must own the token or be an approved operator."
              }
            ]
          },
          "events": {
            "Transfer(address,address,uint256)": {
              "notice": "`_value` tokens have been moved from `from` to `to`"
            }
          },
          "kind": "user",
          "methods": {
            "transfer(address,uint256)": {
              "notice": "Transfers `_value` tokens to address `_to`"
            }
          },
          "version": 1 // NatSpec 版本
        }
      },
      // 必选： 编译器设置。反映编译时 JSON 输入的设置。
      // 查看标准 JSON 输入的 “setting” 字段文档
      "settings": {
        // 对 Solidity 来说是必选的： 文件路径以及为其创建的合约或库的名称。
        "compilationTarget": {
          "myDirectory/myFile.sol": "MyContract"
        },
        // 对 Solidity 来说是必选的。
        "evmVersion": "london",
        // 对 Solidity 来说是必选的： 使用的库合约地址。
        "libraries": {
          "MyLib": "0x123123..."
        },
        "metadata": {
          // 反映输入 json 中使用的设置，默认为“true”
          "appendCBOR": true,
          // 反映输入 json 中使用的设置，默认为“ipfs”
          "bytecodeHash": "ipfs",
          // 反映输入 json 中使用的设置，默认为“false”
          "useLiteralContent": true
        },
<<<<<<< HEAD
        // 可选：优化设置。“enabled” 和 “runs” 字段已弃用，仅用于向后兼容。
=======
        // Optional: Optimizer settings. The fields "enabled" and "runs" are deprecated
        // and are only given for backward-compatibility.
>>>>>>> english/develop
        "optimizer": {
          "details": {
            "constantOptimizer": false,
            "cse": false,
            "deduplicate": false,
<<<<<<< HEAD
            // inliner 默认为“true”
            "inliner": true,
            // jumpdestRemover 默认为“true”
=======
            // inliner defaults to "false"
            "inliner": false,
            // jumpdestRemover defaults to "true"
>>>>>>> english/develop
            "jumpdestRemover": true,
            "orderLiterals": false,
            // peephole 默认为“true”
            "peephole": true,
            "yul": true,
            // 可选：仅当 “yul” 为 “true” 时才出现
            "yulDetails": {
              "optimizerSteps": "dhfoDgvulfnTUtnIf...",
              "stackAllocation": false
            }
          },
          "enabled": true,
          "runs": 500
        },
        // 对 Solidity 来说是必选的：导入重新映射的排序列表。
        "remappings": [ ":g=/dir" ]
      },
      // 必选：编译源文件/源单元，键为文件路径
      "sources": {
        "destructible": {
          // 必选（除非使用了 “url”）：源文件的字面内容
          "content": "contract destructible is owned { function destroy() { if (msg.sender == owner) selfdestruct(owner); } }",
          // 必选：源文件的 keccak256 哈希值
          "keccak256": "0x234..."
        },
        "myDirectory/myFile.sol": {
          // 必选：源文件的 keccak256 哈希值
          "keccak256": "0x123...",
          // 可选：源文件中给出的 SPDX 许可证标识符
          "license": "MIT",
          // 必选（除非使用了 “content”，见上文）：指向源文件的排序 URL，
          // 协议可任意选择，但建议使用 IPFS URL
          "urls": [ "bzz-raw://7d7a...", "dweb:/ipfs/QmN..." ]
        }
      },
      // 必选：元数据格式的版本
      "version": 1
    }

.. warning::
  由于产生的合约的字节码默认包含元数据哈希值，
  对元数据的任何改变都可能导致字节码的改变。
  这包括对文件名或路径的改变，而且由于元数据包括所有使用的源的哈希值，
  一个空白的改变就会导致不同的元数据和不同的字节码。

.. note::
    上面的ABI定义没有固定的顺序。它可以随着编译器的版本而改变。
    不过，从Solidity 0.5.12版本开始，该数组保持一定的顺序。

.. _encoding-of-the-metadata-hash-in-the-bytecode:

在字节码中对元数据哈希值进行编码
=============================================

编译器目前默认将规范元数据文件的 
`IPFS 哈希（in CID v0） <https://docs.ipfs.tech/concepts/content-addressing/#version-0-v0>`_ 
和编译器版本附加到字节码末尾。
也可选择使用 Swarm 哈希值代替 IPFS，或使用实验标志。
以下是所有可能的字段：

.. code-block:: javascript

    {
      "ipfs": "<metadata hash>",
      // 如果编译器设置中的“bytecodeHash”为“bzzr1”，此处不是“ipfs”而是“bzzr1”
      "bzzr1": "<metadata hash>",
      // 以前的版本使用“bzzr0”而不是“bzzr1”
      "bzzr0": "<metadata hash>",
      // 如果使用任何影响代码生成的实验性功能
      "experimental": true,
      "solc": "<compiler version>"
    }

因为我们将来可能会支持以其他方式检索元数据文件，
因此这些信息被存储为 `CBOR <https://tools.ietf.org/html/rfc7049>`__ - 编码。
字节码中的最后两个字节表示 CBOR 编码信息的长度。通过查看这个长度，
可以用 CBOR 解码器对字节码的相关部分进行解码。

请访问 `Metadata Playground <https://playground.sourcify.dev/>`_ 查看实际操作。

<<<<<<< HEAD
SOLC的发布版本使用如上所示的3个字节的版本编码
（主要、次要和补丁版本号各一个字节），
而预发布版本将使用一个完整的版本字符串，包括提交哈希和构建日期。
=======
Whereas release builds of solc use a 3 byte encoding of the version as shown
above (one byte each for major, minor and patch version number), pre-release builds
will instead use a complete version string including commit hash and build date.
>>>>>>> english/develop

命令行标志 ``--no-cbor-metadata`` 可以用来跳过元数据在部署的字节码末端的附加。
同样地，标准JSON输入中的布尔字段 ``settings.metadata.appendCBOR`` 可以设置为false。

.. note::
  CBOR 映射也可能包含其他键，
  因此最好通过查看字节码末尾的 CBOR 长度来完全解码数据，
  并使用适当的 CBOR 分析器。不要依赖以 ``0xa264``
  或 ``0xa2 0x64 'i' 'p' 'f' 's'`` 开头的数据。

自动化接口生成和NatSpec 的使用方法
====================================================

元数据的使用方式如下：一个想要与合约交互的组件
（例如钱包）会检索合约的代码。
它对包含元数据文件的 IPFS/Swarm 哈希的 CBOR 编码部分进行解码。
通过该哈希值，元数据文件被检索出来。该文件被 JSON 解码成一个类似于上述的结构。

然后，该组件可以使用ABI为合约自动生成一个基本的用户界面。

此外，钱包还可以使用 NatSpec 用户文档，在用户与合约进行交互时，
向用户显示一条可读的确认信息，同时请求交易签名进行授权。

有关其他信息，请阅读 :doc:`以太坊自然语言规范（NatSpec）格式 <natspec-format>`。

源代码验证的用法
==================================

如果已固定/发布，则可以从 IPFS/Swarm 获取合约的元数据。
元数据文件还包含源文件的 URL 或 IPFS 哈希值，以及编译设置，
即重现编译所需的一切信息。

有了这些信息，就可以通过重现编译来验证合约的源代码，
并将编译的字节码与已部署合约的字节码进行比较。

由于元数据和源代码的哈希值都是字节码的一部分，因此可以自动验证元数据和源代码。
文件或设置的任何更改都会导致不同的元数据哈希值。
这里的元数据是整个编译过程的指纹。

`Sourcify <https://sourcify.dev>`_ 利用这一特性进行 “完全/完美验证”，
并将文件公开固定在 IPFS 上，以便使用元数据哈希值进行访问。
