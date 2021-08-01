.. Instruction Fetch
.. =================

命令フェッチ
============

.. _front-end:
.. figure:: /figures/front-end.svg
    :alt: BOOM :term:`Front-end`

    The BOOM :term:`Front-end`


.. BOOM instantiates its own :term:`Front-end` , similar to how the Rocket core(s)
.. instantiates its own :term:`Front-end` . This :term:`Front-end` fetches instructions and
.. makes predictions throughout the Fetch stage to redirect the instruction
.. stream in multiple fetch cycles (**F0**, **F1**...). If a misprediction is detected in BOOM’s
.. :term:`Back-end` (execution pipeline), or one of BOOM’s own predictors wants to redirect the pipeline in
.. a different direction, a request is sent to the :term:`Front-end` and it begins
.. fetching along a new instruction path. See :ref:`Branch Prediction` for
.. more information on how branch prediction fits into the Fetch Stage’s pipeline.

BOOM は、Rocket コアが独自の :term:`フロントエンド` をインスタンス化するのと同様に、独自の :term:`フロントエンド` をインスタンス化します。
この :term:`フロントエンド` は、複数のフェッチサイクル (**F0**、**F1**...) で命令ストリームをリダイレクトするために、
フェッチステージを通して命令をフェッチし、予測を行います。
BOOMの :term:`バックエンド` (実行パイプライン)で予測の間違いが検出された場合や、
BOOM自身の予測器の一つがパイプラインを別の方向に向けたいと思った場合、 
:term:`フロントエンド` にリクエストが送られ、新しい命令パスに沿ってフェッチを開始します。
分岐予測がフェッチステージのパイプラインにどのように適合するかについての詳細は、 :ref:`分岐予測` を参照してください。


.. Since superscalar fetch is supported, the :term:`Front-end` retrieves a :term:`Fetch Packet`
.. of instructions from instruction memory and puts them into the
.. :term:`Fetch Buffer` to give to the rest of the pipeline. The :term:`Fetch Packet` also
.. contains other meta-data, such as a valid mask (which instructions in the
.. packet are valid?) and some branch prediction information that is used
.. later in the pipeline. Additionally, the PC and branch prediction information
.. is stored inside of the :term:`Fetch Target Queue` which holds this information
.. for the rest of the pipeline.

スーパースカラフェッチがサポートされているので、 :term:`フロントエンド` は命令メモリから命令の :term:`フェッチパケット` を取得し、 
:term:`フェッチバッファ` に入れてパイプラインの残りの部分に渡します。
フェッチパケットには、有効マスク(パケット内のどの命令が有効か)や、パイプラインの後半で使用される分岐予測情報など、
他のメタデータも含まれています。
さらに、PCと分岐予測情報は、パイプラインの残りの部分のためにこれらの情報を保持する :term:`フェッチターゲットキュー` の中に格納されます。


.. The Rocket Core I-Cache
.. -----------------------

Rocketコア 命令キャッシュ
-------------------------

.. BOOM instantiates the i-cache taken from the Rocket processor source code.
.. The i-cache is a virtually indexed, physically tagged set-associative cache.

BOOM は、Rocket プロセッサのソースコードから取得した命令キャッシュをインスタンス化します。
命令キャッシュは、仮想的にインデックスを付け、
物理アドレスにタグを付けたセットアソシアティブキャッシュです。

.. To save power, the i-cache reads out a fixed number of bytes (aligned)
.. and stores the instruction bits into a register. Further instruction
.. fetches can be managed by this register. The i-cache is only fired up
.. again once the fetch register has been exhausted (or a branch prediction
.. directs the PC elsewhere).

電力を節約するために、命令キャッシュは固定数のバイト(アラインド)を読み出し、
命令ビットをレジスタに格納します。
さらなる命令のフェッチは、このレジスタで管理できます。
命令キャッシュは、フェッチレジスタが使い果たされたとき(または、分岐予測によってPCが別の場所に移動したとき)にのみ、再び起動されます。


.. The i-cache does not (currently) support fetching across cache-lines,
.. nor does it support fetching unaligned relative to the superscalar fetch
.. address. [1]_

また、スーパースカラ・フェッチ・アドレスに対して(現在では)アンアラインでのフェッチもサポートしていません。[1]_ 

.. The i-cache does not (currently) support hit-under-miss. If an i-cache
.. miss occurs, the i-cache will not accept any further requests until the
.. miss has been handled. This is less than ideal for scenarios in which
.. the pipeline discovers a branch mispredict and would like to redirect
.. the i-cache to start fetching along the correct path.

命令キャッシュは(現在は)it-under-missをサポートしていません。
命令キャッシュにミスが発生した場合、ミスが処理されるまで命令キャッシュはそれ以上のリクエストを受け付けません。
これは、パイプラインがブランチの分岐予測失敗を検出し、
正しいパスに沿ってフェッチを開始するために命令キャッシュをリダイレクトしたい場合には理想的ではありません。


.. Fetching Compressed Instructions
.. --------------------------------

Compressed命令のフェッチ
------------------------

.. This section describes how the `RISC-V Compressed ISA extension <https://riscv.org/specifications/>`__
.. was implemented in BOOM. The Compressed ISA Extension, or RVC enables smaller, 16
.. bit encodings of common instructions to decrease the static and dynamic
.. code size. "RVC" comes with a number of features that are of particular
.. interest to micro-architects:

このセクションでは、 `RISC-V Compressed ISA extension <https://riscv.org/specifications/>`__ が BOOM にどのように実装されたかを説明します。
Compressed ISA 拡張 (RVC)は、一般的な命令をより小さな16ビットのエンコーディングにすることで、
スタティックおよびダイナミックなコードサイズを小さくすることができます。
"RVC "には、マイクロアーキテクトにとって特に興味深いいくつかの機能が搭載されています。

.. -  32b instructions have no alignment requirement, and may start on a
..    half-word boundary.
.. 
.. -  All 16b instructions map directly into a longer 32b instruction.

- 32ビットの命令はアライメントの制約が無く、16バイトの境界から開始することができる。

- 全ての16ビット命令はより長い32ビットの命令にマッピングすることができる。

.. During the :term:`Front-end` stages, BOOM retrieves a :term:`Fetch Packet` from the
.. i-cache, quickly decodes the instructions for branch
.. prediction, and pushes the :term:`Fetch Packet` into the :term:`Fetch Buffer`. However,
.. doing this brings up a particular set of issues to manage:

BOOM は :term:`フロントエンド` ステージにおいて、命令キャッシュから :term:`フェッチパケット` を取得し、
分岐予測のために命令を素早くデコードして、 :term:`フェッチパケット` を :term:`フェッチバッファ` にプッシュします。
しかし、これを行うには、管理すべき特殊な問題が発生します。

.. -  Increased decoding complexity (e.g., operands can now move around).
.. 
.. -  Finding *where* the instruction begins.
.. 
.. -  Removing ``+4`` assumptions throughout the code base,
..    particularly with branch handling.
.. 
.. -  Unaligned instructions, in particular, running off cache lines and
..    virtual pages.

- デコードの複雑さが増す(オペランドが位置が変わるようになったため)。

- 命令が *どこから* 始まるかを見つける必要がある。

- コードベース全体での ``+4`` の仮定の削除(特に分岐処理で)。

- アンアラインド命令、特にキャッシュラインや仮想ページからの実行。


.. The last point requires some additional "statefulness" in the :term:`Fetch Unit` ,
.. as fetching all of the pieces of an instruction may take multiple cycles.

最後の点については、命令のすべての部分をフェッチするのに複数のサイクルを要することがあるため、 
:term:`フェッチユニット` に追加の "statefulness" が必要になります。

.. The following describes the implementation of RVC in BOOM by describing
.. the lifetime of a instruction.

以下では、命令の寿命を記述することで、BOOMにおけるRVCの実装を説明します。


.. -  The :term:`Front-end` returns :term:`Fetch Packet` s of :term:`fetchWidth<Fetch Width>` \*16 bits wide. This
..    was supported inherently in the BOOM :term:`Front-end` .
.. 
.. -  Maintain statefulness in **F3**, in the cycle where :term:`Fetch Packet` s
..    are dequeued from the i-cache response queue and enqueued onto the
..    :term:`Fetch Buffer` .
.. 
.. -  **F3** tracks the trailing 16b, PC, and instruction boundaries of the
..    last :term:`Fetch Packet` . These bits are combined with the current
..    :term:`Fetch Packet` and expanded to :term:`fetchWidth<Fetch Width>` \*32 bits for enqueuing onto the
..    :term:`Fetch Buffer` . Predecode determines the start address of every
..    instruction in this :term:`Fetch Packet` and masks the :term:`Fetch Packet` for the
..    :term:`Fetch Buffer` .
.. 
.. -  The :term:`Fetch Buffer` now compacts away invalid, or misaligned instructions
..    when storing to its memory.

- :term:`フロントエンド` は :term:`fetchWidth<Fetch Width>` という 16ビット幅の :term:`フェッチパケット` を返します。
  これは、BOOMの :term:`フロントエンド` で本質的にサポートされていました。

- **F3** では、 :term:`フェッチパケット` が命令キャッシュのレスポンスキューからデキューされ、 
  :term:`フェッチバッファ` にエンキューされるサイクルにおいて、ステートフルネスを維持します。

- **F3** は、最後の :term:`フェッチパケット` の末尾の16ビット、PC、および命令境界を追跡します。
  これらのビットは、現在の :term:`フェッチパケット` と結合され、 :term:`fetchWidth<Fetch Width>` に拡張され、
  :term:`フェッチバッファ` にエンキューされます。
  プリデコードは、この :term:`フェッチパケット` 内のすべての命令の開始アドレスを決定し、 
  :term:`フェッチパケット` を :term:`フェッチバッファ` 用にマスクします。

- :term:`フェッチバッファ` は、メモリに格納する際に、無効な命令やミスアラインメントの命令を圧縮します。


.. The following section describes miscellaneous implementation details.

.. -  A challenging problem is dealing with instructions that cross a
..    :term:`Fetch Boundary`. We track these instructions as belonging to the
..    :term:`Fetch Packet` that contains their higher-order 16 bits. We have to
..    be careful when determining the PC of these instructions, by tracking
..    all instructions which were initially misaligned across a :term:`Fetch Boundary` .

.. -  The pipeline must also track whether an instruction was originally
..    16b or 32b, for calculating ``PC+4`` or ``PC+2``.

次のセクションでは、その他の実装の詳細について説明します。

- 困難な問題は、 :term:`フェッチ境界` を越える命令を扱うことです。
  このような命令は、その上位16ビットを含む :term:`フェッチパケット` に属するものとして追跡します。
  これらの命令のPCを決定する際には、最初に :term:`フェッチ境界` を越えてミスアラインされたすべての命令を追跡することで、
  注意しなければなりません。

- また、パイプラインは、命令が元々16ビットだったのか32ビットだったのかを追跡し、``PC+4`` や ``PC+2`` を計算する必要があります。



.. The Fetch Buffer
.. ----------------

フェッチバッファ
----------------

.. :term:`Fetch Packet` s coming from the i-cache are placed into a :term:`Fetch Buffer` . The :term:`Fetch Buffer` helps to decouple the instruction
.. fetch :term:`Front-end` from the execution pipeline in the :term:`Back-end` .

命令キャッシュから来る :term:`フェッチパケット` は :term:`フェッチバッファ` に入れられます。
フェッチバッファは、命令フェッチ(フロントエンド)をバックエンドの実行パイプラインから切り離すのに役立ちます。

.. The :term:`Fetch Buffer` is parameterizable. The number of entries can be
.. changed and whether the buffer is implemented as a “flow-through"
.. queue [2]_ or not can be toggled.

:term:`フェッチバッファ` はパラメータ化が可能です。エントリの数を変更したり、
バッファを「フロースルー」キュー[2]_ として実装するかどうかを切り替えたりすることができます。


.. The Fetch Target Queue
.. ----------------------

フェッチターゲットキュー
------------------------

.. The :term:`Fetch Target Queue` is a queue that holds the PC
.. received from the i-cache and the branch prediction info associated
.. with that address. It holds this information for the pipeline to
.. reference during the executions of its :term:`Micro-Ops (UOPs)<Micro-Op (UOP)>` . It is dequeued by
.. the ROB once an instruction is committed and is updated during pipeline
.. redirection/mispeculation.

:term:`フェッチターゲットキュー` は、命令キャッシュから受け取った PC と、そのアドレスに関連する分岐予測情報を保持するキューです。
この情報は、パイプラインが :term:`Micro-Ops (UOPs)<Micro-Ops (UOP)>` を実行する際に参照するために保持されます。
これは、命令がコミットされるとROBによってデキューされ、パイプラインのリダイレクション/分岐予測失敗時に更新されます。


.. .. [1] This constraint is due to the fact that a cache-line is not stored
..     in a single row of the memory bank, but rather is striped across a
..     single bank to match the refill size coming from the uncore.
..     Fetching unaligned would require modification of the underlying
..     implementation, such as banking the i-cache such that consecutive
..     chunks of a cache-line could be accessed simultaneously.
.. 
.. .. [2] A flow-through queue allows entries being enqueued to be
..     immediately dequeued if the queue is empty and the consumer is
..     requesting (the packet "flows through" instantly).

.. [1] この制約は、キャッシュラインがメモリバンクの1行に格納されるのではなく、
    外部インタフェースからのリフィルサイズに合わせて1つのバンクにストライプ状に格納されていることに起因します。
    アンアラインドで取得するには、キャッシュラインの連続したチャンクに同時にアクセスできるように命令キャッシュをバンク化するなど、
    基本的な実装を変更する必要があります。

.. [2] フロースルーキューは、キューが空でコンシューマが要求している場合に、
    キューイングされているエントリを直ちにデキューイングすることができます(パケットは瞬時に「フロースルー」されます)。
