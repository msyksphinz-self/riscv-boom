.. The Issue Unit
.. ==============

命令発行ユニット
================

.. The **Issue Queue** s hold dispatched :term:`Micro-Ops (UOPs) <Micro-Op (UOP)>` that have not yet executed.
.. When all of the operands for the :term:`UOP<Micro-Op (UOP)` are ready, the issue slot sets
.. its "request" bit high. The issue select logic then chooses to issue a
.. slot which is asserting its "request" signal. Once a :term:`UOP<Micro-Op (UOP)` is issued,
.. it is removed from the Issue Queue to make room for more dispatched
.. instructions.

**命令発行キュー** は、まだ実行されていないディスパッチされた :term:`Micro-Ops（UOP）<Micro-Ops（UOP）>` を保持します。
:term:`UOP<Micro-Op (UOP)>` のすべてのオペランドの準備が整うと、発行スロットはその"要求"ビットを1に設定します。
命令発行選択回路は、"request "信号をアサートしているスロットの発行を選択します。
:term:`UOP<Micro-Op (UOP)` が発行されると、そのスロットはイシューキューから削除され、
次にディスパッチされた命令のためのスペースが確保されます。

.. BOOM uses a split Issue Queues - instructions of specific types are placed
.. into a unique Issue Queue (integer, floating point, memory).

BOOMでは、分割された命令発行キューを使用しています。
特定のタイプの命令は、固有の発行キュー(整数、浮動小数点、メモリ)に入れられます。

.. Speculative Issue
.. -----------------

投機命令発行
------------

.. Although not yet supported, future designs may choose to speculatively
.. issue :term:`UOPs<Micro-Op (UOP)` for improved performance (e.g., speculating that a load
.. instruction will hit in the cache and thus issuing dependent :term:`UOPs<Micro-Op (UOP)`
.. assuming the load data will be available in the bypass network). In such
.. a scenario, the Issue Queue cannot remove speculatively issued
.. :term:`UOPs<Micro-Op (UOP)` until the speculation has been resolved. If a
.. speculatively-issued :term:`UOP<Micro-Op (UOP)` failure occurs, then all issued :term:`UOPs<Micro-Op (UOP)`
.. that fall within the speculated window must be killed and retried from
.. the Issue Queue. More advanced techniques are also available.

まだサポートされていませんが、将来の設計では、パフォーマンスを向上させるために、
:term:`UOPs<Micro-Op (UOP)` を投機的に発行することが選択されるかもしれません
(例えば、ロード命令がキャッシュにヒットすると推測し、
ロードデータがバイパスネットワークで利用可能であると仮定して、
依存性のある:term:`UOPs<Micro-Op (UOP)` を発行するなど)。
このようなシナリオでは、投機的に発行された :term:`UOPs<Micro-Op (UOP)` は、
投機が解消されるまで発行キューは削除できません。
投機的に発行された :term:`UOP<Micro-Op (UOP)` のハザードが発生した場合、
投機されたウィンドウ内にあるすべての発行済み :term:`UOP<Micro-Op (UOP)` を殺して、
発行キューから再送する必要があります。
さらに高度な技術も用意されています。


.. Issue Slot
.. ----------

発行スロット
------------

.. :numref:`single-issue-slot` shows a single **issue slot** from the
.. Issue Queue. [1]_

:numref:`single-issue-slot` は、発行キューから1つの **発行スロット** を表示します。[1]_

.. Instructions are *dispatched* into the Issue Queue. From here, they
.. wait for all of their operands to be ready ("p" stands for *presence*
.. bit, which marks when an operand is *present* in the register file).

命令はイシュー・キューに *ディスパッチ* されます。
ここからは、すべてのオペランドの準備が整うのを待ちます
("p "は*presence*ビットの略で、オペランドがレジスタファイルに*存在していることを示します)。。

.. Once ready, the issue slot will assert its "request" signal, and wait
   to be *issued*.

準備が整うと、発行スロットは "request "信号をアサートして、発行されるのを待ちます。

.. Issue Select Logic
.. ------------------

命令発行選択論理
----------------

.. _single-issue-slot:
.. figure:: /figures/issue_slot.png
    :alt: Single Issue Slot

    A single issue slot from the Issue Queue.

.. Each issue select logic port is a static-priority encoder that picks
.. that first available :term:`UOP<Micro-Op (UOP)` in the Issue Queue. Each port will only
.. schedule a :term:`UOP<Micro-Op (UOP)` that its port can handle (e.g., floating point
.. :term:`UOPs<Micro-Op (UOP)` will only be scheduled onto the port governing the Floating
.. Point Unit). This creates a cascading priority encoder for ports that
.. can schedule the same :term:`UOPs<Micro-Op (UOP)` as each other.

各命令発行選択論理ポートは、発行キューの中で最初に利用可能な :term:`UOP<Micro-Op (UOP)` を選択する静的優先順位エンコーダです。
各ポートは、そのポートが処理できる:term:`UOP<Micro-Op (UOP)` のみをスケジュールします
(例えば、浮動小数点の :term:`UOP<Micro-Op (UOP)` は、浮動小数点ユニットを管理するポートにのみスケジュールされます)。
これにより、お互いに同じ :term:`UOPs<Micro-Op (UOP)` をスケジューリングできるポートに対して、
カスケード式のプライオリティ・エンコーダーが作成されます。

.. If a **Functional Unit** is unavailable, it de-asserts its available signal
.. and instructions will not be issued to it (e.g., an un-pipelined
.. divider).

**機能ユニット** が利用できない場合、その機能ユニットは利用可能な信号を解除し、
その機能ユニットには命令が発行されません(例: パイプライン化されていない除算器)。

.. Un-ordered Issue Queue
.. -----------------------

順序化されていない発行キュー
----------------------------

.. There are two scheduling policies available in BOOM.

BOOMでは2つのスケジューリングポリシが使用可能です。

.. The first is a MIPS R10K-style Un-ordered Issue
.. Queue. Dispatching instructions are placed
.. into the first available Issue Queue slot and remain there until they
.. are *issued*. This can lead to pathologically poor performance,
.. particularly in scenarios where unpredictable branches are placed into
.. the lower priority slots and are unable to be issued until the ROB fills
.. up and the Issue Window starts to drain. Because instructions following
.. branches are only *implicitly* dependent on the branch, there is no
.. other forcing function that enables the branches to issue earlier,
.. except the filling of the ROB.

1つ目は、MIPS R10KスタイルのUn-ordered Issue Queueです。
ディスパッチされた命令は、最初に利用可能な命令キュースロットに配置され、
その命令が *発行* されるまでそこに留まります。
これは特に、予測できない分岐が優先度の低いスロットに置かれ、
ROBがいっぱいになって発行ウィンドウが空になるまで発行できないようなシナリオでは、
病的なほどパフォーマンスが低下する可能性があります。
分岐に続く命令は、分岐に *暗黙のうちに* 依存しているだけなので、
ROBが満杯になること以外に、分岐を早く発行できる強制的な機能はありません。

.. Age-ordered Issue Queue
.. ------------------------

順序付けされた発行キュー

.. The second available policy is an Age-ordered Issue Queue. Dispatched
.. instructions are placed into the bottom of the Issue Queue (at lowest
.. priority). Every cycle, every instruction is shifted upwards (the Issue
.. queue is a “collapsing queue"). Thus, the oldest instructions will have
.. the highest issue priority. While this increases performance by
.. scheduling older branches and older loads as soon as possible, it comes
.. with a potential energy penalty as potentially every Issue Queue slot
.. is being read and written to on every cycle.

2番目に利用可能なポリシーは、年齢順の発行キューです。
ディスパッチされた命令は、発行キューの一番下(優先順位が一番低い)に置かれます。
サイクルごとに、すべての命令が上にシフトされます(イシューキューは"折りたたみ式キュー"です)。
したがって、最も古い命令が最も高い発行優先度を持つことになります。
これにより、古い分岐や古いロードをできるだけ早くスケジューリングすることでパフォーマンスが向上しますが、
サイクルごとにすべてのイシューキューのスロットが読み書きされる可能性があるため、潜在的なエネルギーペナルティが発生します。

.. Wake-up
.. -------

ウェイクアップ
--------------

.. There are two types of wake-up in BOOM - *fast* wakeup and *slow*
.. wakeup (also called a long latency wakeup). Because ALU :term:`UOPs<Micro-Op (UOP)` can send their write-back data through the
.. bypass network, issued ALU :term:`UOPs<Micro-Op (UOP)` will broadcast their wakeup to the
.. Issue Queue as they are issued.

BOOMのウェイクアップには、 *fast* ウェイクアップと *slow* ウェイクアップ(long latency wakeupともいう)の2種類があります。
ALU :term:`UOPs<Micro-Op (UOP)` はバイパスネットワークを介してライトバックデータを送ることができるため、
発行されたALU :term:`UOPs<Micro-Op (UOP)` は、発行された時点でそのウェイクアップをIssue Queueにブロードキャストします。

.. However, floating-point operations, loads, and variable latency
.. operations are not sent through the bypass network, and instead the
.. wakeup signal comes from the register file ports during the *write-back*
.. stage.

しかし、浮動小数点演算、ロード、可変レイテンシー演算はバイパスネットワークを経由せず、
代わりにライトバックステージでレジスタファイルポートからウェイクアップ信号が送られます。

.. [1]
   Conceptually, a bus is shown for implementing the driving of the
   signals sent to the **Register Read** Stage. In reality BOOM actually
   uses muxes.
