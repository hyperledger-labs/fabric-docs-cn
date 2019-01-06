Endorsement policies - 背书策略
====================

Every chaincode has an endorsement policy which specifies the set of peers on
a channel that must execute chaincode and endorse the execution results in
order for the transaction to be considered valid. These endorsement policies
define the organizations (through their peers) who must "endorse" (i.e., approve
of) the execution of a proposal.

每一个链码都有一个背书策略，用于指定一些执行和背书执行结果的节点，以此来验证一个交易的有效性。这些背书策略定义了组织中（其中的节点）哪些必须“背书”（或者说，同意）一个提案的执行。

.. note :: Recall that **state**, represented by key-value pairs, is separate
           from blockchain data. For more on this, check out our :doc:`ledger/ledger`
           documentation.
           
           重申一下 **状态** 的概念，和区块数据不同，它是由一个键值对表示的。详细信息参考文档 :doc:`ledger/ledger`

As part of the transaction validation step performed by the peers, each validating
peer checks to make sure that the transaction contains the appropriate **number**
of endorsements and that they are from the expected sources (both of these are
specified in the endorsement policy). The endorsements are also checked to make
sure they're valid (i.e., that they are valid signatures from valid certificates).

作为节点验证交易步骤的一部分，每一个验证节点都会进行检查，以确保交易包含适当的背书数量并且都来源于期望的地方（这些都是在背书策略中定义的）。也会检查背书是否正确（例如，会验证签名是否来源于有效的证书）。

Two ways to require endorsement - 请求背书的两种方法
-------------------------------

By default, endorsement policies are specified for a channel's chaincode at
instantiation or upgrade time (that is, one endorsement policy covers all of the
state associated with a chaincode).

一般地，在通道链码初始化或者更新的时候会指定背书策略（也就是说，一个背书策略覆盖一个链码中所有相关的状态）。

However, there are cases where it may be necessary for a particular state (a
particular key-value pair, in other words) to have a different endorsement policy.
This **state-based endorsement** allows the default chaincode-level endorsement
policies to be overridden by a different policy for the specified keys.

然而，有些情况下一些特殊的状态（或者说，一个特殊的键值对）可能会需要不同的背书策略。这里的 **基于状态的背书** 允许一个指定键的背书策略覆盖默认的背书策略。

To illustrate the circumstances in which these two types of endorsement policies
might be used, consider a channel on which cars are being exchanged. The "creation"
--- also known as "issuance" -- of a car as an asset that can be traded (putting
the key-value pair that represents it into the world state, in other words) would
have to satisfy the chaincode-level endorsement policy. To see how to set a
chaincode-level endorsement policy, check out the section below.

为了解释使用这两种类型背书策略的情况，想像一个汽车置换的通道。“创建” ——或者说是“发布”——一辆可以交易的汽车作为资产（换句话说，就是在世界状态中加入一个键值对），需要满足链码级的背书策略。在后边的段落中你可以找到怎么设置链码级背书策略。

If the car requires a specific endorsement policy, it can be defined either when
the car is created or afterwards. There are a number of reasons why it might
be necessary or preferable to set a state-specific endorsement policy. The car
might have historical importance or value that makes it necessary to have the
endorsement of a licensed appraiser. Also, the owner of the car (if they're a
member of the channel) might also want to ensure that their peer signs off on a
transaction. In both cases, **an endorsement policy is required for a particular
asset that is different from the default endorsement policies for the other
assets associated with that chaincode.**

如果一辆车需要指定一个背书策略，在车辆被创建前和创建后都可以设定。有很多种理由解释为什么指定一个特定状态的背书策略是必要的。这辆车可能具有历史重要性或价值，因此有必要得到持有执照的评估师的认可。车辆的主人也想确认评估师的节点在交易上签了名（如果他们在同一个通道）。在这两种情况下， **一个特殊资产需要一个和其相关链码上的其他资产的背书策略不同的背书策略。** 

We'll show you how to define a state-based endorsement policy in a subsequent
section. But first, let's see how we set a chaincode-level endorsement policy.

我们将在下一节中展示如何定义一个基于状态的背书策略。但是在那之前，我们先看一下如何设置一个链码级背书策略。

Setting chaincode-level endorsement policies - 设置链码级背书策略
--------------------------------------------

Chaincode-level endorsement policies can be specified at instantiate time using
either the SDK (for some sample code on how to do this, click
`here <https://github.com/hyperledger/fabric-sdk-node/blob/f8ffa90dc1b61a4a60a6fa25de760c647587b788/test/integration/e2e/e2eUtils.js#L178>`_)
or in the peer CLI using the ``-P`` switch followed by the policy.

链码级背书策略在链码初始化的时候指定，通过 SDK (有一些例子演示了怎么做，点击 
`here <https://github.com/hyperledger/fabric-sdk-node/blob/f8ffa90dc1b61a4a60a6fa25de760c647587b788/test/integration/e2e/e2eUtils.js#L178>`_)或者 peer CLI 命令后边 ``-P`` 加背书策略都可以。

.. note:: Don't worry about the policy syntax (``'Org1.member'``, et all) right
          now. We'll talk more about the syntax in the next section.

          现在不要担心这里的策略语法 （``'Org1.member'`` 等等），我们会在下一部分介绍相关语法。

For example:

例如：

::

    peer chaincode instantiate -C <channelid> -n mycc -P "AND('Org1.member', 'Org2.member')"

This command deploys chaincode ``mycc`` ("my chaincode") with the policy
``AND('Org1.member', 'Org2.member')`` which would require that a member of both
Org1 and Org2 sign the transaction.

这条命令部署了一个名为 ``mycc`` ("my chaincode") 的链码，并设置了一个策略 ``AND('Org1.member', 'Org2.member')`` ，其含义为交易将请求 Org1 和 Org2 的成员签名。

Notice that, if the identity classification is enabled (see :doc:`msp`),
one can use the ``PEER`` role to restrict endorsement to only peers.

注意，如果身份类别启用的话 （参考 :doc:`msp`），你就可以使用 ``PEER`` 角色来限制只能 peer 背书。

For example:

例如：

::

    peer chaincode instantiate -C <channelid> -n mycc -P "AND('Org1.peer', 'Org2.peer')"

A new organization added to the channel after instantiation can query a chaincode
(provided the query has appropriate authorization as defined by channel policies
and any application level checks enforced by the chaincode) but will not be able
to execute or endorse the chaincode. The endorsement policy needs to be modified
to allow transactions to be committed with endorsements from the new organization.

在初始化后新加入的组织可以查询链码（是否有查询权限是链码的通道策略和应用层检查定义的）但是不能执行和背书链码。背书策略需要更改以允许提交的交易可以被新组织背书。

.. note:: if not specified at instantiation time, the endorsement policy
          defaults to "any member of the organizations in the channel".
          For example, a channel with "Org1" and "Org2" would have a default
          endorsement policy of "OR('Org1.member', 'Org2.member')".
          
          如果在初始化的时候没有指定，默认背书策略就是“通道中的组织的所有成员”。例如，在一个有 “Org1” 和 “Org2” 的通道中，默认背书策略就是 "OR('Org1.member', 'Org2.member')" 。

Endorsement policy syntax - 背书策略语法
~~~~~~~~~~~~~~~~~~~~~~~~~

As you can see above, policies are expressed in terms of principals
("principals" are identities matched to a role). Principals are described as
``'MSP.ROLE'``, where ``MSP`` represents the required MSP ID and ``ROLE``
represents one of the four accepted roles: ``member``, ``admin``, ``client``, and
``peer``.

就像你上边看到的，策略就是一组规则（ “规则” 就是角色匹配的身份）。规则描述为 ``'MSP.ROLE'`` ， 这里 ``MSP`` 代表MSP ID ， ``ROLE`` 代表四个可接受角色中的一个： ``member`` ， ``admin`` ， ``client`` ，和 ``peer`` 。


Here are a few examples of valid principals:

这是一些有效规则的例子：

  - ``'Org0.admin'``: any administrator of the ``Org0`` MSP
  - ``'Org1.member'``: any member of the ``Org1`` MSP
  - ``'Org1.client'``: any client of the ``Org1`` MSP
  - ``'Org1.peer'``: any peer of the ``Org1`` MSP

  - ``'Org0.admin'`` ： ``Org0`` 中任意管理员 
  - ``'Org1.member'`` ： ``Org1`` 中任意成员
  - ``'Org1.client'`` ： ``Org1`` 中任意客户端
  - ``'Org1.peer`` ： ``Org1`` 中任意节点

The syntax of the language is:

语法如下：

``EXPR(E[, E...])``

Where ``EXPR`` is either ``AND``, ``OR``, or ``OutOf``, and ``E`` is either a
principal (with the syntax described above) or another nested call to ``EXPR``.

``EXPR`` 是 ``AND`` ， ``OR`` ， 或 ``OutOf`` 其中之一， ``E`` 是一个规则（语法如上）或者另外一个嵌套的 ``EXPR`` 。

For example:

例如：

  - ``AND('Org1.member', 'Org2.member', 'Org3.member')`` requests one signature
    from each of the three principals.
  - ``OR('Org1.member', 'Org2.member')`` requests one signature from either one
    of the two principals.
  - ``OR('Org1.member', AND('Org2.member', 'Org3.member'))`` requests either one
    signature from a member of the ``Org1`` MSP or one signature from a member
    of the ``Org2`` MSP and one signature from a member of the ``Org3`` MSP.
  - ``OutOf(1, 'Org1.member', 'Org2.member')``, which resolves to the same thing
    as ``OR('Org1.member', 'Org2.member')``.
  - Similarly, ``OutOf(2, 'Org1.member', 'B.member')`` is equivalent to
    ``AND('Org1.member', 'Org2.member')``.

  - ``AND('Org1.member', 'Org2.member', 'Org3.member')`` 请求这三个规则中的每一个签名。
  - ``OR('Org1.member', 'Org2.member')`` 请求这两个规则中的任一个。
  - ``OR('Org1.member', AND('Org2.member', 'Org3.member'))`` 请求 ``Org1`` 中的一个成员的签名或者请求 ``Org2`` 和 ``Org3`` 中各一个成员的签名。 
  - ``OutOf(1, 'Org1.member', 'Org2.member')`` ， 和 ``OR('Org1.member', 'Org2.member')`` 一样。
  - 类似地， ``OutOf(2, 'Org1.member', 'B.member')`` 和 ``AND('Org1.member', 'Org2.member')`` 一样。


.. _key-level-endorsement:

Setting key-level endorsement policies - 设置键级背书策略
--------------------------------------

Setting regular chaincode-level endorsement policies is tied to the lifecycle of
the corresponding chaincode. They can only be set or modified when instantiating
or upgrading the corresponding chaincode on a channel.

设置一个规则的链码级背书规则，是绑定在链码的生命周期内的。他们只能在链码所在通道初始化或者升级的时候更改。

In contrast, key-level endorsement policies can be set and modified in a more
granular fashion from within a chaincode. The modification is part of the
read-write set of a regular transaction.

相反，键级的背书策略可以在链码中以更细粒度的方式设置和修改。修改是常规交易的读写集的一部分。

The shim API provides the following functions to set and retrieve an endorsement
policy for/from a regular key.

shim API 提供了下边的函数来设置和检索键上的背书策略。

.. note:: ``ep`` below stands for the "endorsement policy", which can be expressed
          either by using the same syntax described above or by using the
          convenience function described below. Either method will generate a
          binary version of the endorsement policy that can be consumed by the
          basic shim API.
          
          下边的 ``ep`` 代表 "endorsement policy" ，它可以使用上边的语法定义，也可以用下边描述的函数。两种方式都会生成背书策略的二进制版本用来被基础 shim API 使用。

.. code-block:: Go

    SetStateValidationParameter(key string, ep []byte) error
    GetStateValidationParameter(key string) ([]byte, error)

For keys that are part of :doc:`private-data/private-data` in a collection the
following functions apply:

对于集合中属于 :doc:`private-data/private-data` 的键，应用一下函数：

.. code-block:: Go

    SetPrivateDataValidationParameter(collection, key string, ep []byte) error
    GetPrivateDataValidationParameter(collection, key string) ([]byte, error)

To help set endorsement policies and marshal them into validation
parameter byte arrays, the shim provides convenience functions that allow the
chaincode developer to deal with endorsement policies in terms of the MSP
identifiers of organizations:

为了帮助设置背书策略和将他们封送到验证参数字节数组，shim 提供了方便的函数，让链码开发者根据组织中的MSP表示来处理背书策略。

.. code-block:: Go

    type KeyEndorsementPolicy interface {
        // Policy returns the endorsement policy as bytes
        Policy() ([]byte, error)

        // AddOrgs adds the specified orgs to the list of orgs that are required
        // to endorse
        AddOrgs(roleType RoleType, organizations ...string) error

        // DelOrgs delete the specified channel orgs from the existing key-level endorsement
        // policy for this KVS key. If any org is not present, an error will be returned.
        DelOrgs([]string) error

        // DelAllOrgs removes any key-level endorsement policy from this KVS key.
        DelAllOrgs() error

        // ListOrgs returns an array of channel orgs that are required to endorse changes
        ListOrgs() ([]string, error)
    }

For example, to set an endorsement policy for a key where two specific orgs are
required to endorse the key change, pass both org ``MSPIDs`` to ``AddOrgs()``,
and then call ``Policy()`` to construct the endorsement policy byte array that
can be passed to ``SetStateValidationParameter()``.

例如，为一个键设置一个需要两个指定组织背书才能变更的背书策略，传递两个组织的 ``MSPIDs`` 给 ``AddOrgs()`` ，然后调用 ``Policy()`` 来构造要传递给 ``SetStateValidationParameter()`` 的背书规则字节数组。

Validation - 验证
----------

At commit time, setting a value of a key is no different from setting the
endorsement policy of a key --- both update the state of the key and are
validated based on the same rules.

在提交阶段，设置一个键的背书策略和设置键的值没有区别 —— 都是更新键的状态和根据同样的规则验证。

+---------------------+-----------------------------+--------------------------+
| Validation          | no validation parameter set | validation parameter set |
+=====================+=============================+==========================+
| modify value        | check chaincode ep          | check key-level ep       |
+---------------------+-----------------------------+--------------------------+
| modify key-level ep | check chaincode ep          | check key-level ep       |
+---------------------+-----------------------------+--------------------------+

As we discussed above, if a key is modified and no key-level endorsement policy
is present, the chaincode-level endorsement policy applies by default. This is
also true when a key-level endorsement policy is set for a key for the first time
--- the new key-level endorsement policy must first be endorsed according to the
pre-existing chaincode-level endorsement policy.

就像我们上边讨论的，如果一个键更改了而且没有存在键级的背书策略，默认使用链码级的背书策略。第一次设置键级背书策略也是一样 —— 新的键级的背书策略必须先经过已有的链码级背书策略背书。

If a key is modified and a key-level endorsement policy is present, the key-level
endorsement policy overrides the chaincode-level endorsement policy. In practice,
this means that the key-level endorsement policy can be either less restrictive
or more restrictive than the chaincode-level endorsement policy. Because the
chaincode-level endorsement policy must be satisfied in order to set a key-level
endorsement policy for the first time, no trust assumptions have been violated.

如果一个键被更改了，而且存在键级背书策略，键级背书策略就覆盖链码级背书策略。实践中，这意味着键级背书策略可以比链码级背书策略限制更少，也可以更严格。因为首次设置键级背书策略必须经过链码级背书策略认可，因此没有违反信任假设。

If a key's endorsement policy is removed (set to nil), the chaincode-level
endorsement policy becomes the default again.

如果一个键的背书策略移除了（被设置为 nil），链码级背书策略就有成了默认值。

If a transaction modifies multiple keys with different associated key-level
endorsement policies, all of these policies need to be satisfied in order
for the transaction to be valid.

如果一个交易改变了多个关联不同背书策略的键，所有的策略都满足，这个交易才算有效。

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
