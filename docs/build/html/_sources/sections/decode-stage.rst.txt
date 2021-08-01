.. The Decode Stage
.. ================

デコードステージ
================

.. The **Decode** stage takes instructions from the :term:`Fetch Buffer`, decodes them,
.. and allocates the necessary resources as required by each instruction.
.. The **Decode** stage will stall as needed if not all resources are available.

**デコード** ステージは、 :term:`Fetch Buffer` から命令を受け取り、デコードし、各命令に必要なリソースを割り当てます。
**デコード** ステージは、すべてのリソースが利用可能でない場合、必要に応じてストールします。

.. RVC Changes
.. -----------

RVCの変更
---------

.. RVC decode is performed by expanding RVC instructions using Rocket's
.. ``RVCExpander``. This does not change normal functionality of the **Decode** stage.

RVCのデコードは、Rocketの ``RVCExpander`` を使ってRVC命令を展開することで行われます。
これは **デコード** ステージの通常の機能を変更するものではありません。
