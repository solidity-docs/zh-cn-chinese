Solidity
========

Solidity是一门为实现智能合约而创建的高级的，面向对象的编程语言。 智能合约是管理以太坊中账户行为的程序。

<<<<<<< HEAD
Solidity是一种 `带花括号的语言 <https://en.wikipedia.org/wiki/List_of_programming_languages_by_type#Curly-bracket_languages>`_.
这门语言受到了 C++，Python 和 Javascript 语言的影响，设计的目的是能在以太坊虚拟机（EVM）上运行。
你可以在:doc:`语言影响 <language-influences>` 这章找到更多的细节。
=======
Solidity is a `curly-bracket language <https://en.wikipedia.org/wiki/List_of_programming_languages_by_type#Curly-bracket_languages>`_ designed to target the Ethereum Virtual Machine (EVM).
It is influenced by C++, Python and JavaScript. You can find more details about which languages Solidity has been inspired by in the :doc:`language influences <language-influences>` section.
>>>>>>> 800088e38b5835ebdc71e9ba5299a70a5accd7c2

Solidity 是静态类型语言，支持继承、库和复杂的用户定义类型等特性。

下面您将会看到，使用 Solidity 语言，可以为投票、众筹、秘密竞价（盲拍）、多重签名的钱包以及其他应用创建合约。

当你开发智能合约时，你应该使用最新版本的Solidity。除某些特殊情况之外，只有最新版本会收到
`安全修复 <https://github.com/ethereum/solidity/security/policy#supported-versions>`_.
突破性的变化以及定期推出的新特性。 目前，我们使用 0.y.z 版本号 `来表明这种快速的变化 <https://semver.org/#spec-item-4>`_.

.. 警告::

  Solidity最近发布了0.8.x版本，该版本引入了许多重大更新。 点击阅读 :doc:`完整列表 <080-breaking-changes>`.

始终欢迎改进 Solidity 或此文档的想法,
点击:doc:`贡献者指南 <contributing>` 查看更多细节。

.. 提示::

  您可以通过单击左下角的版本弹出菜单并选择首选下载格式，以 PDF、HTML 或 Epub 格式下载此文档

开始入门
---------------

**1. 理解合约基础概念**

如果您不熟悉智能合约的概念，推荐从智能合约的概念开始，可以参考:

* :ref:`一个用Solidity编写的简单的智能合约实例 <simple-smart-contract>` 。
* :ref:`区块链基础 <blockchain-basics>`。
* :ref:`以太坊虚拟机 <the-ethereum-virtual-machine>`。

**2. 开始了解Solidity**

一旦熟悉了基础概念，我们推荐你学习:doc:`"Solidity实例" <solidity-by-example>`以及Solidity详解部分理解语言的核心概念。

**3. 安装Solidity编译器**

有多种方式来安装Solidity编译器,选择你喜欢的选项并按照 :ref:`安装手册 <installing-solidity>`上的步骤来安装。

.. 提示::
  你可以使用 `Remix IDE <https://remix.ethereum.org>` 在浏览器中直接运行代码。Remix 是一个基于 Web 浏览器的 IDE，
  允许您编写、部署和管理 Solidity 智能合约，而无需在本地安装 Solidity。

.. 警告::
    As humans write software, it can have bugs. You should follow established
    software development best-practices when writing your smart contracts. This
    includes code review, testing, audits, and correctness proofs. Smart contract
    users are sometimes more confident with code than their authors, and
    blockchains and smart contracts have their own unique issues to
    watch out for, so before working on production code, make sure you read the
    :ref:`security_considerations` section.

**4. Learn More**

If you want to learn more about building decentralized applications on Ethereum, the
`Ethereum Developer Resources <https://ethereum.org/en/developers/>`_
can help you with further general documentation around Ethereum, and a wide selection of tutorials,
tools and development frameworks.

If you have any questions, you can try searching for answers or asking on the
`Ethereum StackExchange <https://ethereum.stackexchange.com/>`_, or
our `Gitter channel <https://gitter.im/ethereum/solidity/>`_.

.. _translations:

Translations
------------

Community contributors help translate this documentation into several languages.
Note that they have varying degrees of completeness and up-to-dateness. The English
version stands as a reference.

You can switch between languages by clicking on the flyout menu in the bottom-left corner
and selecting the preferred language.

* `French <https://docs.soliditylang.org/fr/latest/>`_
* `Indonesian <https://github.com/solidity-docs/id-indonesian>`_
* `Persian <https://github.com/solidity-docs/fa-persian>`_
* `Japanese <https://github.com/solidity-docs/ja-japanese>`_
* `Korean <https://github.com/solidity-docs/ko-korean>`_
* `Chinese <https://github.com/solidity-docs/zh-cn-chinese/>`_

.. note::

   We recently set up a new GitHub organization and translation workflow to help streamline the
   community efforts. Please refer to the `translation guide <https://github.com/solidity-docs/translation-guide>`_
   for information on how to start a new language or contribute to the community translations.

Contents
========

:ref:`Keyword Index <genindex>`, :ref:`Search Page <search>`

.. toctree::
   :maxdepth: 2
   :caption: Basics

   introduction-to-smart-contracts.rst
   installing-solidity.rst
   solidity-by-example.rst

.. toctree::
   :maxdepth: 2
   :caption: Language Description

   layout-of-source-files.rst
   structure-of-a-contract.rst
   types.rst
   units-and-global-variables.rst
   control-structures.rst
   contracts.rst
   assembly.rst
   cheatsheet.rst
   grammar.rst

.. toctree::
   :maxdepth: 2
   :caption: Compiler

   using-the-compiler.rst
   analysing-compilation-output.rst
   ir-breaking-changes.rst

.. toctree::
   :maxdepth: 2
   :caption: Internals

   internals/layout_in_storage.rst
   internals/layout_in_memory.rst
   internals/layout_in_calldata.rst
   internals/variable_cleanup.rst
   internals/source_mappings.rst
   internals/optimizer.rst
   metadata.rst
   abi-spec.rst

.. toctree::
   :maxdepth: 2
   :caption: Additional Material

   050-breaking-changes.rst
   060-breaking-changes.rst
   070-breaking-changes.rst
   080-breaking-changes.rst
   natspec-format.rst
   security-considerations.rst
   smtchecker.rst
   resources.rst
   path-resolution.rst
   yul.rst
   style-guide.rst
   common-patterns.rst
   bugs.rst
   contributing.rst
   brand-guide.rst
   language-influences.rst
