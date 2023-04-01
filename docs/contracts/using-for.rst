.. index:: ! using for, library, ! operator; user-defined

.. _using-for:

*********
Using For
*********

<<<<<<< HEAD
指令 ``use A for B;`` 可以用来将函数（ ``A``）作为成员函数附加到任何类型（ ``B``）。
这些函数将接收它们被调用的对象作为其第一个参数（就像Python中的 ``self`` 变量）。
=======
The directive ``using A for B`` can be used to attach
functions (``A``) as operators to user-defined value types
or as member functions to any type (``B``).
The member functions receive the object they are called on
as their first parameter (like the ``self`` variable in Python).
The operator functions receive operands as parameters.
>>>>>>> english/develop

它可以在文件级别或者在合约级别的合约内部有效。

第一部分， ``A``，可以是以下之一：

<<<<<<< HEAD
- 文件级或库函数的列表（例如 ``using {f, g, h, L.t} for uint;`` ）-
  只有这些函数将作为成员函数附加到该类型上。
  注意，只有当 ``using for`` 在库合约内时，才能指定私有库函数。
- 一个库合约的名称（例如 ``using L for uint;`` ）-
  该库合约的所有非私有函数都附加在该类型上。
=======
- A list of functions, optionally with an operator name assigned (e.g.
  ``using {f, g as +, h, L.t} for uint``).
  If no operator is specified, the function can be either a library function or a free function and
  is attached to the type as a member function.
  Otherwise it must be a free function and it becomes the definition of that operator on the type.
- The name of a library (e.g. ``using L for uint``) -
  all non-private functions of the library are attached to the type
  as member functions
>>>>>>> english/develop

在文件级别中，第二部分， ``B``，必须是一个明确的类型（没有数据位置指定）。
在合约内部，您也可以用 ``*`` 代替类型（例如 ``using L for *;`` ），
这样做的效果是，库合约 ``L`` 中所有的函数都会被附加到 *所有* 类型上。

<<<<<<< HEAD
如果您指定了一个库合约，那么该库合约中的 *所有* 函数都会被附加上，
即使是那些第一个参数的类型与对象的类型不匹配的函数。
类型会在函数被调用的时候检查，
并执行函数重载解析。

如果您使用一个函数列表（例如 ``用{f, g, h, L.t}表示uint;`` ），
那么类型（ ``uint`` ）必须可以隐式地转换为这些函数的第一个参数。
即使这些函数都没有被调用，也要进行这种检查。
=======
If you specify a library, *all* non-private functions in the library get attached,
even those where the type of the first parameter does not
match the type of the object. The type is checked at the
point the function is called and function overload
resolution is performed.

If you use a list of functions (e.g. ``using {f, g, h, L.t} for uint``),
then the type (``uint``) has to be implicitly convertible to the
first parameter of each of these functions. This check is
performed even if none of these functions are called.
Note that private library functions can only be specified when ``using for`` is inside a library.

If you define an operator (e.g. ``using {f as +} for T``), then the type (``T``) must be a
:ref:`user-defined value type <user-defined-value-types>` and the definition must be a ``pure`` function.
Operator definitions must be global.
The following operators can be defined this way:

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

Note that unary and binary ``-`` need separate definitions.
The compiler will choose the right definition based on how the operator is invoked.
>>>>>>> english/develop

``using A for B;`` 指令只在当前作用域（合约或当前模块/源单元）内有效，
包括其中所有的函数，在使用它的合约或模块之外没有任何效果。

<<<<<<< HEAD
当在文件级别使用该指令并应用于在同一文件中用户定义类型时，
可以在末尾添加 ``global`` 关键字。
这将产生的效果是，函数将附加到类型的每个地方（包括其他文件），
而不仅仅是在 using 语句的作用域内。
=======
When the directive is used at file level and applied to a
user-defined type which was defined at file level in the same file,
the word ``global`` can be added at the end. This will have the
effect that the functions and operators are attached to the type everywhere
the type is available (including other files), not only in the
scope of the using statement.
>>>>>>> english/develop

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

<<<<<<< HEAD
注意，所有的外部库调用实际都是EVM函数调用。
这意味着，如果您传递内存或值类型，将进行拷贝，即使是在 ``self`` 变量的情况下。
唯一不进行拷贝的情况是当使用存储引用变量或调用内部库函数时。
=======
Note that all external library calls are actual EVM function calls. This means that
if you pass memory or value types, a copy will be performed, even in case of the
``self`` variable. The only situation where no copy will be performed
is when storage reference variables are used or when internal library
functions are called.

Another example shows how to define a custom operator for a user-defined type:

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
>>>>>>> english/develop
