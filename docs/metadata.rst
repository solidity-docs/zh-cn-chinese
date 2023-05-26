.. _metadata:

#################
合约的元数据
#################

.. index:: metadata, contract verification

<<<<<<< HEAD
Solidity 编译器自动生成一个 JSON 文件，即合约元数据，
其中包含有关已编译合约的信息。您可以使用此文件来查询编译器版本，使用的源码，ABI 和 NatSpec 文档，
以便更安全地与合约交互并验证其源代码。

编译器默认将元数据文件的IPFS哈希附加到每个合约的字节码末尾（详见下文），
这样您就可以通过认证的方式来检索文件，而不必求助于中心化数据提供者。
其他可用的选项是Swarm哈希值和不将元数据哈希值附加到字节码上。
这些可以通过 :ref:`标准 JSON 接口 <compiler-api>` 来配置。
=======
The Solidity compiler automatically generates a JSON file.
The file contains two kinds of information about the compiled contract:

- How to interact with the contract: ABI, and NatSpec documentation.
- How to reproduce the compilation and verify a deployed contract:
  compiler version, compiler settings, and source files used.

The compiler appends by default the IPFS hash of the metadata file to the end
of the runtime bytecode (not necessarily the creation bytecode) of each contract,
so that, if published, you can retrieve the file in an authenticated way without
having to resort to a centralized data provider. The other available options are
the Swarm hash and not appending the metadata hash to the bytecode. These can be
configured via the :ref:`Standard JSON Interface<compiler-api>`.
>>>>>>> english/develop

您必须将元数据文件发布到IPFS，Swarm或其他服务，
以便其他人可以访问它。您可以通过使用 ``solc --metadata`` 命令
和 ``--output-dir`` 参数来创建该文件。如果没有这个参数，
元数据将被写到标准输出。
元数据包含 IPFS 和 Swarm 对源代码的引用，
所以除了元数据文件外，您还必须上传所有的源文件。
对于IPFS， ``ipfs add`` 返回的 CID 中包含的哈希值（不是文件的直接sha2-256哈希值）
应与字节码中包含的哈希值相匹配。

<<<<<<< HEAD
元数据文件有以下格式。下面的例子是以人类可读的方式呈现的。
正确的元数据格式应该正确地使用引号，
将空白减少到最低限度，并对所有对象的键进行排序，得出唯一的格式。
注释是不被允许的，在此仅用于解释目的。
=======
The metadata file has the following format. The example below is presented in a
human-readable way. Properly formatted metadata should use quotes correctly,
reduce whitespace to a minimum, and sort the keys of all objects in alphabetical order
to arrive at a canonical formatting. Comments are not permitted and are used here only for
explanatory purposes.
>>>>>>> english/develop

.. code-block:: javascript

    {
<<<<<<< HEAD
      // 必选：元数据格式的版本
      "version": "1",
      // 必选：源代码的编程语言，一般会选择规范的“子版本”
      "language": "Solidity",
      // 必选：编译器的详情，内容视语言而定。
      "compiler": {
        // 对 Solidity 来说是必选的：编译器的版本
        "version": "0.8.2+commit.661d1103",
        // 可选： 生成此输出的编译器二进制文件的哈希值
        "keccak256": "0x123..."
      },
      // 必选：编译的源文件／源单元，键值为文件路径
      "sources":
      {
        "myDirectory/myFile.sol": {
          // 必选：源文件的 keccak256 哈希值
          "keccak256": "0x123...",
          // 必选：（除非使用 “content”，见下文）：已排序的源文件的URL，
          // 协议可以是任意的，
          // 但建议使用 IPFS URL
          "urls": [ "bzz-raw://7d7a...", "dweb:/ipfs/QmN..." ],
          // 可选：源文件中给出的 SPDX 许可证标识符。
          "license": "MIT"
        },
        "destructible": {
          // 必选：源文件的 keccak256 哈希值
          "keccak256": "0x234...",
          // 必选（除非定义了“urls”）： 源文件的字面内容
          "content": "contract destructible is owned { function destroy() { if (msg.sender == owner) selfdestruct(owner); } }"
        }
      },
      // 必选：编译器的设置
      "settings":
      {
        // 对 Solidity 来说是必选的：已排序的导入重映射列表
        "remappings": [ ":g=/dir" ],
        // 可选：优化器设置。“enabled” 和 "runs" 这两个字段已被废弃，
        // 这里只是为了向后兼容而给出。
        "optimizer": {
          "enabled": true,
          "runs": 500,
          "details": {
            // 默认值为 “true“
            "peephole": true,
            // 内联器默认值为 “true“
            "inliner": true,
            // 跳转目的地移除器默认为 “true“
            "jumpdestRemover": true,
            "orderLiterals": false,
            "deduplicate": false,
            "cse": false,
            "constantOptimizer": false,
            "yul": true,
            // 可选：只在 “yul“ 为 “true“ 时出现
            "yulDetails": {
              "stackAllocation": false,
              "optimizerSteps": "dhfoDgvulfnTUtnIf..."
            }
          }
        },
        "metadata": {
          // 显示输入 json 中使用的设置，默认为 “true”
          "appendCBOR": true,
          // 显示输入 json 中使用的设置，默认为 “false”
          "useLiteralContent": true,
          // 显示输入json中使用的设置，默认为 “ipfs“
          "bytecodeHash": "ipfs"
        },
        // 对 Solidity 来说是必选的：用以生成该元数据的文件路径和合约名或库名
        "compilationTarget": {
          "myDirectory/myFile.sol": "MyContract"
        },
        // 对 Solidity 来说是必须的：所使用的库合约的地址
        "libraries": {
          "MyLib": "0x123123..."
        }
      },
      // 必选：合约的生成信息
      "output":
      {
        // 必选：合约的 ABI 定义，见 “合约 ABI 规范”
        "abi": [/* ... */],
        // 必选：合约的开发者 NatSpec 文档
        "devdoc": {
          "version": 1 // NatSpec 版本
          "kind": "dev",
          // 合约中 @author NatSpec 字段的内容
          "author": "John Doe",
          // 合约中 @title NatSpec 字段的内容
          "title": "MyERC20: an example ERC20"
          // 合约中 @dev NatSpec 字段的内容
=======
      // Required: Details about the compiler, contents are specific
      // to the language.
      "compiler": {
        // Optional: Hash of the compiler binary which produced this output
        "keccak256": "0x123...",
        // Required for Solidity: Version of the compiler
        "version": "0.8.2+commit.661d1103"
      },
      // Required: Source code language, basically selects a "sub-version"
      // of the specification
      "language": "Solidity",
      // Required: Generated information about the contract.
      "output": {
        // Required: ABI definition of the contract. See "Contract ABI Specification"
        "abi": [/* ... */],
        // Required: NatSpec developer documentation of the contract. See https://docs.soliditylang.org/en/latest/natspec-format.html for details.
        "devdoc": {
          // Contents of the @author NatSpec field of the contract
          "author": "John Doe",
          // Contents of the @dev NatSpec field of the contract
>>>>>>> english/develop
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
<<<<<<< HEAD
              }
              // 方法的 @return NatSpec 字段的内容
=======
              },
              // Contents of the @return NatSpec field.
>>>>>>> english/develop
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
<<<<<<< HEAD
          }
          "events": {
             "Transfer(address,address,uint256)": {
               "details": "Emitted when `value` tokens are moved from one account (`from`) toanother (`to`)."
               "params": {
                 "from": "The sender address"
                 "to": "The receiver address"
                 "value": "The token amount"
               }
             }
          }
        },
        // 必选：合约的用户 NatSpec 文档
        "userdoc": {
          "version": 1 // NatSpec 版本
=======
          },
          // Contents of the @title NatSpec field of the contract
          "title": "MyERC20: an example ERC20",
          "version": 1 // NatSpec version
        },
        // Required: NatSpec user documentation of the contract. See "NatSpec Format"
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
>>>>>>> english/develop
          "kind": "user",
          "methods": {
            "transfer(address,uint256)": {
              "notice": "Transfers `_value` tokens to address `_to`"
            }
          },
          "version": 1 // NatSpec version
        }
      },
      // Required: Compiler settings. Reflects the settings in the JSON input during compilation.
      // Check the documentation of standard JSON input's "settings" field
      "settings": {
        // Required for Solidity: File path and the name of the contract or library this
        // metadata is created for.
        "compilationTarget": {
          "myDirectory/myFile.sol": "MyContract"
        },
        // Required for Solidity.
        "evmVersion": "london",
        // Required for Solidity: Addresses for libraries used.
        "libraries": {
          "MyLib": "0x123123..."
        },
        "metadata": {
          // Reflects the setting used in the input json, defaults to "true"
          "appendCBOR": true,
          // Reflects the setting used in the input json, defaults to "ipfs"
          "bytecodeHash": "ipfs",
          // Reflects the setting used in the input json, defaults to "false"
          "useLiteralContent": true
        },
        // Optional: Optimizer settings. The fields "enabled" and "runs" are deprecated
        // and are only given for backwards-compatibility.
        "optimizer": {
          "details": {
            "constantOptimizer": false,
            "cse": false,
            "deduplicate": false,
            // inliner defaults to "true"
            "inliner": true,
            // jumpdestRemover defaults to "true"
            "jumpdestRemover": true,
            "orderLiterals": false,
            // peephole defaults to "true"
            "peephole": true,
            "yul": true,
            // Optional: Only present if "yul" is "true"
            "yulDetails": {
              "optimizerSteps": "dhfoDgvulfnTUtnIf...",
              "stackAllocation": false
            }
          },
          "enabled": true,
          "runs": 500
        },
        // Required for Solidity: Sorted list of import remappings.
        "remappings": [ ":g=/dir" ]
      },
      // Required: Compilation source files/source units, keys are file paths
      "sources": {
        "destructible": {
          // Required (unless "url" is used): literal contents of the source file
          "content": "contract destructible is owned { function destroy() { if (msg.sender == owner) selfdestruct(owner); } }",
          // Required: keccak256 hash of the source file
          "keccak256": "0x234..."
        },
        "myDirectory/myFile.sol": {
          // Required: keccak256 hash of the source file
          "keccak256": "0x123...",
          // Optional: SPDX license identifier as given in the source file
          "license": "MIT",
          // Required (unless "content" is used, see above): Sorted URL(s)
          // to the source file, protocol is more or less arbitrary, but an
          // IPFS URL is recommended
          "urls": [ "bzz-raw://7d7a...", "dweb:/ipfs/QmN..." ]
        }
      },
      // Required: The version of the metadata format
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

<<<<<<< HEAD
因为我们将来可能会支持其他方式来检索元数据文件，
所以映射 ``{"ipfs": <IPFS 哈希值>, "solc": <编译器版本>}`` 将以
`CBOR <https://tools.ietf.org/html/rfc7049>`_-编码来存储。
由于映射可能包含更多的键（见下文），而且该编码的开头不容易找到，
所以添加两个字节来表述其长度，以大端方式编码。
当前版本的 Solidity 编译器通常在部署的字节码的末尾添加以下内容
=======
The compiler currently by default appends the
`IPFS hash (in CID v0) <https://docs.ipfs.tech/concepts/content-addressing/#version-0-v0>`_
of the canonical metadata file and the compiler version to the end of the bytecode.
Optionally, a Swarm hash instead of the IPFS, or an experimental flag is used.
Below are all the possible fields:
>>>>>>> english/develop

.. code-block:: javascript

<<<<<<< HEAD
    0xa2
    0x64 'i' 'p' 'f' 's' 0x58 0x22 <34字节的IPFS哈希值>
    0x64 's' 'o' 'l' 'c' 0x43 <3字节的版本编码>
    0x00 0x33

因此，为了检索数据，可以检查已部署字节码的末尾以匹配该模式，
并且可以使用 IPFS 哈希值来检索文件（如果固定/发布）。
=======
    {
      "ipfs": "<metadata hash>",
      // If "bytecodeHash" was "bzzr1" in compiler settings not "ipfs" but "bzzr1"
      "bzzr1": "<metadata hash>",
      // Previous versions were using "bzzr0" instead of "bzzr1"
      "bzzr0": "<metadata hash>",
      // If any experimental features that affect code generation are used
      "experimental": true,
      "solc": "<compiler version>"
    }

Because we might support other ways to retrieve the
metadata file in the future, this information is stored
`CBOR <https://tools.ietf.org/html/rfc7049>`_-encoded. The last two bytes in the bytecode
indicate the length of the CBOR encoded information. By looking at this length, the
relevant part of the bytecode can be decoded with a CBOR decoder.

Check the `Metadata Playground <https://playground.sourcify.dev/>`_ to see it in action.
>>>>>>> english/develop

SOLC的发布版本使用如上所示的3个字节的版本编码
（主要、次要和补丁版本号各一个字节），
而预发布版本将使用一个完整的版本字符串，包括提交哈希和构建日期。

命令行标志 ``--no-cbor-metadata`` 可以用来跳过元数据在部署的字节码末端的附加。
同样地，标准JSON输入中的布尔字段 ``settings.metadata.appendCBOR`` 可以设置为false。

.. note::
<<<<<<< HEAD
  CBOR映射也可以包含其他的键，所以最好是完全解码，
  而不是依靠它以 ``0xa264`` 开始。
  例如，如果使用了任何影响代码生成的实验性功能，
  映射也将包含 ``"experimental": true``。

.. note::
  编译器目前默认使用元数据的IPFS哈希值，
  但将来也可能使用bzzr1哈希值或其他哈希值，
  所以不要依赖这个序列以 ``0xa2 0x64 'i' 'p' 'f' 's'`` 开始。
  我们还可能向这个CBOR结构添加额外的数据，
  所以最好的选择是使用一个合适的CBOR解析器。

=======
  The CBOR mapping can also contain other keys, so it is better to fully
  decode the data by looking at the end of the bytecode for the CBOR length,
  and to use a proper CBOR parser. Do not rely on it starting with ``0xa264``
  or ``0xa2 0x64 'i' 'p' 'f' 's'``.
>>>>>>> english/develop

自动化接口生成和NatSpec 的使用方法
====================================================

元数据的使用方式如下：一个想要与合约交互的组件
（例如钱包）会检索合约的代码。
它对包含元数据文件的 IPFS/Swarm 哈希的 CBOR 编码部分进行解码。
通过该哈希值，元数据文件被检索出来。该文件被 JSON 解码成一个类似于上述的结构。

然后，该组件可以使用ABI为合约自动生成一个基本的用户界面。

<<<<<<< HEAD
此外，钱包可以使用 NatSpec 用户文档，每当用户与合约交互时，
就会向用户显示一条可读的确认信息，同时要求对交易签名进行授权。
=======
Furthermore, the wallet can use the NatSpec user documentation to display a
human-readable confirmation message to the user whenever they interact with
the contract, together with requesting authorization for the transaction signature.
>>>>>>> english/develop

有关其他信息，请阅读 :doc:`以太坊自然语言规范（NatSpec）格式 <natspec-format>`。

源代码验证的用法
==================================

<<<<<<< HEAD
为了验证编译，可以通过元数据文件中的链接从IPFS/Swarm检索源码。
正确版本的编译器（应该为“官方”编译器之一）以指定的设置在该输入上被调用。
产生的字节码与创建交易的数据或 ``CREATE`` 操作码数据进行比较。
这将自动验证元数据，因为其哈希值是字节码的一部分。
多余的数据对应于构造器的输入数据，应该根据接口进行解码并呈现给用户。

在资源库 `sourcify <https://github.com/ethereum/sourcify>`_
(`npm package <https://www.npmjs.com/package/source-verify>`_)，
您可以看到如何使用这一功能的示例代码。
=======
If pinned/published, it is possible to retrieve the metadata of the contract from IPFS/Swarm.
The metadata file also contains the URLs or the IPFS hashes of the source files, as well as
the compilation settings, i.e. everything needed to reproduce a compilation.

With this information it is then possible to verify the source code of a contract by
reproducing the compilation, and comparing the bytecode from the compilation with
the bytecode of the deployed contract.

This automatically verifies the metadata since its hash is part of the bytecode, as well
as the source codes, because their hashes are part of the metadata. Any change in the files
or settings would result in a different metadata hash. The metadata here serves
as a fingerprint of the whole compilation.

`Sourcify <https://sourcify.dev>`_ makes use of this feature for "full/perfect verification",
as well as pinning the files publicly on IPFS to be accessed with the metadata hash.
>>>>>>> english/develop
