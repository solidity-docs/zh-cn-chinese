.. index: variable cleanup

*********************
清理变量
*********************

<<<<<<< HEAD
如果一个数值不足 256 位，那么在某些情况下，剩余的位必须被清理。
Solidity 编译器被设计为在执行任何操作之前清除这些剩余位中可能会造成不利影响的潜在垃圾。
例如，在将一个值写入内存之前，剩余的位需要被清除，因为内存的内容可以被用来计算哈希值或作为消息调用的数据发送。
同样地，在将一个值保存到存储中之前，剩余的位需要被清理，否则就会看到被混淆的数值。
=======
Ultimately, all values in the EVM are stored in 256 bit words.
Thus, in some cases, when the type of a value has less than 256 bits,
it is necessary to clean the remaining bits.
The Solidity compiler is designed to do such cleaning before any operations
that might be adversely affected by the potential garbage in the remaining bits.
For example, before writing a value to  memory, the remaining bits need
to be cleared because the memory contents can be used for computing
hashes or sent as the data of a message call.  Similarly, before
storing a value in the storage, the remaining bits need to be cleaned
because otherwise the garbled value can be observed.
>>>>>>> 4100a59ccaf6b921c5c8edbf66537d22d6e3e974

注意，通过内联汇编的访问不被认为是这种操作。
如果您使用内联汇编来访问短于256位的Solidity变量，编译器不保证该值被正确清理。

此外，如果接下来的操作不受影响，我们就不清理这些位。
例如，由于任何非零值都被 ``JUMPI`` 指令认为是 ``true``，
所以在布尔值被用作 ``JUMPI`` 的条件之前，我们不对它们进行清理。

除了上面的设计原则外，Solidity编译器在输入数据被加载到堆栈时也会对其进行清理。

<<<<<<< HEAD
不同的类型有不同的规则来清理无效的值：

+----------------+----------------+--------------------------------+
|      类型      |    有效的值    |         无效的值会导致         |
+================+================+================================+
| n 个成员的枚举 | 0 到 n - 1     | 异常报错（exception）          |
+----------------+----------------+--------------------------------+
| 布尔           | 0 或 1         | 1                              |
+----------------+----------------+--------------------------------+
| 有符号整数     | 以符号开头的字 | 目前会直接打包；未来会抛出异常 |
+----------------+----------------+--------------------------------+
| 无符号整数     | 高位补 0       | 目前会直接打包；未来会抛出异常 |
+----------------+----------------+--------------------------------+
=======
The following table describes the cleaning rules applied to different types,
where ``higher bits`` refers to the remaining bits in case the type has less than 256 bits.

+---------------+---------------+-------------------------+
|Type           |Valid Values   |Cleanup of Invalid Values|
+===============+===============+=========================+
|enum of n      |0 until n - 1  |throws exception         |
|members        |               |                         |
+---------------+---------------+-------------------------+
|bool           |0 or 1         |results in 1             |
+---------------+---------------+-------------------------+
|signed integers|higher bits    |currently silently       |
|               |set to the     |signextends to a valid   |
|               |sign bit       |value, i.e. all higher   |
|               |               |bits are set to the sign |
|               |               |bit; may throw an        |
|               |               |exception in the future  |
+---------------+---------------+-------------------------+
|unsigned       |higher bits    |currently silently masks |
|integers       |zeroed         |to a valid value, i.e.   |
|               |               |all higher bits are set  |
|               |               |to zero; may throw an    |
|               |               |exception in the future  |
+---------------+---------------+-------------------------+

Note that valid and invalid values are dependent on their type size.
Consider ``uint8``, the unsigned 8-bit type, which has the following valid values:

.. code-block:: none

    0000...0000 0000 0000
    0000...0000 0000 0001
    0000...0000 0000 0010
    ....
    0000...0000 1111 1111

Any invalid value will have the higher bits set to zero:

.. code-block:: none

    0101...1101 0010 1010   invalid value
    0000...0000 0010 1010   cleaned value

For ``int8``, the signed 8-bit type, the valid values are:

Negative

.. code-block:: none

    1111...1111 1111 1111
    1111...1111 1111 1110
    ....
    1111...1111 1000 0000

Positive

.. code-block:: none

    0000...0000 0000 0000
    0000...0000 0000 0001
    0000...0000 0000 0010
    ....
    0000...0000 1111 1111

The compiler will ``signextend`` the sign bit, which is 1 for negative and 0 for
positive values, overwriting the higher bits:

Negative

.. code-block:: none

    0010...1010 1111 1111   invalid value
    1111...1111 1111 1111   cleaned value

Positive

.. code-block:: none

    1101...0101 0000 0100   invalid value
    0000...0000 0000 0100   cleaned value
>>>>>>> 4100a59ccaf6b921c5c8edbf66537d22d6e3e974
