.. The Load/Store Unit (LSU)
.. =========================

ロードストアユニット(LSU)
=========================

.. _lsu:
.. figure:: /figures/lsu.png
    :alt: Load Store Unit

    ロードストアユニット
..    The Load/Store Unit

.. The **Load/Store Unit (LSU)** is responsible for deciding when to fire memory
.. operations to the memory system. There are two queues: the **Load
.. Queue (LDQ)**, and the **Store Queue (STQ)**. Load instructions generate a
.. “uopLD" :term:`Micro-Op (UOP)`. When issued, "uopLD" calculates the load address and
.. places its result in the LDQ. Store instructions (may) generate *two*
.. :term:`UOP<Micro-Op (UOP)>` s, “uopSTA" (Store Address Generation) and “uopSTD" (Store Data
.. Generation). The STA :term:`UOP<Micro-Op (UOP)>` calculates the store address and updates the
.. address in the STQ entry. The STD :term:`UOP<Micro-Op (UOP)>` moves the store data into the
.. STQ entry. Each of these :term:`UOP<Micro-Op (UOP)>` s will issue out of the
.. *Issue Window* as soon their operands are ready. See :ref:`Store Micro-Ops`
.. for more details on the store :term:`UOP<Micro-Op (UOP)>` specifics.

**LSU（Load/Store Unit）** は、メモリシステムへのメモリ操作の発火タイミングを決定する役割を担っています。
**ロードキュー(LDQ)** と **ストアキュー(STQ)** の2つのキューがあります。
ロード命令は、"uopLD" :term:`Micro-Op (UOP)` を生成します。
uopLD は発行されるとロードアドレスを計算し、その結果をLDQに格納します。
ストア命令では、"uopSTA"（ストアアドレス生成）と"uopSTD"（ストアデータ生成）の *2つの* :term:`UOP<Micro-Op (UOP)>` を生成します。
STA :term:`UOP<Micro-Op (UOP)>` は、ストアアドレスを計算し、STQエントリのアドレスを更新します。
STD :term:`UOP<Micro-Op (UOP)>` は、ストアデータをSTQエントリに移動させます。
これらの各 :term:`UOP<Micro-Op (UOP)>` は、オペランドの準備が整い次第、 *命令発行ウィンドウ* から発行されます。
ストアの :term:`UOP<Micro-Op (UOP)>` の仕様の詳細については、 :ref:`Store Micro-Ops` を参照してください。


.. Store Instructions
.. ------------------

ストア命令
----------

.. Entries in the Store Queue are allocated in the *Decode* stage (
.. stq(i).valid is set). A “valid" bit denotes when an entry in the STQ holds
.. a valid address and valid data (stq(i).bits.addr.valid and stq(i).bits.data.valid).
.. Once a store instruction is committed, the corresponding entry in the Store
.. Queue is marked as committed. The store is then free to be fired to the
.. memory system at its convenience. Stores are fired to the memory in program
.. order.

ストアキューのエントリは、デコード時に確保されます（stq(i).validがセットされます）。
Validビットは、STQ内のエントリが有効なアドレスと有効なデータを保持していることを示します（stq(i).bits.addr.validおよびstq(i).bits.data.valid）。
ストア命令がコミットされると、ストアキューの対応するエントリがコミットされたとマークされます。
その後、ストアは自由にメモリシステムに送信されます。
ストアはプログラム順にメモリに投入されます。

.. Store Micro-Ops
.. ~~~~~~~~~~~~~~~

ストア Micro-Ops
----------------

.. Stores are inserted into the issue window as a single instruction (as
.. opposed to being broken up into separate addr-gen and data-gen
.. :term:`UOP<Micro-Op (UOP)>` s). This prevents wasteful usage of the expensive issue window
.. entries and extra contention on the issue ports to the LSU. A store in
.. which both operands are ready can be issued to the LSU as a single
.. :term:`UOP<Micro-Op (UOP)>` which provides both the address and the data to the LSU. While
.. this requires store instructions to have access to two register file
.. read ports, this is motivated by a desire to not cut performance in half
.. on store-heavy code. Sequences involving stores to the stack should
.. operate at IPC=1!

ストアは命令発行ウィンドウに1つの命令として挿入されます（別々のaddr-genおよびdata-gen :term:`UOP<Micro-Op (UOP)>`に分割されるのではなく）。
これにより、高価な命令発行ウィンドウ・エントリの無駄な使用や、LSUへの命令発行ポートでの余計な競合を防ぐことができます。
両方のオペランドが準備できているストアは、アドレスとデータの両方をLSUに提供する単一の :term:`UOP<Micro-Op (UOP)>` としてLSUに発行することができます。
これにより、ストア命令は2つのレジスタファイルリードポートにアクセスする必要がありますが、これはストアを多用するコードでパフォーマンスを半減させたくないという思いからです。
スタックへのストアを含むシーケンスは、IPC=1で動作しなければなりません。

.. However, it is common for store addresses to be known well in advance of
.. the store data. Store addresses should be moved to the STQ as soon as
.. possible to allow later loads to avoid any memory ordering failures.
.. Thus, the issue window will emit uopSTA or uopSTD :term:`UOP<Micro-Op (UOP)>` s as required,
.. but retain the remaining half of the store until the second operand is
.. ready.

しかし、ストア・アドレスはストア・データのかなり前から知っているのが普通です。
ストアアドレスはできるだけ早くSTQに移動させ、後でロードできるようにして、メモリ順序の失敗を回避する必要があります。
このように、命令発行ウィンドウは必要に応じてuopSTAまたはuopSTD :term:`UOP<Micro-Op (UOP)>` を発するが、
第2オペランドの準備が整うまでストアの残り半分を保持する。

.. Load Instructions
.. -----------------

ロード命令
----------

.. Entries in the Load Queue (LDQ) are allocated in the *Decode* stage
.. (``ldq(i).valid``). In **Decode**, each load entry is also given a *store
.. mask* (``ldq(i).bits.st\_dep\_mask``), which marks which stores in the Store
.. Queue the given load depends on. When a store is fired to memory and
.. leaves the Store Queue, the appropriate bit in the *store mask* is cleared.

ロードキュー(LDQ)のエントリは、 *デコード* の段階で割り当てられます (``ldq(i).valid``)。
**デコード** ステージ中に、書くロードエントリは *ストアマスク* (``ldq(i).bits.st\_dep\_mask``) が割り当てられます。
これはストアキュー内のどのストアに依存しているかを示すものです。
ストアがメモリに実行されてストアキューから離れると、 *ストアマスク* の適切なビットがクリアされます。

.. Once a load address has been computed and placed in the LDQ, the
.. corresponding *valid* bit is set (``ldq(i).addr.valid``).

ロードアドレスが計算されてLDQに配置されると、対応する*valid*ビットが設定されます（``ldq(i).addr.valid``）。

.. Loads are optimistically fired to memory on arrival to the LSU (getting
.. loads fired early is a huge benefit of out–of–order pipelines).
.. Simultaneously, the load instruction compares its address with all of
.. the store addresses that it depends on. If there is a match, the memory
.. request is killed. If the corresponding store data is present, then the
.. store data is *forwarded* to the load and the load marks itself as
.. having *succeeded*. If the store data is not present, then the load goes
.. to *sleep*. Loads that have been put to sleep are retried at a later
.. time. [1]_

ロードは、LSUに到着した時点で、最適な方法でメモリに実行されます（ロードが早期にファイアーされることは、アウトオブオーダー・パイプラインの大きなメリットです）。
同時に、ロード命令は自分のアドレスと、依存するすべてのストアアドレスを比較します。
一致した場合、そのメモリ要求はキャンセルされます。
対応するストアデータが存在すれば、そのストアデータはロードに *転送* され、ロードは *成功* したと判断します。
ストアデータが存在しない場合は、ロードは *スリープ* に入ります。スリープ状態になったロードは、後から再試行されます。[1]_

.. The BOOM Memory Model
.. ---------------------

BOOMのメモリモデル
------------------

.. BOOM follows the RVWMO memory consistency model.

BOOMはRVWMOメモリコンシステンシモデルに従います。

.. BOOM currently exhibits the following behavior:

現在BOOMは以下のような動作をしています。

.. #. Write -> Read constraint is relaxed (newer loads may execute before
..    older stores).
.. 
.. #. Read -> Read constraint is maintained (loads to the same address
..    appear in order).
.. 
.. #. A thread can read its own writes early.

#. 書き込み→読み込みの制約が緩和された（新しいロードが古いストアより先に実行されることがある 新しいロードが古いストアより先に実行される可能性があります)。

#. 読み出し→読み出しの制約は維持されます（同じアドレスへのロードは 順番に現れる）。

#. スレッドは自分の書き込みを早く読むことができます。

.. Ordering Loads to the Same Address
.. ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

同一アドレスに対するロードの順序
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. The RISC-V WMO memory model requires that loads to the same address be ordered.
.. [2]_ This requires loads to search against other loads for potential address conflicts.
.. If a younger load executes before an older load with a matching address, the
.. younger load must be replayed and the instructions after it in the pipeline flushed.
.. However, this scenario is only required if a cache coherence probe event
.. snooped the core’s memory, exposing the reordering to the other threads.
.. If no probe events occurred, the load re-ordering may safely occur.

RISC-VのWMOメモリモデルでは、同じアドレスへのロードは順番に行う必要があります。[2]_ 
これにより、ロードはアドレス衝突の可能性がないか、他のロードに対して検索する必要があります。
アドレスが一致する古いロードの前に若いロードが実行された場合、若いロードは再生され、パイプライン内の後続の命令はフラッシュされなければなりません。
しかし、このシナリオが必要になるのは、キャッシュコヒーレンスのプローブイベントがコアのメモリをスヌープして、
他のスレッドに並び替えを暴露した場合だけです。
プローブイベントが発生しなければ、ロードリオーダリングは安全に行われるでしょう。

.. Memory Ordering Failures
.. ------------------------

メモリのオーダリングの失敗
--------------------------

.. The Load/Store Unit has to be careful regarding
.. store -> load dependences. For the best performance,
.. loads need to be fired to memory as soon as possible.

ロード/ストアユニットでは、ストア→ロードの依存関係に注意しなければなりません。
最高のパフォーマンスを得るためには、ロードをできるだけ早くメモリに投入する必要があります。

.. code-block:: bash

    sw x1 -> 0(x2)
    ld x3 <- 0(x4)

.. However, if x2 and x4 reference the same memory address, then the load
.. in our example *depends* on the earlier store. If the load issues to
.. memory before the store has been issued, the load will read the wrong
.. value from memory, and a *memory ordering failure* has occurred. On an
.. ordering failure, the pipeline must be flushed and the Rename Map Tables
.. reset. This is an incredibly expensive operation.

しかし、x2とx4が同じメモリアドレスを参照している場合、この例のロードは、先に発行されたストアに *依存* しています。
ストアが発行される前にロードがメモリに発行された場合、ロードはメモリから間違った値を読み出すことになり、 *メモリ順序付けの失敗* が発生します。
順番に失敗した場合、パイプラインをフラッシュし、リネームマップテーブルをリセットしなければなりません。これは非常に高価な処理です。

.. To discover ordering failures, when a store commits, it checks the
.. entire LDQ for any address matches. If there is a match, the store
.. checks to see if the load has *executed*, and if it got its data from
.. memory or if the data was forwarded from an older store. In either case,
.. a memory ordering failure has occurred.

順番付けの失敗を発見するために、ストアがコミットするときに、LDQ全体でアドレスの一致をチェックします。
一致した場合、ストアはロードが実行されたかどうか、またデータがメモリから取得されたものか、古いストアから転送されたものかを確認します。
いずれの場合も、メモリオーダーの失敗が発生しています。

.. See :numref:`lsu` for more information about the Load/Store Unit.

ロード/ストアユニットの詳細については、 :numref:`lsu` を参照してください。

.. [1]
   より高性能なプロセッサでは、ロードがスリープ状態になった原因を追跡し、
   ロードをブロックしているされた原因が解消された時点で負荷を起こします。

..    Higher-performance processors will track *why* a load was put to
..    sleep and wake it up once the blocking cause has been alleviated.

.. [2]
   技術的には、 *fence.r.r* を使用して、依存性のあるロードをリオーダするマシン上でのソフトウェアの正しい実行を提供することができます。
   しかし、ISAが依存性ロードのリオーダリングを禁止する理由は2つあります。1）他のポピュラーなISAではこのような緩和を認めていないため、
   ソフトウェアをRISC-Vに移植する際に、新たな課題が生じる可能性があること、
   2）慎重なソフトウェアは、適切な *fence* 命令を自由に使いすぎて、ソフトウェアの速度低下を招く可能性があること、です。
   ありがたいことに、順序付きの依存性ロードを強制することは、実際にはそれほどコストがかからないかもしれません。
   まず、ロードアドレスは早い段階で判明しており、どのような場合でもインオーダーで実行される可能性があります。
   第二に、順序違いのロードは、キャッシュコヒーレンスプローブのキャッシュ内でのみ問題となるため、パフォーマンス上のペナルティは無視できるでしょう。
   ロードは、ストアが既に使用しているLAQのCAM検索ポートと同じものを使用できます。
   1サイクルに1つのロードとストアのアドレス計算をサポートする場合には問題となる可能性がありますが、余分なCAMサーチポートはバンキングによって軽減されるか、
   より多くのキャッシュバンド幅をサポートするために必要な他のハードウェアコストに比べて小さいものとなります。


..   Technically, a *fence.r.r* could be used to provide the correct
..   execution of software on machines that reorder dependent loads.
..   However, there are two reasons for an ISA to disallow re-ordering of
..   dependent loads: 1) no other popular ISA allows this relaxation, and
..   thus porting software to RISC-V could face extra challenges, and 2)
..   cautious software may be too liberal with the appropriate *fence*
..   instructions causing a slow-down in software. Thankfully, enforcing
..   ordered dependent loads may not actually be very expensive. For one,
..   load addresses are likely to be known early - and are probably likely
..   to execute in-order anyways. Second, misordered loads are only a
..   problem in the cache of a cache coherence probe, so performance
..   penalty is likely to be negligible. The hardware cost is also
..   negligible - loads can use the same CAM search port on the LAQ that
..   stores must already use. While this may become an issue when
..   supporting one load and one store address calculation per cycle, the
..   extra CAM search port can either be mitigated via banking or will be
..   small compared to the other hardware costs required to support more
..   cache bandwidth.
