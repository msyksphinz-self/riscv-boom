The Berkeley Out-of-Order Machine (BOOM)
========================================

.. figure:: /figures/boom-pipeline-detailed.png
    :scale: 25 %
    :align: center
    :alt: Detailed BOOM Pipeline

    Detailed BOOM Pipeline. \*'s denote where the core can be configured.

.. The Berkeley Out-of-Order Machine (BOOM) is heavily inspired by the MIPS R10000 [1]_ and the Alpha 21264 [2]_ out–of–order processors.
.. Like the MIPS R10000 and the Alpha 21264, BOOM is a unified physical register file design (also known as “explicit register renaming").


Berkeley Out-of-Order Machine (BOOM)は，MIPS R10000 [1]_ やAlpha 21264 [2]_ のアウトオブオーダー・プロセッサに大きく影響を受けています。
MIPS R10000やAlpha 21264と同様に、BOOMも統一された物理レジスタファイル設計です(「明示的レジスタリネーミング」としても知られています)。


.. BOOM implements the open-source `RISC-V ISA <riscv.org>`__ and utilizes the `Chisel <chisel-lang>`__ hardware construction language to construct generator for the core.
.. A generator can be thought of a generialized RTL design.
.. A standard RTL design can be viewed as a single instance of a generator design.
.. Thus, BOOM is a family of out-of-order designs rather than a single instance of a core.
.. Additionally, to build an SoC with a BOOM core, BOOM utilizes the `Rocket Chip <https://github.com/chipsalliance/rocket-chip>`__ SoC generator as a library to reuse different micro-architecture structures (TLBs, PTWs, etc).

BOOMは、オープンソースの `RISC-V ISA <riscv.org>`__ を実装し、
`Chisel <chisel-lang>`__ というハードウェア構築言語を利用して、コアのジェネレータを構築します。
ジェネレータは、RTL設計の一般化と考えることができます。
標準的なRTLデザインは、ジェネレーターデザインの1つのインスタンスとみなすことができます。
したがって、BOOMは、コアの単一のインスタンスではなく、アウトオブオーダーデザインのファミリーです。
さらに、BOOMコアでSoCを構築するために、BOOMは、異なるマイクロアーキテクチャ構造(TLB、PTWなど)を再利用するためのライブラリとして、
`Rocket Chip <https://github.com/chipsalliance/rocket-chip>`__ SoCジェネレータを利用しています。

.. [1] Yeager, Kenneth C. "The MIPS R10000 superscalar microprocessor." IEEE micro 16.2 (1996): 28-41.

.. [2] Kessler, Richard E. "The alpha 21264 microprocessor." IEEE micro 19.2 (1999): 24-36.
