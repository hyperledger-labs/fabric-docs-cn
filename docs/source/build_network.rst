Building Your First Network-创建你的第一个fabric网络
======================================================

.. note:: These instructions have been verified to work against the
​          latest stable Docker images and the pre-compiled
​          setup utilities within the supplied tar file. If you run
​          these commands with images or tools from the current master
​          branch, it is possible that you will see configuration and panic
​          errors.
​
​		  
.. note::

  这些说明已经被验证，它可以在最新稳定版的docker镜像和提供tar文件的预编译的安装实用程序中工作。如果你在当前分支下，通过镜像或者工具使用这些命令，可能会有一些配置或者panic错误。

The build your first network (BYFN) scenario provisions a sample Hyperledger
Fabric network consisting of two organizations, each maintaining two peer
nodes, and a "solo" ordering service.


在你构建的第一个网络(BYFN)场景中，提供了一个包含两个组织的Hyperledger Fabric网络。每个组织包含两个peer节点，一个"solo"模式的排序服务。

Install prerequisites  - 安装准备
----------------------------------

Before we begin, if you haven't already done so, you may wish to check that
you have all the :doc:`prereqs` installed on the platform(s)
on which you'll be developing blockchain applications and/or operating
Hyperledger Fabric.

在我们开始之前，如果你什么都没做，你也许应该在你想要部署学习区块链和/或者操作Hyperledger Fabric网络的平台上，检查你是否做了预置环境的安装 :doc:`prereqs`。

You will also need to :doc:`install`. You will notice
that there are a number of samples included in the ``fabric-samples``
repository. We will be using the ``first-network`` sample. Let's open that
sub-directory now.

你还需要去安装一些示例，二进制文件和docker镜像 :doc:`install`。`fabric-samples`中包含了许多示例。我们将使用 `first-network`作为例子。现在我们一起打开这个子目录。

.. code:: bash

  cd fabric-samples/first-network

.. note:: The supplied commands in this documentation
​          **MUST** be run from your ``first-network`` sub-directory
​          of the ``fabric-samples`` repository clone.  If you elect to run the
​          commands from a different location, the various provided scripts
​          will be unable to find the binaries.




.. note:: ​	
这个文档里提供的命令都需要运行在你克隆的``fabric-samples``项目的子目录 ``first-network``里。如果你选择从不同的位置运行命令，提供的那些脚本将无法找到二进制文件。

Want to run it now?-想要现在运行吗？
-------------------

We provide a fully annotated script - ``byfn.sh`` - that leverages these Docker
images to quickly bootstrap a Hyperledger Fabric network comprised of 4 peers
representing two different organizations, and an orderer node. It will also
launch a container to run a scripted execution that will join peers to a
channel, deploy and instantiate chaincode and drive execution of transactions
against the deployed chaincode.

我们提供了一个被全部注释的脚本 - ``byfn.sh`` - 它可以通过镜像快速启动一个Hyperledger Fabric网络，这个网络由代表两个组织的四个peer节点，一个排序节点组成。它还将启动一个容器用于运行一个将peer节点加入channel、部署并且实例化链码服务以及驱动已经部署的链码执行交易的脚本。

Here's the help text for the ``byfn.sh`` script:

以下是该脚本 ``byfn.sh``的帮助文档：

.. code:: bash

  Usage:
​    byfn.sh <mode> [-c <channel name>] [-t <timeout>] [-d <delay>] [-f <docker-compose-file>] [-s <dbtype>] [-l <language>] [-i <imagetag>] [-v]
​      <mode> - one of 'up', 'down', 'restart', 'generate' or 'upgrade'
​        - 'up' - bring up the network with docker-compose up
​        - 'down' - clear the network with docker-compose down
​        - 'restart' - restart the network
​        - 'generate' - generate required certificates and genesis block
​        - 'upgrade'  - upgrade the network from v1.0.x to v1.1
​      -c <channel name> - channel name to use (defaults to "mychannel")
​      -t <timeout> - CLI timeout duration in seconds (defaults to 10)
​      -d <delay> - delay duration in seconds (defaults to 3)
​      -f <docker-compose-file> - specify which docker-compose file use (defaults to docker-compose-cli.yaml)
​      -s <dbtype> - the database backend to use: goleveldb (default) or couchdb
​      -l <language> - the chaincode language: golang (default) or node
​      -i <imagetag> - the tag to be used to launch the network (defaults to "latest")
​      -v - verbose mode
​    byfn.sh -h (print this message)

  Typically, one would first generate the required certificates and
  genesis block, then bring up the network. e.g.:

	  byfn.sh generate -c mychannel
	  byfn.sh up -c mychannel -s couchdb
	      byfn.sh up -c mychannel -s couchdb -i 1.1.0-alpha
	  byfn.sh up -l node
	  byfn.sh down -c mychannel
	      byfn.sh upgrade -c mychannel

  Taking all defaults:
​	  byfn.sh generate
​	  byfn.sh up
​	  byfn.sh down

If you choose not to supply a channel name, then the
script will use a default name of ``mychannel``.  The CLI timeout parameter
(specified with the -t flag) is an optional value; if you choose not to set
it, then the CLI will give up on query requests made after the default
setting of 10 seconds.

如果你选择不提供通道名称，脚本会使用默认的通道名称mychannel。CLI的超时参数(用-t标志标识)是可选的.如果你不设置它，Cli 会放弃在默认设置的十秒之后进行查询请求

Generate Network Artifacts -生成网络构件
^^^^^^^^^^^^^^^^^^^^^^^^^^

Ready to give it a go? Okay then! Execute the following command:

准备好了没？OK，执行下面的命令：

.. code:: bash

  ./byfn.sh generate

You will see a brief description as to what will occur, along with a yes/no command line
prompt. Respond with a ``y`` or hit the return key to execute the described action.

伴随命令行提示yes/no，你会看到将要发生什么的一些简要说明。输入Y或者返回键来执行描述的动作。

.. code:: bash

  Generating certs and genesis block for with channel 'mychannel' and CLI timeout of '10'
  Continue? [Y/n] y
  proceeding ...
  /Users/xxx/dev/fabric-samples/bin/cryptogen

  ##########################################################
  ##### Generate certificates using cryptogen tool #########
  ##########################################################
  org1.example.com
  2017-06-12 21:01:37.334 EDT [bccsp] GetDefault -> WARN 001 Before using BCCSP, please call InitFactories(). Falling back to bootBCCSP.
  ...

  /Users/xxx/dev/fabric-samples/bin/configtxgen
  ##########################################################
  #########  Generating Orderer Genesis block ##############
  ##########################################################
  2017-06-12 21:01:37.558 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
  2017-06-12 21:01:37.562 EDT [msp] getMspConfig -> INFO 002 intermediate certs folder not found at [/Users/xxx/dev/byfn/crypto-config/ordererOrganizations/example.com/msp/intermediatecerts]. Skipping.: [stat /Users/xxx/dev/byfn/crypto-config/ordererOrganizations/example.com/msp/intermediatecerts: no such file or directory]
  ...
  2017-06-12 21:01:37.588 EDT [common/configtx/tool] doOutputBlock -> INFO 00b Generating genesis block
  2017-06-12 21:01:37.590 EDT [common/configtx/tool] doOutputBlock -> INFO 00c Writing genesis block

  #################################################################
  ### Generating channel configuration transaction 'channel.tx' ###
  #################################################################
  2017-06-12 21:01:37.634 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
  2017-06-12 21:01:37.644 EDT [common/configtx/tool] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
  2017-06-12 21:01:37.645 EDT [common/configtx/tool] doOutputChannelCreateTx -> INFO 003 Writing new channel tx

  #################################################################
  #######    Generating anchor peer update for Org1MSP   ##########
  #################################################################
  2017-06-12 21:01:37.674 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
  2017-06-12 21:01:37.678 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
  2017-06-12 21:01:37.679 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update

  #################################################################
  #######    Generating anchor peer update for Org2MSP   ##########
  #################################################################
  2017-06-12 21:01:37.700 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
  2017-06-12 21:01:37.704 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
  2017-06-12 21:01:37.704 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update

This first step generates all of the certificates and keys for our various
network entities, the ``genesis block`` used to bootstrap the ordering service,
and a collection of configuration transactions required to configure a
:ref:`Channel`.

第一步为我们各种网络实体生成证书和秘钥。初始区块 ``genesis block``用于引导排序服务，也包含了一组用于配置 :ref:`Channel`所需要的一组配置交易集合。

Bring Up the Network - 启动网络
^^^^^^^^^^^^^^^^^^^^

Next, you can bring the network up with one of the following commands:

接下来，你可以用下面的命令启动网络：

.. code:: bash

  ./byfn.sh up

The above command will compile Golang chaincode images and spin up the corresponding
containers.  Go is the default chaincode language, however there is also support
for `Node.js <https://fabric-shim.github.io/>`__ chaincode.  If you'd like to run through this tutorial with node
chaincode, pass the following command instead:



上面的命令会编译Golang智能合约的镜像并且在对应的镜像中启动。Go语言是默认的智能合约语言，但是它也支持Node.js`Node.js <https://fabric-shim.github.io/>`.如果你想要在这个教程里运行node智能合约，你可以通过下面的命令替代：

.. code:: bash

  # we use the -l flag to specify the chaincode language
  # forgoing the -l flag will default to Golang

  ./byfn.sh up -l node

.. note:: View the `Hyperledger Fabric Shim <https://fabric-shim.github.io/ChaincodeStub.html>`__
​          documentation for more info on the node.js chaincode shim APIs.

.. note::

​	查看 `Hyperledger Fabric Shim <https://fabric-shim.github.io/ChaincodeStub.html>` 文档获取更多关于node.js 智能合约的 shim API 信息。

Once again, you will be prompted as to whether you wish to continue or abort.
Respond with a ``y`` or hit the return key:

再一次，您将被提示是否要继续或中止。用y或者按下返回键表示响应。

.. code:: bash

  Starting with channel 'mychannel' and CLI timeout of '10'
  Continue? [Y/n]
  proceeding ...
  Creating network "net_byfn" with the default driver
  Creating peer0.org1.example.com
  Creating peer1.org1.example.com
  Creating peer0.org2.example.com
  Creating orderer.example.com
  Creating peer1.org2.example.com
  Creating cli

____    _____      _      ____    _____
  / ___|  |_   _|    / \    |  _ \  |_   _|
  \___ \    | |     / _ \   | |_) |   | |
   ___) |   | |    / ___ \  |  _ <    | |
  |____/    |_|   /_/   \_\ |_| \_\   |_|

  Channel name : mychannel
  Creating channel...

The logs will continue from there. This will launch all of the containers, and
then drive a complete end-to-end application scenario. Upon successful
completion, it should report the following in your terminal window:

日志会从那里继续。这一步会启动所有的容器，然后驱动一个完整的 end-to-end 应用场景。完成后，它应该在您的终端窗口中报告以下内容:



.. code:: bash

    Query Result: 90
    2017-05-16 17:08:15.158 UTC [main] main -> INFO 008 Exiting.....
    ===================== Query successful on peer1.org2 on channel 'mychannel' =====================
    
    ===================== All GOOD, BYFN execution completed =====================


     _____   _   _   ____
    | ____| | \ | | |  _ \
    |  _|   |  \| | | | | |
    | |___  | |\  | | |_| |
    |_____| |_| \_| |____/

You can scroll through these logs to see the various transactions. If you don't
get this result, then jump down to the :ref:`Troubleshoot` section and let's see
whether we can help you discover what went wrong.



你可以滚动这些日志去查看各种交易。如果你没有获得这个结果，请移步疑难解答部分 :ref:`Troubleshoot`，看看我们是否可以帮助你发现问题。



Bring Down the Network-关闭网络
^^^^^^^^^^^^^^^^^^^^^^

Finally, let's bring it all down so we can explore the network setup one step
at a time. The following will kill your containers, remove the crypto material
and four artifacts, and delete the chaincode images from your Docker Registry:

最后，让我们把他停下来，这样我们可以一步步探索网络设置。接下来的命令会结束掉你所有的容器，移除加密的材料和4个配置信息。并且从Docker仓库删除chinacode镜像。

.. code:: bash

  ./byfn.sh down

Once again, you will be prompted to continue, respond with a ``y`` or hit the return key:

再一次，您将被提示是否要继续或中止。用y或者按下返回键表示响应。

.. code:: bash

  Stopping with channel 'mychannel' and CLI timeout of '10'
  Continue? [Y/n] y
  proceeding ...
  WARNING: The CHANNEL_NAME variable is not set. Defaulting to a blank string.
  WARNING: The TIMEOUT variable is not set. Defaulting to a blank string.
  Removing network net_byfn
  468aaa6201ed
  ...
  Untagged: dev-peer1.org2.example.com-mycc-1.0:latest
  Deleted: sha256:ed3230614e64e1c83e510c0c282e982d2b06d148b1c498bbdcc429e2b2531e91
  ...

If you'd like to learn more about the underlying tooling and bootstrap mechanics,
continue reading.  In these next sections we'll walk through the various steps
and requirements to build a fully-functional Hyperledger Fabric network.

如果你想要了解更多关于底层工具和引导材料的信息，继续阅读。在接下来的章节，我们将浏览构建一个功能完整的Hyperledger Fabric 网络的各个步骤和要求。

.. note:: The manual steps outlined below assume that the ``CORE_LOGGING_LEVEL`` in
​          the ``cli`` container is set to ``DEBUG``. You can set this by modifying
​          the ``docker-compose-cli.yaml`` file in the ``first-network`` directory.
​          e.g.

.. note:: 

​	下面列出的手动步骤设置假想在 ``cli``容器中的 ``CORE_LOGGING_LEVEL``设置为``DEBUG``。你可以通过编辑 在``first-network``中的``docker-compose-cli.yaml``文件来设置他。

​		

          .. code::
    
            cli:
              container_name: cli
              image: hyperledger/fabric-tools:$IMAGE_TAG
              tty: true
              stdin_open: true
              environment:
                - GOPATH=/opt/gopath
                - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
                - CORE_LOGGING_LEVEL=DEBUG
                #- CORE_LOGGING_LEVEL=INFO

Crypto Generator - 加密生成器
----------------

We will use the ``cryptogen`` tool to generate the cryptographic material
(x509 certs and signing keys) for our various network entities.  These certificates are
representative of identities, and they allow for sign/verify authentication to
take place as our entities communicate and transact.

我们将使用``cryptogen``工具生成各种网络实体的加密材料（x509证书）。这些证书是身份的代表，在实体之间交流和交易的时候，它们允许对身份验证进行签名/验证。

How does it work? - 它是怎么工作的？
^^^^^^^^^^^^^^^^^

Cryptogen consumes a file - ``crypto-config.yaml`` - that contains the network
topology and allows us to generate a set of certificates and keys for both the
Organizations and the components that belong to those Organizations.  Each
Organization is provisioned a unique root certificate (``ca-cert``) that binds
specific components (peers and orderers) to that Org.  By assigning each
Organization a unique CA certificate, we are mimicking a typical network where
a participating :ref:`Member` would use its own Certificate Authority.
Transactions and communications within Hyperledger Fabric are signed by an
entity's private key (``keystore``), and then verified by means of a public
key (``signcerts``).



Cryptogen 通过一个包含网络拓扑的文件``crypto-config.yaml``，为所有组织和属于这些组织的组件生成一组证书和秘钥。每一个组织被分配一个唯一的根证书(``ca-cert``)，它绑定该组织的特定组件(peers and orderers)。通过为每个组织分配一个惟一的CA证书，我们模拟了一个典型的网络，一个参与成员 :ref:`Member`将使用它自己的认证授权。超级账本中的事务和通信是由一个实体的私钥（(``keystore``）签名的，然后通过公钥（``signcerts``）验证。

You will notice a ``count`` variable within this file.  We use this to specify
the number of peers per Organization; in our case there are two peers per Org.
We won't delve into the minutiae of `x.509 certificates and public key
infrastructure <https://en.wikipedia.org/wiki/Public_key_infrastructure>`__
right now. If you're interested, you can peruse these topics on your own time.

在这个文件里你会发现一个 ``count``变量。我们通过它来指定每个组织的peer节点数量。在我们的案例里每隔组织有两个peer节点。我们现在不会深入研究`x.509 certificates and public key
infrastructure <https://en.wikipedia.org/wiki/Public_key_infrastructure>`__细节。如果你有兴趣，你可以在自己的时间细读这些主题。

Before running the tool, let's take a quick look at a snippet from the
``crypto-config.yaml``. Pay specific attention to the "Name", "Domain"
and "Specs" parameters under the ``OrdererOrgs`` header:

在运行该工具之前，我们快速浏览一下``crypto-config.yaml``的一段代码。特别注意``OrdererOrgs`` 头结点下“Name”，Domain"和 "Specs"参数。

.. code:: bash

  OrdererOrgs:
  #---------------------------------------------------------

  # Orderer
  # --------------------------------------------------------
  - Name: Orderer
    Domain: example.com
    CA:
    ​    Country: US
    ​    Province: California
    ​    Locality: San Francisco
    #   OrganizationalUnit: Hyperledger Fabric
    #   StreetAddress: address for org # default nil
    #   PostalCode: postalCode for org # default nil
    # ------------------------------------------------------
    # "Specs" - See PeerOrgs below for complete description
  # -----------------------------------------------------
    Specs:
      - Hostname: orderer
  # -------------------------------------------------------
  # "PeerOrgs" - Definition of organizations managing peer nodes
   # ------------------------------------------------------
  PeerOrgs:
  # -----------------------------------------------------
  # Org1
  # ----------------------------------------------------
  - Name: Org1
    Domain: org1.example.com
    EnableNodeOUs: true

The naming convention for a network entity is as follows -
"{{.Hostname}}.{{.Domain}}".  So using our ordering node as a
reference point, we are left with an ordering node named -
``orderer.example.com`` that is tied to an MSP ID of ``Orderer``.  This file
contains extensive documentation on the definitions and syntax.  You can also
refer to the :doc:`msp` documentation for a deeper dive on MSP.



网络实体的命名约定如下:“{{. hostname}}.{{. domain}}”。因此，使用我们的order节点作为参考点，我们只剩下一个order节点—``orderer.example.com``，它与Orderer的MSP ID绑定在一起。

After we run the ``cryptogen`` tool, the generated certificates and keys will be
saved to a folder titled ``crypto-config``.

在我们运行``cryptogen``工具之后，生成的证书和密钥将是保存到一个名为``crypto-config``的文件夹中。

Configuration Transaction Generator - 配置交易生成器
-----------------------------------

The ``configtxgen tool`` is used to create four configuration artifacts:

  * orderer ``genesis block``,
  * channel ``configuration transaction``,
  * and two ``anchor peer transactions`` - one for each Peer Org.

 ``configtxgen tool`` 用来创建四个配置构件:

- order节点的初始区块 ``genesis block``,
- 通道配置事务``configuration transaction``,
- 两个锚节点交易 ``anchor peer transactions`` - 一个对应一个Peer组织。

Please see :doc:`commands/configtxgen` for a complete description of this tool's functionality.

有关此工具的完整说明，请参阅 :doc:`commands/configtxgen`

The orderer block is the :ref:`Genesis-Block` for the ordering service, and the
channel configuration transaction file is broadcast to the orderer at :ref:`Channel` creation
time.  The anchor peer transactions, as the name might suggest, specify each
Org's :ref:`Anchor-Peer` on this channel.



order block 是 排序服务的初始区块`Genesis-Block`，channel configuration transaction在 :ref:`Channel`创建的时候广播给排序服务。 anchor peer transactions，正如名称所示，指定了每个组织在此channel上的:ref:`Anchor-Peer`

How does it work? -它是怎么工作的？
^^^^^^^^^^^^^^^^^

Configtxgen consumes a file - ``configtx.yaml`` - that contains the definitions
for the sample network. There are three members - one Orderer Org (``OrdererOrg``)
and two Peer Orgs (``Org1`` & ``Org2``) each managing and maintaining two peer nodes.
This file also specifies a consortium - ``SampleConsortium`` - consisting of our
two Peer Orgs.  Pay specific attention to the "Profiles" section at the top of
this file.  You will notice that we have two unique headers. One for the orderer genesis
block - ``TwoOrgsOrdererGenesis`` - and one for our channel - ``TwoOrgsChannel``.



Configtxgen 使用一个文件- ``configtx.yaml``，这个文件包含了一个示例网络的定义。它拥有三个成员：一个Order组织（``OrdererOrg``） 和两个 Peer 组织(``Org1`` & ``Org2``)，这两个peer组织每个都管理和维护两个peer节点。

These headers are important, as we will pass them in as arguments when we create
our artifacts.



这些标题很重要，因为在我们创建我们的网络各项构件的时侯它们将作为传递的参数。

.. note:: Notice that our ``SampleConsortium`` is defined in
​          the system-level profile and then referenced by
​          our channel-level profile.  Channels exist within
​          the purview of a consortium, and all consortia
​          must be defined in the scope of the network at
​          large.



.. note::

​	注意我们的 ``SampleConsortium`` 在系统级配置文件中定义，并且在通道级的配置文件中关联引用。管道存在于联盟的范围内，所有的联盟必须定义在整个网络范围内。

This file also contains two additional specifications that are worth
noting. Firstly, we specify the anchor peers for each Peer Org
(``peer0.org1.example.com`` & ``peer0.org2.example.com``).  Secondly, we point to
the location of the MSP directory for each member, in turn allowing us to store the
root certificates for each Org in the orderer genesis block.  This is a critical
concept. Now any network entity communicating with the ordering service can have
its digital signature verified.



该文件还包含两个值得注意的附加规范。第一，我们为每个组织指定了锚节点（``peer0.org1.example.com`` & ``peer0.org2.example.com``）。第二，我们为每个成员指定MSP文件位置，进而让我们可以在order的初始区块中存储每个组织的根证书。这是一个关键概念。现在每个和order service 服务通信的网络实体都有它自己的被验证过的数字签名证书。





Run the tools
-------------

You can manually generate the certificates/keys and the various configuration
artifacts using the ``configtxgen`` and ``cryptogen`` commands. Alternately,
you could try to adapt the byfn.sh script to accomplish your objectives.

Manually generate the artifacts
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can refer to the ``generateCerts`` function in the byfn.sh script for the
commands necessary to generate the certificates that will be used for your
network configuration as defined in the ``crypto-config.yaml`` file. However,
for the sake of convenience, we will also provide a reference here.

First let's run the ``cryptogen`` tool.  Our binary is in the ``bin``
directory, so we need to provide the relative path to where the tool resides.

.. code:: bash

    ../bin/cryptogen generate --config=./crypto-config.yaml

You should see the following in your terminal:

.. code:: bash

  org1.example.com
  org2.example.com

The certs and keys (i.e. the MSP material) will be output into a directory - ``crypto-config`` -
at the root of the ``first-network`` directory.

Next, we need to tell the ``configtxgen`` tool where to look for the
``configtx.yaml`` file that it needs to ingest.  We will tell it look in our
present working directory:

.. code:: bash

    export FABRIC_CFG_PATH=$PWD

Then, we'll invoke the ``configtxgen`` tool to create the orderer genesis block:

.. code:: bash

    ../bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block

You should see an output similar to the following in your terminal:

.. code:: bash

  2017-10-26 19:21:56.301 EDT [common/tools/configtxgen] main -> INFO 001 Loading configuration
  2017-10-26 19:21:56.309 EDT [common/tools/configtxgen] doOutputBlock -> INFO 002 Generating genesis block
  2017-10-26 19:21:56.309 EDT [common/tools/configtxgen] doOutputBlock -> INFO 003 Writing genesis block

.. note:: The orderer genesis block and the subsequent artifacts we are about to create
​          will be output into the ``channel-artifacts`` directory at the root of this
​          project.

.. _createchanneltx:

Create a Channel Configuration Transaction
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Next, we need to create the channel transaction artifact. Be sure to replace ``$CHANNEL_NAME`` or
set ``CHANNEL_NAME`` as an environment variable that can be used throughout these instructions:

.. code:: bash

    # The channel.tx artifact contains the definitions for our sample channel
    
    export CHANNEL_NAME=mychannel  && ../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME

You should see an output similar to the following in your terminal:

.. code:: bash

  2017-10-26 19:24:05.324 EDT [common/tools/configtxgen] main -> INFO 001 Loading configuration
  2017-10-26 19:24:05.329 EDT [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
  2017-10-26 19:24:05.329 EDT [common/tools/configtxgen] doOutputChannelCreateTx -> INFO 003 Writing new channel tx

Next, we will define the anchor peer for Org1 on the channel that we are
constructing. Again, be sure to replace ``$CHANNEL_NAME`` or set the environment variable
for the following commands.  The terminal output will mimic that of the channel transaction artifact:

.. code:: bash

    ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP

Now, we will define the anchor peer for Org2 on the same channel:

.. code:: bash

    ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP

Start the network
-----------------

.. note:: If you ran the ``byfn.sh`` example above previously, be sure that you
​          have brought down the test network before you proceed (see
​          `Bring Down the Network`_).

We will leverage a script to spin up our network. The
docker-compose file references the images that we have previously downloaded,
and bootstraps the orderer with our previously generated ``genesis.block``.

We want to go through the commands manually in order to expose the
syntax and functionality of each call.

First let's start our network:

.. code:: bash

    docker-compose -f docker-compose-cli.yaml up -d

If you want to see the realtime logs for your network, then do not supply the ``-d`` flag.
If you let the logs stream, then you will need to open a second terminal to execute the CLI calls.

.. _peerenvvars:

Environment variables
^^^^^^^^^^^^^^^^^^^^^

For the following CLI commands against ``peer0.org1.example.com`` to work, we need
to preface our commands with the four environment variables given below.  These
variables for ``peer0.org1.example.com`` are baked into the CLI container,
therefore we can operate without passing them.  **HOWEVER**, if you want to send
calls to other peers or the orderer, then you can provide these
values accordingly by editing the  ``docker-compose-base.yaml`` before starting the
container. Modify the following four environment variables to use a different
peer and org.

.. code:: bash

    # Environment variables for PEER0
    
    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    CORE_PEER_LOCALMSPID="Org1MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt

.. _createandjoin:

Create & Join Channel
^^^^^^^^^^^^^^^^^^^^^

Recall that we created the channel configuration transaction using the
``configtxgen`` tool in the :ref:`createchanneltx` section, above. You can
repeat that process to create additional channel configuration transactions,
using the same or different profiles in the ``configtx.yaml`` that you pass
to the ``configtxgen`` tool. Then you can repeat the process defined in this
section to establish those other channels in your network.

We will enter the CLI container using the ``docker exec`` command:

.. code:: bash

        docker exec -it cli bash

If successful you should see the following:

.. code:: bash

        root@0d78bb69300d:/opt/gopath/src/github.com/hyperledger/fabric/peer#

If you do not want to run the CLI commands against the default peer
``peer0.org1.example.com``, replace the values of ``peer0`` or ``org1`` in the
four environment variables and run the commands:

.. code:: bash

    # Environment variables for PEER0
    
    export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    export CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    export CORE_PEER_LOCALMSPID="Org1MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt

Next, we are going to pass in the generated channel configuration transaction
artifact that we created in the :ref:`createchanneltx` section (we called
it ``channel.tx``) to the orderer as part of the create channel request.

We specify our channel name with the ``-c`` flag and our channel configuration
transaction with the ``-f`` flag. In this case it is ``channel.tx``, however
you can mount your own configuration transaction with a different name.  Once again
we will set the ``CHANNEL_NAME`` environment variable within our CLI container so that
we don't have to explicitly pass this argument. Channel names must be all lower
case, less than 250 characters long and match the regular expression
``[a-z][a-z0-9.-]*``.


.. code:: bash

        export CHANNEL_NAME=mychannel
    
        # the channel.tx file is mounted in the channel-artifacts directory within your CLI container
        # as a result, we pass the full path for the file
        # we also pass the path for the orderer ca-cert in order to verify the TLS handshake
        # be sure to export or replace the $CHANNEL_NAME variable appropriately
    
        peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

.. note:: Notice the ``--cafile`` that we pass as part of this command.  It is
​          the local path to the orderer's root cert, allowing us to verify the
​          TLS handshake.

This command returns a genesis block - ``<channel-ID.block>`` - which we will use to join the channel.
It contains the configuration information specified in ``channel.tx``  If you have not
made any modifications to the default channel name, then the command will return you a
proto titled ``mychannel.block``.

.. note:: You will remain in the CLI container for the remainder of
​          these manual commands. You must also remember to preface all commands
​          with the corresponding environment variables when targeting a peer other than
​          ``peer0.org1.example.com``.

Now let's join ``peer0.org1.example.com`` to the channel.

.. code:: bash

        # By default, this joins ``peer0.org1.example.com`` only
        # the <channel-ID.block> was returned by the previous command
        # if you have not modified the channel name, you will join with mychannel.block
        # if you have created a different channel name, then pass in the appropriately named block
    
         peer channel join -b mychannel.block

You can make other peers join the channel as necessary by making appropriate
changes in the four environment variables we used in the :ref:`peerenvvars`
section, above.

Rather than join every peer, we will simply join ``peer0.org2.example.com`` so that
we can properly update the anchor peer definitions in our channel.  Since we are
overriding the default environment variables baked into the CLI container, this full
command will be the following:

.. code:: bash

  CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer channel join -b mychannel.block

Alternatively, you could choose to set these environment variables individually
rather than passing in the entire string.  Once they've been set, you simply need
to issue the ``peer channel join`` command again and the CLI container will act
on behalf of ``peer0.org2.example.com``.

Update the anchor peers
^^^^^^^^^^^^^^^^^^^^^^^

The following commands are channel updates and they will propagate to the definition
of the channel.  In essence, we adding additional configuration information on top
of the channel's genesis block.  Note that we are not modifying the genesis block, but
simply adding deltas into the chain that will define the anchor peers.

Update the channel definition to define the anchor peer for Org1 as ``peer0.org1.example.com``:

.. code:: bash

  peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org1MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

Now update the channel definition to define the anchor peer for Org2 as ``peer0.org2.example.com``.
Identically to the ``peer channel join`` command for the Org2 peer, we will need to
preface this call with the appropriate environment variables.

.. code:: bash

  CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp CORE_PEER_ADDRESS=peer0.org2.example.com:7051 CORE_PEER_LOCALMSPID="Org2MSP" CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/Org2MSPanchors.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

Install & Instantiate Chaincode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note:: We will utilize a simple existing chaincode. To learn how to write
​          your own chaincode, see the :doc:`chaincode4ade` tutorial.

Applications interact with the blockchain ledger through ``chaincode``.  As
such we need to install the chaincode on every peer that will execute and
endorse our transactions, and then instantiate the chaincode on the channel.

First, install the sample Go or Node.js chaincode onto one of the four peer nodes.  These commands
place the specified source code flavor onto our peer's filesystem.

.. note:: You can only install one version of the source code per chaincode name
​          and version.  The source code exists on the peer's file system in the
​          context of chaincode name and version; it is language agnostic.  Similarly
​          the instantiated chaincode container will be reflective of whichever
​          language has been installed on the peer.

**Golang**

.. code:: bash

    # this installs the Go chaincode
    peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go/

**Node.js**

.. code:: bash

    # this installs the Node.js chaincode
    # make note of the -l flag; we use this to specify the language
    peer chaincode install -n mycc -v 1.0 -l node -p /opt/gopath/src/github.com/chaincode/chaincode_example02/node/

Next, instantiate the chaincode on the channel. This will initialize the
chaincode on the channel, set the endorsement policy for the chaincode, and
launch a chaincode container for the targeted peer.  Take note of the ``-P``
argument. This is our policy where we specify the required level of endorsement
for a transaction against this chaincode to be validated.

In the command below you’ll notice that we specify our policy as
``-P "AND ('Org1MSP.peer','Org2MSP.peer')"``. This means that we need
“endorsement” from a peer belonging to Org1 **AND** Org2 (i.e. two endorsement).
If we changed the syntax to ``OR`` then we would need only one endorsement.

**Golang**

.. code:: bash

    # be sure to replace the $CHANNEL_NAME environment variable if you have not exported it
    # if you did not install your chaincode with a name of mycc, then modify that argument as well
    
    peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"

**Node.js**

.. note::  The instantiation of the Node.js chaincode will take roughly a minute.
​           The command is not hanging; rather it is installing the fabric-shim
​           layer as the image is being compiled.

.. code:: bash

    # be sure to replace the $CHANNEL_NAME environment variable if you have not exported it
    # if you did not install your chaincode with a name of mycc, then modify that argument as well
    # notice that we must pass the -l flag after the chaincode name to identify the language
    
    peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -l node -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "AND ('Org1MSP.peer','Org2MSP.peer')"

See the `endorsement
policies <http://hyperledger-fabric.readthedocs.io/en/latest/endorsement-policies.html>`__
documentation for more details on policy implementation.

If you want additional peers to interact with ledger, then you will need to join
them to the channel, and install the same name, version and language of the
chaincode source onto the appropriate peer's filesystem.  A chaincode container
will be launched for each peer as soon as they try to interact with that specific
chaincode.  Again, be cognizant of the fact that the Node.js images will be slower
to compile.

Once the chaincode has been instantiated on the channel, we can forgo the ``l``
flag.  We need only pass in the channel identifier and name of the chaincode.

Query
^^^^^

Let's query for the value of ``a`` to make sure the chaincode was properly
instantiated and the state DB was populated. The syntax for query is as follows:

.. code:: bash

  # be sure to set the -C and -n flags appropriately

  peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

Invoke
^^^^^^

Now let's move ``10`` from ``a`` to ``b``.  This transaction will cut a new block and
update the state DB. The syntax for invoke is as follows:

.. code:: bash

    # be sure to set the -C and -n flags appropriately
    
    peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}'

Query
^^^^^

Let's confirm that our previous invocation executed properly. We initialized the
key ``a`` with a value of ``100`` and just removed ``10`` with our previous
invocation. Therefore, a query against ``a`` should reveal ``90``. The syntax
for query is as follows.

.. code:: bash

  # be sure to set the -C and -n flags appropriately

  peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

We should see the following:

.. code:: bash

   Query Result: 90

Feel free to start over and manipulate the key value pairs and subsequent
invocations.

.. _behind-scenes:

What's happening behind the scenes?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note:: These steps describe the scenario in which
​          ``script.sh`` is run by './byfn.sh up'.  Clean your network
​          with ``./byfn.sh down`` and ensure
​          this command is active.  Then use the same
​          docker-compose prompt to launch your network again

-  A script - ``script.sh`` - is baked inside the CLI container. The
   script drives the ``createChannel`` command against the supplied channel name
   and uses the channel.tx file for channel configuration.

-  The output of ``createChannel`` is a genesis block -
   ``<your_channel_name>.block`` - which gets stored on the peers' file systems and contains
   the channel configuration specified from channel.tx.

-  The ``joinChannel`` command is exercised for all four peers, which takes as
   input the previously generated genesis block.  This command instructs the
   peers to join ``<your_channel_name>`` and create a chain starting with ``<your_channel_name>.block``.

-  Now we have a channel consisting of four peers, and two
   organizations.  This is our ``TwoOrgsChannel`` profile.

-  ``peer0.org1.example.com`` and ``peer1.org1.example.com`` belong to Org1;
   ``peer0.org2.example.com`` and ``peer1.org2.example.com`` belong to Org2

-  These relationships are defined through the ``crypto-config.yaml`` and
   the MSP path is specified in our docker compose.

-  The anchor peers for Org1MSP (``peer0.org1.example.com``) and
   Org2MSP (``peer0.org2.example.com``) are then updated.  We do this by passing
   the ``Org1MSPanchors.tx`` and ``Org2MSPanchors.tx`` artifacts to the ordering
   service along with the name of our channel.

-  A chaincode - **chaincode_example02** - is installed on ``peer0.org1.example.com`` and
   ``peer0.org2.example.com``

-  The chaincode is then "instantiated" on ``peer0.org2.example.com``. Instantiation
   adds the chaincode to the channel, starts the container for the target peer,
   and initializes the key value pairs associated with the chaincode.  The initial
   values for this example are ["a","100" "b","200"]. This "instantiation" results
   in a container by the name of ``dev-peer0.org2.example.com-mycc-1.0`` starting.

-  The instantiation also passes in an argument for the endorsement
   policy. The policy is defined as
   ``-P "AND ('Org1MSP.peer','Org2MSP.peer')"``, meaning that any
   transaction must be endorsed by a peer tied to Org1 and Org2.

-  A query against the value of "a" is issued to ``peer0.org1.example.com``. The
   chaincode was previously installed on ``peer0.org1.example.com``, so this will start
   a container for Org1 peer0 by the name of ``dev-peer0.org1.example.com-mycc-1.0``. The result
   of the query is also returned. No write operations have occurred, so
   a query against "a" will still return a value of "100".

-  An invoke is sent to ``peer0.org1.example.com`` to move "10" from "a" to "b"

-  The chaincode is then installed on ``peer1.org2.example.com``

-  A query is sent to ``peer1.org2.example.com`` for the value of "a". This starts a
   third chaincode container by the name of ``dev-peer1.org2.example.com-mycc-1.0``. A
   value of 90 is returned, correctly reflecting the previous
   transaction during which the value for key "a" was modified by 10.

What does this demonstrate?
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Chaincode **MUST** be installed on a peer in order for it to
successfully perform read/write operations against the ledger.
Furthermore, a chaincode container is not started for a peer until an ``init`` or
traditional transaction - read/write - is performed against that chaincode (e.g. query for
the value of "a"). The transaction causes the container to start. Also,
all peers in a channel maintain an exact copy of the ledger which
comprises the blockchain to store the immutable, sequenced record in
blocks, as well as a state database to maintain a snapshot of the current state.
This includes those peers that do not have chaincode installed on them
(like ``peer1.org1.example.com`` in the above example) . Finally, the chaincode is accessible
after it is installed (like ``peer1.org2.example.com`` in the above example) because it
has already been instantiated.

How do I see these transactions?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Check the logs for the CLI Docker container.

.. code:: bash

        docker logs -f cli

You should see the following output:

.. code:: bash

      2017-05-16 17:08:01.366 UTC [msp] GetLocalMSP -> DEBU 004 Returning existing local MSP
      2017-05-16 17:08:01.366 UTC [msp] GetDefaultSigningIdentity -> DEBU 005 Obtaining default signing identity
      2017-05-16 17:08:01.366 UTC [msp/identity] Sign -> DEBU 006 Sign: plaintext: 0AB1070A6708031A0C08F1E3ECC80510...6D7963631A0A0A0571756572790A0161
      2017-05-16 17:08:01.367 UTC [msp/identity] Sign -> DEBU 007 Sign: digest: E61DB37F4E8B0D32C9FE10E3936BA9B8CD278FAA1F3320B08712164248285C54
      Query Result: 90
      2017-05-16 17:08:15.158 UTC [main] main -> INFO 008 Exiting.....
      ===================== Query successful on peer1.org2 on channel 'mychannel' =====================
    
      ===================== All GOOD, BYFN execution completed =====================


       _____   _   _   ____
      | ____| | \ | | |  _ \
      |  _|   |  \| | | | | |
      | |___  | |\  | | |_| |
      |_____| |_| \_| |____/

You can scroll through these logs to see the various transactions.

How can I see the chaincode logs?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Inspect the individual chaincode containers to see the separate
transactions executed against each container. Here is the combined
output from each container:

.. code:: bash

        $ docker logs dev-peer0.org2.example.com-mycc-1.0
        04:30:45.947 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
        ex02 Init
        Aval = 100, Bval = 200
    
        $ docker logs dev-peer0.org1.example.com-mycc-1.0
        04:31:10.569 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
        ex02 Invoke
        Query Response:{"Name":"a","Amount":"100"}
        ex02 Invoke
        Aval = 90, Bval = 210
    
        $ docker logs dev-peer1.org2.example.com-mycc-1.0
        04:31:30.420 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
        ex02 Invoke
        Query Response:{"Name":"a","Amount":"90"}

Understanding the Docker Compose topology
-----------------------------------------

The BYFN sample offers us two flavors of Docker Compose files, both of which
are extended from the ``docker-compose-base.yaml`` (located in the ``base``
folder).  Our first flavor, ``docker-compose-cli.yaml``, provides us with a
CLI container, along with an orderer, four peers.  We use this file
for the entirety of the instructions on this page.

.. note:: the remainder of this section covers a docker-compose file designed for the
​          SDK.  Refer to the `Node SDK <https://github.com/hyperledger/fabric-sdk-node>`__
​          repo for details on running these tests.

The second flavor, ``docker-compose-e2e.yaml``, is constructed to run end-to-end tests
using the Node.js SDK.  Aside from functioning with the SDK, its primary differentiation
is that there are containers for the fabric-ca servers.  As a result, we are able
to send REST calls to the organizational CAs for user registration and enrollment.

If you want to use the ``docker-compose-e2e.yaml`` without first running the
byfn.sh script, then we will need to make four slight modifications.
We need to point to the private keys for our Organization's CA's.  You can locate
these values in your crypto-config folder.  For example, to locate the private
key for Org1 we would follow this path - ``crypto-config/peerOrganizations/org1.example.com/ca/``.
The private key is a long hash value followed by ``_sk``.  The path for Org2
would be - ``crypto-config/peerOrganizations/org2.example.com/ca/``.

In the ``docker-compose-e2e.yaml`` update the FABRIC_CA_SERVER_TLS_KEYFILE variable
for ca0 and ca1.  You also need to edit the path that is provided in the command
to start the ca server.  You are providing the same private key twice for each
CA container.

Using CouchDB
-------------

The state database can be switched from the default (goleveldb) to CouchDB.
The same chaincode functions are available with CouchDB, however, there is the
added ability to perform rich and complex queries against the state database
data content contingent upon the chaincode data being modeled as JSON.

To use CouchDB instead of the default database (goleveldb), follow the same
procedures outlined earlier for generating the artifacts, except when starting
the network pass ``docker-compose-couch.yaml`` as well:

.. code:: bash

    docker-compose -f docker-compose-cli.yaml -f docker-compose-couch.yaml up -d

**chaincode_example02** should now work using CouchDB underneath.

.. note::  If you choose to implement mapping of the fabric-couchdb container
​           port to a host port, please make sure you are aware of the security
​           implications. Mapping of the port in a development environment makes the
​           CouchDB REST API available, and allows the
​           visualization of the database via the CouchDB web interface (Fauxton).
​           Production environments would likely refrain from implementing port mapping in
​           order to restrict outside access to the CouchDB containers.

You can use **chaincode_example02** chaincode against the CouchDB state database
using the steps outlined above, however in order to exercise the CouchDB query
capabilities you will need to use a chaincode that has data modeled as JSON,
(e.g. **marbles02**). You can locate the **marbles02** chaincode in the
``fabric/examples/chaincode/go`` directory.

We will follow the same process to create and join the channel as outlined in the
:ref:`createandjoin` section above.  Once you have joined your peer(s) to the
channel, use the following steps to interact with the **marbles02** chaincode:

-  Install and instantiate the chaincode on ``peer0.org1.example.com``:

.. code:: bash

       # be sure to modify the $CHANNEL_NAME variable accordingly for the instantiate command
    
       peer chaincode install -n marbles -v 1.0 -p github.com/chaincode/marbles02/go
       peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org0MSP.peer','Org1MSP.peer')"

-  Create some marbles and move them around:

.. code:: bash

        # be sure to modify the $CHANNEL_NAME variable accordingly
    
        peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble1","blue","35","tom"]}'
        peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble2","red","50","tom"]}'
        peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble3","blue","70","tom"]}'
        peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarble","marble2","jerry"]}'
        peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarblesBasedOnColor","blue","jerry"]}'
        peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["delete","marble1"]}'

-  If you chose to map the CouchDB ports in docker-compose, you can now view
   the state database through the CouchDB web interface (Fauxton) by opening
   a browser and navigating to the following URL:

   ``http://localhost:5984/_utils``

You should see a database named ``mychannel`` (or your unique channel name) and
the documents inside it.

.. note:: For the below commands, be sure to update the $CHANNEL_NAME variable appropriately.

You can run regular queries from the CLI (e.g. reading ``marble2``):

.. code:: bash

      peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["readMarble","marble2"]}'

The output should display the details of ``marble2``:

.. code:: bash

       Query Result: {"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}

You can retrieve the history of a specific marble - e.g. ``marble1``:

.. code:: bash

      peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["getHistoryForMarble","marble1"]}'

The output should display the transactions on ``marble1``:

.. code:: bash

      Query Result: [{"TxId":"1c3d3caf124c89f91a4c0f353723ac736c58155325f02890adebaa15e16e6464", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"tom"}},{"TxId":"755d55c281889eaeebf405586f9e25d71d36eb3d35420af833a20a2f53a3eefd", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"jerry"}},{"TxId":"819451032d813dde6247f85e56a89262555e04f14788ee33e28b232eef36d98f", "Value":}]

You can also perform rich queries on the data content, such as querying marble fields by owner ``jerry``:

.. code:: bash

      peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarblesByOwner","jerry"]}'

The output should display the two marbles owned by ``jerry``:

.. code:: bash

       Query Result: [{"Key":"marble2", "Record":{"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}},{"Key":"marble3", "Record":{"color":"blue","docType":"marble","name":"marble3","owner":"jerry","size":70}}]


Why CouchDB
-------------
CouchDB is a kind of NoSQL solution. It is a document-oriented database where document fields are stored as key-value maps. Fields can be either a simple key-value pair, list, or map.
In addition to keyed/composite-key/key-range queries which are supported by LevelDB, CouchDB also supports full data rich queries capability, such as non-key queries against the whole blockchain data,
since its data content is stored in JSON format and fully queryable. Therefore, CouchDB can meet chaincode, auditing, reporting requirements for many use cases that not supported by LevelDB.

CouchDB can also enhance the security for compliance and data protection in the blockchain. As it is able to implement field-level security through the filtering and masking of individual attributes within a transaction, and only authorizing the read-only permission if needed.

In addition, CouchDB falls into the AP-type (Availability and Partition Tolerance) of the CAP theorem. It uses a master-master replication model with ``Eventual Consistency``.
More information can be found on the
`Eventual Consistency page of the CouchDB documentation <http://docs.couchdb.org/en/latest/intro/consistency.html>`__.
However, under each fabric peer, there is no database replicas, writes to database are guaranteed consistent and durable (not ``Eventual Consistency``).

CouchDB is the first external pluggable state database for Fabric, and there could and should be other external database options. For example, IBM enables the relational database for its blockchain.
And the CP-type (Consistency and Partition Tolerance) databases may also in need, so as to enable data consistency without application level guarantee.


A Note on Data Persistence
--------------------------

If data persistence is desired on the peer container or the CouchDB container,
one option is to mount a directory in the docker-host into a relevant directory
in the container. For example, you may add the following two lines in
the peer container specification in the ``docker-compose-base.yaml`` file:

.. code:: bash

       volumes:
        - /var/hyperledger/peer0:/var/hyperledger/production

For the CouchDB container, you may add the following two lines in the CouchDB
container specification:

.. code:: bash

       volumes:
        - /var/hyperledger/couchdb0:/opt/couchdb/data

.. _Troubleshoot:

Troubleshooting
---------------

-  Always start your network fresh.  Use the following command
   to remove artifacts, crypto, containers and chaincode images:

   .. code:: bash

      ./byfn.sh down

   .. note:: You **will** see errors if you do not remove old containers
   ​          and images.

-  If you see Docker errors, first check your docker version (:doc:`prereqs`),
   and then try restarting your Docker process.  Problems with Docker are
   oftentimes not immediately recognizable.  For example, you may see errors
   resulting from an inability to access crypto material mounted within a
   container.

   If they persist remove your images and start from scratch:

   .. code:: bash

       docker rm -f $(docker ps -aq)
       docker rmi -f $(docker images -q)

-  If you see errors on your create, instantiate, invoke or query commands, make
   sure you have properly updated the channel name and chaincode name.  There
   are placeholder values in the supplied sample commands.


-  If you see the below error:

   .. code:: bash

       Error: Error endorsing chaincode: rpc error: code = 2 desc = Error installing chaincode code mycc:1.0(chaincode /var/hyperledger/production/chaincodes/mycc.1.0 exits)

   You likely have chaincode images (e.g. ``dev-peer1.org2.example.com-mycc-1.0`` or
   ``dev-peer0.org1.example.com-mycc-1.0``) from prior runs. Remove them and try
   again.

   .. code:: bash

       docker rmi -f $(docker images | grep peer[0-9]-peer[0-9] | awk '{print $3}')

-  If you see something similar to the following:

   .. code:: bash

      Error connecting: rpc error: code = 14 desc = grpc: RPC failed fast due to transport failure
      Error: rpc error: code = 14 desc = grpc: RPC failed fast due to transport failure

   Make sure you are running your network against the "1.0.0" images that have
   been retagged as "latest".

-  If you see the below error:

   .. code:: bash

     [configtx/tool/localconfig] Load -> CRIT 002 Error reading configuration: Unsupported Config Type ""
     panic: Error reading configuration: Unsupported Config Type ""

   Then you did not set the ``FABRIC_CFG_PATH`` environment variable properly.  The
   configtxgen tool needs this variable in order to locate the configtx.yaml.  Go
   back and execute an ``export FABRIC_CFG_PATH=$PWD``, then recreate your
   channel artifacts.

-  To cleanup the network, use the ``down`` option:

   .. code:: bash

       ./byfn.sh down

-  If you see an error stating that you still have "active endpoints", then prune
   your Docker networks.  This will wipe your previous networks and start you with a
   fresh environment:

   .. code:: bash

        docker network prune

   You will see the following message:

   .. code:: bash

      WARNING! This will remove all networks not used by at least one container.
      Are you sure you want to continue? [y/N]

   Select ``y``.

-  If you see an error similar to the following:

   .. code:: bash

      /bin/bash: ./scripts/script.sh: /bin/bash^M: bad interpreter: No such file or directory

   Ensure that the file in question (**script.sh** in this example) is encoded
   in the Unix format. This was most likely caused by not setting
   ``core.autocrlf`` to ``false`` in your Git configuration (see
   :ref:`windows-extras`). There are several ways of fixing this. If you have
   access to the vim editor for instance, open the file:

   .. code:: bash

      vim ./fabric-samples/first-network/scripts/script.sh

   Then change its format by executing the following vim command:

   .. code:: bash

      :set ff=unix

.. note:: If you continue to see errors, share your logs on the
​          **fabric-questions** channel on
​          `Hyperledger Rocket Chat <https://chat.hyperledger.org/home>`__
​          or on `StackOverflow <https://stackoverflow.com/questions/tagged/hyperledger-fabric>`__.

.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
