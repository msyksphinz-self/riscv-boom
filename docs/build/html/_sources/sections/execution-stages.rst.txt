.. The Execute Pipeline
.. ====================

実行パイプライン
================

.. _dual-issue-pipeline:
.. figure:: /figures/execution-pipeline-2w.png
    :alt: Dual Issue Pipeline

    2命令発行のBOOMのパイプライン例。最初の発行ポートは、ALU演算、FPU演算、整数乗算命令を受け付けることができるExecute Unit #0に、
    :term:`UOP<Micro-Op (UOP)`をスケジューリングします。
    2番目の発行ポートは、ALU演算、整数除算命令(非パイプライン)、ロード/ストア演算をスケジュールします。
    ALU演算は、依存する命令にバイパスすることができます。なお、実行ユニット#0のALUには、FPUやiMulユニットとのレイテンシーを合わせるためにパイプラインレジスタが追加されており、
    書き込みポートのスケジューリングが容易になっています。
    各 :term:`Execution Unit` は、それ専用の1つの命令発行ポートを持っていますが、
    その中にはいくつかの下位レベルの :term:`機能ユニット` が含まれています。

..    An example pipeline for a dual-issue BOOM. The first issue port schedules :term:`UOP<Micro-Op (UOP)`s onto
..    Execute Unit #0, which can accept ALU operations, FPU operations, and integer multiply instructions.
..    The second issue port schedules ALU operations, integer divide instructions (unpipelined), and load/store
..    operations. The ALU operations can bypass to dependent instructions. Note that the ALU in Execution Unit #0 is
..    padded with pipeline registers to match latencies with the FPU and iMul units to make scheduling for the
..    write-port trivial. Each :term:`Execution Unit` has a single issue-port dedicated to it but contains within it a number
..    of lower-level :term:`Functional Unit`s.

.. The **Execution Pipeline** covers the execution and write-back of :term:`Micro-Ops (UOPs)<Micro-Op (UOP)>`.
.. Although the :term:`UOPs<Micro-Op (UOP)` will travel down the pipeline one after the other
.. (in the order they have been issued), the :term:`UOPs<Micro-Op (UOP)` themselves are
.. likely to have been issued to the Execution Pipeline out-of-order.
.. :numref:`dual-issue-pipeline` shows an example Execution Pipeline for a
.. dual-issue BOOM.

**実行パイプライン** は、 :term:`Micro-Ops (UOPs)<Micro-Ops (UOP)>` の実行とライトバックをカバーしています。
:term:`UOPs<Micro-Op (UOP)>` はパイプラインを次々と通過しますが、 :term:`UOPs<Micro-Op (UOP)>` 自体は実行パイプラインに順番通りではない形で発行されている可能性があります。 
:numref:`dual-issue-pipeline` は2命令発行の BOOM の実行パイプラインの例を示しています。

.. Execution Units
.. ---------------

実行ユニット
------------

.. _example-fu:
.. figure:: /figures/execution-unit.png
    :alt: Example :term:`Execution Unit`

    例 :term:`実行ユニット`。この例では、整数のALU(依存する命令に結果をバイパスできる)と、
    動作中にビジー状態になるパイプライン化されていない分周器を示しています。
    両方の :term:`機能ユニット` は、1つの書き込みポートを共有しています。
    実行ユニットは、キル信号と分岐解決信号の両方を受け取り、必要に応じて内部の機能ユニットに渡します。

..    An example :term:`Execution Unit`. This particular example shows an integer ALU (that can bypass
..    results to dependent instructions) and an unpipelined divider that becomes busy during operation. Both
..    :term:`Functional Unit`s share a single write-port. The :term:`Execution Unit` accepts both kill signals and branch resolution
..    signals and passes them to the internal :term:`Functional Unit` s as required.


.. An :term:`Execution Unit` is a module that a single issue port will schedule
.. :term:`UOPs<Micro-Op (UOP)` onto and contains some mix of :term:`Functional Unit` s. Phrased in
.. another way, each issue port from the **Issue Queue** talks to one and only
.. one :term:`Execution Unit`. An :term:`Execution Unit` may contain just a single simple
.. integer ALU, or it could contain a full complement of floating point
.. units, a integer ALU, and an integer multiply unit.

:term:`実行ユニット` とは、1つの命令発行ポートが :term:`UOPs<Micro-Op (UOP)` をスケジュールするモジュールで、 
:term:`機能ユニット` のいくつかの組み合わせを含みます。
別の言い方をすると、 **命令発行キュー** からの各命令発行ポートは、1つだけの :term:`実行ユニット` と通信します。
1つの :term:`Execution Unit` は1つの単純な整数ALUだけを含むかもしれませんし、
完全な浮動小数点ユニット、整数ALU、整数乗算ユニットを含むかもしれません。


.. The purpose of the :term:`Execution Unit` is to provide a flexible abstraction
.. which gives a lot of control over what kind of :term:`Execution Unit` s the
.. architect can add to their pipeline

:term:`実行ユニット` の目的は、アーキテクトがパイプラインにどのような種類の :term:`実行ユニット` を追加できるかについて、
多くのコントロールを与える柔軟な抽象化を提供することです。


.. Scheduling Readiness
.. ~~~~~~~~~~~~~~~~~~~~

スケジューリングの準備
~~~~~~~~~~~~~~~~~~~~~~

.. An :term:`Execution Unit` provides a bit-vector of the :term:`Functional Unit` s it has
.. available to the issue scheduler. The issue scheduler will only schedule
.. :term:`UOPs<Micro-Op (UOP)` that the :term:`Execution Unit` supports. For :term:`Functional Unit` s that
.. may not always be ready (e.g., an un-pipelined divider), the appropriate
.. bit in the bit-vector will be disabled (See :numref:`dual-issue-pipeline`).

:term:`実行ユニット` は、利用可能な :term:`機能ユニット` のビットベクターを命令発行スケジューラに提供します。
命令発行スケジューラは、その :term:`実行ユニット` がサポートする :term:`UOPs<Micro-Op (UOP)`のみをスケジュールします。
常に準備ができていない可能性のある :term:`Functional Unit`  (例えば、パイプライン化されていない除算器)については、
ビットベクタの適切なビットが無効になります(参照 :numref:`dual-issue-pipeline`)。


.. Functional Unit
.. ----------------

機能ユニット
------------

.. _abstract-fu:
.. figure:: /figures/abstract-functional-unit.png
    :alt: Abstract :term:`Functional Unit`

    抽象的なパイプライン化された :term:`機能ユニット` クラスです。
    専門家によって書かれた低レベルの :term:`機能ユニット` の中でインスタンス化されます。
    :term:`UOPs<Micro-Op (UOP)`は、低レベルの :term:`機能ユニット` を出るときに、
    そのレスポンスをゲートオフすることで個別に殺されます。

..    The abstract Pipelined :term:`Functional Unit` class. An expert-written, low-level :term:`Functional Unit`
..    is instantiated within the :term:`Functional Unit`. The request and response ports are abstracted and bypass and
..    branch speculation support is provided. :term:`UOPs<Micro-Op (UOP)` are individually killed by gating off their response as they
..    exit the low-level :term:`Functional Unit` .

.. :term:`Functional Unit` s are the muscle of the CPU, computing the necessary
.. operations as required by the instructions. :term:`Functional Unit` s typically
.. require a knowledgable domain expert to implement them correctly and
.. efficiently.

:term:`機能ユニット` はCPUの筋肉であり、命令に応じて必要な演算を行います。 
:term:`機能ユニット` を正しく効率的に実装するには、知識豊富なドメインエキスパートが必要です。

.. For this reason, BOOM uses an abstract :term:`Functional Unit` class to "wrap"
.. expert-written, low-level :term:`Functional Unit` s from the Rocket repository
.. (see :ref:`Rocket Chip SoC Generator`). However, the expert-written :term:`Functional Unit` s
.. created for the Rocket in-order processor make assumptions about
.. in-order issue and commit points (namely, that once an instruction has
.. been dispatched to them it will never need to be killed). These
.. assumptions break down for BOOM.

このような理由から、BOOMは抽象的な :term:`機能ユニット` クラスを使用して、Rocket リポジトリから
エキスパートが書いた低レベルの :term:`機能ユニット` を「ラップ」しています( :ref:`Rocket Chip SoC Generator` 参照)。
しかし、Rocketのインオーダープロセッサ用に作成されたエキスパートが書いた :term:`機能ユニット` は、
インオーダーの発行ポイントとコミットポイントについて仮定しています(つまり、一旦命令がそれらにディスパッチされたら、決してキルする必要はないということです)。
この仮定はBOOMでは崩れます。

.. However, instead of re-writing or forking the :term:`Functional Unit` s, BOOM
.. provides an abstract :term:`Functional Unit` class (see :numref:`abstract-fu`)
.. that “wraps" the lower-level functional
.. units with the parameterized auto-generated support code needed to make
.. them work within BOOM. The request and response ports are abstracted,
.. allowing :term:`Functional Unit` s to provide a unified, interchangeable
.. interface.

しかし、BOOM は :term:`機能ユニット` を書き直したりフォークしたりするのではなく、抽象的な :term:`機能ユニット` クラス( :numref:`abstract-fu` 参照)を提供しています。
このクラスは、低レベルの機能ユニットを BOOM 内で動作させるために必要なパラメータ化された自動生成サポートコードで「ラップ」します。
リクエストポートとレスポンスポートは抽象化されているので、 :term:`機能ユニット` クラスは統一された交換可能なインターフェースを提供することができます。

.. Pipelined Functional Units
.. ~~~~~~~~~~~~~~~~~~~~~~~~~~

パイプライン化された機能ユニット
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. A pipelined :term:`Functional Unit` can accept a new :term:`UOP<Micro-Op (UOP)` every cycle. Each
.. :term:`UOP<Micro-Op (UOP)` will take a known, fixed latency.

パイプライン化された :term:`機能ユニット` は、1サイクルごとに新しい :term:`UOP<Micro-Op (UOP)` を受け入れることができます。
それぞれの :term:`UOP<Micro-Op (UOP)` は、既知の固定されたレイテンシーをとります。

.. Speculation support is provided by auto-generating a pipeline that
.. passes down the :term:`UOP<Micro-Op (UOP)` meta-data and *branch mask* in parallel with
.. the :term:`UOP<Micro-Op (UOP)` within the expert-written :term:`Functional Unit` . If a :term:`UOP<Micro-Op (UOP)` is
.. misspeculated, it’s response is de-asserted as it exits the functional
.. unit.

投機実行のサポートは、専門家によって書かれた :term:`機能ユニット` 内の :term:`UOP<Micro-Op (UOP)` メタデータと *分岐マスク* を並行して渡すパイプラインを自動的に生成することによって提供されます。
もし :term:`UOP<Micro-Op (UOP)` が誤って指定された場合、その応答は機能ユニットを出るときに無効にされます。

.. An example pipelined :term:`Functional Unit` is shown in :numref:`abstract-fu`.

パイプライン化された :term:`機能ユニット` の例を :numref:`abstract-fu` に示します。


.. Un-pipelined Functional Units
.. ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

非パイプラインの機能ユニット
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. Un-pipelined :term:`Functional Unit` s (e.g., a divider) take an variable (and
.. unknown) number of cycles to complete a single operation. Once occupied,
.. they de-assert their ready signal and no additional :term:`UOPs<Micro-Op (UOP)` may be
.. scheduled to them.

パイプライン化されていない :term:`機能ユニット` (例：除算器)は、1つの操作を完了するために可変の(そして未知の)サイクル数を要します。
一旦占有されると、レディ信号のアサートが解除され、追加の :term:`UOP<Micro-Op (UOP)` がスケジューリングされることはありません。

.. Speculation support is provided by tracking the **branch mask** of the
.. :term:`UOP<Micro-Op (UOP)` in the :term:`Functional Unit`.

投機実行のサポートは、 :term:`UOP<Micro-Op (UOP)` の **ブランチマスク** を :term:`機能ユニット` でトラッキングすることで行われます。

.. The only requirement of the expert-written un-pipelined :term:`Functional Unit`
.. is to provide a *kill* signal to quickly remove misspeculated
.. :term:`UOPs<Micro-Op (UOP)`. [1]_

専門家によって書かれたパイプライン化されていない :term:`機能ユニット` の唯一の要件は、
誤って仕様化された :term:`UOPs<Micro-Op (UOP)` を素早く取り除くための *kill* シグナルを提供することです。[1]_

.. _fu-hierarchy:
.. figure:: /figures/functional-unit-hierarchy.png
    :alt: Functional Unit Hierarchy

    破線の楕円は専門家によって書かれた低レベルの :term:`機能ユニット` であり、四角は低レベルの :term:`機能ユニット` をインスタンス化する具象クラスであり、
    八角は汎用的な投機のサポートと BOOM パイプラインとのインターフェイスを提供する抽象クラスです。
    浮動小数点の除算と平方根のユニットは ``Pipelined`` と ``Unpipelined`` のどちらの抽象クラスにも
    きれいに収まらないので、``FunctionalUnit`` のスーパークラスを直接継承しています。

..    The dashed ovals are the low-level :term:`Functional Unit` s written by experts, the squares are
..    concrete classes that instantiate the low-level :term:`Functional Unit` s, and the octagons are abstract classes that
..    provide generic speculation support and interfacing with the BOOM pipeline. The floating point divide
..    and squart-root unit doesn’t cleanly fit either the ``Pipelined`` nor ``Unpipelined`` abstract class, and so directly
..    inherits from the ``FunctionalUnit`` super class.

.. Branch Unit & Branch Speculation
.. --------------------------------

分岐ユニット & 分岐命令の投機実行
---------------------------------

.. The :term:`Branch Unit` handles the resolution of all branch and jump
.. instructions.

:term:`分岐ユニット` は、すべての分岐命令とジャンプ命令の解決を行います。

.. All :term:`UOPs<Micro-Op (UOP)` that are "inflight" in the pipeline (have an allocated ROB
.. entry) are given a branch mask, where each bit in the branch mask
.. corresponds to an un-executed, inflight branch that the :term:`UOP<Micro-Op (UOP)` is
.. speculated under. Each branch in *Decode* is allocated a branch tag,
.. and all following :term:`UOPs<Micro-Op (UOP)` will have the corresponding bit in the
.. branch mask set (until the branch is resolved by the :term:`Branch Unit`).

パイプラインの中で "インフライト "である(割り当てられたROBエントリを持つ)すべての :term:`UOPs<Micro-Op (UOP)` には分岐マスクが与えられます。
分岐マスクの各ビットは、 :term:`UOPs<Micro-Op (UOP)` が予測されるされる未実行のインフライト分岐に対応しています。
*デコード* の各分岐命令には分岐タグが割り当てられ、それに続くすべての :term:`UOPs<Micro-Op (UOP)` には分岐マスクの対応するビットが設定されます
( :term:`Branch Unit` でブランチが解決されるまで)。


.. If the branches (or jumps) have been correctly speculated by the
.. :term:`Front-end`, then the :term:`Branch Unit` s only action is to broadcast the
.. corresponding branch tag to *all* inflight :term:`UOPs<Micro-Op (UOP)` that the branch has
.. been resolved correctly. Each :term:`UOP<Micro-Op (UOP)` can then clear the corresponding
.. bit in its branch mask, and that branch tag can then be allocated to a
.. new branch in the *Decode* stage.

分岐(またはジャンプ)が :term:`フロントエンド` によって正しく推測された場合、 :term:`分岐ユニット` の唯一のアクションは、
ブランチが正しく解決されたことを *全ての* インフライトの :term:`UOPs<Micro-Op (UOP)` に対応する分岐タグをブロードキャストすることです。
各 :term:`UOP<Micro-Op (UOP)` は、その分岐マスクの対応するビットをクリアすることができ、
その分岐タグは、その後、*Decode* ステージで新しいブランチに割り当てることができます。

.. If a branch (or jump) is misspeculated, the :term:`Branch Unit` must redirect
.. the PC to the correct target, kill the :term:`Front-end` and :term:`Fetch Buffer`, and
.. broadcast the misspeculated branch tag so that all dependent, inflight
.. :term:`UOPs<Micro-Op (UOP)` may be killed. The PC redirect signal goes out immediately, to
.. decrease the misprediction penalty. However, the *kill* signal is
.. delayed a cycle for critical path reasons.

もし分岐(またはジャンプ)の予測が間違っていた場合、 :term:`分岐ユニット` は PC を正しいターゲットにリダイレクトし、
:term:`フロントエンド` と :term:`フェッチバッファ` を殺し、
依存しているすべての機内の :term:`UOPs<Micro-Op (UOP)` を殺すことができるように、予測が間違っている分岐タグをブロードキャストしなければなりません。
PCリダイレクト信号は、分岐予測ミスのペナルティを減らすために、すぐに出力されます。
しかし、*kill* 信号はクリティカルパス上の理由から1サイクル遅れます。

.. The :term:`Front-end` must pass down the pipeline the appropriate branch
.. speculation meta-data, so that the correct direction can be reconciled
.. with the prediction. Jump Register instructions are evaluated by
.. comparing the correct target with the PC of the next instruction in the
.. ROB (if not available, then a misprediction is assumed). Jumps are
.. evaluated and handled in the :term:`Front-end` (as their direction and target
.. are both known once the instruction can be decoded).

:term:`フロントエンド` は、正しい方向を予測と一致させるために、
適切な分岐推測のメタデータをパイプラインに渡さなければなりません。
ジャンプレジスタ命令は、正しいターゲットとROB内の次の命令のPCを比較して評価されます(利用できない場合は、分岐予測ミスが想定されます)。
ジャンプは :term:`フロントエンド` で評価され、処理されます(命令がデコードできるようになると、その方向とターゲットの両方が判明するため)。

.. BOOM (currently) only supports having one :term:`Branch Unit` .

BOOMは（現在のところ）1つの :term:`分岐ユニット` を持つことのみをサポートしています。

.. Load/Store Unit
.. ---------------

ロードストアユニット
--------------------

.. The **Load/Store Unit (LSU)** handles the execution of load, store, atomic,
.. and fence operations.

**ロード/ストアユニット(LSU)** は、ロード、ストア、アトミック、フェンスの各オペレーションの実行を担当します。

.. BOOM (currently) only supports having one LSU (and thus can only send
.. one load or store per cycle to memory). [2]_

BOOMは(現在)、1つのLSUを持つことしかサポートしていません(したがって、1サイクルあたり1つのロードまたはストアをメモリに送ることしかできません)。[2]_

.. See :ref:`The Load/Store Unit (LSU)` for more details on the LSU.

LSUの詳細については :ref:`The Load/Store Unit (LSU)` を参照してください。


.. Floating Point Units
.. --------------------

浮動小数点ユニット
------------------

.. _fp-fu:
.. figure:: /figures/functional-unit-fpu.png
    :scale: 15 %
    :align: center
    :alt: Functional Unit for FPU

    FPUのクラス階層を示します。専門家によって書かれたコードは、hardfloatとrocketのリポジトリに含まれています。
    "FPU"クラスはRocketコンポーネントをインスタンス化し、
    それ自体はさらに抽象的な :term:`機能ユニット` クラスによってラップされています(これはアウトオブオーダーの投機サポートを提供します)。


..    The class hierarchy of the FPU is shown. The expert-written code is contained within
..    the hardfloat and rocket repositories. The "FPU" class instantiates the Rocket components, which itself
..    is further wrapped by the abstract :term:`Functional Unit` classes (which provides the out-of-order speculation
..    support).

.. The low-level floating point units used by BOOM come from the Rocket
.. processor (https://github.com/chipsalliance/rocket-chip) and hardfloat
.. (https://github.com/ucb-bar/berkeley-hardfloat) repositories. Figure
.. :numref:`fp-fu` shows the class hierarchy of the FPU.

BOOM で使用される低レベルの浮動小数点ユニットは、Rocket プロセッサ (https://github.com/chipsalliance/rocket-chip) と 
hardfloat (https://github.com/ucb-bar/berkeley-hardfloat) のリポジトリから取得しています。
図 :numref:`fp-fu` は、FPU のクラス階層を示しています。

.. To make the scheduling of the write-port trivial, all of the pipelined
.. FP units are padded to have the same latency. [3]_

書き込みポートのスケジューリングが容易になるように、パイプライン化されたFPユニットはすべて同じレイテンシになるようにパディングされています。[3]_

.. Floating Point Divide and Square-root Unit
.. ------------------------------------------

浮動小数点除算と平方根ユニット
------------------------------

.. BOOM fully supports floating point divide and square-root operations
.. using a single **FDiv/Sqrt** (or fdiv for short). BOOM accomplishes this by
.. instantiating a double-precision unit from the hardfloat repository. The
.. unit comes with the following features/constraints:

BOOM は、単一の **FDiv/Sqrt** (略して fdiv) を用いた浮動小数点の除算と平方根演算を完全にサポートしています。
BOOM は hardfloat リポジトリから倍精度ユニットをインスタンス化することでこれを実現しています。
このユニットには以下のような機能・制約があります。

.. -  expects 65-bit recoded double-precision inputs
.. 
.. -  provides a 65-bit recoded double-precision output
.. 
.. -  can execute a divide operation and a square-root operation
..    simultaneously
.. 
.. -  operations are unpipelined and take an unknown, variable latency
.. 
.. -  provides an *unstable* FIFO interface

- 65ビットの再コード化された倍精度入力を期待する

- 65ビットに再コード化された倍精度の出力を提供する

- 除算と平方根の演算を同時に実行可能

- 演算はパイプライン化されておらず、未知の可変レイテンシーを要する

- *不安定* なFIFOインターフェース

.. Single-precision operations have their operands upscaled to
.. double-precision (and then the output downscaled). [4]_

単精度演算は、オペランドが倍精度にアップスケールされ、出力はダウンスケールされます。[4]_

.. Although the unit is unpipelined, it does not fit cleanly into the
.. Pipelined/Unpipelined abstraction used by the other :term:`Functional Unit` s
.. (see :numref:`fu-hierarchy`). This is because the unit provides
.. an unstable FIFO interface: although the unit may provide a *ready*
.. signal on Cycle ``i``, there is no guarantee that it will continue
.. to be *ready* on Cycle ``i+1``, even if no operations are enqueued.
.. This proves to be a challenge, as the Issue Queue may attempt to issue
.. an instruction but cannot be certain the unit will accept it once it
.. reaches the unit on a later cycle.

このユニットは非パイプラインですが、他の :term:`機能ユニット` で使用されている Pipelined/Unpipelined の抽象化にはきれいに収まりません
( :numref:`fu-hierarchy` 参照)。
これは、このユニットが不安定なFIFOインターフェースを提供しているからです。
ユニットはサイクル``i``で *ready* 信号を提供しているかもしれませんが、たとえ操作がキューに入っていなくても、
サイクル ``i+1`` で *ready* であり続けるという保証はありません。
これは、命令発行キューが命令を発行しようとしても、
後のサイクルでユニットに届いたときにユニットがそれを受け入れるかどうか確信が持てないため、難しい問題となります。

.. The solution is to add extra buffering within the unit to hold
.. instructions until they can be released directly into the unit. If the
.. buffering of the unit fills up, back pressure can be safely applied to
.. the **Issue Queue**. [5]_

解決策としては、ユニット内に追加のバッファリングを追加して、命令がユニットに直接リリースされるまでの間、
命令を保持することです。ユニットのバッファリングが一杯になったら、
バックプレッシャーをかけて **命令発行キュー** に安全にアクセスできます。[5]_


.. Parameterization
.. ----------------

パラメータ化
------------

.. BOOM provides flexibility in specifying the issue width and the mix of
.. :term:`Functional Unit` s in the execution pipeline. See ``src/main/scala/exu/execution-units.scala``
.. for a detailed view on how to instantiate the execution pipeline in BOOM.

BOOMは命令発行の幅や実行パイプラインの中の :term:`機能ユニット` の組み合わせを柔軟に指定することができます。
BOOMの実行パイプラインをどのようにインスタンス化するかについての詳細な見解は、 ``src/main/scala/exu/execution-units.scala`` を参照してください。

.. Additional parameterization, regarding things like the latency of the FP
.. units can be found within the configuration settings (``src/main/common/config-mixins.scala``).

FPユニットのレイテンシーなどに関する追加のパラメータ設定は、コンフィギュレーション設定(``src/main/common/config-mixins.scala``)の中にあります。

.. Control/Status Register Instructions
.. ------------------------------------

Control/Statusレジスタ操作命令
------------------------------

.. A set of **Control/Status Register (CSR)** instructions allow the atomic
.. read and write of the Control/Status Registers. These architectural
.. registers are separate from the integer and floating registers, and
.. include the cycle count, retired instruction count, status, exception
.. PC, and exception vector registers (and many more!). Each CSR has its
.. own required privilege levels to read and write to it and some have
.. their own side-effects upon reading (or writing).

**CSR(Control/Status Register)** 命令群により、コントロール/ステータス・レジスタのアトミックな読み出し/書き込みが可能になりました。
これらのアーキテクチャ・レジスタは、整数レジスタやフローティング・レジスタとは別に、サイクル・カウント、リタイア命令カウント、ステータス、例外PC、例外ベクタ・レジスタ(その他多数)を含んでいます。
各CSRには、読み書きに必要な特権レベルがあり、読み書き時に独自の副作用が発生するものもあります。


.. BOOM (currently) does not rename *any* of the CSRs, and in addition to
.. the potential side-effects caused by reading or writing a CSR, **BOOM
.. will only execute a CSR instruction non-speculatively.** [6]_ This is
.. accomplished by marking the CSR instruction as a "unique" (or
.. "serializing") instruction - the ROB must be empty before it may proceed
.. to the Issue Queue (and no instruction may follow it until it has
.. finished execution and been committed by the ROB). It is then issued by
.. the Issue Queue, reads the appropriate operands from the Physical
.. Register File, and is then sent to the CSRFile. [7]_ The CSR instruction
.. executes in the CSRFile and then writes back data as required to the
.. Physical Register File. The CSRFile may also emit a PC redirect and/or
.. an exception as part of executing a CSR instruction (e.g., a syscall).

BOOMは(現在)CSRの名前を変更しません。
また、CSRの読み書きによって生じる潜在的な副作用に加えて、**BOOMはCSR命令を非特定的にしか実行しません** [6]_ 
これは、CSR命令を「ユニーク」(または「シリアライズ」)な命令としてマークすることで実現します。
その後、命令発行キューで発行され、物理レジスタ・ファイルから適切なオペランドを読み込み、CSRファイルに送られます。[7]_
CSR命令はCSRFileで実行され、必要に応じて物理レジスタファイルにデータを書き戻します。
CSRFileは、CSR命令（例：syscall）の実行の一部として、PCリダイレクトや例外を発することもあります。

.. The Rocket Custom Co-Processor Interface (RoCC)
.. -----------------------------------------------

.. The **RoCC interface** accepts a RoCC command and up to two register inputs
.. from the Control Processor’s scalar register file. The RoCC command is
.. actually the entire RISC-V instruction fetched by the Control Processor
.. (a "RoCC instruction"). Thus, each RoCC queue entry is at least
.. ``2\*XPRLEN + 32`` bits in size (additional RoCC instructions may use the
.. longer instruction formats to encode additional behaviors).

**RoCCインタフェース** は，コントロール・プロセッサのスカラ・レジスタ・ファイルから、RoCCコマンドと最大2つのレジスタ入力を受け付ける。
RoCCコマンドは、コントロール・プロセッサによってフェッチされたRISC-V命令全体("RoCC命令")となります。
したがって，各RoCCキューのエントリは，最低でも ``2\*XPRLEN + 32`` ビットのサイズになります
(追加のRoCC命令は，より長い命令フォーマットを使用して追加の動作をエンコードすることができます)。

.. As BOOM does not store the instruction bits in the ROB, a separate data
.. structure (A "RoCC Shim") holds the
.. instructions until the RoCC instruction can be committed and the RoCC
.. command sent to the co-processor.

BOOMはROBに命令ビットを格納しないので、別のデータ構造(A "RoCC Shim")は、
RoCC命令がコミットされ、RoCCコマンドがコプロセッサに送信されるまで、命令を保持します。

.. The source operands will also require access to BOOM’s register file.
.. RoCC instructions are dispatched to the Issue Window, and scheduled
.. so that they may access the read ports of the register file once the
.. operands are available. The operands are then written into the RoCC
.. Shim, which stores the operands and the instruction
.. bits until they can be sent to the co-processor. This requires
.. significant state.

また、ソース・オペランドは、BOOMのレジスタ・ファイルにアクセスする必要がある。
RoCC命令はイシューウィンドウにディスパッチされ、オペランドが利用可能になった時点でレジスタファイルのリードポートにアクセスできるようにスケジューリングされます。
その後、オペランドはRoCC Shimに書き込まれ、コプロセッサに送信されるまでオペランドと命令ビットが保存されます。
これには重要な状態が必要です。

.. After issue to RoCC, we track a queue of in-flight RoCC instructions,
.. since we need to translate the logical destination register identifier
.. from the RoCC response into the previously renamed physical destination
.. register identifier.

RoCCへの発行後、飛行中のRoCC命令のキューを追跡します。
これは、RoCCレスポンスからの論理的なデスティネーション・レジスタ識別子を、
以前に名前を変更した物理的なデスティネーション・レジスタ識別子に変換する必要があるためです。

.. Currently the RoCC interface does not support interrupts, exceptions,
.. reusing the BOOM FPU, or direct access to the L1 data cache. This should
.. all be straightforward to add, and will be completed as demand arises.

現在、RoCCインターフェイスは、割り込み、例外処理、BOOM FPUの再利用、L1データキャッシュへの直接アクセスなどをサポートしていません。
これらはすべて簡単に追加できるはずで、需要があれば完成させる予定です。

.. [1]
   This constraint could be relaxed by waiting for the un-pipelined unit
   to finish before de-asserting its busy signal and suppressing the
   *valid* output signal.

.. [2]
   Relaxing this constraint could be achieved by allowing multiple LSUs
   to talk to their own bank(s) of the data-cache, but the added
   complexity comes in allocating entries in the LSU before knowing the
   address, and thus which bank, a particular memory operation pertains
   to.

.. [3]
   Rocket instead handles write-port scheduling by killing and
   refetching the offending instruction (and all instructions behind it)
   if there is a write-port hazard detected. This would be far more
   heavy-handed to do in BOOM.

.. [4]
   It is cheaper to perform the SP-DP conversions than it is to
   instantiate a single-precision fdivSqrt unit.

.. [5]
   It is this ability to hold multiple inflight instructions within the
   unit simultaneously that breaks the “only one instruction at a time"
   assumption required by the UnpipelinedFunctionalUnit abstract class.

.. [6]
   There is a lot of room to play with regarding the CSRs. For example,
   it is probably a good idea to rename the register (dedicated for use
   by the supervisor) as it may see a lot of use in some kernel code and
   it causes no side-effects.

.. [7]
   The CSRFile is a Rocket component.

