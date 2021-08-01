.. The BOOM Pipeline
.. =================

BOOMパイプライン
================

.. _boom-pipeline:
.. figure:: /figures/boom-pipeline.svg
    :alt: Simplified BOOM Pipeline with Stages

..    Simplified BOOM Pipeline with Stages

    ステージ毎に単純化したBOOMパイプライン

.. Overview
.. --------

概要
----

.. Conceptually, BOOM is broken up into 10 stages: **Fetch**, **Decode**,
.. **Register Rename**, **Dispatch**, **Issue**, **Register Read**, **Execute**, **Memory**,
.. **Writeback** and **Commit**. However, many of those stages are
.. combined in the current implementation, yielding **seven** stages:
.. **Fetch**, **Decode/Rename**, **Rename/Dispatch**, **Issue/RegisterRead**, **Execute**,
.. **Memory** and **Writeback** (**Commit** occurs asynchronously, so it is not counted as part of the “pipeline").
.. :numref:`boom-pipeline` shows a simplified BOOM pipeline that has all of the pipeline stages listed.

概念的に、BOOMは10のステージに分かれています。 **フェッチ**、 **デコード**、
**レジスタリネーム**、 **ディスパッチ**、 **命令発行**、 **レジスタ読み込み**、 **実行**、 **メモリアクセス**。
**ライトバック**、 **コミット** の順になります。しかし、これらのステージの多くは、現在の実装では
現在の実装では、これらのステージの多くが組み合わされ、 **7** ステージとなっています。
**フェッチ**、 **デコード・リネーム**、 **リネーム・ディスパッチ**、 **命令発行・レジスタ読み込み**、 **実行**。
**メモリ** と **ライトバック** ( **コミット** は非同期に行われるため、「パイプライン」の一部としてはカウントされません)。
:numref:`boom-pipeline` は、すべてのパイプラインステージが記載されている簡略化された BOOM パイプラインを示しています。

.. Stages
.. ------

ステージ
--------

.. Fetch
.. ^^^^^

フェッチ
^^^^^^^^

.. Instructions are *fetched* from instruction memory and
.. pushed into a FIFO queue, known as the :term:`Fetch Buffer` . Branch
.. prediction also occurs in this stage, redirecting the fetched
.. instructions as necessary. [1]_

命令は、命令メモリから*フェッチ*され、 :term:`フェッチバッファ` と呼ばれるFIFOキューに押し込まれます。
分岐予測もこの段階で行われ、フェッチされた命令を必要に応じてリダイレクトします。[1]_


.. Decode
.. ^^^^^^

デコード
^^^^^^^^

.. **Decode** pulls instructions out of the :term:`Fetch Buffer` and
.. generates the appropriate :term:`Micro-Op(s) (UOPs)<Micro-Op (UOP)>` to place into the
.. pipeline. [2]_

**デコード** は :term:`Fetch Buffer` から命令を取り出し、
適切な :term:`Micro-Op(s) (UOPs)<Micro-Op (UOP)>` を生成してパイプラインに配置します。[2]_


.. Rename
.. ^^^^^^

リネーム
^^^^^^^^

.. The ISA, or "logical", register specifiers (e.g. x0-x31) are
.. then *renamed* into "physical" register specifiers.

ISA、つまり「論理的」なレジスタ指定(例：x0-x31)は、「物理的」なレジスタ指定子に*リネーム*されます。

.. Dispatch
.. ^^^^^^^^

ディスパッチ
^^^^^^^^^^^^

.. The :term:`UOP<Micro-Op (UOP)>` is then *dispatched*, or written, into
.. a set of :term:`Issue Queue` s.

その後、 :term:`UOP<Micro-Op (UOP)>` は、 :term:`命令キュー` のセットに *ディスパッチ* されるか、または書き込まれます。

.. Issue
.. ^^^^^

命令発行
^^^^^^^^

.. :term:`UOPs<Micro-Op (UOP)>` sitting in a :term:`Issue Queue` wait until all of
.. their operands are ready and are then *issued*. [3]_ This is
.. the beginning of the out–of–order piece of the pipeline.

:term:`Issue Queue` に置かれた `UOPs<Micro-Op (UOP)>` は、
そのオペランドの準備がすべてととのうまで待機し、その後 *発行* されます。[3]_ 
これがパイプラインのアウトオブオーダー部分の始まりです。

.. Register Read
.. ^^^^^^^^^^^^^

レジスタ読み込み
^^^^^^^^^^^^^^^^

.. Issued :term:`UOPs<Micro-Op (UOP)>` s first *read* their register operands from the unified
.. **Physical Register File** (or from the **Bypass Network**)...

発酵された :term:`UOPs<Micro-Op (UOP)>` sは、まずレジスタ・オペランドを統一された
**物理レジスタファイル** (または **バイパスネットワーク** )から *読み込み* ます。

.. Execute
.. ^^^^^^^

実行
^^^^

.. ... and then enter the **Execute** stage where the functional
.. units reside. Issued memory operations perform their address
.. calculations in the **Execute** stage, and then store the
.. calculated addresses in the **Load/Store Unit** which resides in the
.. **Memory** stage.

発酵された命令は、機能ユニットが存在する **実行** ステージに入ります。
発行されたメモリ・オペレーションは、**実行** ステージでアドレス計算を行い、
**メモリ** ステージにある **ロードストアユニット** に計算されたアドレスを格納します。


.. Memory
.. ^^^^^^

メモリ
^^^^^^

.. The **Load/Store Unit** consists of three queues: a **Load Address Queue
.. (LAQ)**, a **Store Address Queue (SAQ)**, and a **Store Data Queue (SDQ)**.
.. Loads are fired to memory when their address is present in the
.. **LAQ**. Stores are fired to memory at **Commit** time (and
.. naturally, stores cannot be *committed* until both their
.. address and data have been placed in the **SAQ** and **SDQ**).

**ロード/ストアユニット** は、 **ロードアドレスキュー(LAQ)** 、
**ストアアドレスキュー(SAQ)** 、 **ストアデータキュー(SDQ)** の3つのキューで構成されています。
ロードは，そのアドレスが **LAQ** に存在するときにメモリに送られます。
ストアは、**コミット** 時にメモリに起動されます
(当然ながら、ストアのアドレスとデータの両方が **SAQ** と **SDQ** に配置されるまで、ストアを*コミット*することはできません)。


.. Writeback
.. ^^^^^^^^^

ライトバック
^^^^^^^^^^^^

.. ALU operations and load operations are *written* back to the
.. **Physical Register File**.

ALUオペレーションとロード操作は **物理レジスタファイル** に **ライトバック** されます。

.. Commit
.. ^^^^^^

コミット
^^^^^^^^

.. The **Reorder Buffer (ROB)**, tracks the status of each instruction
.. in the pipeline. When the head of the **ROB** is not-busy, the **ROB**
.. *commits* the instruction. For stores, the **ROB** signals to the
.. store at the head of the **Store Queue (SAQ/SDQ)** that it can now write its
.. data to memory.

**リオーダバッファ(ROB)** は、パイプライン内の各命令の状態を追跡します。
ROBの先頭がビジーでない場合、ROBはその命令をコミットします。
ストアの場合、 **ROB** は **ストアキュー(SAQ/SDQ)** の先頭にあるストアに、
データをメモリに書き込めるようになったことを知らせます。


.. Branch Support
.. --------------

分岐のサポート
--------------

.. BOOM supports full branch speculation and branch prediction. Each
.. instruction, no matter where it is in the pipeline, is accompanied by a
.. **Branch Tag** that marks which branches the instruction is "speculated
.. under". A mispredicted branch requires killing all instructions that
.. depended on that branch. When a branch instructions passes through
.. **Rename**, copies of the **Register Rename Table** and the **Free
.. List** are made. On a mispredict, the saved processor state is
.. restored.

BOOMは完全な投機実行と分岐予測をサポートしています。
各命令は、それがパイプラインのどの位置にあるかに関わらず、 **分岐タグ** を伴っており、
その命令がどの分岐に対して"投機実行されている"のかを示しています。
分岐予測を誤ると、その分岐に依存していたすべての命令を殺す必要があります。
分岐命令が **リネーム** を通過する際には、**レジスタリネームテーブル** と **フリーリスト** のコピーが作成されます。
分岐予測に失敗した際には、保存されていたプロセッサの状態が復元されます。


.. Detailed BOOM Pipeline
.. ----------------------

詳細なBOOMのパイプライン
------------------------

.. Although :numref:`boom-pipeline` shows a simplified BOOM pipeline, BOOM supports RV64GC and the privileged ISA
.. which includes single-precision and double-precision floating point, atomics support, and page-based virtual memory.
.. A more detailed diagram is shown below in :numref:`boom-pipeline-detailed`.


:numref:`boom-pipeline` は 簡略化されたBOOM パイプラインを示していますが、
BOOM は RV64GC と特権 ISA をサポートしており、単精度および倍精度の浮動小数点、アトミックのサポート、
ページベースの仮想メモリが含まれています。
より詳細な図は、以下の :numref:`boom-pipeline-detailed` にあります。

.. _boom-pipeline-detailed:
.. figure:: /figures/boom-pipeline-detailed.png
    :alt: Detailed BOOM Pipeline

    Detailed BOOM Pipeline. \*'s denote where the core can be configured.

.. .. [1] While the :term:`Fetch Buffer` is ``N``-entries deep, it can instantly read
..     out the first instruction on the front of the FIFO. Put another way,
..     instructions don’t need to spend ``N`` cycles moving their way through
..     the :term:`Fetch Buffer` if there are no instructions in front of
..     them.

.. [1] :term:`フェッチバッファ` は ``N`` エントリの深さがありますが、FIFO の先頭命令を即座に読み出すことができます。
       別の言い方をすれば、命令の前に命令がなければ、
       命令が :term:`フェッチバッファ` を通過するのに ``N`` サイクルを費やす必要はありません。


.. .. [2] Because RISC-V is a RISC ISA, currently all instructions generate
..     only a single :term:`Micro-Op (UOP)` . More details on how store :term:`UOPs<Micro-Op (UOP)>` are
..     handled can be found in :ref:`The Memory System and the Data-cache Shim`.

.. [2] RISC-VはRISC ISAであるため、現在、すべての命令は単一の :term:`Micro-Op (UOP)` のみを生成します。
       ストアの :term:`UOPs<Micro-Op (UOP)>` がどのように処理されるかについての詳細は、 
       :ref:`The Memory System and the Data-cache Shim` にあります。


.. .. [3] More precisely, :term:`Micro-Ops (UOPs)<Micro-Op (UOP)>` that are ready assert their request, and the
..     issue scheduler within the Issue Queue chooses which :term:`UOPs<Micro-Op (UOP)>` to issue that cycle.

.. [3] より正確には、準備が整った :term:`Micro-Ops (UOP)<Micro-Ops (UOP)>` が要求をアサートし、命令キュー内の命令スケジューラが
       そのサイクルで発行する :term:`UOP<Micro-Ops (UOP)>` を選択します。
