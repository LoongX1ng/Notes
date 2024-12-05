# Dalvik bytecode format
https://source.android.google.cn/docs/core/runtime/dalvik-bytecode

## General design
1 The machine model and calling conventions are meant to approximately imitate common real architectures and C-style calling conventions:

    机器模型和调用规范旨在大致模仿常见的真实架构和 C 风格的调用规范：

    1.1 The machine is register-based, and frames are fixed in size upon creation. Each frame consists of a particular number of registers (specified by the method) as well as any adjunct data needed to execute the method, such as (but not limited to) the program counter and a reference to the` .dex `file that contains the method.

        机器基于寄存器，而帧的大小在创建时确定后就固定不变。每一帧由特定数量的寄存器（由相应method指定）以及执行该method所需的所有辅助数据构成，例如（但不限于）程序计数器和对包含该method的`.dex `文件的引用。

    1.2 When used for bit values (such as integers and floating point numbers), registers are considered 32 bits wide. Adjacent register pairs are used for 64-bit values. There is no alignment requirement for register pairs.

        当用于比特值（例如整数和浮点数）时，寄存器会被视为 32 比特宽。如果值是 64 比特，则使用两个相邻的寄存器。对于寄存器对儿，没有对齐要求。

    1.3 When used for object references, registers are considered wide enough to hold exactly one such reference.

        当用于对象引用时，寄存器会被视为其宽度正好能够容纳一个此类引用。

    1.4 In terms of bitwise representation,`(Object) null == (int) 0`.

        对于按比特的表示，`(Object) null == (int) 0`。

    1.5 The N arguments to a method land in the last N registers of the method's invocation frame, in order. Wide arguments consume two registers. Instance methods are passed a` this `reference as their first argument.

    如果一个method有 N 个参数，则在该方法的调用帧的最后 N 个寄存器中按顺序传递这些参数。宽的参数占用两个寄存器。向实例method传入一个` this `引用作为其第一个参数。

2 The storage unit in the instruction stream is a 16-bit unsigned quantity. Some bits in some instructions are ignored / must-be-zero.

  指令流中的存储单元是 16 位无符号数。某些指令中的某些比特会被忽略/必须为 0。

3 Instructions aren't gratuitously limited to a particular type. For example, instructions that move 32-bit register values without interpretation don't have to specify whether they are moving ints or floats.

    指令并非一定限于特定类型。例如，在不做任何interpretation的情况下move 32 位寄存器值的指令不一定非得指定移动对象的类型是整数还是浮点数。

4 There are separately enumerated and indexed constant pools for references to strings, types, fields, and methods.

    至于对字符串、类型、字段和方法的引用，有已分别枚举且已编好索引的常量池。

5 Bitwise literal data is represented in-line in the instruction stream.

    按比特的字面数据在指令流中内联表示。

6 Because, in practice, it is uncommon for a method to need more than 16 registers, and because needing more than eight registers is reasonably common, many instructions are limited to only addressing the first 16 registers. When reasonably possible, instructions allow references to up to the first 256 registers. In addition, some instructions have variants that allow for much larger register counts, including a pair of catch-all move instructions that can address registers in the range` v0 `–` v65535 `. In cases where an instruction variant isn't available to address a desired register, it is expected that the register contents get moved from the original register to a low register (before the operation) and/or moved from a low result register to a high register (after the operation).

   在实践中，一个method需要 16 个以上的寄存器不太常见，而需要 8 个以上的寄存器却相当普遍，因此很多指令仅限于寻址前 16 个寄存器。在合理的可能情况下，指令允许引用最多前 256 个寄存器。此外，某些指令还具有允许更多寄存器的变体，包括可寻址method` v0 `-` v65535 `范围内的寄存器的一对儿 catch-all move 指令。如果指令变体不能用于寻址所需的寄存器，寄存器内容会（在运算前）从原始寄存器移动到低位寄存器和/或（在运算后）从低位结果寄存器移动到高位寄存器。

7 There are several "pseudo-instructions" that are used to hold variable-length data payloads, which are referred to by regular instructions (for example, `fill-array-data`). Such instructions must never be encountered during the normal flow of execution. In addition, the instructions must be located on even-numbered bytecode offsets (that is, 4-byte aligned). In order to meet this requirement, dex generation tools must emit an extra nop instruction as a spacer if such an instruction would otherwise be unaligned. Finally, though not required, it is expected that most tools will choose to emit these instructions at the ends of methods, since otherwise it would likely be the case that additional instructions would be needed to branch around them.

    有几个“伪指令”可用于容纳被常规指令（例如，`fill-array-data`）引用的可变长度数据负载。在正常执行流程中绝对不会遇到这类指令。此外，这类指令必须位于偶数字节码偏移（即以 4 字节对齐）上。为了满足这一要求，如果这类指令未对齐，则 dex 生成工具必须填充额外的 nop 指令。最后，虽然并非必须这样做，但是大多数工具会选择在method的末尾填充这些额外的指令，否则可能需要额外的指令才能围绕这些method进行分支。

8 When installed on a running system, some instructions may be altered, changing their format, as an install-time static linking optimization. This is to allow for faster execution once linkage is known. See the associated [instruction formats document](https://source.android.google.cn/docs/core/runtime/instruction-formats?hl=zh-cn) for the suggested variants. The word "suggested" is used advisedly; it is not mandatory to implement these.

    如果安装到正在运行的系统中，某些指令可能会被改变，因为在其安装过程中执行的静态链接优化可能会更改它们的格式。这样做可以在链接已知之后加快执行的速度。有关被建议的变体，请参阅相关的[指令格式文档](https://source.android.google.cn/docs/core/runtime/instruction-formats?hl=zh-cn)。特意使用“建议”一词是因为并非必须强制实现这些变体。

9 Human-syntax and mnemonics:

    符合人类语言习惯的语法与助记符：

    9.1 Dest-then-source ordering for arguments.

        对参数进行 Dest-then-source 排序。

    9.2 Some opcodes have a disambiguating name suffix to indicate the type(s) they operate on:

        一些opcode具有消除歧义的名称后缀，这些后缀表示它们运算的对象的类型：

        9.2.1 Type-general 32-bit opcodes are unmarked.

            常规类型的 32 比特未标记opcode。

        9.2.2 Type-general 64-bit opcodes are suffixed with `-wide`.

            常规类型的 64 比特opcode以` -wide `为后缀。

        9.2.3 Type-specific opcodes are suffixed with their type (or a straightforward abbreviation), one of: `-boolean` `-byte` `-char` `-short` `-int` `-long` `-float` `-double` `-object` `-string` `-class` `-void`.

            特定类型的opcode以其类型（或简单缩写）为后缀，这些类型包括：`-boolean` `-byte` `-char` `-short` `-int` `-long` `-float` `-double` `-object` `-string` `-class` `-void`。

    9.3 Some opcodes have a disambiguating suffix to distinguish otherwise-identical operations that have different instruction layouts or options. These suffixes are separated from the main names with a slash ("`/`") and mainly exist at all to make there be a one-to-one mapping with static constants in the code that generates and interprets executables (that is, to reduce ambiguity for humans).

        一些opcode具有消除歧义的后缀，这些后缀用于区分除指令layout或option不同之外其他完全相同的运算。这些后缀与主要名称之间以斜杠（“`/`”）分开，主要目的是使生成和解析可执行文件的代码中存在与静态常量的一对一映射关系（即，让人读的更清楚明白）。

    9.4 In the descriptions here, the width of a value (indicating, e.g., the range of a constant or the number of registers possibly addressed) is emphasized by the use of a character per four bits of width.

        在本文档的说明部分，我们使用 4 比特宽的字符来强调一个值的宽度（例如，指示常量的范围或可能寻址的寄存器的数量）。

    9.5 For example, in the instruction "`move-wide/from16 vAA, vBBBB`":

        例如，在指令“`move-wide/from16 vAA, vBBBB`”中：

        9.5.1 "`move`" is the base opcode, indicating the base operation (move a register's value).

            “`move`”为基础opcode，表示基础运算（移动寄存器的值）。

        9.5.2 "`wide`" is the name suffix, indicating that it operates on wide (64 bit) data.

            “`wide`”为名称后缀，表示指令对宽（64 比特）数据的进行运算。

        9.5.3 "`from16`" is the opcode suffix, indicating a variant that has a 16-bit register reference as a source.

            “`from16`”为opcode后缀，表示具有 16 位寄存器引用作为源的变体。

        9.5.4 "`vAA`" is the destination register (implied by the operation; again, the rule is that destination arguments always come first), which must be in the range `v0` – `v255`.

            “`vAA`”为目标寄存器（隐含在运算中；并且，规定目标参数始终在前），取值范围为 `v0` - `v255`。

        9.5.5 "`vBBBB`" is the source register, which must be in the range `v0` – `v65535`.

            “`vBBBB`”是源寄存器，取值范围为 `v0` - `v65535`。

10 See the [instruction formats document](https://source.android.google.cn/docs/core/runtime/instruction-formats) for more details about the various instruction formats (listed under "Op & Format") as well as details about the opcode syntax.

    请参阅[指令格式文档](https://source.android.google.cn/docs/core/runtime/instruction-formats)，详细了解各种指令格式（在“Op & Format”下列出）以及opcode语法。

11 See the [`.dex` file format document](https://source.android.google.cn/docs/core/runtime/dex-format) for more details about where the bytecode fits into the bigger picture.

    请参阅 [`.dex` 文件格式文档](https://source.android.google.cn/docs/core/runtime/dex-format)，详细了解字节码如何融入整个编码环境。
