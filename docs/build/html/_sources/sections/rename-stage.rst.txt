.. The Rename Stage
.. ================

リネームステージ
================

.. The **Rename** stage maps the *ISA* (or *logical*) register specifiers of
.. each instruction to *physical* register specifiers.

**リネーム** ステージでは、各命令の *ISA* (または *論理* )レジスタ指定子を
*物理* レジスタ指定子にマッピングします。

.. The Purpose of Renaming
.. -----------------------

リネームの目的
--------------

.. *Renaming* is a technique to rename the *ISA* (or *logical*) register
.. specifiers in an instruction by mapping them to a new space of
.. *physical* registers. The goal to *register renaming* is to break the
.. output-dependencies (WAW) and anti-dependences (WAR) between instructions, leaving
.. only the true dependences (RAW). Said again, but in architectural
.. terminology, register renaming eliminates write-after-write (WAW) and
.. write-after-read (WAR) hazards, which are artifacts introduced by a)
.. only having a limited number of ISA registers to use as specifiers and
.. b) loops, which by their very nature will use the same register
.. specifiers on every loop iteration.

*リネーム* は、命令の *ISA* (または *論理* )レジスタ指定子を、
新しい *物理* レジスタの空間にマッピングすることで、名前を変更する技術です。
レジスタリネームの目的は、命令間の出力依存性(WAW)と逆依存性(WAR)を断ち切り、
真の依存性(RAW)だけを残すことです。
これは、a) 指定子として使用できるISAレジスタの数が限られていること、
b)ループはその性質上、ループの反復ごとに同じレジスタ指定子を使用する、ということに対して効果があります。

.. The Explicit Renaming Design
.. ----------------------------

明示的なリネームのデザイン
--------------------------

.. _prf-vs-data-in-rob:
.. figure:: /figures/prf-and-arf.png
    :alt: PRF vs Data-in-ROB design

    物理レジスタ(左)とROB中データのデザイン(右)

..    A PRF design (left) and a data-in-ROB design (right)

.. BOOM is an "explicit renaming" or "physical register file" out-of-order
.. core design. A **Physical Register File**, containing many more registers
.. than the ISA dictates, holds both the committed architectural register
.. state and speculative register state. The **Rename Map Table** s contain the
.. information needed to recover the committed state. As instructions are
.. renamed, their register specifiers are explicitly updated to point to
.. physical registers located in the Physical Register File. [1]_

BOOMは、「明示的リネーム」または「物理レジスタファイル」によるアウトオブオーダーのコアデザインです。
**物理レジスタファイル** は、ISAが規定するよりも多くのレジスタを含み、
コミットされたアーキテクチャのレジスタ状態と投機的なレジスタ状態の両方を保持します。
**リネームマップテーブル** には、コミットされた状態を回復するために必要な情報が含まれています。
命令がリネームされると、そのレジスタ指定子は物理レジスタファイルにある物理レジスタを指すように明示的に更新されます。 [1]_

.. This is in contrast to an "implicit renaming" or "data-in-ROB"
.. out-of-order core design. The **Architectural Register File (ARF)** only
.. holds the committed register state, while the ROB holds the speculative
.. write-back data. On commit, the ROB transfers the speculative data to
.. the ARF[2]_

これは、「暗黙のリネーム」や「データインROB」と呼ばれるアウトオブオーダーのコアデザインとは対照的でです。
**ARF (アーキテクチャレジスタファイル: Architectural Register File)** はコミットされたレジスタの状態のみを保持し、
ROBは投機的なライトバックデータを保持しています。
コミット時には、ROBは投機的データを ARF[2]_ に転送します。


.. The Rename Map Table
.. --------------------

リネームマップテーブル
----------------------

.. _rename-stage:
.. figure:: /figures/rename-pipeline.png
    :alt: The Rename Stage

    **リネームステージ**。論理レジスタ指定子は**リネームマップテーブル**を読み込んで物理レジスタ指定子を取得します。
    スーパースカラのリネームでは、マップテーブルの変更は依存する命令にバイパスされなければなりません。
    その後、物理ソース指定子はビジーテーブルを読むことができます。
    Stale指定子は、命令が後にコミットしたときにどの物理レジスタが解放されるかを追跡するために使用されます。
    物理レジスタファイルのP0は常に0です。

..    The **Rename Stage**. Logical register specifiers read the **Rename Map Table** to get their physical specifier.
..    For superscalar rename, any changes to the Map Tables must be bypassed to dependent instructions. The
..    physical source specifiers can then read the Busy Table. The Stale specifier is used to track which physical
..    register will be freed when the instruction later commits. P0 in the Physical Register File is always 0.

.. The **Rename Map Table (abbreviated as Map Table)** holds the speculative mappings from ISA registers
.. to physical registers.

**リネームマップテーブル** (マップテーブルと略す)は、ISAレジスタから物理レジスタへの投機的なマッピングを保持します。

.. Each branch gets its own copy of the Rename Map Table[3]_ On a branch
.. mispredict, the Rename Map Table can be reset instantly from the mispredicting
.. branch’s copy of the Rename Map Table

各分岐命令はリネームマップテーブルのコピーを取得します[3]。
分岐の予測が外れた場合、リネームマップテーブルは予測が外れた分岐命令の
リネームマップテーブルのコピーから即座にリセットすることができます。

.. As the RV64G ISA uses fixed locations of the register specifiers (and no
.. implicit register specifiers), the Map Table can be read before the
.. instruction is decoded! And hence the **Decode** and **Rename** stages can be combined.

RV64G ISAでは、レジスタ指定子の位置が固定されているため (暗黙のレジスタ指定子はない)、
命令がデコードされる前にマップテーブルを読み取ることができます。
そのため、 **デコード** と **リネーム** のステージを組み合わせることができます。


.. Resets on Exceptions and Flushes
.. ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

例外とフラッシュによるリセット
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. An additional, optional "Committed Map Table" holds the rename map for
.. the committed architectural state. If enabled, this allows single-cycle
.. reset of the pipeline during flushes and exceptions (the current map
.. table is reset to the Committed Map Table). Otherwise, pipeline flushes
.. require multiple cycles to "unwind" the ROB to write back in the rename
.. state at the commit point, one ROB row per cycle.

オプションで追加される「コミットマップテーブル」は、コミットされたアーキテクチャ状態のリネームマップを保持します。
この機能を有効にすると、フラッシュ時や例外発生時にパイプラインを1サイクルでリセットすることができます
(現在のマップテーブルがコミットされたマップテーブルにリセットされます)。
そうでない場合、パイプラインのフラッシュには、コミットポイントでリネーム状態に書き戻すためにROBを「アンワインド」するための複数サイクルが必要となり、
1サイクルに1つのROB行が必要となります。

.. The Busy Table
.. --------------

ビジーテーブル
--------------

.. The **Busy Table** tracks the readiness status of each physical register. If
.. all physical operands are ready, the instruction will be ready to be
.. issued.

**ビジーテーブル** は、各物理レジスタの準備状態を追跡します。
すべての物理オペランドの準備が整っていれば、命令の発行が可能になります。


.. The Free List
.. -------------

フリーリスト
------------

.. The **Free List** tracks the physical registers that are currently un-used
.. and is used to allocate new physical registers to instructions passing
.. through the *Rename* stage.

**フリーリスト** は、現在使用されていない物理レジスタを追跡し、
*リネーム* ステージを通過した命令に新しい物理レジスタを割り当てるために使用されます。

.. The Free List is implemented as a bit-vector. A priority decoder can
.. then be used to find the first free register. BOOM uses a cascading
.. priority decoder to allocate multiple registers per cycle. [4]_

フリーリストはビットベクターとして実装されます。
優先度デコーダは、最初のフリーレジスタを見つけるために使用されます。
BOOMでは、カスケード式の優先度デコーダを使用して、1サイクルに複数のレジスタを割り当てています。[4]_

.. On every branch (or JALR), the Rename Map Tables are snapshotted to
.. allow single-cycle recovery on a branch misprediction. Likewise, the
.. Free List also sets aside a new "Allocation List", initialized to zero.
.. As new physical registers are allocated, the Allocation List for each
.. branch is updated to track all of the physical registers that have been
.. allocated after the branch. If a misspeculation occurs, its Allocation
.. List is added back to the Free List by *OR'ing* the branch’s Allocation
.. List with the Free List. [5]_

分岐命令(またはJALR)ごとに、リネームマップテーブルがスナップショットされ、
分岐予測の誤りを1サイクルで回復できるようになっています。
同様に、フリーリストもゼロに初期化された新しい「アロケーション・リスト」を確保します。
新しい物理レジスタが割り当てられると、各分岐の「アロケーションリスト」が更新され、
分岐後に割り当てられたすべての物理レジスタが追跡されます。
誤検知が発生した場合は、そのブランチの「アロケーションリスト」と「フリーリスト」を *論理和* することで、
「フリーリスト」に戻します。[5]_

.. Stale Destination Specifiers
.. ----------------------------

古くなったリネームレジスタ
--------------------------

.. For instructions that will write a register, the Map Table is read to
.. get the *stale physical destination specifier* ("stale pdst"). Once the
.. instruction commits, the *stale pdst* is returned to the Free List, as
.. no future instructions will read it.

レジスタを書き込む命令では、マップテーブルが読み込まれ、 *古くなった物理レジスタ指定子* ("stale pdst")が取得されます。
命令がコミットされると、以降の命令では読み取れないため、 *stale pdst* はフリーリストに戻されます。


.. [1]
   MIPS R10k、Alpha 21264、Intel Sandy Bridge、ARM Cortex A15の各コアは、明示的なリネームを採用したアウトオブオーダーのコアの例です。

..   The MIPS R10k, Alpha 21264, Intel Sandy Bridge, and ARM
..   Cortex A15 cores are all example of explicit renaming out-of-order
..   cores.

.. [2]
   Pentium 4 とARM Cortex A57は「暗黙的なリネーム」を採用したデザインの一例です。

..   The Pentium 4 and the ARM Cortex A57 are examples of *implicit
..   renaming* designs.

.. [3]
   より広いパイプラインのための別の設計では、1サイクルあたり最大1つのスナップショットのみを作成することを好むかもしれませんが、
   これには :term:`フェッチパケット` 内の任意の命令に対する正確なマッピングを推論するための追加の複雑さが伴います。

..   An alternate design for wider pipelines may prefer to only make up to
..   one snapshot per cycle, but this comes with additional complexity to
..   deduce the precise mappings for any given instruction within the
..   :term:`Fetch Packet`.

.. [4]
   2命令幅の **リネームステージ** では逆方向から始まる2つのプライオリティデコーダを使うことができます。

..   A two-wide **Rename** stage could use two priority decoders starting from
..   opposite ends.

.. [5]
   概念的には、分岐はフリーリストを「スナップショット」していると説明されることが多いです(分岐予測ミスが発生した時点での現在のフリーリストと*ORしている)。
   しかし、スナップショットでは、スナップショット発生時に割り当てられた物理レジスタが、その後解放され、分岐予測ミスが検出される前に再割り当てされた場合を考慮することができません。
   この場合、スナップショットも現在のフリーリストもその物理レジスタが解放されたことを知らないため、物理レジスタがリークしてしまいます。
   最終的には、十分な数の物理レジスタを確保するためにプロセッサが減速し、マシンが停止してしまいます。
   これが自伝的に聞こえるのは、原作者(Chris)がコンピュータ・アーキテクチャの講義を信頼していたからかもしれませんが、まあ...。

..   Conceptually, branches are often described as "snapshotting" the Free
..   List (along with an *OR'ing* with the current Free List at the time
..   of the misprediction). However, snapshotting fails to account for
..   physical registers that were allocated when the snapshot occurs, then
..   become freed, then becomes re-allocated before the branch mispredict
..   is detected. In this scenario, the physical register gets leaked, as
..   neither the snapshot nor the current Free List know that it had been
..   freed. Eventually, the processor slows as it struggles to maintain
..   enough inflight physical registers, until finally the machine comes
..   to a halt. If this sounds autobiographical because the original author
..   (Chris) may have trusted computer architecture lectures, well...
