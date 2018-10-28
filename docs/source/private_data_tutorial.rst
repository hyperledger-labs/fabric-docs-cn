
Using Private Data in Fabric-在Fabric中使用私有数据
============================

This tutorial will demonstrate the use of collections to provide storage
and retrieval of private data on the blockchain network for authorized peers
of organizations.

本教程将演示在区块链网络上，组织中授权过的peer节点如何使用集合提供私有数据的存储和检索。

The information in this tutorial assumes knowledge of private data
stores and their use cases. For more information, check out :doc:`private-data/private-data`.

本教程中的信息假定您了解私有数据存储及其用例。 有关更多信息，请查看：doc：`private-data / private-data`。

The tutorial will take you through the following steps for practice defining,
configuring and using private data with Fabric:

本教程将指导您完成以下步骤，以便在Fabric中定义，配置和使用私有数据：

#. :ref:`pd-build-json`
#. :ref:`pd-read-write-private-data`
#. :ref:`pd-install-instantiate_cc`
#. :ref:`pd-store-private-data`
#. :ref:`pd-query-authorized`
#. :ref:`pd-query-unauthorized`
#. :ref:`pd-purge`
#. :ref:`pd-indexes`

This tutorial will use the `marbles private data sample <https://github.com/hyperledger/fabric-samples/tree/master/chaincode/marbles02_private>`__
--- running on the Building Your First Network (BYFN) tutorial network --- to
demonstrate how to create, deploy, and use a collection of private data.
The marbles private data sample will be deployed to the :doc:`build_network`
(BYFN) tutorial network. You should have completed the task :doc:`install`;
however, running the BYFN tutorial is not a prerequisite for this tutorial.
Instead the necessary commands are provided throughout this tutorial to use the
network. We will describe what is happening at each step, making it possible to
understand the tutorial without actually running the sample.

被教程将使用 `marbles private data sample <https://github.com/hyperledger/fabric-samples/tree/master/chaincode/marbles02_private>`__运行在构建你的第一个网络（BYFN）的教程网络上--- 去演示如何创建，部署和使用私有数据集。marbles私有数据例子将会被部署到:doc:`build_network`
(BYFN)示例网络上。你应该先完成安装任务 :doc:`install`;然而运行BYFN教程不是本教程的前提，相反，本教程提供了必要的命令来使用网络。我们将描述在每一步发生了什么，使得在没有实际运行示例的情况下理解教程成为可能。

.. _pd-build-json:

Build a collection definition JSON file-构件集合定义在JSON文件中
------------------------------------------

The first step in privatizing data on a channel is to build a collection
definition which defines access to the private data.

将通道上的数据私有化的第一步是构建一个集合定义，用于定义对私有数据的访问。

The collection definition describes who can persist data, how many peers the
data is distributed to, how many peers are required to disseminate the private
data, and how long the private data is persisted in the private database. Later,
we will demonstrate how chaincode APIs ``PutPrivateData`` and ``GetPrivateData``
are used to map the collection to the private data being secured.

这个集合定义了谁可以持久化数据。数据分配到多少个peers，多少peer节点被要求传播这些私有数据和这些私有数据被持久化在这些私有数据库中多久。随后我们将演示如何使用chaincode API``PutPrivateData``和``GetPrivateData``将集合映射到受保护的私有数据。

A collection definition is composed of five properties:

一个集合定义由五个属性组成：

.. _blockToLive:

- ``name``: Name of the collection.
- ``name``: 集合的名称。
- ``policy``: Defines the organization peers allowed to persist the collection data.
- ``policy``: 定义组织的peer节点允许持久化集合数据。
- ``requiredPeerCount``: Number of peers required to disseminate the private data as
  a condition of the endorsement of the chaincode
- ``requiredPeerCount``: 被要求传播私有数据的peer节点数量，作为链码背书的条件。
- ``maxPeerCount``: For data redundancy purposes, the number of other peers
  that the current endorsing peer will attempt to distribute the data to.
  If an endorsing peer goes down, these other peers are available at commit time
  if there are requests to pull the private data.
- ``maxPeerCount``: 出于数据冗余的目的，当前背书的peer会试图分发到的其他peer节点的数量。
- ``blockToLive``: For very sensitive information such as pricing or personal information,
  this value represents how long the data should live on the private database in terms
  of blocks. The data will live for this specified number of blocks on the private database
  and after that it will get purged, making this data obsolete from the network.
  To keep private data indefinitely, that is, to never purge private data, set
  the ``blockToLive`` property to ``0``.
- ``blockToLive``: 对一些特别敏感的信息，如价格或者私人信息，此值表示数据在区块的角度，应在私有数据库上存在多长时间。这些数据将在私有数据库上对那些指定数量的块有效，之后将被清除，从而使这些数据从网络中过时。要无限期地保留私有数据，即永远不要清除私有数据，请设置``blockToLive``属性为``0``。

To illustrate usage of private data, the marbles private data example contains
two private data collection definitions: ``collectionMarbles``
and ``collectionMarblePrivateDetails``. The ``policy`` property in the
``collectionMarbles`` definition allows all members of  the channel (Org1 and
Org2) to have the private data in a private database. The
``collectionMarblesPrivateDetails`` collection allows only members of Org1 to
have the private data in their private database.

为了说明私有数据的使用，marbles私有数据示例包含了两个私有数据集定义： ``collectionMarbles``
和``collectionMarblePrivateDetails``。``collectionMarbles``定义中的``policy``属性允许通道的所有成员（Org1和Org2）在私有数据库中拥有私有数据。``collectionMarblesPrivateDetails``集合只允许Org1的成员
私有数据库中包含私有数据。

For more information on building a policy definition refer to the :doc:`endorsement-policies`
topic.

更多有关构建一个策略的定义请查阅:doc:`endorsement-policies`。

.. code-block:: JSON

 // collections_config.json

 [
   {
​        "name": "collectionMarbles",
​        "policy": "OR('Org1MSP.member', 'Org2MSP.member')",
​        "requiredPeerCount": 0,
​        "maxPeerCount": 3,
​        "blockToLive":1000000
   },

   {
​        "name": "collectionMarblePrivateDetails",
​        "policy": "OR('Org1MSP.member')",
​        "requiredPeerCount": 0,
​        "maxPeerCount": 3,
​        "blockToLive":3
   }
 ]

The data to be secured by these policies is mapped in chaincode and will be
shown later in the tutorial.

这些策略要保护的数据映射在链码中，稍后将在本教程中显示。

This collection definition file is deployed on the channel when its associated
chaincode is instantiated on the channel using the `peer chaincode instantiate command <http://hyperledger-fabric.readthedocs.io/en/latest/commands/peerchaincode.html#peer-chaincode-instantiate>`__.More details on this process are provided in Section 3 below.

当使用命令`peer chaincode instantiate command <http://hyperledger-fabric.readthedocs.io/en/latest/commands/peerchaincode.html#peer-chaincode-instantiate>`__在通道上实例化其关联的链码时，此集合定义文件将部署在通道上。有关此过程的更多详细信息，请参见下面的第3节。

.. _pd-read-write-private-data:

Read and Write private data using chaincode APIs-用chaincode  APIs读写私有数据
------------------------------------------------

The next step in understanding how to privatize data on a channel is to build
the data definition in the chaincode.  The marbles private data sample divides
the private data into two separate data definitions according to how the data will
be accessed.

理解如何在通道上私有化数据的下一步是在链代码中构建数据定义。marbles 私有数据示例根据数据的访问方式将私有数据划分为两个单独的数据定义。

.. code-block:: GO

 // Peers in Org1 and Org2 will have this private data in a side database
 type marble struct {
   ObjectType string `json:"docType"`
   Name       string `json:"name"`
   Color      string `json:"color"`
   Size       int    `json:"size"`
   Owner      string `json:"owner"`
 }

 // Only peers in Org1 will have this private data in a side database
 type marblePrivateDetails struct {
   ObjectType string `json:"docType"`
   Name       string `json:"name"`
   Price      int    `json:"price"`
 }

 Specifically access to the private data will be restricted as follows:

具体访问私人数据将受到如下限制：

 - ``name, color, size, and owner`` will be visible to all members of the channel (Org1 and Org2)
 - ``name, color, size, and owner`` 将会对通道（Org1和Org2）的所有成员可见。
 - ``price`` only visible to members of Org1
 - ``price`` 仅仅对Org1的成员可见。

Thus two different sets of private data are defined in the marbles private data
sample. The mapping of this data to the collection policy which restricts its
access is controlled by chaincode APIs. Specifically, reading and writing
private data using a collection definition is performed by calling ``GetPrivateData()``
and ``PutPrivateData()``, which can be found `here <https://github.com/hyperledger/fabric/blob/master/core/chaincode/shim/interfaces.go#L179>`_.

因此，在marbles 私有数据示例中定义了两组不同的私有数据。这个数据到限制其访问的集合策略的映射由链码APIs控制。具体来说，使用集合定义读取和写入私有数据是通过调用``GetPrivateData()``
and ``PutPrivateData()``来实现的，可以在这里找到： `<https://github.com/hyperledger/fabric/blob/master/core/chaincode/shim/interfaces.go#L179>`_.

The following diagrams illustrate the private data model used by the marbles private data sample.

下图说明了marbles 私有数据示例使用的私有数据模型：

 .. image:: images/SideDB-org1.png

 .. image:: images/SideDB-org2.png

Reading collection data-读取集合数据

~~~~~~~~~~~~~~~~~~~~~~~~

Use the chaincode API ``GetPrivateData()`` to query private data in the
database.  ``GetPrivateData()`` takes two arguments, the **collection name**
and the data key. Recall the collection  ``collectionMarbles`` allows members of
Org1 and Org2 to have the private data in a side database, and the collection
``collectionMarblePrivateDetails`` allows only members of Org1 to have the
private data in a side database. For implementation details refer to the
following two `marbles private data functions <https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02_private/go/marbles_chaincode_private.go>`__:
------------
使用链码API  ``GetPrivateData()`` 在数据库中查询私有数据。 ``GetPrivateData()``需要两个参数，集合名称和数据的键。
 * **readMarble** for querying the values of the ``name, color, size and owner`` attributes
  * **readMarble** 用于查询名称，颜色，大小和所有者属性的值。
 * **readMarblePrivateDetails** for querying the values of the ``price`` attribute
 * **readMarblePrivateDetails** 用于查询价格的值。
When we issue the database queries using the peer commands later in this tutorial,
we will call these two functions.
------------
当我们在本教程后面使用peer命令发出数据库查询时，我们将调用这两个函数。

Writing private data-写入私有数据
​~~~~~~~~~~~~~~~~~~~~

Use the chaincode API ``PutPrivateData()`` to store the private data
into the private database. The API also requires the name of the collection.
Since the marbles private data sample includes two different collections, it is called
twice in the chaincode:
通过链码API``PutPrivateData()``来把私有数据些人私有数据库。这个API同事还要求提供数据集的名称。由于marbles私有数据例中包含了两个不同的数据集，所在它在链码中被调用两次：

1. Write the private data ``name, color, size and owner`` using the
   collection named ``collectionMarbles``.
   ------------
   使用集合名称``collectionMarbles``写入私有数据 ``name, color, size and owner``.
2. Write the private data ``price`` using the collection named
   ``collectionMarblePrivateDetails``.
   ------------
  使用集合名称``collectionMarblePrivateDetails``写入私有数据 ``price``.

For example, in the following snippet of the ``initMarble`` function,
``PutPrivateData()`` is called twice, once for each set of private data.
------------
举例来说，在``initMarble``函数的以下片段中，``PutPrivateData（）``被调用两次，每组私有数据集一次。

.. code-block:: GO

  // ==== Create marble object and marshal to JSON ====
	objectType := "marble"
	marble := &marble{objectType, marbleName, color, size, owner}
	marbleJSONasBytes, err := json.Marshal(marble)
	if err != nil {
		return shim.Error(err.Error())
	}
	//Alternatively, build the marble json string manually if you don't want to use struct marshalling
	//marbleJSONasString := `{"docType":"Marble",  "name": "` + marbleName + `", "color": "` + color + `", "size": ` + strconv.Itoa(size) + `, "owner": "` + owner + `"}`
	//marbleJSONasBytes := []byte(str)

	// === Save marble to state ===
	err = stub.PutPrivateData("collectionMarbles", marbleName, marbleJSONasBytes)
	if err != nil {
		return shim.Error(err.Error())
	}

	// ==== Save marble private details ====
	objectType = "marblePrivateDetails"
	marblePrivateDetails := &marblePrivateDetails{objectType, marbleName, price}
	marblePrivateDetailsBytes, err := json.Marshal(marblePrivateDetails)
	if err != nil {
		return shim.Error(err.Error())
	}
	err = stub.PutPrivateData("collectionMarblePrivateDetails", marbleName, marblePrivateDetailsBytes)
	if err != nil {
		return shim.Error(err.Error())
 }

To summarize, the policy definition above for our ``collection.json``
allows all peers in Org1 and Org2 can store and transact (endorse, commit,
query) with the marbles private data ``name, color, size, owner`` in their
private database. But only peers in Org1 can can store and transact with
the ``price`` private data in an additional private database.
总而言之，上面在collection.json中定义的策略允许Org1和Org2中的所有peer都可以在其私有数据库中存储和交易（认可，提交，查询）marbles私有数据名称，颜色，大小，所有者。 但只有Org1中的peer可以在另外的私有数据库中存储和交易价格私有数据。

As an additional data privacy benefit, since a collection is being used,
only the private data hashes go through orderer, not the private data itself,
keeping private data confidential from orderer.
作为一个另外的私有数据的优势，既然一个数据集被使用，只有那个数据集的hash通过orderer，而不是数据集本身，从而使私有数据对orderer保密。

Start the network-启动网络
-----------------

Now we are ready to step through some commands which demonstrate using private
data.
现在我们准备使用私有数据的命令逐步完成一些演示。

 :guilabel:`Try it yourself`

 Before installing and instantiating the marbles private data chaincode below,
 we need to start the BYFN network. For the sake of this tutorial, we want to
 operate from a known initial state. The following command will kill any active
 or stale docker containers and remove previously generated artifacts.
 Therefore let's run the following command to clean up any previous
 environments:
 在下面安装和实例化marbles私有数据链代码之前，我们需要启动BYFN网络。为了本教程的缘故，我们希望从已知的初始状态开始操作。以下命令将终止所有活动或过时的docker容器并删除以前生成的构件。因此，让我们运行以下命令来清理以前的所有环境：

 .. code:: bash

    cd fabric-samples/first-network
    ./byfn.sh -m down


 Start up the BYFN network with CouchDB by running the following command:
 使用一下命令启动一个使用CouchDB的BYFN网络：

 .. code:: bash

    ./byfn.sh up -c mychannel -s couchdb

 This will create a simple Fabric network consisting of a single channel named
 ``mychannel`` with two organizations (each maintaining two peer nodes) and an
 ordering service while using CouchDB as the state database. Either LevelDB
 or CouchDB may be used with collections. CouchDB was chosen to demonstrate
 how to use indexes with private data.
 这将创建一个简单的Fabric网络，该网络由一个名为``mychannel``的通道组成，其中包含两个组织（每个组织维护两个peer节点）和一个orderer服务，同时使用CouchDB作为状态数据库。LevelDB或CouchDB可以与集合一起使用。 选择CouchDB来演示如何将索引与私有数据一起使用。
 

 .. note:: For collections to work, it is important to have cross organizational
           gossip configured correctly. Refer to our documentation on :doc:`gossip`,
           paying particular attention to the section on "anchor peers". Our tutorial
           does not focus on gossip given it is already configured in the BYFN sample,
           but when configuring a channel, the gossip anchors peers are critical to
           configure for collections to work properly.
   注意：要使集合起作用，必须正确配置跨组织的gossip。 请参阅我们的文档：doc：`gossip`。特别注意“anchor peers”的章节部分。我们的教程并没有关注gossip，因为它已经在BYFN例子中配置了。但是当配置一个通道的时候， gossip anchors peers对于配置数据集以使其正常工作至关重要。
           

.. _pd-install-instantiate_cc:

Install and instantiate chaincode with a collection-使用集合安装和初始化链码
---------------------------------------------------

Client applications interact with the blockchain ledger through chaincode. As
such we need to install and instantiate the chaincode on every peer that will
execute and endorse our transactions. Chaincode is installed onto a peer and
then instantiated onto the channel using :doc:`peer-commands`.
客户端通过链码和区块链账本交互。因此我们需要在我们将处理和背书我们的交易的peer节点安装和初始化链码。链码安装在peer节点然后使用:doc:`peer-commands`实例化到链码上。


Install chaincode on all peers - 安装链码到所有的peers
​~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As discussed above, the BYFN network includes two organizations, Org1 and Org2,
with two peers each. Therefore the chaincode has to be installed on four peers:
就像上面讨论的一样，BYFN网络包含了两个组织，Org1和Org2，每个组织包含两个peer节点。因此，链码必须安装在四个peer上：

- peer0.org1.example.com
- peer1.org1.example.com
- peer0.org2.example.com
- peer1.org2.example.com

Use the `peer chaincode install <http://hyperledger-fabric.readthedocs.io/en/master/commands/peerchaincode.html?%20chaincode%20instantiate#peer-chaincode-install>`__ command to install the Marbles chaincode on each peer.
使用 `peer chaincode install <http://hyperledger-fabric.readthedocs.io/en/master/commands/peerchaincode.html?%20chaincode%20instantiate#peer-chaincode-install>` 命令在每个peer上安装Marbles链码。

 :guilabel:`Try it yourself`

 Assuming you have started the BYFN network, enter the CLI container.
 假设您已经启动了BYFN网络，进入CLI容器。

 .. code:: bash

    docker exec -it cli bash

 Your command prompt will change to something similar to:
 您的命令提示符将更改为类似于：

 ``root@81eac8493633:/opt/gopath/src/github.com/hyperledger/fabric/peer#``

 1. Use the following command to install the Marbles chaincode from the git
    repository onto the peer ``peer0.org1.example.com`` in your BYFN network.
    (By default, after starting the BYFN network, the active peer is set to:
    ``CORE_PEER_ADDRESS=peer0.org1.example.com:7051``):
    使用下面的命令，安装Marbles链码从git仓库到BYFN网络的peer ``peer0.org1.example.com``节点，

    .. code:: bash

       peer chaincode install -n marblesp -v 1.0 -p github.com/chaincode/marbles02_private/go/

    When it is complete you should see something similar to:
    当完成的时候你会看到类似于：

    .. code:: bash

       install -> INFO 003 Installed remotely response:<status:200 payload:"OK" >

 2. Use the CLI to switch the active peer to the second peer in Org1 and
    install the chaincode. Copy and paste the following entire block of
    commands into the CLI container and run them.
    使用CLI去切换到Org组织的第二个活跃的peer安装链码。将以下整个命令块复制并粘贴到CLI容器中并运行它们。

    .. code:: bash

       export CORE_PEER_ADDRESS=peer1.org1.example.com:7051
       peer chaincode install -n marblesp -v 1.0 -p github.com/chaincode/marbles02_private/go/

 3. Use the CLI to switch to Org2. Copy and paste the following block of
    commands as a group into the peer container and run them all at once.
    使用CLI切换到Org2。将以下命令块作为一个组复制并粘贴到peer容器中，并立即运行它们。

    .. code:: bash

       export CORE_PEER_LOCALMSPID=Org2MSP
       export PEER0_ORG2_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
       export CORE_PEER_TLS_ROOTCERT_FILE=$PEER0_ORG2_CA
       export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp

 4. Switch the active peer to the first peer in Org2 and install the chaincode:
 切换到Org2中的第一个活跃节点然后安装链码：

    .. code:: bash

       export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
       peer chaincode install -n marblesp -v 1.0 -p github.com/chaincode/marbles02_private/go/

 5. Switch the active peer to the second peer in org2 and install the chaincode:
  切换到Org2中的第二个活跃节点然后安装链码：

    .. code:: bash

       export CORE_PEER_ADDRESS=peer1.org2.example.com:7051
       peer chaincode install -n marblesp -v 1.0 -p github.com/chaincode/marbles02_private/go/

Instantiate the chaincode on the channel -在通道上实例化链码
​~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use the `peer chaincode instantiate <http://hyperledger-fabric.readthedocs.io/en/master/commands/peerchaincode.html?%20chaincode%20instantiate#peer-chaincode-instantiate>`__
command to instantiate the marbles chaincode on a channel. To configure
the chaincode collections on the channel, specify the flag ``--collections-config``
along with the name of the collections JSON file, ``collections_config.json`` in our
example.
使用`peer chaincode instantiate <http://hyperledger-fabric.readthedocs.io/en/master/commands/peerchaincode.html?%20chaincode%20instantiate#peer-chaincode-instantiate>`__命令在通道上实例化marbles链码。为了在通道上配置链码数据集，指定标识 ``--collections-config``和在我们例子中的数据集的JSON文件名称：``collections_config.json``。

 :guilabel:`Try it yourself`

 Run the following commands to instantiate the marbles private data
 chaincode on the BYFN channel ``mychannel``.
 运行下面的命令，在BYFN网络的通道``mychannel``上实例化marbles私有数据链码。

 .. code:: bash

   export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
   peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile $ORDERER_CA -C mychannel -n marblesp -v 1.0 -c '{"Args":["init"]}' -P "OR('Org1MSP.member','Org2MSP.member')" --collections-config  $GOPATH/src/github.com/chaincode/marbles02_private/collections_config.json

 .. note:: When specifying the value of the ``--collections-config`` flag, you will
           need to specify the fully qualified path to the collections_config.json file.For example: ``--collections-config  $GOPATH/src/github.com/chaincode/marbles02_private/collections_config.json``
            注意：单卖给你指定 ``--collections-config``标示的时候，你组要指定collections_config.json 文件的完整路径。例如：$GOPATH/src/github.com/chaincode/marbles02_private/collections_config.json``

 When the instantiation completes successfully you should see something similar to:
 当我们成功完成实例化的时候你回看到类似的信息：

 .. code:: bash

    [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
    [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc

 .. _pd-store-private-data:

Store private data-存储私有数据
------------------

Acting as a member of Org1, who is authorized to transact with all of the private data
in the marbles private data sample, switch back to an Org1 peer and
submit a request to add a marble:
作为Org1的成员，在marbles私有数据例中有权利与所有数据交易。切换到Org1的peer然后提交一个请求去添加marble。

 :guilabel:`Try it yourself`

 Copy and paste the following set of commands to the CLI command line.
 复制粘贴下面的设置命令到CLI容器的命令行中：

 .. code:: bash

    export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    export CORE_PEER_LOCALMSPID=Org1MSP
    export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    export PEER0_ORG1_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org1.example.com/tls/ca.crt

 Invoke the marbles ``initMarble`` function which
 creates a marble with private data ---  name ``marble1`` owned by ``tom`` with a color
 ``blue``, size ``35`` and price of ``99``. Recall that private data **price**
 will be stored separately from the public data **name, owner, color, size**.
 For this reason, the ``initMarble`` function calls the ``PutPrivateData()`` API
 twice to persist the private data, once using each collection.
 调用marbles的 ``initMarble``方法，它会创建一个带有私有数据的marble-名字是``marble1``，所有者是 ``tom``，颜色是 ``blue``, 大小 ``35``,价格``99``.回想一下，私有数据**price**将和公共数据 **name, owner, color, size**分开存储。由于这个原因，``initMarble``函数两次调用``PutPrivateData()``去实例化私有数据，一次使用一个集合。

 .. code:: bash

   peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["initMarble","marble1","blue","35","tom","99"]}'

 You should see results similar to:
 你会看到类似的结果：

 ``[chaincodeCmd] chaincodeInvokeOrQuery->INFO 001 Chaincode invoke successful. result: status:200``

.. _pd-query-authorized:

Query the private data as an authorized peer-作为一个授权的peer查询私有数据
--------------------------------------------

Our collection definition allows all members of Org1 and Org2
to have the ``name, color, size, owner`` private data in their side database,
but only peers in Org1 can have the ``price`` private data in their side
database. As an authorized peer in Org1, we will query both sets of private data.
我们的集合定义运行Org1和Org2的所有成员可以在他们的本地存储中存储``name, color, size, owner`` 私有数据，但只有Org1的peer可以在它们的本地存储 ``price``私有数据。

The first ``query`` command calls the ``readMarble`` function which passes
``collectionMarbles`` as an argument.
第一个``query``命令调用``readMarble``函数，它将``collectionMarbles``作为参数传递。

.. code:: GO

   // ===============================================
   // readMarble - read a marble from chaincode state
   // ===============================================

   func (t *SimpleChaincode) readMarble(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	    var name, jsonResp string
      var err error
      if len(args) != 1 {
	 	    return shim.Error("Incorrect number of arguments. Expecting name of the marble to query")
	     }

  name = args[0]
   valAsbytes, err := stub.GetPrivateData("collectionMarbles", name) //get the marble from chaincode state

	  if err != nil {
       jsonResp = "{\"Error\":\"Failed to get state for " + name + "\"}"
       return shim.Error(jsonResp)
     } else if valAsbytes == nil {
       jsonResp = "{\"Error\":\"Marble does not exist: " + name + "\"}"
       return shim.Error(jsonResp)
     }

   return shim.Success(valAsbytes)
   }

The second ``query`` command calls the ``readMarblereadMarblePrivateDetails``
function which passes ``collectionMarblePrivateDetails`` as an argument.
第二个``query``命令调用``readMarblereadMarblePrivateDetails``，它将``collectionMarblePrivateDetails``作为参数传递的函数。

.. code:: GO

   // ===============================================
   // readMarblereadMarblePrivateDetails - read a marble private details from chaincode state
   // ===============================================

   func (t *SimpleChaincode) readMarblePrivateDetails(stub shim.ChaincodeStubInterface, args []string) pb.Response {
   var name, jsonResp string
   var err error

   if len(args) != 1 {
     return shim.Error("Incorrect number of arguments. Expecting name of the marble to query")
    }

   name = args[0]
   valAsbytes, err := stub.GetPrivateData("collectionMarblePrivateDetails", name) //get the marble private details from chaincode state

   if err != nil {
     jsonResp = "{\"Error\":\"Failed to get private details for " + name + ": " + err.Error() + "\"}"
     return shim.Error(jsonResp)
    } else if valAsbytes == nil {
     jsonResp = "{\"Error\":\"Marble private details does not exist: " + name + "\"}"
     return shim.Error(jsonResp)
    }
   return shim.Success(valAsbytes)
   }

Now :guilabel:`Try it yourself`

 Query for the ``name, color, size and owner`` private data of ``marble1`` as a member of Org1.
 作为Org1的一个成员查询``marble1``的私有数据 ``name, color, size and owner``。

 .. code:: bash

    peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarble","marble1"]}'

 You should see the following result:

 .. code:: bash

    {"color":"blue","docType":"marble","name":"marble1","owner":"tom","size":35}

 Query for the ``price`` private data of ``marble1`` as a member of Org1.

 .. code:: bash

    peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'

 You should see the following result:

 .. code:: bash

    {"docType":"marblePrivateDetails","name":"marble1","price":99}

.. _pd-query-unauthorized:

Query the private data as an unauthorized peer-作为一个未授权的peer查询私有数据
----------------------------------------------

Now we will switch to a member of Org2 which has the marbles private data
``name, color, size, owner`` in its side database, but does not have the
marbles ``price`` private data in its side database. We will query for both
sets of private data.
现在我们会切换到Org2的一个成员，它在本地数据库存储了私有数据``name, color, size, owner`` ，但是没有 ``price``。我们将查询这两组私有数据。
Switch to a peer in Org2 - 切换到Org2的peer
~~~~~~~~~~~~~~~~~~~~~~~~

From inside the docker container, run the following commands to switch to
the peer which is unauthorized to the marbles ``price`` private data.

从docker容器内部，运行以下命令切换到未经授权使用marbles ``price``私有数据的peer。

 :guilabel:`Try it yourself`

 .. code:: bash

    export CORE_PEER_ADDRESS=peer0.org2.example.com:7051
    export CORE_PEER_LOCALMSPID=Org2MSP
    export PEER0_ORG2_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    export CORE_PEER_TLS_ROOTCERT_FILE=$PEER0_ORG2_CA
    export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp

Query private data Org2 is authorized to-被授权查询私有数据Org2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Peers in Org2 should have the first set of marbles private data (``name,
color, size and owner``) in their side database and can access it using the
``readMarble()`` function which is called with the ``collectionMarbles``
argument.
Org2的peer节点应该在它们的本地数据库有第一组marbles的私有数据 (``name,
color, size and owner``)，并且可以使用``collectionMarbles``作为参数的``readMarble（）``函数来访问它。

 :guilabel:`Try it yourself`

 .. code:: bash

    peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarble","marble1"]}'

 You should see something similar to the following result:
 你会看到类似下面的结果：

 .. code:: json

    {"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"tom"}

Query private data Org2 is not authorized to-未被授权查询私有数据Org2
​~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Peers in Org2 do not have the marbles ``price`` private data in their side database.
When they try to query for this data, they get back a hash of the key matching
the public state but will not have the private state.
Org2中的peer在它们的本地仓库没有存储marbles的 ``price`` 。当它们试图查询这个值的时候，它们找回与公共状态匹配的密钥的哈希值，但不会拥有私有状态的。

 :guilabel:`Try it yourself`

 .. code:: bash

    peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'

 You should see a result similar to:
 你会看到类似下面的结果：

 .. code:: json

    {"Error":"Failed to get private details for marble1: GET_STATE failed:
    transaction ID: b04adebbf165ddc90b4ab897171e1daa7d360079ac18e65fa15d84ddfebfae90:
    Private data matching public hash version is not available. Public hash
    version = &version.Height{BlockNum:0x6, TxNum:0x0}, Private data version =
    (*version.Height)(nil)"}"

Members of Org2 will only be able to see the public hash of the private data.
Org2的成员将智能看到私有数据的公共hash。

.. _pd-purge:

Purge Private Data-清楚私有数据
------------------

For use cases where private data only needs to be on the ledger until it can be
replicated into an off-chain database, it is possible to "purge" the data after
a certain set number of blocks, leaving behind only hash of the data that serves
as immutable evidence of the transaction.
对于私有数据只需要在账本上直到可以复制到离线数据库中的用例，可以在一定数量的块之后“清除”数据，只留下数据的哈希值。作为交易的不可改变的证据。

There may be private data including personal or confidential
information, such as the pricing data in our example, that the transacting
parties don't want disclosed to other organizations on the channel. Thus, it
has a limited lifespan, and can be purged after existing unchanged on the
blockchain for a designated number of blocks using the ``blockToLive`` property
in the collection definition.
可能存在私人数据，包括个人或机密信息，例如我们示例中的定价数据，交易方不希望在渠道上向其他组织披露。 因此，它具有有限的寿命，并且可以在区块链中使用集合定义中的“blockToLive”属性在指定数量的块上保持不变之后进行清除。

Our ``collectionMarblePrivateDetails`` definition has a ``blockToLive``
property value of three meaning this data will live on the side database for
three blocks and then after that it will get purged. Tying all of the pieces
together, recall this collection definition  ``collectionMarblePrivateDetails``
is associated with the ``price`` private data in the  ``initMarble()`` function
when it calls the ``PutPrivateData()`` API and passes the
``collectionMarblePrivateDetails`` as an argument.
我们的``collectionMarblePrivateDetails``定义有一个``blockToLive``property值
三，意味着这个数据将存在于拥有三个块的本地数据库中，超过它将被清除。 将所有部分绑定在一起，回想一下这个集合定义``collectionMarblePrivateDetails``与``initMarble（）``函数中的``price``私有数据相关联，当它调用``PutPrivateData（）``API时 传递``collectionMarblePrivateDetails``作为参数。

We will step through adding blocks to the chain, and then watch the price
information get purged by issuing four new transactions (Create a new marble,
followed by three marble transfers) which adds four new blocks to the chain.
After the fourth transaction (third marble transfer), we will verify that the
price private data is purged.
我们将逐步向链中添加块，然后通过发出四个新的交易（三个marble转移后创建一个新的marble）来观察价格信息被清除，这将为链添加四个新块。 在第四次交易（第三次marble转移）之后，我们将验证价格私人数据是否被清除。

 :guilabel:`Try it yourself`

 Switch back to peer0 in Org1 using the following commands. Copy and paste the
 following code block and run it inside your peer container:
 用下面的命令切换到Org1的peer0节点。复制粘贴下面的代码块并在peer容器中运行它：

 .. code:: bash

    export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    export CORE_PEER_LOCALMSPID=Org1MSP
    export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    export PEER0_ORG1_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org1.example.com/tls/ca.crt

 Open a new terminal window and view the private data logs for this peer by
 running the following command:
 打开一个新的终端，通过下面名查看这个peer的私有数据日志：

 .. code:: bash

    docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'

 You should see results similar to the following. Note the highest block number
 in the list. In the example below, the highest block height is ``4``.
 你回看到类似的结果。注意这个列表中的最高区块数量，在下面的列表中，区块的最大高度是``4``。

 .. code:: bash

    [pvtdatastorage] func1 -> INFO 023 Purger started: Purging expired private data till block number [0]
    [pvtdatastorage] func1 -> INFO 024 Purger finished
    [kvledger] CommitWithPvtData -> INFO 022 Channel [mychannel]: Committed block [0] with 1 transaction(s)
    [kvledger] CommitWithPvtData -> INFO 02e Channel [mychannel]: Committed block [1] with 1 transaction(s)
    [kvledger] CommitWithPvtData -> INFO 030 Channel [mychannel]: Committed block [2] with 1 transaction(s)
    [kvledger] CommitWithPvtData -> INFO 036 Channel [mychannel]: Committed block [3] with 1 transaction(s)
    [kvledger] CommitWithPvtData -> INFO 03e Channel [mychannel]: Committed block [4] with 1 transaction(s)

 Back in the peer container, query for the **marble1** price data by running the
 following command. (A Query does not create a new transaction on the ledger
 since no data is transacted).
 回到peer容器，通过下面命令查看**marble1**的价格数据（由于没有数据处理，因此查询不会在账本上创建新事务）。

 .. code:: bash

    peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'

 You should see results similar to:
 你会看到类似的信息：

 .. code:: bash

    {"docType":"marblePrivateDetails","name":"marble1","price":99}

 The ``price`` data is still on the private data ledger.
 ``price``数据依然存在私有数据账本上。

 Create a new **marble2** by issuing the following command. This transaction
 creates a new block on the chain.
 提交下面的命令来创建一个新的 **marble2** 。这个交易在链上创建一个新的交易。

 .. code:: bash

    peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["initMarble","marble2","blue","35","tom","99"]}'

 Switch back to the Terminal window and view the private data logs for this peer
 again. You should see the block height increase by 1.
 切回到widow终端，再次查看peer的私有数据日志。你会看到区块高度加1.

 .. code:: bash

    docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'

 Back in the peer container, query for the **marble1** price data again by
 running the following command:
 回到peer容器，再次通过下面的命令查询**marble1**的价格数据。

 .. code:: bash

    peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'

 The private data has not been purged, therefore the results are unchanged from
 previous query:
 这个数据还没有并清除，因此结果与先前的查询相同：

 .. code:: bash

    {"docType":"marblePrivateDetails","name":"marble1","price":99}

 Transfer marble2 to "joe" by running the following command. This transaction
 will add a second new block on the chain.
 通过运行以下命令将marble2传输到“joe”。 此事务将在链上添加第二个新块。

 .. code:: bash

    peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["transferMarble","marble2","joe"]}'

 Switch back to the Terminal window and view the private data logs for this peer
 again. You should see the block height increase by 1.
 切换到window的终端然后再次查看peer的私有数据日志。你会看到区块高度加1.

 .. code:: bash

    docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'

 Back in the peer container, query for the marble1 price data by running
 the following command:
 在回到peer容器，通过下面的命令查看marble1的价格数据。

 .. code:: bash

    peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'

 You should still be able to see the price private data.
 你依然能看到价格的私有数据：

 .. code:: bash

    {"docType":"marblePrivateDetails","name":"marble1","price":99}

 Transfer marble2 to "tom" by running the following command. This transaction
 will create a third new block on the chain.
 通过运行以下命令将marble2传输到“tom”。 此事务将在链上创建第三个新块。

 .. code:: bash

    peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["transferMarble","marble2","tom"]}'

 Switch back to the Terminal window and view the private data logs for this peer
 again. You should see the block height increase by 1.
  切换到window的终端然后再次查看peer的私有数据日志。你会看到区块高度加1.

 .. code:: bash

    docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'

 Back in the peer container, query for the marble1 price data by running
 the following command:
 在回到peer容器，通过下面的命令查看marble1的价格数据。

 .. code:: bash

    peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'

 You should still be able to see the price data.
 你依然能看到价格的私有数据：

 .. code:: bash

    {"docType":"marblePrivateDetails","name":"marble1","price":99}

 Finally, transfer marble2 to "jerry" by running the following command. This
 transaction will create a fourth new block on the chain. The ``price`` private
 data should be purged after this transaction.
 最后，通过运行以下命令将marble2转移到“jerry”。 此事务将在链上创建第四个新块。 此交易后应清除“价格”私人数据。

 .. code:: bash

    peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marblesp -c '{"Args":["transferMarble","marble2","jerry"]}'

 Switch back to the Terminal window and view the private data logs for this peer
 again. You should see the block height increase by 1.
 切换到window的终端然后再次查看peer的私有数据日志。你会看到区块高度加1.

 .. code:: bash

    docker logs peer0.org1.example.com 2>&1 | grep -i -a -E 'private|pvt|privdata'

 Back in the peer container, query for the marble1 price data by running the following command:
 在回到peer容器，通过下面的命令查看marble1的价格数据。

 .. code:: bash

    peer chaincode query -C mychannel -n marblesp -c '{"Args":["readMarblePrivateDetails","marble1"]}'

 Because the price data has been purged, you should no longer be able to see
 it. You should see something similar to:
 因为价格私有数据已经被清除，你将不会在看到它。你会看到类似下面的输出：

 .. code:: bash

    Error: endorsement failure during query. response: status:500
    message:"{\"Error\":\"Marble private details does not exist: marble1\"}"

.. _pd-indexes:

Using indexes with private data-使用私有数据索引
-------------------------------

Indexes can also be applied to private data collections, by packaging indexes in
the ``META-INF/statedb/couchdb/collections/<collection_name>/indexes`` directory
alongside the chaincode. An example index is available `here <https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02_private/go/META-INF/statedb/couchdb/collections/collectionMarbles/indexes/indexOwner.json>`__ .
通过在链码旁边的“META-INF / statedb / couchdb / collections / <collection_name> / indexes``目录中打包索引，索引也可以应用于私有数据集合。 一个示例索引可用`here <https://github.com/hyperledger/fabric samples / blob / master / chaincode / marbles02_private / go / META INF / statedb / couchdb / collections / collectionMarbles / indexes / indexOwner.json>`__。

For deployment of chaincode to production environments, it is recommended
to define any indexes alongside chaincode so that the chaincode and supporting
indexes are deployed automatically as a unit, once the chaincode has been
installed on a peer and instantiated on a channel. The associated indexes are
automatically deployed upon chaincode instantiation on the channel when
the  ``--collections-config`` flag is specified pointing to the location of
the collection JSON file.
为了将链码部署到生产环境，建议在链码旁边定义任何索引，以便一旦链码安装在peer并在通道上实例化，链码和支持索引作为一个单元自动部署。 当指定``--collections config``标志指向集合JSON文件的位置时，关联的索引在通道上的链码实例化时自动部署。

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~