
Using CouchDB--使用CouchDB
=============

This tutorial will describe the steps required to use the CouchDB as the state
database with Hyperledger Fabric. By now, you should be familiar with Fabric
concepts and have explored some of the samples and tutorials.

本教程将介绍在Hyperledger Fabric中使用CouchDB用作状态数据库所需的步骤。 到现在为止，你应该熟悉Fabric概念并探讨了一些示例和教程。

The tutorial will take you through the following steps:

本教程将包括下面几个步骤：

#. :ref:`cdb-enable-couch`

开启CouchDB

#. :ref:`cdb-create-index`

创建索引

#. :ref:`cdb-add-index`

添加索引到链码文件夹

#. :ref:`cdb-install-instantiate`

安装实例化链码

#. :ref:`cdb-query`

查询CouchDB

#. :ref:`cdb-update-index`

更新CouchDB 索引

#. :ref:`cdb-delete-index`

删除CouchDB索引

For a deeper dive into CouchDB refer to :doc:`couchdb_as_state_database`
and for more information on the Fabric ledger refer to the `Ledger <ledger/ledger.html>`_
topic. Follow the tutorial below for details on how to leverage CouchDB in your
blockchain network.

要深入了解CouchDB，请参考:doc:`couchdb_as_state_database。了解更多超级账本的信息参考 `Ledger <ledger/ledger.html>`_。教程下面会详细介绍如何在你的区块链旺罗冲使用CouchDB。



Throughout this tutorial we will use the `Marbles sample <https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02/go/marbles_chaincode.go>`__
as our use case to demonstrate how to use CouchDB with Fabric and will deploy
Marbles to the :doc:`build_network` (BYFN) tutorial network. You should have
completed the task :doc:`install`. However, running the BYFN tutorial is not
a prerequisite for this tutorial, instead the necessary commands are provided
throughout this tutorial to use the network.

在本教程中，我们会使用Marbles sample <https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02/go/marbles_chaincode.go>`__作为我们的例子来演示如何在Fabric中使用CouchDB，然后会在BYFN网络中部署Marbles。你应该已经完成了这部分:doc:`install。然而，运行BYFN不是本教程的必要条件，提供了一些网络中本教程的一些必要命令。

Why CouchDB? -为什么是CouchDB？
~~~~~~~~~~~~

Fabric supports two types of peer databases. LevelDB is the default state
database embedded in the peer node and stores chaincode data as simple
key-value pairs and supports key, key range, and composite key queries only.
CouchDB is an optional alternate state database that supports rich
queries when chaincode data values are modeled as JSON. Rich queries are more
flexible and efficient against large indexed data stores, when you want to
query the actual data value content rather than the keys. CouchDB is a JSON
document datastore rather than a pure key-value store therefore enabling
indexing of the contents of the documents in the database.
Fabric支持两种节点数据库。LevelDB是默认的嵌在peer节点，存储链码数据。它支持简单的键值对，仅支持键，键范围和复合键查询。CouchDB是一个可选的备用状态数据库，当链代码数据值建模为JSON时，它支持富查询。 当您要查询实际数据值内容而不是键时，富查询对大型索引数据存储更灵活，更有效。 CouchDB是一个JSON文档数据存储区而不是纯键值存储区，因此可以索引数据库中文档的内容。

In order to leverage the benefits of CouchDB, namely content-based JSON
queries,your data must be modeled in JSON format. You must decide whether to use
LevelDB or CouchDB before setting up your network. Switching a peer from using
LevelDB to CouchDB is not supported due to data compatibility issues. All peers
on the network must use the same database type. If you have a mix of JSON and
binary data values, you can still use CouchDB, however the binary values can
only be queried based on key, key range, and composite key queries.

为了利用CouchDB的优势，即基于内容的JSON查询，您的数据必须以JSON格式建模。 在设置网络之前，您必须决定是使用LevelDB还是CouchDB。 由于数据兼容性问题，不支持将peer从使用LevelDB切换到CouchDB。 网络上的所有peer都必须使用相同的数据库类型。 如果混合使用JSON和二进制数据值，仍可以使用CouchDB，但只能根据键，键范围和组合键查询查询二进制值。

.. _cdb-enable-couch:

Enable CouchDB in Hyperledger Fabric -在Hyperledger Fabric 中启用CouchDB
​~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CouchDB runs as a separate database process alongside the peer, therefore there
are additional considerations in terms of setup, management, and operations.
A docker image of `CouchDB <https://hub.docker.com/r/hyperledger/fabric-couchdb/>`__
is available and we recommend that it be run on the same server as the
peer. You will need to setup one CouchDB container per peer
and update each peer container by changing the configuration found in
``core.yaml`` to point to the CouchDB container. The ``core.yaml``
file must be located in the directory specified by the environment variable
FABRIC_CFG_PATH:
CouchDB作为独立的数据库进程与peer节点一起运行，因此在设置，管理和操作方面还有其他注意事项。 可以使用`CouchDB <https://hub.docker.com/r/hyperledger/fabric couchdb />`__的docker镜像，我们建议它在运行在相同的peer服务上。 您需要为每个peer设置一个CouchDB容器，并通过更改``core.yaml``中的配置来更新每个peer容器，以指向CouchDB容器。 ``core.yaml``文件必须位于环境变量FABRIC_CFG_PATH指定的目录中：

* For docker deployments, ``core.yaml`` is pre-configured and located in the peer
  container ``FABRIC_CFG_PATH`` folder. However when using docker environments,
  you typically pass environment variables by editing the
  ``docker-compose-couch.yaml``  to override the core.yaml
  对于docker部署，``core.yaml``是预配置的，位于peer容器的``FABRIC_CFG_PATH``文件夹中。 但是，在使用docker环境时，通常通过编辑``docker-compose-couch.yaml``来覆盖core.yaml来传递环境变量。

* For native binary deployments, ``core.yaml`` is included with the release artifact
  distribution.
  对于本地二进制部署，发布构件分发中包含``core.yaml``。

Edit the ``stateDatabase`` section of ``core.yaml``. Specify ``CouchDB`` as the
``stateDatabase`` and fill in the associated ``couchDBConfig`` properties. For
more details on configuring CouchDB to work with fabric, refer `here <http://hyperledger-fabric.readthedocs.io/en/master/couchdb_as_state_database.html#couchdb-configuration>`__.
To view an example of a core.yaml file configured for CouchDB, examine the
BYFN ``docker-compose-couch.yaml`` in the ``HyperLedger/fabric-samples/first-network``
directory.

编辑``core.yaml``的``stateDatabase``部分。 指定``CouchDB``作为``stateDatabase``并填写相关的``couchDBConfig``属性。 有关配置CouchDB和fabric工作的更多详细信息，请参阅“here <http：// hyperledger fabric.readthedocs.io/en/master/couchdb_as_state_database html＃couchdb-configuration>`__。 要查看为CouchDB配置的core.yaml文件的示例，请检查``HyperLedger / fabric samples / first-network``目录中的BYFN`“docker-compose-couch.yaml``。

.. _cdb-create-index:

Create an index-创建索引
​~~~~~~~~~~~~~~~

Why are indexes important?-为什么这些索引很重要

Indexes allow a database to be queried without having to examine every row
with every query, making them run faster and more efficiently. Normally,
indexes are built for frequently occurring query criteria allowing the data to
be queried more efficiently. To leverage the major benefit of CouchDB -- the
ability to perform rich queries against JSON data -- indexes are not required,
but they are strongly recommended for performance. Also, if sorting is required
in a query, CouchDB requires an index of the sorted fields.
索引允许查询数据库，而不必检查每个查询的每一行，使它们运行得更快，更有效。 通常，索引是针对频繁出现的查询条件构建的，允许更有效地查询数据。 要利用CouchDB的主要优势 - 
能够对JSON数据执行丰富的查询 - 不需要索引，但强烈建议使用它们来提高性能。 此外，如果查询中需要排序，CouchDB需要排序字段的索引。

.. note::

   Rich queries that do not have an index will work but may throw a warning
   in the CouchDB log that the index was not found. However, if a rich query
   includes a sort specification, then an index on that field is required;
   otherwise, the query will fail and an error will be thrown.
   没有索引的富查询将起作用，但可能会在CouchDB日志中发出未找到索引的警告。 但是，如果富查询包含排序规范，则需要该字段的索引;否则，查询将失败并将引发错误。

To demonstrate building an index, we will use the data from the `Marbles
sample <https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02/go/marbles_chaincode.go>`__.
In this example, the Marbles data structure is defined as:

为了演示构建索引，我们将使用“Marbles”中的数据
示例<https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02/go/marbles_chaincode.go>`__。
在此示例中，Marbles数据结构定义为：

.. code:: javascript

  type marble struct {
	   ObjectType string `json:"docType"` //docType is used to distinguish the various types of objects in state database
	   Name       string `json:"name"`    //the field tags are needed to keep case from bouncing around
	   Color      string `json:"color"`
           Size       int    `json:"size"`
           Owner      string `json:"owner"`
  }


In this structure, the attributes (``docType``, ``name``, ``color``, ``size``,
``owner``) define the ledger data associated with the asset. The attribute
``docType`` is a pattern used in the chaincode to differentiate different data
types that may need to be queried separately. When using CouchDB, it
recommended to include this ``docType`` attribute to distinguish each type of
document in the chaincode namespace. (Each chaincode is represented as its own
CouchDB database, that is, each chaincode has its own namespace for keys.)

在这种结构中，属性（``docType``，``name``，``color``，``size``，``owner``）定义与资产相关的账本数据。属性“docType”是链码中使用的模式，用于区分可能需要单独查询的不同数据类型。 当使用CouchDB时，它建议包含这个``docType``属性来区分chaincode命名空间中的每种类型的文档。 （每个链代码都表示为自己的CouchDB数据库，也就是说，每个链代码都有自己的密钥命名空间。）

With respect to the Marbles data structure, ``docType`` is used to identify
that this document/asset is a marble asset. Potentially there could be other
documents/assets in the chaincode database. The documents in the database are
searchable against all of these attribute values.

关于Marbles数据结构，``docType``用于标识此文档/资产是marble资产。 可能在链代码数据库中可能存在其他文档/资产。 数据库中的文档可以针对所有这些属性值进行搜索。

When defining an index for use in chaincode queries, each one must be defined
in its own text file with the extension `*.json` and the index definition must
be formatted in the CouchDB index JSON format.

在定义用于链代码查询的索引时，每个索引必须在其自己的文本文件中定义，扩展名为`* .json`，索引定义必须以CouchDB索引的JSON格式进行格式化。

To define an index, three pieces of information are required:
要定义索引需要下面三个信息：

  * `fields`: these are the frequently queried fields
  * `fields`: 频繁查询的部分
  * `name`: name of the index
  * `name`: 索引名称
  * `type`: always json in this context
  * `type`: 总是json

For example, a simple index named ``foo-index`` for a field named ``foo``.

例如，一个名为``foo-index``的简单索引，用于名为``foo``的字段。
.. code:: json

    {
        "index": {
            "fields": ["foo"]
        },
        "name" : "foo-index",
        "type" : "json"
    }

Optionally the design document  attribute ``ddoc`` can be specified on the index
definition. A `design document <http://guide.couchdb.org/draft/design.html>`__ is
CouchDB construct designed to contain indexes. Indexes can be grouped into
design documents for efficiency but CouchDB recommends one index per design
document.
可选地，可以在索引定义上指定设计文档属性“ddoc”。 `设计文档<http://guide.couchdb.org/draft/design.html>`__是用于包含索引的CouchDB构造。 索引可以分组到设计文档中以提高效率，但CouchDB建议每个设计文档使用一个索引。

.. tip:: When defining an index it is a good practice to include the ``ddoc``
         attribute and value along with the index name. It is important to
         include this attribute to ensure that you can update the index later
         if needed. Also it gives you the ability to explicitly specify which
         index to use on a query.
         定义索引时，最好将``ddoc``属性和值与索引名一起包含在内。 包含此属性非常重要，以确保您可以在以后需要时更新索引。 此外，它还使您能够显式指定要在查询上使用的索引。


Here is another example of an index definition from the Marbles sample with
the index name ``indexOwner`` using multiple fields ``docType`` and ``owner``
and includes the ``ddoc`` attribute:
下面是Marbles示例中索引定义的另一个示例，索引名称为``indexOwner``，使用多个字段``docType``和``owner``并包含``ddoc``属性：

.. _indexExample:

.. code:: json

  {
    "index":{
        "fields":["docType","owner"] // Names of the fields to be queried
    },
    "ddoc":"indexOwnerDoc", // (optional) Name of the design document in which the index will be created.
    "name":"indexOwner",
    "type":"json"
  }

In the example above, if the design document ``indexOwnerDoc`` does not already
exist, it is automatically created when the index is deployed. An index can be
constructed with one or more attributes specified in the list of fields and
any combination of attributes can be specified. An attribute can exist in
multiple indexes for the same docType. In the following example, ``index1``
only includes the attribute ``owner``, ``index2`` includes the attributes
``owner and color`` and ``index3`` includes the attributes ``owner, color and
size``. Also, notice each index definition has its own ``ddoc`` value, following
the CouchDB recommended practice.

在上面的示例中，如果设计文档``indexOwnerDoc``尚不存在，则在部署索引时会自动创建它。 可以使用字段列表中指定的一个或多个属性构造索引，并且可以指定任何属性组合。 对于同一docType，属性可以存在于多个索引中。 在下面的例子中，``index1``只包含属性``owner``，``index2``包含属性``owner and color``和``index3``包含属性``owner，color和size``。 另外，请注意每个索引定义都有自己的``ddoc``值，遵循CouchDB建议的做法。

.. code:: json

  {
    "index":{
        "fields":["owner"] // Names of the fields to be queried
    },
    "ddoc":"index1Doc", // (optional) Name of the design document in which the index will be created.
    "name":"index1",
    "type":"json"
  }

  {
    "index":{
        "fields":["owner", "color"] // Names of the fields to be queried
    },
    "ddoc":"index2Doc", // (optional) Name of the design document in which the index will be created.
    "name":"index2",
    "type":"json"
  }

  {
    "index":{
        "fields":["owner", "color", "size"] // Names of the fields to be queried
    },
    "ddoc":"index3Doc", // (optional) Name of the design document in which the index will be created.
    "name":"index3",
    "type":"json"
  }


In general, you should model index fields to match the fields that will be used
in query filters and sorts. For more details on building an index in JSON
format refer to the `CouchDB documentation <http://docs.couchdb.org/en/latest/api/database/find.html#db-index>`__.

通常，您应该为索引字段建模以匹配将在查询过滤器和排序中使用的字段。 有关以JSON格式构建索引的更多详细信息，请参阅`CouchDB文档<http://docs.couchdb.org/en/latest/api/database/find.html#db-index>`__。

A final word on indexing, Fabric takes care of indexing the documents in the
database using a pattern called ``index warming``. CouchDB does not typically
index new or updated documents until the next query. Fabric ensures that
indexes stay 'warm' by requesting an index update after every block of data is
committed.  This ensures queries are fast because they do not have to index
documents before running the query. This process keeps the index current
and refreshed every time new records are added to the state database.

关于索引的最后一句话，Fabric负责使用名为``index warming``的模式索引数据库中的文档。 在下一个查询之前，CouchDB通常不会索引新文档或更新的文档。 Fabric通过在提交每个数据块之后请求索引更新来确保索引保持“warm”。 这可以确保查询很快，因为它们不必在运行查询之前索引文档。 每次将新记录添加到状态数据库时，此过程都会使索引保持最新并刷新。

.. _cdb-add-index:


Add the index to your chaincode folder-将索引添加到链码文件夹
​~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once you finalize an index, it is ready to be packaged with your chaincode for
deployment by being placed alongside it in the appropriate metadata folder.

完成索引后，可以将其与您的链代码打包在一起，以便将其放在相应的元数据文件夹中。

If your chaincode installation and instantiation uses the Hyperledger
Fabric Node SDK, the JSON index files can be located in any folder as long
as it conforms to this `directory structure <https://fabric-sdk-node.github.io/tutorial-metadata-chaincode.html>`__.
During the chaincode installation using the client.installChaincode() API,
include the attribute (``metadataPath``) in the `installation request <https://fabric-sdk-node.github.io/global.html#ChaincodeInstallRequest>`__.
The value of the metadataPath is a string representing the absolute path to the
directory structure containing the JSON index file(s).

如果您的链代码安装和实例化使用Hyperledger Fabric Node SDK，则JSON索引文件可以位于任何文件夹中，只要它符合此目录结构<https://fabric-sdk-node.github.io/tutorial- metadata chaincode.html>`__。在使用client.installChaincode（）API进行链代码安装期间，在`installation request <https：// fabric sdk-node.github.io/中包含属性（``metadataPath``）。global.html＃ChaincodeInstallRequest>`__。 metadataPath的值是一个字符串，表示包含JSON索引文件的目录结构的绝对路径。

Alternatively, if you are using the
:doc:`peer-commands` to install and instantiate the chaincode, then the JSON
index files must be located under the path ``META-INF/statedb/couchdb/indexes``
which is located inside the directory where the chaincode resides.

或者，如果您使用：doc：`peer-commands`来安装和实例化链代码，那么JSON索引文件必须位于目录内的路径``METAINF / statedb / couchdb / indexes``下,就是链码所在的位置。

The `Marbles sample <https://github.com/hyperledger/fabric-samples/tree/master/chaincode/marbles02/go>`__  below illustrates how the index
is packaged with the chaincode which will be installed using the peer commands.

下面的`Marbles示例<https://github.com/hyperledger/fabric-samples/tree/master/chaincode/marbles02/go>`__说明了索引如何与将使用peer命令安装的chaincode打包在一起。

.. image:: images/couchdb_tutorial_pkg_example.png
  :scale: 100%
  :align: center
  :alt: Marbles Chaincode Index Package


Start the network --启动网络
-----------------

 :guilabel:`Try it yourself`

 Before installing and instantiating the marbles chaincode, we need to start
 up the BYFN network. For the sake of this tutorial, we want to operate
 from a known initial state. The following command will kill any active
 or stale docker containers and remove previously generated artifacts.
 Therefore let's run the following command to clean up any
 previous environments:
 
 在安装和实例化marbles链码之前，我们需要启动BYFN网络。 为了本教程的目的，我们希望从已知的初始状态进行操作。 以下命令将终止所有活动或过时的docker容器并删除以前生成的工件。 因此，让我们运行以下命令来清理以前的所有环境：

 .. code:: bash

  cd fabric-samples/first-network
  ./byfn.sh -m down


 Now start up the BYFN network with CouchDB by running the following command:
 现在用下面的命令来用CouchDB启动BYFN网络：

 .. code:: bash

   ./byfn.sh up -c mychannel -s couchdb

 This will create a simple Fabric network consisting of a single channel named
 `mychannel` with two organizations (each maintaining two peer nodes) and an
 ordering service while using CouchDB as the state database.
 这将创建一个简单的Fabric网络，其中包含一个名为`mychannel`的通道，其中包含两个组织（每个组织维护两个peer节点）和一个order排序服务，同时使用CouchDB作为状态数据库。

.. _cdb-install-instantiate:

Install and instantiate the Chaincode-安装实例化链码
​~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Client applications interact with the blockchain ledger through chaincode. As
such we need to install the chaincode on every peer that will
execute and endorse our transactions and and instantiate the chaincode on the
channel. In the previous section, we demonstrated how to package the chaincode
so they should be ready for deployment.

客户端应用程序通过链代码与区块链账本交互。 因此，我们需要在每个将执行和背书我们的事务的peer节点上安装链码，并在通道上实例化链代码。 在上一节中，我们演示了如何打包链码，以便它们可以为部署做好准备。

Chaincode is installed onto a peer and then instantiated onto the channel using
:doc:`peer-commands`.
链码安装在peer节点上，然后使用:doc:`peer-commands`实例化到通道上。


1. Use the `peer chaincode install <http://hyperledger-fabric.readthedocs.io/en/master/commands/peerchaincode.html?%20chaincode%20instantiate#peer-chaincode-install>`__ command to install the Marbles chaincode on a peer.

使用`peer chaincode install <http://hyperledger-fabric.readthedocs.io/en/master/commands/peerchaincode.html?%20chaincode%20instantiate#peer-chaincode-install>`__命令在peer节点上安装Marbles链码。

 :guilabel:`Try it yourself`

 Assuming you have started the BYFN network, navigate into the CLI
 container using the command:
 假设您已启动BYFN网络，请使用以下命令进入到CLI容器：

 .. code:: bash

      docker exec -it cli bash

 Use the following command to install the Marbles chaincode from the git
 repository onto a peer in your BYFN network. The CLI container defaults
 to using peer0 of org1:
 使用以下命令将Marnles链码从git存储库安装到BYFN网络中的peer节点。 CLI容器默认使用org1的peer0：

 .. code:: bash

      peer chaincode install -n marbles -v 1.0 -p github.com/chaincode/marbles02/go

2. Issue the `peer chaincode instantiate <http://hyperledger-fabric.readthedocs.io/en/master/commands/peerchaincode.html?%20chaincode%20instantiate#peer-chaincode-instantiate>`__ command to instantiate the
chaincode on a channel.
通过`peer chaincode instantiate <http://hyperledger-fabric.readthedocs.io/en/master/commands/peerchaincode.html?%20chaincode%20instantiate#peer-chaincode-instantiate>`__命令来实例化通道上的链码。

 :guilabel:`Try it yourself`

 To instantiate the Marbles sample on the BYFN channel ``mychannel``
 run the following command:
 要在BYFN通道“mychannel”上实例化Marbles示例，请运行以下命令：

 .. code:: bash

    export CHANNEL_NAME=mychannel
    peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org0MSP.peer','Org1MSP.peer')"

Verify index was deployed--验证索引已部署
-------------------------

Indexes will be deployed to each peer's CouchDB state database once the
chaincode is both installed on the peer and instantiated on the channel. You
can verify that the CouchDB index was created successfully by examining the
peer log in the Docker container.

一旦链码安装在peer上并在通道上实例化，索引将被部署到每个peer的CouchDB状态数据库。 您可以通过检查Docker容器中的peer日志来验证是否已成功创建CouchDB索引。

 :guilabel:`Try it yourself`

 To view the logs in the peer docker container,
 open a new Terminal window and run the following command to grep for message
 confirmation that the index was created.
 
 要查看peer节点的docker容器中的日志，请打开一个新的终端窗口并运行以下命令以grep查找已创建索引的消息。

 ::

   docker logs peer0.org1.example.com  2>&1 | grep "CouchDB index"


 You should see a result that looks like the following:
 你会看到类似下面的结果：

 ::

    [couchdb] CreateIndex -> INFO 0be Created CouchDB index [indexOwner] in state database [mychannel_marbles] using design document [_design/indexOwnerDoc]

 .. note:: If Marbles was not installed on the BYFN peer ``peer0.org1.example.com``,
          you may need to replace it with the name of a different peer where
          Marbles was installed.
          如果在BYFN 的peer ``peer0.org1.example.com``上没有安装Marbles，您可能需要将其替换为安装了Marbles的其他peer节点的名称。

.. _cdb-query:

Query the CouchDB State Database-查询CouchDB状态数据库
​~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now that the index has been defined in the JSON file and deployed alongside
the chaincode, chaincode functions can execute JSON queries against the CouchDB
state database, and thereby peer commands can invoke the chaincode functions.
现在索引已在JSON文件中定义并与链码一起部署，链码函数可以对CouchDB状态数据库执行JSON查询，因此peer命令可以调用链码函数。

Specifying an index name on a query is optional. If not specified, and an index
already exists for the fields being queried, the existing index will be
automatically used.

在查询上指定索引名称是可选的。 如果未指定，并且已查询的字段已存在索引，则将自动使用现有索引。

.. tip:: It is a good practice to explicitly include an index name on a
         query using the ``use_index`` keyword. Without it, CouchDB may pick a
         less optimal index. Also CouchDB may not use an index at all and you
         may not realize it, at the low volumes during testing. Only upon
         higher volumes you may realize slow performance because CouchDB is not
         using an index and you assumed it was.
使用``use_index``关键字在查询中显式包含索引名称是一个好习惯。 没有它，CouchDB可能会选择一个不太理想的索引。 此外，CouchDB可能根本不使用索引，您可能没有意识到它，当测试低容量的时候。 只有在较高的容量的时候，您可能会发现性能较慢，因为CouchDB没有使用索引而您认为它使用了。

Build the query in chaincode-在链码中创建查询
----------------------------

You can perform complex rich queries against the chaincode data values using
the CouchDB JSON query language within chaincode. As we explored above, the
`marbles02 sample chaincode <https://github.com/hyperledger/fabric-samples/blob/master/chaincode/marbles02/go/marbles_chaincode.go>`__
includes an index and rich queries are defined in the functions - ``queryMarbles``
and ``queryMarblesByOwner``:
您可以通过链码中的CouchDB JSON查询语言对链码数据值执行复杂的富查询。 正如我们上面所探讨的那样，`marbles02示例链码<https://github.com/hyperledger/fabric samples / blob / master / chaincode / marbles02 / go / marbles_cha ncode.go>`__包含索引，并且富查询定义于 函数 - ``queryMarbles``和``queryMarblesByOwner``：

  * **queryMarbles** --

      Example of an **ad hoc rich query**. This is a query
      where a (selector) string can be passed into the function. This query
      would be useful to client applications that need to dynamically build
      their own selectors at runtime. For more information on selectors refer
      to `CouchDB selector syntax <http://docs.couchdb.org/en/latest/api/database/find.html#find-selectors>`__.

ad hoc富查询的示例。 这是一个查询，其中（选择器）字符串可以传递给函数。 此查询对于需要在运行时动态构建自己的选择器的客户端应用程序非常有用。 有关选择器的更多信息，请参阅CouchDB选择器语法<http://docs.couchdb.org/en/latest/api/database/find.html#find-selectors>`__。

  * **queryMarblesByOwner** --

      Example of a parameterized query where the
      query logic is baked into the chaincode. In this case the function accepts
      a single argument, the marble owner. It then queries the state database for
      JSON documents matching the docType of “marble” and the owner id using the
      JSON query syntax.
      参数化查询的示例，其中查询逻辑被复制到链代码中。 在这种情况下，函数接受单个参数，即marble的所有者。 然后，它使用JSON查询语法在状态数据库中查询与“marble”的docType和所有者id匹配的JSON文档。


Run the query using the peer command-使用peer命令执行查询
------------------------------------

In absence of a client application to test rich queries defined in chaincode,
peer commands can be used. Peer commands run from the command line inside the
docker container. We will customize the `peer chaincode query <http://hyperledger-fabric.readthedocs.io/en/master/commands/peerchaincode.html?%20chaincode%20query#peer-chaincode-query>`__
command to use the Marbles index ``indexOwner`` and query for all marbles owned
by "tom" using the ``queryMarbles`` function.

如果没有客户端应用程序来测试链码中定义的富查询，则可以使用peer命令。 peer命令从docker容器内的命令行运行。 我们将自定义`peer chaincode query <http：// hyperledger fabric.readthedocs.io/en/master/commands/peerchaincode.html?%20chaincode%20query#peer-chaincode-query>`__命令来使用Marbles索引 `indexOwner``并使用``queryMarbles``函数查询“tom”拥有的marbles。

 :guilabel:`Try it yourself`

 Before querying the database, we should add some data. Run the following
 command in the peer container to create a marble owned by "tom":
 在查询数据库之前，我们应该添加一些数据。 在对等容器中运行以下命令以创建由“tom”拥有的marble：

 .. code:: bash

   peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble1","blue","35","tom"]}'

 After an index has been deployed during chaincode instantiation, it will
 automatically be utilized by chaincode queries. CouchDB can determine which
 index to use based on the fields being queried. If an index exists for the
 query criteria it will be used. However the recommended approach is to
 specify the ``use_index`` keyword on the query. The peer command below is an
 example of how to specify the index explicitly in the selector syntax by
 including the ``use_index`` keyword:
 
 在链码实例化期间部署索引之后，链码查询将自动使用它。 CouchDB可以根据要查询的字段确定要使用的索引。 如果查询条件存在索引，则将使用该索引。 但是，建议的方法是在查询中指定``use_index``关键字。 下面的peer命令是一个如何通过包含``use_index``关键字在选择器语法中显式指定索引的示例：

 .. code:: bash

   // Rich Query with index name explicitly specified:
   peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarbles", "{\"selector\":{\"docType\":\"marble\",\"owner\":\"tom\"}, \"use_index\":[\"_design/indexOwnerDoc\", \"indexOwner\"]}"]}'

Delving into the query command above, there are three arguments of interest:
深入研究上面的查询命令，有三个感兴趣的参数：

*  ``queryMarbles``
  Name of the function in the Marbles chaincode. Notice a `shim <https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim>`__
  ``shim.ChaincodeStubInterface`` is used to access and modify the ledger. The
  ``getQueryResultForQueryString()`` passes the queryString to the shim API ``getQueryResult()``.
  
  Marbles链码中函数的名称。 注意一下`shim <https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim>`__ ``shim.ChaincodeStubInterface``用于访问和修改账本。该 ``getQueryResultForQueryString（）``将queryString传递给shim API``getQueryResult（）``。

.. code:: bash

  func (t *SimpleChaincode) queryMarbles(stub shim.ChaincodeStubInterface, args []string) pb.Response {

	  //   0
	  // "queryString"
	   if len(args) < 1 {
		   return shim.Error("Incorrect number of arguments. Expecting 1")
	   }

	   queryString := args[0]

	   queryResults, err := getQueryResultForQueryString(stub, queryString)
	   if err != nil {
		 return shim.Error(err.Error())
	   }
	   return shim.Success(queryResults)
  }

*  ``{"selector":{"docType":"marble","owner":"tom"}``
  This is an example of an **ad hoc selector** string which finds all documents
  of type ``marble`` where the ``owner`` attribute has a value of ``tom``.
  这是一个** ad hoc selector **字符串的示例，它查找所有类型为``marble``的文档，其中``owner``属性的值为``tom``。


*  ``"use_index":["_design/indexOwnerDoc", "indexOwner"]``
  Specifies both the design doc name  ``indexOwnerDoc`` and index name
  ``indexOwner``. In this example the selector query explicitly includes the
  index name, specified by using the ``use_index`` keyword. Recalling the
  index definition above :ref:`CreateAnIndex`, it contains a design doc,
  ``"ddoc":"indexOwnerDoc"``. With CouchDB, if you plan to explicitly include
  the index name on the query, then the index definition must include the
  ``ddoc`` value, so it can be referenced with the ``use_index`` keyword.
  
  指定设计文档名称``indexOwnerDoc``和索引名称``indexOwner``。 在此示例中，选择器查询显式包含使用``use_index``关键字指定的索引名称。 回顾上面的索引定义：ref：`CreateAnIndex`，它包含一个设计文档，``“ddoc”：“indexOwnerDoc”``。 使用CouchDB，如果您计划在查询中明确包含索引名称，那么索引定义必须包含``ddoc``值，因此可以使用``use_index``关键字引用它。


The query runs successfully and the index is leveraged with the following results:
查询成功运行，并利用索引得到以下结果：

.. code:: json

  Query Result: [{"Key":"marble1", "Record":{"color":"blue","docType":"marble","name":"marble1","owner":"tom","size":35}}]

.. _cdb-update-index:

Update an Index-更新索引
​~~~~~~~~~~~~~~~

It may be necessary to update an index over time. The same index may exist in
subsequent versions of the chaincode that gets installed. In order for an index
to be updated, the original index definition must have included the design
document ``ddoc`` attribute and an index name. To update an index definition,
use the same index name but alter the index definition. Simply edit the index
JSON file and add or remove fields from the index. Fabric only supports the
index type JSON, changing the index type is not supported. The updated
index definition gets redeployed to the peer’s state database when the chaincode
is installed and instantiated. Changes to the index name or ``ddoc`` attributes
will result in a new index being created and the original index remains
unchanged in CouchDB until it is removed.
随着时间推移可能需要更新索引。 安装的链码的后续版本中可能存在相同的索引。 为了更新索引，原始索引定义必须包含设计文档ddoc属性和索引名称。 要更新索引定义，请使用相同的索引名称，但更改索引定义。 只需编辑索引JSON文件，然后在索引中添加或删除字段。 Fabric仅支持索引类型JSON，不支持更改索引类型。 在安装和实例化链代码时，更新的索引定义将重新部署到对等方的状态数据库。 对索引名称或ddoc属性的更改将导致创建新索引，并且原始索引在CouchDB中保持不变，直到将其删除。

.. note:: If the state database has a significant volume of data, it will take
          some time for the index to be re-built, during which time chaincode
          invokes that issue queries may fail or timeout.
          如果状态数据库具有大量数据，则重建索引将花费一些时间，在此期间链代码调用查询可能失败或超时。

Iterating on your index definition - 迭代索引定义
----------------------------------

If you have access to your peer's CouchDB state database in a development
environment, you can iteratively test various indexes in support of
your chaincode queries. Any changes to chaincode though would require
redeployment. Use the `CouchDB Fauxton interface <http://docs.couchdb.org/en/latest/fauxton/index.html>`__ or a command
line curl utility to create and update indexes.

如果您可以在开发环境中访问peer的CouchDB状态数据库，则可以迭代测试各种索引以支持您的链码查询。 但是，对链码的任何更改都需要重新部署。 使用CouchDB Fauxton接口 <http://docs.couchdb.org/en/latest/fauxton/index.html>`__或命令行curl实用程序来创建和更新索引。

.. note:: The Fauxton interface is a web UI for the creation, update, and
          deployment of indexes to CouchDB. If you want to try out this
          interface, there is an example of the format of the Fauxton version
          of the index in Marbles sample. If you have deployed the BYFN network
          with CouchDB, the Fauxton interface can be loaded by opening a browser
          and navigating to ``http://localhost:5984/_utils``.
Fauxton接口是用于创建，更新和部署CouchDB索引的Web UI。 如果你想试试这个界面，有一个Marbles样本中索引的Fauxton版本格式的例子。 如果已使用CouchDB部署BYFN网络，则可以通过打开浏览器并导航到http：// localhost：5984 / _utils来加载Fauxton接口。

Alternatively, if you prefer not use the Fauxton UI, the following is an example
of a curl command which can be used to create the index on the database
``mychannel_marbles``:
或者，如果您不想使用Fauxton UI，则以下是curl命令的示例，该命令可用于在数据库“mychannel_marbles”上创建索引：
     // Index for docType, owner.
     // Example curl command line to define index in the CouchDB channel_chaincode database

.. code:: bash

   curl -i -X POST -H "Content-Type: application/json" -d
          "{\"index\":{\"fields\":[\"docType\",\"owner\"]},
            \"name\":\"indexOwner\",
            \"ddoc\":\"indexOwnerDoc\",
            \"type\":\"json\"}" http://hostname:port/mychannel_marbles/_index

.. note:: If you are using BYFN configured with CouchDB, replace hostname:port
	  with ``localhost:5984``.

.. _cdb-delete-index:

Delete an Index-删除索引
​~~~~~~~~~~~~~~~

Index deletion is not managed by Fabric tooling. If you need to delete an index,
manually issue a curl command against the database or delete it using the
Fauxton interface.
索引删除不受Fabric工具管理。 如果需要删除索引，请手动向数据库发出curl命令或使用Fauxton接口将其删除。

The format of the curl command to delete an index would be:
删除索引的curl命令的格式为：

.. code:: bash

   curl -X DELETE http://localhost:5984/{database_name}/_index/{design_doc}/json/{index_name} -H  "accept: */*" -H  "Host: localhost:5984"


To delete the index used in this tutorial, the curl command would be:
要删除本教程中使用的索引，curl命令将是：

.. code:: bash

   curl -X DELETE http://localhost:5984/mychannel_marbles/_index/indexOwnerDoc/json/indexOwner -H  "accept: */*" -H  "Host: localhost:5984"



.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/

~~~~~~~~~~~~