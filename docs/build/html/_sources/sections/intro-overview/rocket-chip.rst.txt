.. Rocket Chip SoC Generator
.. =========================

Rocket Chip SoCジェネレータ
===========================

.. _boom-chip:
.. figure:: /figures/chip.png
    :alt: BOOM Chip

    A single-core "BOOM-chip", with no L2 last-level cache

.. As BOOM is just a core, an entire SoC infrastructure must be provided.
.. BOOM was developed to use the open-source `Rocket Chip SoC generator <https://github.com/chipsalliance/rocket-chip>`__.
.. The **Rocket Chip generator** can instantiate a wide range of SoC designs, including cache-coherent
.. multi-tile designs, cores with and without accelerators, and chips with or without a last-level shared cache.
.. It comes bundled with a 5-stage in-order core, called **Rocket**, by default.
.. BOOM uses the Rocket Chip infrastructure to instantiate it's core/tile complex (tile is a core, L1D/I$, and PTW) instead of a
.. Rocket tile.

BOOMは単なるコアなので、SoCのインフラ全体を提供する必要があります。
BOOMは、オープンソースの `Rocket Chip SoC generator <https://github.com/chipsalliance/rocket-chip>`__ を使用するために開発されました。
**Rocket Chip ジェネレータ** は、キャッシュコヒーレントなマルチタイルデザイン、アクセラレータの有無にかかわらずコア、
最終レベルの共有キャッシュの有無にかかわらずチップなど、幅広いSoCデザインをインスタンス化することができます。
Rocket Chip SoC ジェネレータには、デフォルトで **Rocket**と 呼ばれる5段のインオーダーコアがバンドルされています。
BOOMはRocketタイルではなく、Rocketチップのインフラを利用して、
コアとタイルの複合体(タイルはコア、L1D/I$、PTW)をインスタンス化しています。

.. To get more information, please visit the `Chipyard Rocket Chip documentation <https://chipyard.readthedocs.io/en/dev/Generators/Rocket-Chip.html>`__.

より詳細な情報は、 `Chipyard Rocket Chip documentation <https://chipyard.readthedocs.io/en/dev/Generators/Rocket-Chip.html>`__ を参照してください。

.. The Rocket Core - a Library of Processor Components!
.. ----------------------------------------------------

Rocketコア - プロセッサコンポーネントのライブラリ!
--------------------------------------------------

.. From BOOM's point of view, the Rocket core can be thought of as a
.. "Library of Processor Components." There are a number of modules created
.. for Rocket that are also used by BOOM - the functional units, the
.. caches, the translation look-aside buffers (TLBs), the page table walker (PTW), and
.. more. Throughout this document you will find references to these
.. Rocket components and descriptions on how they fit into BOOM.

BOOM の視点から見ると、Rocket コアは "プロセッサコンポーネントのライブラリ" と考えることができます。
機能ユニット、キャッシュ、TLB(Translation Look-aside Buffers)、PTW(Page Table Walker)など、
BOOMでも使用されるRocket用のモジュールが多数あります。
このドキュメントでは、これらのRocketコンポーネントを参照し、それらがどのようにBOOMに組み込まれているかを説明しています。

.. To get more information about the Rocket core, please visit the `Chipyard Rocket Core documentation <https://chipyard.readthedocs.io/en/dev/Generators/Rocket.html>`__.

Rocketコアの詳細については、 `Chipyard Rocket Core documentation <https://chipyard.readthedocs.io/en/dev/Generators/Rocket.html>`__ をご覧ください。


.. .. Note:: Both Chipyard links point to the ``dev`` documentation of Chipyard to get the most recent documentation changes.

.. Note:: どちらのChipyardのリンクも、最新のドキュメントの変更点を得るために、Chipyardの ``dev`` ドキュメントを指しています。
