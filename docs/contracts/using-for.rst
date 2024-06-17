.. index:: ! using for, library, ! operator;user-defined, function;free

.. _using-for:

*********
Using For
*********

指令 ``using A for B`` 可以用来将函数（ ``A``）作为操作符附加到用户定义的值类型，
或者作为任何类型（ ``B``）的成员函数。
成员函数将它们被调用的对象作为第一个参数接收（像 Python 中的 ``self`` 变量）。
操作符函数将操作数作为参数接收。

它可以在文件级别或者在合约级别的合约内部有效。

第一部分， ``A``，可以是以下之一：

- 一个函数列表，可以选择性地分配操作符名称（例如 ``using {f, g as +, h, L.t} for uint``）。
  如果没有指定操作符，函数可以是库函数或自由函数，并作为成员函数附加到类型上。
  否则，它必须是一个自由函数，并成为该类型上该操作符的定义。
- 一个库合约的名称（例如 ``using L for uint`` ）-
  该库合约的所有非私有函数都作为成员函数附加在该类型上。

在文件级别中，第二部分， ``B``，必须是一个明确的类型（没有数据位置指定）。
在合约内部，您也可以用 ``*`` 代替类型（例如 ``using L for *;`` ），
这样做的效果是，库合约 ``L`` 中所有的函数都会被附加到 *所有* 类型上。

如果您指定了一个库合约，那么该库合约中的 *所有* 非私有函数都会被附加到该类型上，
即使是那些第一个参数的类型与对象的类型不匹配的函数。
类型会在函数被调用的时候检查，
并执行函数重载解析。

如果您使用一个函数列表（例如 ``using {f, g, h, L.t} for uint`` ），
那么类型（ ``uint`` ）必须可以隐式地转换为这些函数的第一个参数。
即使这些函数都没有被调用，也要进行这种检查。
请注意，只有当 ``using for`` 位于库合约内时，才能指定私有库函数。

如果您定义了一个操作符（例如 ``using {f as +} for T``），那么类型（ ``T``）必须是一个
:ref:`用户定义的值类型 <user-defined-value-types>`，并且定义必须是一个 ``pure`` 函数。
操作符定义必须是全局的。
以下操作符可以用这种方式定义：

+------------+----------+---------------------------------------------+
| Category   | Operator | Possible signatures                         |
+============+==========+=============================================+
| Bitwise    | ``&``    | ``function (T, T) pure returns (T)``        |
|            +----------+---------------------------------------------+
|            | ``|``    | ``function (T, T) pure returns (T)``        |
|            +----------+---------------------------------------------+
|            | ``^``    | ``function (T, T) pure returns (T)``        |
|            +----------+---------------------------------------------+
|            | ``~``    | ``function (T) pure returns (T)``           |
+------------+----------+---------------------------------------------+
| Arithmetic | ``+``    | ``function (T, T) pure returns (T)``        |
|            +----------+---------------------------------------------+
|            | ``-``    | ``function (T, T) pure returns (T)``        |
|            +          +---------------------------------------------+
|            |          | ``function (T) pure returns (T)``           |
|            +----------+---------------------------------------------+
|            | ``*``    | ``function (T, T) pure returns (T)``        |
|            +----------+---------------------------------------------+
|            | ``/``    | ``function (T, T) pure returns (T)``        |
|            +----------+---------------------------------------------+
|            | ``%``    | ``function (T, T) pure returns (T)``        |
+------------+----------+---------------------------------------------+
| Comparison | ``==``   | ``function (T, T) pure returns (bool)``     |
|            +----------+---------------------------------------------+
|            | ``!=``   | ``function (T, T) pure returns (bool)``     |
|            +----------+---------------------------------------------+
|            | ``<``    | ``function (T, T) pure returns (bool)``     |
|            +----------+---------------------------------------------+
|            | ``<=``   | ``function (T, T) pure returns (bool)``     |
|            +----------+---------------------------------------------+
|            | ``>``    | ``function (T, T) pure returns (bool)``     |
|            +----------+---------------------------------------------+
|            | ``>=``   | ``function (T, T) pure returns (bool)``     |
+------------+----------+---------------------------------------------+

注意，一元和二元的 ``-`` 需要单独定义。
编译器会根据操作符的调用方式选择正确的定义。

``using A for B;`` 指令只在当前作用域（合约或当前模块/源单元）内有效，
包括其中所有的函数，在使用它的合约或模块之外没有任何效果。

当在文件级别使用该指令并应用于在同一文件中用户定义类型时，
可以在末尾添加 ``global`` 关键字。
这将使函数和操作符附加到该类型的任何可用位置（包括其他文件），
而不仅仅是在 using 语句的范围内。

下面我们将使用文件级函数来重写 :ref:`libraries` 部分中的 set 示例。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.13;

    struct Data { mapping(uint => bool) flags; }
    // 现在我们给这个类型附加上函数。
    // 附加的函数可以在模块的其他部分使用。
    // 如果您导入了该模块，
    // 您必须在那里重复using指令，例如
    //   import "flags.sol" as Flags;
    //   using {Flags.insert, Flags.remove, Flags.contains}
    //     for Flags.Data;
    using {insert, remove, contains} for Data;

    function insert(Data storage self, uint value)
        returns (bool)
    {
        if (self.flags[value])
            return false; // 已经存在
        self.flags[value] = true;
        return true;
    }

    function remove(Data storage self, uint value)
        returns (bool)
    {
        if (!self.flags[value])
            return false; // 不存在
        self.flags[value] = false;
        return true;
    }

    function contains(Data storage self, uint value)
        view
        returns (bool)
    {
        return self.flags[value];
    }


    contract C {
        Data knownValues;

        function register(uint value) public {
            // 这里， Data 类型的所有变量都有与之相对应的成员函数。
            // 下面的函数调用和 `Set.insert(knownValues, value)` 的效果完全相同。
            require(knownValues.insert(value));
        }
    }

也可以通过这种方式来扩展内置类型。
在这个例子中，我们将使用一个库合约。

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.13;

    library Search {
        function indexOf(uint[] storage self, uint value)
            public
            view
            returns (uint)
        {
            for (uint i = 0; i < self.length; i++)
                if (self[i] == value) return i;
            return type(uint).max;
        }
    }
    using Search for uint[];

    contract C {
        uint[] data;

        function append(uint value) public {
            data.push(value);
        }

        function replace(uint from, uint to) public {
            // 这将执行库合约中的函数调用
            uint index = data.indexOf(from);
            if (index == type(uint).max)
                data.push(to);
            else
                data[index] = to;
        }
    }

注意，所有的外部库调用实际都是EVM函数调用。
这意味着，如果传递内存或值类型，即使是 ``self`` 变量，也会执行复制。
只有在使用存储引用变量或调用内部库函数时，才不会执行复制。

另一个展示了如何为用户定义的类型定义一个自定义操作符的例子：

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.19;

    type UFixed16x2 is uint16;

    using {
        add as +,
        div as /
    } for UFixed16x2 global;

    uint32 constant SCALE = 100;

    function add(UFixed16x2 a, UFixed16x2 b) pure returns (UFixed16x2) {
        return UFixed16x2.wrap(UFixed16x2.unwrap(a) + UFixed16x2.unwrap(b));
    }

    function div(UFixed16x2 a, UFixed16x2 b) pure returns (UFixed16x2) {
        uint32 a32 = UFixed16x2.unwrap(a);
        uint32 b32 = UFixed16x2.unwrap(b);
        uint32 result32 = a32 * SCALE / b32;
        require(result32 <= type(uint16).max, "Divide overflow");
        return UFixed16x2.wrap(uint16(a32 * SCALE / b32));
    }

    contract Math {
        function avg(UFixed16x2 a, UFixed16x2 b) public pure returns (UFixed16x2) {
            return (a + b) / UFixed16x2.wrap(200);
        }
    }
