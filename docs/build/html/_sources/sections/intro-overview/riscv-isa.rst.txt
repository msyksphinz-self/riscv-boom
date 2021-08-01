.. The RISC-V ISA
.. ==============

RISC-V命令セットアーキテクチャ
==============================

.. The `RISC-V ISA <riscv.org>`__ is a widely adopted open-source ISA suited for a variety of applications.
.. It includes a base ISA as well as multiple optional extensions that implement different features.
.. BOOM implements the RV64GC variant of the RISC-V ISA (otherwise known as IMAFDC) [1]_. This includes the
.. MAFDC extensions and the privileged specification (multiply/divide, AMOs,
.. load-reserve/store-conditional, single-precision and double-precision IEEE
.. 754-2008 floating point).

`RISC-V ISA <riscv.org>`__ は、広く採用されているオープンソースのISAで、
さまざまなアプリケーションに適しています。
BOOM は RISC-V ISA の RV64GCバリエーション (別称 IMAFDC) [1]_ を実装しています。
これには，MAFDC 拡張と特権的仕様(乗算/除算、AMO、ロード・リザーブ/ストア・コンディショナル、
単精度および倍 精度の IEEE 754-2008 浮動小数点)が含まれています．

.. RISC-V provides the following features which make it easy to target with
.. high-performance designs:

RISC-Vには、以下のような特徴があり、高性能な設計に対応しやすくなっています。

.. * **Relaxed memory model**
..    * This greatly simplifies the **Load/Store Unit (LSU)**, which does not need to
..      have loads snoop other loads nor does coherence traffic need to snoop
..      the LSU, as required by sequential consistency.


* **リラックスメモリモデル**
    * これにより、ロードが他のロードをスヌープする必要も、コヒーレンス・トラフィックがLSUをスヌープする必要もなく、 
      **Load/Store Unit (LSU)** が大幅に簡素化されます(シーケンシャル・コンシステンシーでは必要)。


.. * **Accrued Floating Point (FP) exception flags**
..    * The FP status register does not need to be renamed, nor can FP
..      instructions throw exceptions themselves.

* **正確な浮動小数点(FP) 例外フラグ**
    * FPステータスレジスタのリネームを行う必要はありませんし、
      FP命令自体が例外を発生させることもありません．

.. * **No integer side-effects**
..    * All integer ALU operations exhibit no side-effects, other than the writing
..      of the destination register. This prevents the need to rename
..      additional condition state.

* **整数命令の副作用無し**
    * すべての整数ALU演算は、書き込みレジスタの書き込み以外には、副作用がありません。
      これにより、追加のコンディション・ステートのリネームの必要がなくなります。


.. * **No cmov or predication**
..    * Although predication can lower the branch predictor complexity of
..      small designs, it greatly complicates out-of-order pipelines, including the
..      addition of a third read port for integer operations.

* **CMOVとプレディケーション命令の不採用**
    * プレディケーションは、小規模な設計であれば分岐予測器の複雑さを軽減することができますが、
      整数演算用に3つ目の読み出しポートを追加するなど、
      アウトオブオーダーパイプラインを大幅に複雑化させます。


.. * **No implicit register specifiers**
..    * Even JAL requires specifying an explicit register. This simplifies rename
..      logic, which prevents either the need to know the instruction first
..      before accessing the rename tables, or it prevents adding more ports
..      to remove the instruction decode off the critical path.

* **暗黙的なレジスタ指定を含まない**
    * JALでさえ、明示的にレジスタを指定する必要があります。これにより、リネームロジックがシンプルになり、
      リネームテーブルにアクセスする前にまず命令を知る必要がなくなったり、
      命令デコードをクリティカルパスから外すためにポートを増やすことがなくなります。

.. * **Registers rs1, rs2, rs3, rd are always in the same place**
..    * This allows decode and rename to proceed in parallel.

* **rs1, rs2, rs3, rdレジスタのフィールド位置は常に同一**
    * これによりデコードとリネームを同時に実行することができます。

.. More information about the RISC-V ISA can be found at http://riscv.org.

RISC-V ISAに関するより詳細な情報は http://riscv.org より参照できます。

.. .. [1] Currently, BOOM does not implement the proposed "V" vector extension.

.. [1] 現在、BOOMは "V"拡張を実装していません。
