.. Branch Prediction
.. =================

分岐予測
========

.. _front-end-bpu:
.. figure:: /figures/front-end.svg
    :alt: BOOM Front-end

    The BOOM Front-end

.. This chapter discusses how BOOM predicts branches and then resolves
.. these predictions.

この章では、BOOMがどのように分岐を予測し、その予測を解決するかについて説明します。

.. BOOM uses two levels of branch prediction - a fast :term:`Next-Line Predictor (NLP)`
.. and a slower but more complex :term:`Backing Predictor (BPD)` [1]_. In this case,
.. the :term:`NLP<Next-Line Predictor (NLP)>` is a Branch Target Buffer and the :term:`BPD<Backing Predictor (BPD)>`
.. is a more complicated structure like a GShare predictor.

BOOMは、高速な :term:`Next-Line Predictor (NLP)` と、より遅いが複雑な :term:`Backing Predictor (BPD)` という2つのレベルの分岐予測を使用します [1]_ 。
この場合、 :term:`NLP<Next-Line Predictor (NLP)>` は、Branch Target Bufferで、 :term:`BPD<Backing Predictor (BPD)>` は、GShare Predictorのようなより複雑な構造です。


.. toctree::
    :maxdepth: 2
    :caption: Branch Prediction:

    nl-predictor
    backing-predictor

.. .. [1] Unfortunately, the terminology in the literature gets a bit
..     muddled here in what to call different types and levels of branch
..     predictor. Literature has references to different structures; "micro-BTB" versus "BTB", "NLP" versus "BHT",
..     and "cache-line predictor" versus "overriding predictor". Although
..     the Rocket core calls its own predictor the "BTB", BOOM
..     refers to it as the :term:`Next-Line Predictor (NLP)` , to denote
..     that it is a combinational predictor that provides single-cycle
..     predictions for fetching "the next line", and the Rocket BTB
..     encompasses far more complexity than just a "branch target buffer"
..     structure. Likewise, the name :term:`Backing Predictor (BPD)` was chosen to avoid
..     being overly descriptive of the internal design (is it a simple BHT?
..     Is it tagged? Does it override the :term:`NLP<Next-Line Predictor (NLP)>` ?) while being accurate. If you have recommendations for
..     better names, feel free to reach out!

.. [1] 残念なことに、異なるタイプやレベルの分岐予測器を何と呼ぶかについて、文献の用語が少し混乱しています。
    文献には、"マイクロBTB"と"BTB"、"NLP"と"BHT"、"キャッシュラインプレディクター"と"オーバーライドプレディクター"など、さまざまな構造に関する記述があります。
    Rocketコアは独自の予測器を"BTB"と呼んでいますが、BOOMでは :term:`Next-Line Predictor (NLP)` と呼び、
    「次の行」をフェッチするためのシングルサイクル予測を提供する組み合わせ予測器であることを示しています。
    また、Rocket BTBは単なる"ブランチターゲットバッファ"構造よりもはるかに複雑な構造を含んでいます。
    同様に、 :term:`Backing Predictor (BPD)`という名称は、内部設計を過度に説明しないようにするために選ばれました
    (それは単純なBHTなのか？タグ付けされているのか？ `NLP<Next-Line Predictor (NLP)>` をオーバーライドするのか？)
    「もっといい名前があるよ！」という方は、お気軽にご連絡ください。
