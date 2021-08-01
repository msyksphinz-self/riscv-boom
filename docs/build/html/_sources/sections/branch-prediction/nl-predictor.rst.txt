The Next-Line Predictor (NLP)
=============================

.. BOOM core's :term:`Front-end` fetches
.. instructions and predicts every cycle where to fetch the next
.. instructions. If a misprediction is detected in BOOM's :term:`Back-end`, or
.. BOOM’s own :term:`Backing Predictor (BPD)` wants to redirect the pipeline in a
.. different direction, a request is sent to the :term:`Front-end` and it begins
.. fetching along a new instruction path.

BOOMコアの :term:`フロントエンド` は命令をフェッチし、次の命令をどこでフェッチするかをサイクルごとに予測します。
BOOMの :term:`バックエンド` で予測の間違いが検出されたり、
BOOM自身の :term:`Backing Predictor (BPD)` がパイプラインを別の方向に向かわせたいと考えた場合、
:term:`フロントエンド` にリクエストが送られ、新しい命令パスに沿ってフェッチを開始します。

.. The :term:`Next-Line Predictor (NLP)` takes in the current PC being used to
.. fetch instructions (the :term:`Fetch PC`) and predicts combinationally
.. where the next instructions should be fetched for the next cycle. If
.. predicted correctly, there are no pipeline bubbles.

Next-Line Predictor (NLP)`は、命令をフェッチするために使用されている現在のPC（ :term:`Fetch PC` ）を取り込み、次のサイクルでどこに次の命令をフェッチすべきかを組み合わせて予測します。予測が正しければ、パイプライン・バブルは発生しません。


.. The :term:`NLP<Next-Line Predictor (NLP)>` is an amalgamation of a fully-associative **Branch
.. Target Buffer (BTB)**, :term:`Bi-Modal Table (BIM)` and a **Return Address Stack (RAS)** which work together
.. to make a fast, but reasonably accurate prediction.

:term:`NLP<Next-Line Predictor (NLP)>` は、完全に関連付けられた **BTB (Branch Target Buffer)**、 :term:`Bi-Modal Table (BIM)`、 **RAS (Return Address Stack)** を組み合わせたもので、高速かつ適度に正確な予測を行うことができます。

.. NLP Predictions
.. ---------------

NLP予測
-------

.. The :term:`Fetch PC` first performs a tag match to find a uniquely
.. matching BTB entry. If a hit occurs, the BTB entry will make a
.. prediction in concert with the RAS as to whether there is a
.. branch, jump, or return found in the :term:`Fetch Packet` and which
.. instruction in the :term:`Fetch Packet` is to blame. The :term:`BIM<Bi-Modal Table (BIM)>` is used to
.. determine if that prediction made was a branch taken or not taken.
.. The BTB entry also contains a predicted PC target, which is used
.. as the :term:`Fetch PC` on the next cycle.

:term:`フェッチPC` は、まずタグマッチを実行して、一意に一致するBTBエントリを見つけます。
ヒットした場合、BTBエントリはRASと連携して、 :term:`フェッチパケット` で見つかった分岐、ジャンプ、リターンがあるかどうか、 
:term:`フェッチパケットt` のどの命令が原因かを予測します。
`BIM<Bi-Modal Table (BIM)>` は、その予測されたものが、分岐が取られたものか、取られなかったものかを判断するために使用されます。
BTBのエントリーには予測されたPCターゲットも含まれており、これは次のサイクルで :term:`フェッチPC` として使用されます。


.. _nlp-predictor-unit:
.. figure:: /figures/btb.png
    :scale: 35 %
    :align: center
    :alt: The :term:`Next-Line Predictor`

..    The :term:`Next-Line Predictor (NLP)` Unit. The :term:`Fetch PC` scans the BTB’s "PC tags" for a match.
..    If a match is found (and the entry is valid), the :term:`Bi-Modal Table (BIM)` and RAS are consulted for the final verdict. If the entry
..    is a "ret" (return instruction), then the target comes from the RAS. If the entry is a unconditional "jmp"
..    (jump instruction), then the :term:`BIM<Bi-Modal Table (BIM)>` is not consulted. The "bidx", or branch index, marks which instruction
..    in a superscalar :term:`Fetch Packet` is the cause of the control flow prediction. This is necessary to mask off the
..    other instructions in the :term:`Fetch Packet` that come over the taken branch

	:term:`Next-Line Predictor (NLP)`ユニット。:term:`フェッチPC` は、BTBの「PCタグ」をスキャンしてマッチするものを探します。
	一致するものが見つかった場合（そしてエントリーが有効な場合）、 :term:`Bi-Modal Table (BIM)`とRASを参照して最終的な判断を下します。
	エントリが "ret"(return instruction)であれば、ターゲットはRASから来ます。
	エントリが無条件の "jmp"（ジャンプ命令）の場合は、 :term:`BIM<Bi-Modal Table (BIM)>` は参照されません。
	"bidx"、つまりブランチインデックスは、スーパースカラの :term:`フェッチパケット` の中のどの命令が制御フロー予測の原因であるかを示します。
	"bidx"は、スーパースカラの :term:`フェッチパケット` の中のどの命令が制御フロー予測の原因であるかを示すものです。

.. The hysteresis bits in the :term:`BIM<Bi-Modal Table (BIM)>` are only used
.. on a BTB entry *hit* and if the predicting instruction is a branch.

:term:`BIM<Bi-Modal Table (BIM)>` のヒステリシスビットは、BTBエントリ *hit* のときと、
予測される命令が分岐のときにのみ使用されます。

.. If the BTB entry contains a *return* instruction, the RAS stack is
.. used to provide the predicted return PC as the next :term:`Fetch PC`. The
.. actual RAS management (of when to or the stack) is governed externally.

BTBエントリに *return* 命令が含まれている場合、RASスタックは予測されたリターンPCを次の :term:`フェッチ` として提供するために使用されます。
実際のRASの管理(いつスタックを使用するか)は外部で管理されます。

.. For area-efficiency, the high-order bits of the PC tags and PC targets
.. are stored in a compressed file.

面積効率を上げるために、PCタグとPCターゲットの高次ビットは圧縮ファイルに格納されます。

.. NLP Updates
.. -----------

NLPの更新
---------

.. Each branch passed down the pipeline remembers not only its own PC, but
.. also its :term:`Fetch PC` (the PC of the head instruction of its :term:`Fetch Packet` ). [2]_

パイプラインを通過した各分岐は、自分のPCだけでなく、その :term:`フェッチPC` ( :term:`フェッチパケット` の先頭命令のPC) も記憶しています。[2]_

.. BTB Updates
.. ^^^^^^^^^^^

BTBの更新
^^^^^^^^^

.. The BTB is updated *only* when the :term:`Front-end` is redirected to
.. *take* a branch or jump by either the :term:`Branch Unit` (in the
.. Execute stage) or the :term:`BPD<Backing Predictor (BPD)>` (later in the **Fetch** stages). [3]_

BTBは、 :term:`フロントエンド` が、 :term:`分岐ユニット` (実行ステージ)または :term:`BPD<Backing Predictor(BPD)>`
(**Fetch** ステージの後半)によって、分岐またはジャンプを取るようにリダイレクトされたときにのみ、 更新されます。[3]_

.. If there is no BTB entry corresponding to the taken branch or jump, an
.. new entry is allocated for it.

分岐成立した分岐命令やジャンプに対応するBTBエントリがない場合は、
新しいエントリが割り当てられます。

.. RAS Updates
.. ^^^^^^^^^^^

RASの更新
^^^^^^^^^

.. The RAS is updated during the Fetch stages once the
.. instructions in the :term:`Fetch Packet` have been decoded. If the taken
.. instruction is a call [4]_ , the return address is pushed onto the RAS. If
.. the taken instruction is a return, then the RAS is popped.

RASは、フェッチ段階で、 :term:`フェッチパケット` に含まれる命令がデコードされると更新されます。
命令が関数呼び出し [4]_ の場合は、リターンアドレスがRASにプッシュされます。
命令がリターンであれば、RASはポップされます。

.. Superscalar Predictions
.. ^^^^^^^^^^^^^^^^^^^^^^^

スーパスカラの分岐予測
^^^^^^^^^^^^^^^^^^^^^^

.. When the :term:`NLP<Next-Line Predictor (NLP)>` makes a prediction, it is actually using the BTB to tag
.. match against the predicted branch’s :term:`Fetch PC`, and not the PC of
.. the branch itself. The :term:`NLP<Next-Line Predictor (NLP)>` must predict across the entire :term:`Fetch Packet`
.. which of the many possible branches will be the dominating
.. branch that redirects the PC. For this reason, we use a given branch’s
.. :term:`Fetch PC` rather than its own PC in the BTB tag match. [5]_

:term:`NLP<Next-Line Predictor (NLP)>` が予測を行う場合、実際にはBTBを使用して、
予測されたブランチの :term:`フェッチPC` に対してタグマッチを行っており、分岐命令自体のPCではありません。
:term:`NLP<Next-Line Predictor (NLP)>` は、多くの可能性のある分岐命令のうち、
どの命令がPCをリダイレクトする支配的な分岐になるかを、 :term:`フェッチパケット` 全体にわたって予測しなければなりません。
このような理由から，BTBタグマッチでは，あるブランチのPCではなく，そのブランチの :term:`フェッチPC` を使用します。[5]_

.. .. [2] In reality, only the very lowest bits must be saved, as the
..     higher-order bits will be the same.

.. [2] 実際には、上位のビットは同じであるため、最下位のビットのみを保存する必要があります。

.. .. [3] The BTB relies on a little cleverness - when redirecting the
..     PC on a misprediction, this new :term:`Fetch PC` is the same as the
..     update PC that needs to be written into a new BTB entry’s
..     target PC field. This "coincidence" allows the PC compression
..     table to use a single search port - it is simultaneously reading the
..     table for the next prediction while also seeing if the new Update
..     PC already has the proper high-order bits allocated for it.

.. [3] BTBでは、ちょっとした工夫をしています。分岐予測失敗によりPCをリダイレクトする際、
    この新しい :term:`フェッチPC` は、新しいBTBエントリのターゲットPCフィールドに書き込まれる必要のあるアップデートPCと同じものです。
    この「偶然の一致」により、PC圧縮テーブルは単一の検索ポートを使用することができます。
    つまり、次の予測のためにテーブルを読むと同時に、
    新しいアップデートPCに適切な高次ビットがすでに割り当てられているかどうかを確認します。


.. .. [4] While RISC-V does not have a dedicated call instruction, it can be
..     inferred by checking for a JAL or JALR instruction with a writeback
..     destination to x1 (aka, the return address register).

.. [4] RISC-Vには専用のコール命令はありませんが、x1(別名：リターンアドレスレジスタ)への
    ライトバック先を持つJALまたはJALR命令をチェックすることで、コールを推測することができます。


.. .. [5] Each BTB entry corresponds to a single :term:`Fetch PC`, but it is
..     helping to predict across an entire :term:`Fetch Packet`. However, the
..     BTB entry can only store meta-data and target-data on a single
..     control-flow instruction. While there are certainly pathological
..     cases that can harm performance with this design, the assumption is
..     that there is a correlation between which branch in a :term:`Fetch Packet`
..     is the dominating branch relative to the :term:`Fetch PC`,
..     and - at least for narrow fetch designs - evaluations of this design
..     has shown it is very complexity-friendly with no noticeable loss in
..     performance. Some other designs instead choose to provide a whole
..     bank of BTBs for each possible instruction in the :term:`Fetch Packet` .

.. [5] 各BTBエントリは、単一の :term:`フェッチPC` に対応していますが、 :term:`フェッチパケット` 全体を予測するのに役立ちます。
    しかし、BTBエントリは、1つの制御フロー命令上でメタデータとターゲットデータしか格納できません。
    この設計では、確かに性能に悪影響を及ぼす病的なケースもありますが、
    :term:`フェッチパケット` 内のどのブランチが :term: `フェッチPC` に対して支配的なブランチであるかという相関関係があることが前提となっており、
    少なくともナローフェッチの設計においては、この設計の評価では、性能に目立った損失がなく、
    非常に複雑になりにくいことが示されています。
    他のいくつかのデザインでは、代わりに :term:`フェッチパケット` の各命令に対してBTBのバンク全体を提供することを選択しています。
