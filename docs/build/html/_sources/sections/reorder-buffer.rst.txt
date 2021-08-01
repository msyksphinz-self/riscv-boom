.. _rob-dispatch-stage:

.. The Reorder Buffer (ROB) and the Dispatch Stage
.. ===============================================

リオーダバッファ(ROB)とディスパッチステージ
===========================================

.. The **Reorder Buffer (ROB)** tracks the state of all inflight instructions in the pipeline.
.. The role of the ROB is to provide the illusion to the programmer that
.. his program executes in-order. After instructions are *decoded* and
.. *renamed*, they are then *dispatched* to the ROB and the **Issue Queue** and
.. marked as *busy*. As instructions finish execution, they inform the ROB
.. and are marked *not busy*. Once the "head" of the ROB is no longer busy,
.. the instruction is *committed*, and it’s architectural state now
.. visible. If an exception occurs and the excepting instruction is at the
.. head of the ROB, the pipeline is flushed and no architectural changes
.. that occurred after the excepting instruction are made visible. The ROB
.. then redirects the PC to the appropriate exception handler.

**リオーダバッファ** は、パイプライン内のすべてのインフライト命令の状態を追跡します。
ROBの役割は、プログラマに自分のプログラムがインオーダーで実行されているように見せることです。
命令がデコードされ、リネームされると、その命令はROBと **命令キュー** に *ディスパッチ* され、 *busy* とマークされます。
命令の実行が終了すると、ROBに通知され、 *not busy* とマークされます。
ROBの"先頭"がビジー状態でなくなると、その命令は "コミット "され、アーキテクチャの状態が見えるようになります。
例外が発生し、例外命令がROBの先頭にある場合は、パイプラインがフラッシュされ、例外命令の後に発生したアーキテクチャの変更は表示されません。
そして、ROBはPCを適切な例外ハンドラにリダイレクトします。

.. The ROB Organization
.. --------------------

ROBの構成
---------

.. _rob:
.. figure:: /figures/rob.png
    :alt: The Reorder Buffer

    3命令発行の2ワイドBOOMのリオーダバッファです。ディスパッチされたUops(dis uops)はROBの下側(rob tail)に書き込まれ、
    コミットされたUops(com uops)は上側(rob head)からコミットされ、Renameの状態を更新します。
    実行を終えたUops（wb uops）は、ビジービットをクリアします。
    注：ディスパッチされたUOPは、同じROBの行にまとめて書き込まれ、
    メモリ上で連続して配置されるため、1台のPCで行全体を表現することができます。

..    The Reorder Buffer for a two-wide BOOM with three-issue. Dispatched uops (dis uops) are
..    written at the bottom of the ROB (rob tail), while committed uops (com uops) are committed from the top,
..    at rob head, and update the rename state. Uops that finish executing (wb uops) clear their busy bit. Note:
..    the dispatched uops are written into the same ROB row together, and are located consecutively in memory
..    allowing a single PC to represent the entire row.

.. The ROB is, conceptually, a circular buffer that tracks all inflight
.. instructions in-order. The oldest instruction is pointed to by the
.. *commit head*, and the newest instruction will be added at the *rob
.. tail*.

ROBは、概念的には、すべてのインフライト命令を順に追跡するリングバッファです。
最も古い命令は *commit head* で指し示され、最新の命令は *rob tail* で追加されます。

.. To facilitate superscalar *dispatch* and *commit*, the ROB is
.. implemented as a circular buffer with ``W`` banks (where ``W``
.. is the *dispatch* and *commit* width of the machine [1]_). This
.. organization is shown in :numref:`rob`.

スーパースカラの *ディスパッチ* と *コミット* を容易にするために、ROBは ``W`` バンクを持つ円形バッファとして実装されています
(ここで ``W`` はマシンの *dispatch* と *commit* の幅です [1]_ )。
この構成を :numref:`rob` に示します。

.. At *dispatch*, up to ``W`` instructions are written from the :term:`Fetch Packet`
.. into an ROB row, where each instruction is written to a
.. different bank across the row. As the instructions within a :term:`Fetch Packet`
.. are all consecutive (and aligned) in memory, this allows a
.. single PC to be associated with the entire :term:`Fetch Packet` (and the
.. instruction’s position within the :term:`Fetch Packet` provides the low-order
.. bits to its own PC). While this means that branching code will leave
.. bubbles in the ROB, it makes adding more instructions to the ROB very
.. cheap as the expensive costs are amortized across each ROB row.

*ディスパッチ* では、最大で ``W`` 個の命令が :term:`フェッチパケット` から ROB の行に書き込まれ、
各命令は行間の異なるバンクに書き込まれます。
1つの :term:`フェッチパケット` 内の命令はすべてメモリ内で連続している(アラインされている)ため、
これにより1つのPCを :term:`フェッチパケット` 全体に関連付けることができます
(また、 :term:`フェッチパケット` 内の命令の位置は、自身のPCに低次ビットを提供します)。
これは、分岐コードがROB内にバブルを残すことを意味しますが、
高価なコストが各ROB行で償却されるため、ROBへの命令の追加が非常に安くなります。

.. ROB State
.. ---------

ROBの状態
---------

.. Each ROB entry contains relatively little state:

各ROBのエントリには小さなステートが含まれます:

.. -  is entry valid?
.. 
.. -  is entry busy?
.. 
.. -  is entry an exception?
.. 
.. -  branch mask (which branches is this entry still speculated under?
.. 
.. -  rename state (what is the logical destination and the stale physical
..    destination?)
.. 
.. -  floating-point status updates
.. 
.. -  other miscellaneous data (e.g., helpful for statistic tracking)

- エントリが有効か？

- エントリがBusy状態か？

- エントリで例外が発生しているか？

- 分岐マスク(エントリ中のどの分岐命令がまだ投機中であるか？)

- リネーム状態(論理書き込みアドレスと古い書き込み物理レジスタ)

- 浮動小数点ステータスアップデート

- 他の様々なデータ(例えば統計のトラッキングに有益なもの)


.. The PC and the branch prediction information is stored on a per-row
.. basis (see :ref:`PC Storage`). The **Exception State** only tracks the
.. oldest known excepting instruction (see :ref:`Exception State`).

PCと分岐予測の情報は、行ごとに保存されます( :ref:`PC Storage` 参照)。
**例外状態** は、最も古い既知の例外命令のみを追跡します( :ref:`Exception State` 参照)。

.. Exception State
.. ~~~~~~~~~~~~~~~

例外の状態
~~~~~~~~~~

.. The ROB tracks the oldest excepting instruction. If this instruction
.. reaches the head of the ROB, then an exception is thrown.

ROBは、最も古い例外命令を追跡します。この命令がROBの先頭に到達すると、例外が発生します。


.. Each ROB entry is marked with a single-bit to signify whether or not the
.. instruction has encountered exceptional behavior, but the additional
.. exception state (e.g., the bad virtual address and the exception cause)
.. is only tracked for the oldest known excepting instruction. This saves
.. considerable state by not storing this on a per entry basis.

各ROBエントリには、その命令に例外的な動作が発生したかどうかを示す1ビットのマークが付けられますが、
追加の例外状態(不良仮想アドレスや例外原因など)は、既知の最古の例外命令についてのみ追跡されます。
これは、エントリごとに保存しないことで、かなりのステートを節約できます。

.. PC Storage
.. ~~~~~~~~~~

PCの保存
~~~~~~~~

.. The ROB must know the PC of every inflight instruction. This information
.. is used in the following situations:

ROBはすべてのインフライト命令のPCを知っていなければなりません。
この情報は以下のような状況で使用されます。

.. -  Any instruction could cause an exception, in which the "exception pc"
..    (epc) must be known.
.. 
.. -  Branch and jump instructions need to know their own PC for for target
..    calculation.
.. 
.. -  Jump-register instructions must know both their own PC **and the PC
..    of the following instruction** in the program to verify if the
..    :term:`Front-end` predicted the correct JR target.

- どのような命令でも例外が発生する可能性があり、その場合は"例外PC"(epc)を知る必要があります。

- 分岐命令やジャンプ命令は、対象となる計算のために自分のPCを知る必要があります。

- ジャンプレジスタ命令は、自分自身のPCと、 **プログラム中の次の命令** のPCの両方を知っていなければなりません。

.. This information is incredibly expensive to store. Instead of passing
.. PCs down the pipeline, branch and jump instructions access the ROB's "PC
.. File" during the **Register-read** stage for use in the :term:`Branch Unit`. Two
.. optimizations are used:

この情報を保存するには非常にコストがかかります。
分岐命令やジャンプ命令は、PCをパイプラインに渡す代わりに、 **レジスタリード** の段階でROBの"PCファイル"にアクセスし、 
:term:`分岐ユニット` で使用します。
2つの最適化が使われています。

.. -  only a single PC is stored per ROB row.
.. 
.. -  the PC File is stored in two banks, allowing a single read-port to
..    read two consecutive entries simultaneously (for use with JR
..    instructions).

- 単一のPCのみがROBの行に保存されます。

- PCファイルは2つのバンクに分かれており、1つのリードポートで連続した2つのエントリーを同時に読み取ることができます(JR命令で使用)。


.. The Commit Stage
.. ----------------

コミットステージ
----------------

.. When the instruction at the *commit head* is no longer busy (and it is
.. not excepting), it may be *committed*, i.e., its changes to the
.. architectural state of the machine are made visible. For superscalar
.. commit, the entire ROB row is analyzed for *not busy* instructions (and
.. thus, up to the entire ROB row may be committed in a single cycle). The
.. ROB will greedily commit as many instructions as it can per row to
.. release resource as soon as possible. However, the ROB does not
.. (currently) look across multiple rows to find commit-able instructions.

*コミットヘッド* にある命令がビジーでなくなると(そしてそれは例外ではない)、
その命令は *コミット* されるかもしれません。
スーパースカラのコミットでは、ROBの行全体が*not busy*命令について分析されます(そのため、1サイクルでROBの行全体までコミットされる可能性があります)。
ROBは、できるだけ早くリソースを解放するために、行ごとにできるだけ多くの命令を貪欲にコミットします。
しかし、ROBは複数の行をまたいでコミット可能な命令を探すことは(現在は)ありません。

.. Only once a store has been committed may it be sent to memory. For
.. superscalar committing of stores, the **Load/Store Unit (LSU)** is told "how many stores" may
.. be marked as committed. The LSU will then drain the committed stores to
.. memory as it sees fit.

ストアがコミットされて初めて、そのストアをメモリに送ることができます。
スーパースカラでストアをコミットする場合、 **Load/Store Unit (LSU)** に「いくつのストアをコミットしてよいか」を伝えます。
LSUはコミットされたストアを必要に応じてメモリに排出します。

.. When an instruction (that writes to a register) commits, it then frees
.. the *stale physical destination register*. The *stale pdst* is then free
.. to be re-allocated to a new instruction.

(レジスタに書き込む)命令がコミットすると、 *stale physical destination register* を解放します。
その後、 *stale pdst* は自由になり、新しい命令に再割り当てされます。


.. Exceptions and Flushes
.. ----------------------

例外とフラッシュ
----------------

.. Exceptions are handled when the instruction at the *commit head* is
.. excepting. The pipeline is then flushed and the ROB emptied. The **Rename
.. Map Tables** must be reset to represent the true, non-speculative
.. *committed* state. The :term:`Front-end` is then directed to the appropriate PC.
.. If it is an architectural exception, the excepting instruction’s PC
.. (referred to as the *exception vector*) is sent to the Control/Status
.. Register (CSR) file. If it is a micro-architectural exception (e.g., a
.. load/store ordering misspeculation) the failing instruction is refetched
.. and execution can begin anew.

例外は、 *commit head* にある命令が除くときに処理されます。
パイプラインはフラッシュされ、ROBは空になります。
**リネームマップテーブル** をリセットして、真の、非投機的な *committed* 状態を表す必要があります。
そして、 :term:`フロントエンド` は適切なPCに指示されます。
アーキテクチャ上の例外であれば、例外となる命令のPC( *例外ベクトル* と呼ばれます)がControl/Status Register(CSR)ファイルに送られます。
マイクロアーキテクチャ的な例外(ロード/ストア順序の誤記など)の場合は、失敗した命令がリフェッチされ、
新たに実行を開始することができます。


.. Parameterization - Rollback versus Single-cycle Reset
.. ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

パラメータ：ロールバックと1サイクルリセット
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. The behavior of resetting the Rename Map Tables is parameterizable. The first
.. option is to rollback the ROB one row per cycle to unwind the rename
.. state (this is the behavior of the MIPS
.. R10k). For each instruction, the *stale
.. physical destination* register is written back into the Map Table for
.. its *logical destination* specifier.

リネームマップテーブルをリセットする際の動作はパラメータ化可能です。
最初のオプションは、ROBをサイクルごとに1行ずつロールバックしてリネーム状態を解除することです(これはMIPS R10kの動作です)。
各命令に対して、 *stale物理的デスティネーション* レジスタは、その *論理的デスティネーション* 指定のためにマップテーブルに書き戻されます。


.. A faster single-cycle reset is available. This is accomplished by using
.. another rename snapshot that tracks the *committed* state of the rename
.. tables. This *Committed Map Table* is updated as instructions
.. commit. [2]_

より高速なシングルサイクルのリセットが可能です。
これは、リネームテーブルの *committed* 状態を追跡する別のリネームスナップショットを使用することで実現されます。
この *コミットマップテーブル* は命令がコミットすると更新されます。[2]_

.. Causes
.. ~~~~~~

例外要因
~~~~~~~~

The RV64G ISA provides relatively few exception sources:

    Load/Store Unit
        - page faults

    :term:`Branch Unit`
        - misaligned fetches

    **Decode** Stage
        - all other exceptions and interrupts can be handled before the
          instruction is dispatched to the ROB

.. Note that memory ordering speculation errors also originate from the
.. Load/Store Unit, and are treated as exceptions in the BOOM pipeline
.. (actually they only cause a pipeline “retry").

なお、メモリ順序の推測エラーもLoad/Storeユニットから発生し、
BOOMパイプラインでは例外として扱われます(実際にはパイプラインの「リトライ」が発生するだけです)。


Point of No Return (PNR)
------------------------

.. The point-of-no-return head runs ahead of the ROB commit head, marking the
.. next instruction which might be misspeculated or generate an exception.
.. These include unresolved branches and untranslated memory operations.
.. Thus, the instructions *ahead* of the commit head and *behind* the PNR
.. head are guaranteed to be *non-speculative*, even if they have not yet
.. written back.

Point-of-no-returnヘッドは、ROBコミットヘッドの前を走り、
誤判定や例外が発生する可能性のある次の命令をマークします。
これには、未解決の分岐や翻訳されていないメモリ操作などが含まれます。
このように、コミットヘッドの *前* とPNRヘッドの *後* にある命令は、
たとえまだ書き戻されていなくても、 *非仕様* であることが保証されます。

.. Currently the PNR is only used for RoCC instructions. RoCC co-processors
.. typically expect their instructions in-order, and do not tolerate misspeculation.
.. Thus we can only issue a instruction to our co-processor when it has past the
.. PNR head, and thus is no longer speculative.

現在、PNRはRoCC命令にのみ使用されています。
RoCCコプロセッサは、通常、命令をインオーダーで実行することを期待しており、
投機ミスを許容しません。
そのため、コプロセッサへの命令発行は、その命令がPNRヘッドを通過して、推測可能でなくなってから行うことになります。

.. [1]
   This design sets up the *dispatch* and *commit* widths of BOOM to be
   the same. However, that is not necessarily a fundamental constraint,
   and it would be possible to orthogonalize the *dispatch* and *commit*
   widths, just with more added control complexity.

.. [2]
   The tradeoff here is between longer latencies on exceptions versus an
   increase in area and wiring.
