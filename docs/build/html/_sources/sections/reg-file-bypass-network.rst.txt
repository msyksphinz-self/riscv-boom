.. The Register Files and Bypass Network
.. =====================================

レジスタファイルとバイパスネットワーク
======================================

.. _full-boom-pipeline:
.. figure:: /figures/boom-pipeline.svg
    :alt: Multi-Issue Pipeline

    複数命令発行パイプラインの例。整数レジスタファイルには、存在する実行ユニットに対して、6つのリードポートと3つのライトポートが必要です。
    FPレジスタファイルには，3つのリードポートと2つのライトポートが必要です。
    FP演算とメモリ演算は、整数レジスタファイルとFPレジスタファイルの両方に対して、long latency書き込みポートを共有します。
    書き込みポートのスケジューリングを容易にするために、ALUのパイプラインはFPUのレイテンシに合わせて長くなっています。
    ALUは、これらのステージのいずれかから、レジスタリードステージの依存する命令にバイパスすることができます。

..    An example multi-issue pipeline. The integer register file needs 6 read ports and 3 write ports for the
..    execution units present. The FP register file needs 3 read ports and 2 write ports. FP and memory
..    operations share a long latency write port to both the integer and FP
..    register file. To make scheduling of the write port trivial, the ALU’s pipeline is lengthened to match
..    the FPU latency. The ALU is able to bypass from any of these stages to dependent instructions in the
..    Register Read stage.

.. BOOM is a unified, **Physical Register File (PRF)** design. The register
.. files hold both the committed and speculative state. Additionally,
.. there are two register files: one for integer and one for floating point
.. register values. The **Rename Map Tables** track which physical register corresponds
.. to which ISA register.

BOOMは、統一された **PRF(Physical Register File)** デザインです。
レジスタファイルには、コミットされた状態と投機的な状態の両方が保持されます。
さらに、2つのレジスタファイルがあります：1つは整数レジスタ値用、もう1つは浮動小数点レジスタ値用です。
**リネームマップテーブル** は、どの物理レジスタがどのISAレジスタに対応するかを追跡します。

.. BOOM uses the Berkeley hardfloat floating point units which use an
.. internal 65-bit operand format
.. (https://github.com/ucb-bar/berkeley-hardfloat). Therefore, all physical
.. floating point registers are 65-bits.

BOOMは、内部65ビットのオペランドフォーマット(https://github.com/ucb-bar/berkeley-hardfloat) を使用するBerkeleyのHardfloat浮動小数点ユニットを使用しています。
したがって、すべての物理的な浮動小数点レジスタは65ビットです。

.. Register Read
.. -------------

レジスタ読み込み
----------------

.. The register file statically provisions all of the register read ports
.. required to satisfy all issued instructions. For example, if *issue port
.. #0* corresponds to an integer ALU and *issue port #1* corresponds to memory
.. unit, then the first two register read ports will statically serve the
.. ALU and the next two register read ports will service the memory unit for four
.. total read ports.

レジスタファイルは、発行されたすべての命令を満たすために必要なすべてのレジスタリードポートを静的に規定します。
例えば、 *発行ポート#0* が整数のALUに対応し、 *発行ポート#1* がメモリユニットに対応する場合、
最初の2つのレジスタリードポートがALUに、次の2つのレジスタリードポートがメモリユニットに、
合計4つのリードポートが静的に提供されます。

.. Dynamic Read Port Scheduling
.. ~~~~~~~~~~~~~~~~~~~~~~~~~~~~

動的な読み込みポートスケジューリング
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. Future designs can improve area-efficiency by provisioning fewer
.. register read ports and using dynamically scheduling to arbitrate for
.. them. This is particularly helpful as most instructions need only one
.. operand. However, it does add extra complexity to the design, which is
.. often manifested as extra pipeline stages to arbitrate and detect
.. structural hazards. It also requires the ability to kill issued
.. :term:`Micro-Ops (UOPs)<Micro-Op (UOP)` and re-issue them from the **Issue Queue** on a later cycle.

将来の設計では、より少ない数のレジスタリードポートを用意し、動的にスケジューリングすることで、
面積効率を向上させることができます。
これは、ほとんどの命令が1つのオペランドしか必要としない場合に特に有効です。
しかし、この方法では設計が複雑になり、アービトレーションや構造上の問題を検出するために
パイプラインのステージが増えることになります。
また、発行された :term:`Micro-Ops (UOP)<Micro-Ops (UOP)>` を kill し、
後のサイクルで **発行キュー** から再発行する機能も必要です。

.. Bypass Network
.. --------------

バイパスネットワーク
--------------------

.. ALU operations can be issued back-to-back by having the write-back
.. values forwarded through the **Bypass Network**. Bypassing occurs at the end
.. of the **Register Read** stage.

**バイパスネットワーク** を介してライトバック値を転送することで、
ALU演算を連続して実行することができます。
バイパスは **レジスタ読み込み** ステージの最後に行われます。
