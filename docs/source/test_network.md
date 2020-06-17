# Using the Fabric test network

After you have downloaded the Hyperledger Fabric Docker images and samples, you
can deploy a test network by using scripts that are provided in the
`fabric-samples` repository. You can use the test network to learn about Fabric
by running nodes on your local machine. More experienced developers can use the
network to test their smart contracts and applications. The network is meant to
be used only as a tool for education and testing. It should not be used as a
template for deploying a production network. The test network is being introduced
in Fabric v2.0 as the long term replacement for the `first-network` sample.

The sample network deploys a Fabric network with Docker Compose. Because the
nodes are isolated within a Docker Compose network, the test network is not
configured to connect to other running fabric nodes.

**Note:** These instructions have been verified to work against the
latest stable Docker images and the pre-compiled setup utilities within the
supplied tar file. If you run these commands with images or tools from the
current master branch, it is possible that you will encounter errors.

# 使用Fabric测试网络

在已经下载 Hyperledger Fabric Docker 镜像和演示项目后，你可以使用演示代码部署测试网络，演示代码在`fabric-samples`仓库中。你可以通过本机上运行测试网络学习Fabric。有经验的开发者也会使用测试网络测试他们的智能合约或者应用程序。这个测试网络只能用于学习和测试。这个测试网络在 Fabric V2.0被引进，用于替代`first-network`示例。

这个示例测试网络使用 Dokcer Compose部署一个 Fabric 网络。因为节点之间通过 Docker Compose network 进行隔离，这个测试网络配有配置为连接其他正在运行的 fabric 节点。

**注意** 这份指导文件已经使用最新的稳定版本的 Docker 镜像和在 tar 文件中提供的预编译安装套件完成测试。如果你使用 master 分支中的最新提交对应的镜像或工具，可能会发生错误。

## Before you begin

Before you can run the test network, you need to clone the `fabric-samples`
repository and download the Fabric images. Make sure that that you have installed
the [Prerequisites](prereqs.html) and [Installed the Samples, Binaries and Docker Images](install.html).

## 开始之前

在你运行测试网络之前，你需要 clone `fabric-samples` 镜像并下载 Fabric 镜像。请确定你已经安装 [先决条件](prereqs.html) 并且已经安装了 [示例项目， 二进制程序以及 Docker 镜像](install.html)。

## Bring up the test network

You can find the scripts to bring up the network in the `test-network` directory
of the ``fabric-samples`` repository. Navigate to the test network directory by
using the following command:
```
cd fabric-samples/test-network
```

In this directory, you can find an annotated script, ``network.sh``, that stands
up a Fabric network using the Docker images on your local machine. You can run
``./network.sh -h`` to print the script help text:

```
Usage:
  network.sh <Mode> [Flags]
    <Mode>
      - 'up' - bring up fabric orderer and peer nodes. No channel is created
      - 'up createChannel' - bring up fabric network with one channel
      - 'createChannel' - create and join a channel after the network is created
      - 'deployCC' - deploy the fabcar chaincode on the channel
      - 'down' - clear the network with docker-compose down
      - 'restart' - restart the network

    Flags:
    -ca <use CAs> -  create Certificate Authorities to generate the crypto material
    -c <channel name> - channel name to use (defaults to "mychannel")
    -s <dbtype> - the database backend to use: goleveldb (default) or couchdb
    -r <max retry> - CLI times out after certain number of attempts (defaults to 5)
    -d <delay> - delay duration in seconds (defaults to 3)
    -l <language> - the programming language of the chaincode to deploy: go (default), javascript, or java
    -v <version>  - chaincode version. Must be a round number, 1, 2, 3, etc
    -i <imagetag> - the tag to be used to launch the network (defaults to "latest")
    -verbose - verbose mode
  network.sh -h (print this message)

 Possible Mode and flags
  network.sh up -ca -c -r -d -s -i -verbose
  network.sh up createChannel -ca -c -r -d -s -i -verbose
  network.sh createChannel -c -r -d -verbose
  network.sh deployCC -l -v -r -d -verbose

 Taking all defaults:
	network.sh up

 Examples:
  network.sh up createChannel -ca -c mychannel -s couchdb -i 2.0.0-beta
  network.sh createChannel -c channelName
  network.sh deployCC -l javascript
```

From inside the `test-network` directory, run the following command to remove
any containers or artifacts from any previous runs:
```
./network.sh down
```

You can then bring up the network by issuing the following command. You will
experience problems if you try to run the script from another directory:
```
./network.sh up
```

This command creates a Fabric network that consists of two peer nodes, one
ordering node. No channel is created when you run `./network.sh up`, though we
will get there in a [future step](#creating-a-channel). If the command completes
successfully, the script will print out logs similar to these of the nodes being
created.
```
Creating network "net_test" with the default driver
Creating volume "net_orderer.example.com" with default driver
Creating volume "net_peer0.org1.example.com" with default driver
Creating volume "net_peer0.org2.example.com" with default driver
Creating orderer.example.com    ... done
Creating peer0.org2.example.com ... done
Creating peer0.org1.example.com ... done
CONTAINER ID        IMAGE                               COMMAND             CREATED             STATUS                  PORTS                              NAMES
8d0c74b9d6af        hyperledger/fabric-orderer:latest   "orderer"           4 seconds ago       Up Less than a second   0.0.0.0:7050->7050/tcp             orderer.example.com
ea1cf82b5b99        hyperledger/fabric-peer:latest      "peer node start"   4 seconds ago       Up Less than a second   0.0.0.0:7051->7051/tcp             peer0.org1.example.com
cd8d9b23cb56        hyperledger/fabric-peer:latest      "peer node start"   4 seconds ago       Up 1 second             7051/tcp, 0.0.0.0:9051->9051/tcp   peer0.org2.example.com
```

If you don't get this result, jump down to [Troubleshooting](#troubleshooting)
for help on what might have gone wrong. By default, the network uses the
cryptogen tool to bring up the network. However, you can also
[bring up the network with Certificate Authorities](#bring-up-the-network-with-certificate-authorities).

## 启动测试网络

你可以在 `fabric-samples` 目录中的 `test-network` 目录下找到启动测试网络的脚本。使用下面的命令切换到测试网络的目录。

```
cd fabric-samples/test-network
```

在这个目录中，你可以找到一个名字叫做 `network.sh` 的脚本，它使用 Docker 镜像创建一个 Fabric 网络。你可以执行 `./network.sh -h` 获取帮助文档。

```
Usage:
  network.sh <Mode> [Flags]
    <Mode>
      - 'up' - bring up fabric orderer and peer nodes. No channel is created
      - 'up createChannel' - bring up fabric network with one channel
      - 'createChannel' - create and join a channel after the network is created
      - 'deployCC' - deploy the fabcar chaincode on the channel
      - 'down' - clear the network with docker-compose down
      - 'restart' - restart the network

    Flags:
    -ca <use CAs> -  create Certificate Authorities to generate the crypto material
    -c <channel name> - channel name to use (defaults to "mychannel")
    -s <dbtype> - the database backend to use: goleveldb (default) or couchdb
    -r <max retry> - CLI times out after certain number of attempts (defaults to 5)
    -d <delay> - delay duration in seconds (defaults to 3)
    -l <language> - the programming language of the chaincode to deploy: go (default), javascript, or java
    -v <version>  - chaincode version. Must be a round number, 1, 2, 3, etc
    -i <imagetag> - the tag to be used to launch the network (defaults to "latest")
    -verbose - verbose mode
  network.sh -h (print this message)

 Possible Mode and flags
  network.sh up -ca -c -r -d -s -i -verbose
  network.sh up createChannel -ca -c -r -d -s -i -verbose
  network.sh createChannel -c -r -d -verbose
  network.sh deployCC -l -v -r -d -verbose

 Taking all defaults:
	network.sh up

 Examples:
  network.sh up createChannel -ca -c mychannel -s couchdb -i 2.0.0-beta
  network.sh createChannel -c channelName
  network.sh deployCC -l javascript
```

在 `test-network` 目录中，执行下面的命令移除任何以前的测试网络启动的容器或内容：
```
./network.sh down
```

现在你可以使用下面的命令启动一个测试网络。如果你在其他目录执行这个命令会发生错误：
```
./network.sh up
```

这个命令创建一个拥有两个普通节点和一个排序节点的 Fabric 网络。执行完 `./network.sh up` 之后并不会创建任何频道，所以接下来我们要执行 [关键步骤](#creating-a-channel)。如果命令执行成功，脚本会打印类似下面的日志来表明已经创建的节点。
```
Creating network "net_test" with the default driver
Creating volume "net_orderer.example.com" with default driver
Creating volume "net_peer0.org1.example.com" with default driver
Creating volume "net_peer0.org2.example.com" with default driver
Creating orderer.example.com    ... done
Creating peer0.org2.example.com ... done
Creating peer0.org1.example.com ... done
CONTAINER ID        IMAGE                               COMMAND             CREATED             STATUS                  PORTS                              NAMES
8d0c74b9d6af        hyperledger/fabric-orderer:latest   "orderer"           4 seconds ago       Up Less than a second   0.0.0.0:7050->7050/tcp             orderer.example.com
ea1cf82b5b99        hyperledger/fabric-peer:latest      "peer node start"   4 seconds ago       Up Less than a second   0.0.0.0:7051->7051/tcp             peer0.org1.example.com
cd8d9b23cb56        hyperledger/fabric-peer:latest      "peer node start"   4 seconds ago       Up 1 second             7051/tcp, 0.0.0.0:9051->9051/tcp   peer0.org2.example.com
```

如果你得到不同的结果，跳转到 [疑难解答](#troubleshooting) 获取可能导致问题的原因。默认情况下，脚本使用 cryptogen 工具创建需要的凭证并启动测试网络。如果有需要，你也可以[使用CA证书启动网络](#bring-up-the-network-with-certificate-authorities)。

### The components of the test network

After your test network is deployed, you can take some time to examine its
components. Run the following command to list all of Docker containers that
are running on your machine. You should see the three nodes that were created by
the `network.sh` script:
```
docker ps -a
```

Each node and user that interacts with a Fabric network needs to belong to an
organization that is a network member. The group of organizations that are
members of a Fabric network are often referred to as the consortium. The test
network has two consortium members, Org1 and Org2. The network also includes one
orderer organization that maintains the ordering service of the network.

[Peers](peers/peers.html) are the fundamental components of any Fabric network.
Peers store the blockchain ledger and validate transactions before they are
committed to the ledger. Peers run the smart contracts that contain the business
logic that is used to manage the assets on the blockchain ledger.

Every peer in the network needs to belong to a member of the consortium. In the
test network, each organization operates one peer each, `peer0.org1.example.com`
and `peer0.org2.example.com`.

Every Fabric network also includes an [ordering service](orderer/ordering_service.html).
While peers validate transactions and add blocks of transactions to the
blockchain ledger, they do not decide on the order of transactions or include
them into new blocks. On a distributed network, peers may be running far away
from each other and not have a common view of when a transaction was created.
Coming to consensus on the order of transactions is a costly process that would
create overhead for the peers.

An ordering service allows peers to focus on validating transactions and
committing them to the ledger. After ordering nodes receive endorsed transactions
from clients, they come to consensus on the order of transactions and then add
them to blocks. The blocks are then distributed to peer nodes, which add the
blocks the blockchain ledger. Ordering nodes also operate the system channel
that defines the capabilities of a Fabric network, such as how blocks are made
and which version of Fabric that nodes can use. The system channel defines which
organizations are members of the consortium.

The sample network uses a single node Raft ordering service that is operated by
the ordering organization. You can see the ordering node running on your machine
as `orderer.example.com`. While the test network only uses a single node ordering
service, a real network would have multiple ordering nodes, operated by one or
multiple orderer organizations. The different ordering nodes would use the Raft
consensus algorithm to come to agreement on the order of transactions across
the network.

### 测试网络组成

当测试网络部署之后，你可以花一些时间研究一下它的组成。运行下面的命令列出本机所有的 Docker 容器。你可以看到3个由 `network.sh` 脚本创建的节点：
```
docker ps -a
```

任何与 Fabric 网络交互的节点和用户都应该属于作为网络成员的组织。多个作为 Fabric 网络的组织又可以组成联盟。这个测试网络有两个联盟的成员，Org1 和 Org2。这个测试网络还包含了一个排序组织，它维护者一个用于排序的服务。

节点是 Fabric 网络的基本组成。节点保存区块链账本并且在交易提交到账本之前对交易进行验证。节点还运行包含了商业逻辑的智能合约以管理区块链账本上的资产。

网络中的每一个节点都应该属于联盟的成员。在这个测试网络中，每个组织操作一个节点`peer0.org1.example.com`和 `peer0.org2.example.com`。

每一个 Fabric 网络包含一个 [排序服务](orderer/ordering_service.html)。节点对交易进行验证并添加到区块链账单上，他们不需要决定交易的顺序或者哪个区块包含哪些交易。在一个分布式的网络中，节点和节点之间可能距离很远，并且对何时创建交易没有相同的试图。要对交易的顺序达成一致需要远超节点能够承受的资源消耗。

使用排序服务允许节点专注于验证交易和将交易提交到账单上。当排序节点从客户端接收到核验过的交易后，他们对交易进行一致性排序后将交易添加到区块上。然后这个区块会分发到其他节点，其他节点就可以把区块添加到区块链账单中。排序节点同时也操作着定义了 Fabric 网络能力的系统频道，比如控制区块如何建、Fabric 节点应该使用什么版本。系统频道定义了哪些组织是联盟的成员。

这个测试网络使用的是只有一个节点的排序组织。你可以在本机看到一个正在运行的排序节点`orderer.example.com`。尽管测试网络只是用一个节点提供排序服务，一个真正的网络应该有多个排序节点，这些节点有一个或多个排序组织控制。不同的排序节点会使用 Raft 一致性算法在网络上产生一个一致的交易顺序。

## Creating a channel

Now that we have peer and orderer nodes running on our machine, we can use the
script to create a Fabric channel for transactions between Org1 and Org2.
Channels are a private layer of communication between specific network members.
Channels can be used only by organizations that are invited to the channel, and
are invisible to other members of the network. Each channel has a separate
blockchain ledger. Organizations that have been invited "join" their peers to
the channel to store the channel ledger and validate the transactions on the
channel.

You can use the `network.sh` script to create a channel between Org1 and Org2
and join their peers to the channel. Run the following command to create a
channel with the default name of `mychannel`:
```
./network.sh createChannel
```
If the command was successful, you can see the following message printed in your
logs:
```
========= Channel successfully joined ===========
```

You can also use the channel flag to create a channel with custom name. As an
example, the following command would create a channel named `channel1`:
```
./network.sh createChannel -c channel1
```

The channel flag also allows you to create multiple channels by specifying
different channel names. After you create `mychannel` or `channel1`, you can use
the command below to create a second channel named `channel2`:
```
./network.sh createChannel -c channel2
```

If you want to bring up the network and create a channel in a single step, you
can use the `up` and `createChannel` modes together:
```
./network.sh up createChannel
```

## 创建一个频道

现在本机已经有了对等节点和排序节点在运行，我们可以使用这个脚本为 Org1 和 Org2 创建一个 Fabric 频道。频道是特定网络成员私有的通讯层。频道只能被加入频道的组织使用，而对网络中的其他成员不可见。每一个频道都有一个独立的区块链账单。被邀请的组织将他们的节点加入到频道中可以保存频道的账单并且验证频道中的交易。

你可以使用 `network.sh` 脚本创建一个频道，并将 Org1 和 Org2 的节点加入到频道中。执行下面的命令使用默认的名称 `mychannel` 创建频道：
```
./network.sh createChannel
```

如果命令执行成功，你可以在日志中看到下面的信息：
```
========= Channel successfully joined ===========
```

你也可以使用频道标记明明创建一个有自定义名称的频道。比如，下面的命令可以创建一个名称为`channel1`的频道：
```
./network.sh createChannel -c channel1
```

使用频道标记你可以创建多个使用频道名称区分的频道。你创建完 `mychannel` 和 `channel1`之后，你可以使用下面的命令创建另一个名为 `channel2`的频道：
```
./network.sh createChannel -c channel2
```

如果你想一步启动网络并创建频道，你可以同时使用 `up` 和 `creataeChannel`：
```
./network.sh up createChannel
```

## Starting a chaincode on the channel

After you have created a channel, you can start using [smart contracts](smartcontract/smartcontract.html) to
interact with the channel ledger. Smart contracts contain the business logic
that governs assets on the blockchain ledger. Applications run by members of the
network can invoke smart contracts to create assets on the ledger, as well as
change and transfer those assets. Applications also query smart contracts to
read data on the ledger.

To ensure that transactions are valid, transactions created using smart contracts
typically need to be signed by multiple organizations to be committed to the
channel ledger. Multiple signatures are integral to the trust model of Fabric.
Requiring multiple endorsements for a transaction prevents one organization on
a channel from tampering with the ledger on their peer or using business logic
that was not agreed to. To sign a transaction, each organization needs to invoke
and execute the smart contract on their peer, which then signs the output of the
transaction. If the output is consistent and has been signed by enough
organizations, the transaction can be committed to the ledger. The policy that
specifies the set organizations on the channel that need to execute the smart
contract is referred to as the endorsement policy, which is set for each
chaincode as part of the chaincode definition.

In Fabric, smart contracts are deployed on the network in packages referred to
as chaincode. A Chaincode is installed on the peers of an organization and then
deployed to a channel, where it can then be used to endorse transactions and
interact with the blockchain ledger. Before a chaincode can be deployed to a
channel, the members of the channel need to agree on a chaincode definition that
establishes chaincode governance. When the required number of organizations
agree, the chaincode definition can be committed to the channel, and the
chaincode is ready to be used.

After you have used the `network.sh` to create a channel, you can start a
chaincode on the channel using the following command:
```
./network.sh deployCC
```
The `deployCC` subcommand will install the **fabcar** chaincode on
``peer0.org1.example.com`` and ``peer0.org2.example.com`` and then deploy
the chaincode on the channel specified using the channel flag (or `mychannel`
if no channel is specified). If are deploying a chaincode for the first time, the
script will install the chaincode dependencies. By default, The script installs
the Golang version of the fabcar chaincode. However, you can use the language
flag, `-l`, to install the Java or javascript versions of the chaincode.

After the **fabcar** chaincode definition has been committed to the channel, the
script initializes the chaincode by invoking the `init` function and then invokes
the chaincode to put an initial list of cars on the ledger. The script then
queries the chaincode to verify the that the data was added. If the chaincode was
installed, deployed, and invoked correctly, you should see the following list of
cars printed in your logs:
```
[{"Key":"CAR0", "Record":{"make":"Toyota","model":"Prius","colour":"blue","owner":"Tomoko"}},{"Key":"CAR1", "Record":{"make":"Ford","model":"Mustang","colour":"red","owner":"Brad"}},{"Key":"CAR2", "Record":{"make":"Hyundai","model":"Tucson","colour":"green","owner":"Jin Soo"}},{"Key":"CAR3", "Record":{"make":"Volkswagen","model":"Passat","colour":"yellow","owner":"Max"}},{"Key":"CAR4", "Record":{"make":"Tesla","model":"S","colour":"black","owner":"Adriana"}},{"Key":"CAR5", "Record":{"make":"Peugeot","model":"205","colour":"purple","owner":"Michel"}},{"Key":"CAR6", "Record":{"make":"Chery","model":"S22L","colour":"white","owner":"Aarav"}},{"Key":"CAR7", "Record":{"make":"Fiat","model":"Punto","colour":"violet","owner":"Pari"}},{"Key":"CAR8", "Record":{"make":"Tata","model":"Nano","colour":"indigo","owner":"Valeria"}},{"Key":"CAR9", "Record":{"make":"Holden","model":"Barina","colour":"brown","owner":"Shotaro"}}]
===================== Query successful on peer0.org1 on channel 'mychannel' =====================
```

## 在频道上运行链码

创建完频道之后，你可以开始使用智能合约与频道账单交互了。智能合约包含了调度区块链账单上资源的商业逻辑。网络上的成员运行应用程序调用只能合约在账单上创建资源，修改资源并交易资源。应用程序也向智能合约发送请求一度去账单上的数据。

要确认交易是否合法，使用智能合约创建的交易需要被多个组织签名同意才能提交到账单中。多个签名对于 Fabric 的信任模型是不可或缺的。交易需要多个背书的基础可以阻止频道上的组织贿赂账单上的对等节点或者使用商业逻辑不允许的逻辑。要签署一个交易，每个组织需要在他们的节点上调用并执行智能合约对交易进行签名。如果输出一致并且有足够的组织签名，交易就可以在账单上提交。指定哪些组织需要执行智能合约的策略成为背书策略，它是只能合约的一部分。

在 Fabric 中，部署在网络中的智能合约包叫做链码。链码安装在组织的对等节点上以部署到频道中，链码可以用来为交易背书或与区块链账单进行交互。在链码部署到频道之前，需要频道中的成员同意链码的定义，这样可以对链码进行管理。当足够多个组织同意之后，链码的定义可以为提交到频道中，然后链码可以被使用。

当你使用 `network.sh` 创建频道之后，你可以使用下面的命令在频道上创建一个链码：
```
./network.sh deployCC
```

`deployCC`子命令会在`peer0.org1.example.com`和`peer0.org2.example.com`安装**fabcar**链码然后在骗到标记（如果没有指定频道会使用`mychannel`）指定的频道上部署这个链码。如果是第一个部署链码，这个脚本会安装链码的依赖。默认情况下，脚本会安装Golang版本的fabcar链码。你也可以使用了语言标记，`-l`，安装Java或javascript版本的链码。

当**fabcar**链码的定义提交到频道之后，脚本会调用 `init` 函数初始化链码然后链码会在账单上初始化一个汽车列表。然后脚本会请求链码以验证数据已经被添加。如果链码已经正确的安装、部署和调用，你应该可以在日志中看到下面的汽车列表：
```
[{"Key":"CAR0", "Record":{"make":"Toyota","model":"Prius","colour":"blue","owner":"Tomoko"}},{"Key":"CAR1", "Record":{"make":"Ford","model":"Mustang","colour":"red","owner":"Brad"}},{"Key":"CAR2", "Record":{"make":"Hyundai","model":"Tucson","colour":"green","owner":"Jin Soo"}},{"Key":"CAR3", "Record":{"make":"Volkswagen","model":"Passat","colour":"yellow","owner":"Max"}},{"Key":"CAR4", "Record":{"make":"Tesla","model":"S","colour":"black","owner":"Adriana"}},{"Key":"CAR5", "Record":{"make":"Peugeot","model":"205","colour":"purple","owner":"Michel"}},{"Key":"CAR6", "Record":{"make":"Chery","model":"S22L","colour":"white","owner":"Aarav"}},{"Key":"CAR7", "Record":{"make":"Fiat","model":"Punto","colour":"violet","owner":"Pari"}},{"Key":"CAR8", "Record":{"make":"Tata","model":"Nano","colour":"indigo","owner":"Valeria"}},{"Key":"CAR9", "Record":{"make":"Holden","model":"Barina","colour":"brown","owner":"Shotaro"}}]
===================== Query successful on peer0.org1 on channel 'mychannel' =====================
```

## Interacting with the network

After you bring up the test network, you can use the `peer` CLI to interact
with your network. The `peer` CLI allows you to invoke deployed smart contracts,
update channels, or install and deploy new smart contracts from the CLI.

Make sure that you are operating from the `test-network` directory. If you
followed the instructions to [install the Samples, Binaries and Docker Images](install.html),
You can find the `peer` binaries in the `bin` folder of the `fabric-samples`
repository. Use the following command to add those binaries to your CLI Path:
```
export PATH=${PWD}/../bin:${PWD}:$PATH
```
You also need to set the `FABRIC_CFG_PATH` to point to the `core.yaml` file in
the `fabric-samples` repository:
```
export FABRIC_CFG_PATH=$PWD/../config/
```
You can now set the environment variables that allow you to operate the `peer`
 CLI as Org1:
```
# Environment variables for Org1

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

The `CORE_PEER_TLS_ROOTCERT_FILE` and `CORE_PEER_MSPCONFIGPATH` environment
variables point to the Org1 crypto material in the `organizations` folder.

If you used `./network.sh deployCC` to install and start the fabcar chaincode,
you can now query the ledger from your CLI. Run the following command to get the
list of cars that were added to your channel ledger:
```
peer chaincode query -C mychannel -n fabcar -c '{"Args":["queryAllCars"]}'
```

If the command is successful, you can see the same list of cars that were printed
in the logs when you ran the script:
```
[{"Key":"CAR0", "Record":{"make":"Toyota","model":"Prius","colour":"blue","owner":"Tomoko"}},{"Key":"CAR1", "Record":{"make":"Ford","model":"Mustang","colour":"red","owner":"Brad"}},{"Key":"CAR2", "Record":{"make":"Hyundai","model":"Tucson","colour":"green","owner":"Jin Soo"}},{"Key":"CAR3", "Record":{"make":"Volkswagen","model":"Passat","colour":"yellow","owner":"Max"}},{"Key":"CAR4", "Record":{"make":"Tesla","model":"S","colour":"black","owner":"Adriana"}},{"Key":"CAR5", "Record":{"make":"Peugeot","model":"205","colour":"purple","owner":"Michel"}},{"Key":"CAR6", "Record":{"make":"Chery","model":"S22L","colour":"white","owner":"Aarav"}},{"Key":"CAR7", "Record":{"make":"Fiat","model":"Punto","colour":"violet","owner":"Pari"}},{"Key":"CAR8", "Record":{"make":"Tata","model":"Nano","colour":"indigo","owner":"Valeria"}},{"Key":"CAR9", "Record":{"make":"Holden","model":"Barina","colour":"brown","owner":"Shotaro"}}]
```

Chaincodes are invoked when a network member wants to transfer or change an
asset on the ledger. Use the following command to change the owner of a car on
the ledger by invoking the fabcar chaincode:
```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls true --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n fabcar --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"changeCarOwner","Args":["CAR9","Dave"]}'
```

If the command is successful, you should see the following response:
```
2019-12-04 17:38:21.048 EST [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200
```

Because the endorsement policy for the fabcar chaincode requires the transaction
to be signed by Org1 and Org2, the chaincode invoke command needs to target both
`peer0.org1.example.com` and `peer0.org1.example.com` using the `--peerAddresses`
flag. Because TLS is enabled for the network, the command also needs to reference
the TLS certificate for each peer using the `--tlsRootCertFiles` flag.

After we invoke the chaincode, we can use another query to see how the invoke
changed the assets on the blockchain ledger. Since we already queried the Org1
peer, we can take this opportunity to query the chaincode running on the Org2
peer. Set the following environment variables to operate as Org2:
```
# Environment variables for Org2

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

You can now query the fabcar chaincode running on `peer0.org2.example.com`:
```
peer chaincode query -C mychannel -n fabcar -c '{"Args":["queryAllCars"]}'
```

The result will show that `"CAR9"` was transferred to Dave:
```
[{"Key":"CAR0", "Record":{"make":"Toyota","model":"Prius","colour":"blue","owner":"Tomoko"}},{"Key":"CAR1", "Record":{"make":"Ford","model":"Mustang","colour":"red","owner":"Brad"}},{"Key":"CAR2", "Record":{"make":"Hyundai","model":"Tucson","colour":"green","owner":"Jin Soo"}},{"Key":"CAR3", "Record":{"make":"Volkswagen","model":"Passat","colour":"yellow","owner":"Max"}},{"Key":"CAR4", "Record":{"make":"Tesla","model":"S","colour":"black","owner":"Adriana"}},{"Key":"CAR5", "Record":{"make":"Peugeot","model":"205","colour":"purple","owner":"Michel"}},{"Key":"CAR6", "Record":{"make":"Chery","model":"S22L","colour":"white","owner":"Aarav"}},{"Key":"CAR7", "Record":{"make":"Fiat","model":"Punto","colour":"violet","owner":"Pari"}},{"Key":"CAR8", "Record":{"make":"Tata","model":"Nano","colour":"indigo","owner":"Valeria"}},{"Key":"CAR9", "Record":{"make":"Holden","model":"Barina","colour":"brown","owner":"Dave"}}]
```

## 与网络进行交互

启动测试网络之后，你可以使用`peer`命令与你的网络进行交互。使用`peer`命令可以调用已经部署的职能和月，更新频道，或者安装以及部署新的智能合约。

确定你是在`test-network`目录中进行操作。如果你已经按照[安装示例、二进制程序和Docker镜像](install.html)进行操作，你可以在`fabric-samples`仓库中的`bin`目录中找到`peer`二进制程序。使用下面的命令把这些二进制程序添加到你的Path中：
```
export PATH=${PWD}/../bin:${PWD}:$PATH
```
你同样可以使用`FABRIC_CFG_PATH`环境变量指定`fabric-samples`仓库中的`core.yaml`文件：
```
export FABRIC_CFG_PATH=$PWD/../config/
```
现在你可以通过设置环境变量来以Org1的身份来操作`peer`：
```
# Environment variables for Org1

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

`CORE_PEER_TLS_ROOTCERT_FILE` 和 `CORE_PEER_MSPCONFIGPATH` 指定了Org1在`organizations`目录中的用于加密的素材。

如果你已经使用 `./network.sh deployCC` 安装并启动了fabcar链码，现在你可以使用命令行工具请求账目了。使用下面的命令获取已经添加到频道账目中的汽车列表：
```
peer chaincode query -C mychannel -n fabcar -c '{"Args":["queryAllCars"]}'
```

如果命令执行成功了，执行下面的脚本，你会看到像下面这样的汽车列表：
```
[{"Key":"CAR0", "Record":{"make":"Toyota","model":"Prius","colour":"blue","owner":"Tomoko"}},{"Key":"CAR1", "Record":{"make":"Ford","model":"Mustang","colour":"red","owner":"Brad"}},{"Key":"CAR2", "Record":{"make":"Hyundai","model":"Tucson","colour":"green","owner":"Jin Soo"}},{"Key":"CAR3", "Record":{"make":"Volkswagen","model":"Passat","colour":"yellow","owner":"Max"}},{"Key":"CAR4", "Record":{"make":"Tesla","model":"S","colour":"black","owner":"Adriana"}},{"Key":"CAR5", "Record":{"make":"Peugeot","model":"205","colour":"purple","owner":"Michel"}},{"Key":"CAR6", "Record":{"make":"Chery","model":"S22L","colour":"white","owner":"Aarav"}},{"Key":"CAR7", "Record":{"make":"Fiat","model":"Punto","colour":"violet","owner":"Pari"}},{"Key":"CAR8", "Record":{"make":"Tata","model":"Nano","colour":"indigo","owner":"Valeria"}},{"Key":"CAR9", "Record":{"make":"Holden","model":"Barina","colour":"brown","owner":"Shotaro"}}]
```

当一个网络成员想要修改账目中的资源，可以调用链码实现。使用下面的命令来调用链码以修改一辆汽车的所有者：
```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls true --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n fabcar --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"changeCarOwner","Args":["CAR9","Dave"]}'
```

如果命令执行成功，你应该能看到下面的响应：
```
2019-12-04 17:38:21.048 EST [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200
```

因为fabcar链码的背书策略要求交易同时被Org1和Org2签名，调用链码的命令需要使用`--peerAddresses`参数同时指定`peer0.org1.example.com` 和 `peer0.org2.example.com`。因为网络启用了TLS，执行命令时需要使用`--tlsRootCertFiles` 参数为每个节点指定TLS证书。

在我们调用了链码之后，我们可以使用另外一个请求看一看区块链账目上资源的修改。我们已经请求国Org1的节点了，我们现在可以去Org2的节点上请求链码。在Org2上设置环境变量：
```
# Environment variables for Org2

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

现在你请求运行在`peer0.org2.example.com`上的链码：
```
peer chaincode query -C mychannel -n fabcar -c '{"Args":["queryAllCars"]}'
```
从结果上能看到`CAR9`已经转交给Dave了：
```
[{"Key":"CAR0", "Record":{"make":"Toyota","model":"Prius","colour":"blue","owner":"Tomoko"}},{"Key":"CAR1", "Record":{"make":"Ford","model":"Mustang","colour":"red","owner":"Brad"}},{"Key":"CAR2", "Record":{"make":"Hyundai","model":"Tucson","colour":"green","owner":"Jin Soo"}},{"Key":"CAR3", "Record":{"make":"Volkswagen","model":"Passat","colour":"yellow","owner":"Max"}},{"Key":"CAR4", "Record":{"make":"Tesla","model":"S","colour":"black","owner":"Adriana"}},{"Key":"CAR5", "Record":{"make":"Peugeot","model":"205","colour":"purple","owner":"Michel"}},{"Key":"CAR6", "Record":{"make":"Chery","model":"S22L","colour":"white","owner":"Aarav"}},{"Key":"CAR7", "Record":{"make":"Fiat","model":"Punto","colour":"violet","owner":"Pari"}},{"Key":"CAR8", "Record":{"make":"Tata","model":"Nano","colour":"indigo","owner":"Valeria"}},{"Key":"CAR9", "Record":{"make":"Holden","model":"Barina","colour":"brown","owner":"Dave"}}]
```

## Bring down the network

When you are finished using the test network, you can bring down the network
with the following command:
```
./network.sh down
```

The command will stop and remove the node and chaincode containers, delete the
organization crypto material, and remove the chaincode images from your Docker
Registry. The command also removes the channel artifacts and docker volumes from
previous runs, allowing you to run `./network.sh up` again if you encountered
any problems.

## 关闭网络

当你使用完测试网络之后，你可以使用下面的命令关闭测试网络：
```
./network.sh down
```

这个命令会会停止并移除节点和链码容器，删除组织的加密素材，并且从你的Doocker仓库中移除链码的镜像。如果你再次运行`./network.sh up`的时候遇到问题，也可以运行这个命令移除上次运行测试网络的docker卷和频道产物。

## Bring up the network with Certificate Authorities

Hyperledger Fabric uses public key infrastructure (PKI) to verify the actions of
all network participants. Every node, network administrator, and user submitting
transactions need to have a public certificate and private key to verify their
identity. These identities need to have a valid root of trust, establishing
that the certificates and keys were issued by an organization that is a member
of the network.

The `network.sh` script must create all of the crypto material that is required
to deploy and operate the network before it creates the peer and ordering nodes.

By default, the script uses a tool called cryptogen to create the certificates
and keys. The tool is provided for development and testing, and can quickly
create the required crypto material for Fabric organizations with a valid root
of trust. When you run `./network.sh up`, you can see the cryptogen tool creating
the certificates and keys for Org1, Org2, and the Orderer Org.

```
creating Org1, Org2, and ordering service organization with crypto from 'cryptogen'

/Usr/fabric-samples/test-network/../bin/cryptogen

##########################################################
##### Generate certificates using cryptogen tool #########
##########################################################

##########################################################
############ Create Org1 Identities ######################
##########################################################
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-org1.yaml --output=organizations
org1.example.com
+ res=0
+ set +x
##########################################################
############ Create Org2 Identities ######################
##########################################################
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-org2.yaml --output=organizations
org2.example.com
+ res=0
+ set +x
##########################################################
############ Create Orderer Org Identities ###############
##########################################################
+ cryptogen generate --config=./organizations/cryptogen/crypto-config-orderer.yaml --output=organizations
+ res=0
+ set +x
```

However, `network.sh` also provides the option to bring up the network using
Certificate Authorities (CAs). In a production network, each organization would
operate a CA (or multiple intermediate CAs) that creates the identities that
belong to their organization. All of the identities created by a CA run by the
organization would share the same root of trust. Although it takes more time to
run the test network using CAs than to use cryptogen, bringing up a network using
CAs can provide an introduction to a production network would be deployed.
Standing up the Fabric CAs also provides you with the ability to enroll
a client identity using the Fabric SDKs and create a certificate and private key
that can be used by your application. Both cryptogen and the Fabric CAs generate
the crypto material for each organization in the `organizations` folder.

If you would like to bring up a network using Fabric CAs, first run the following
command to bring down any running networks:
```
./network.sh down
```

You can then bring up the network with the CA flag:
```
./network.sh up -ca
```

After you issue the command, you can see the script bringing up three CAs, one
for each organization in the network.
```
##########################################################
##### Generate certificates using Fabric CA's ############
##########################################################
Creating network "net_default" with the default driver
Creating ca_org2    ... done
Creating ca_org1    ... done
Creating ca_orderer ... done
```

The script then uses the Fabric CA client to register the users that belong to
each organization and generate the certificates and keys for each identity. You
can find the commands that are used to set up the network in the `registerEnroll.sh`
script in the `organizations/fabric-ca` directory. To learn more about how you
would use the Fabric CA to deploy a Fabric network, visit the
[Fabric CA operations guide](https://hyperledger-fabric-ca.readthedocs.io/en/latest/operations_guide.html).
You can learn more about how Fabric uses PKI by visiting the [identity](identity/identity.html) and [membership](membership/membership.html) concept topics.

## What's happening behind the scenes?

If you are interested in learning more about the sample network, you can
investigate the files and scripts in the `test-network` directory. The steps
below provide a guided tour of what happens when you issue the command of
`./network.sh up`.

- `./network.sh` creates the certificates and keys for two peer organizations
  and the orderer organization. By default, the script uses the cryptogen tool
  using the configuration files located in the `organizations/cryptogen` folder.
  If you use the `-ca` flag to create Certificate Authorities, the script uses
  Fabric CA server configuration files and `registerEnroll.sh` script located in
  the `organizations/fabric-ca` folder. Both cryptogen and the Fabric CAs create
  the crypto material and MSP folders for all three organizations in the
  `organizations` folder.

- The script uses configtxgen tool to create the system channel genesis block.
  Configtxgen consumes the `TwoOrgsOrdererGenesis`  channel profile in the
  `configtx/configtx.yaml` file to create the genesis block. The block is stored
  in the `system-genesis-block` folder.

- Once the organization crypto material and the system channel genesis block have
  been generated, the `network.sh` can bring up the nodes of the netwowrk. The
  script uses the ``docker-compose-test-net.yaml`` file in the `docker` folder
  to create the peer and orderer nodes. The `docker` folder also contains the
  ``docker-compose-e2e.yaml`` file that brings up the nodes of the network
  alongside three Fabric CAs. This file is meant to be used to run end-to-end
  tests by the Fabric SDK. Refer to the [Node SDK](https://github.com/hyperledger/fabric-sdk-node)
  repo for details on running these tests.

- If you use the `createChannel` subcommand, `./network.sh` runs the
  `createChannel.sh` script in the `scripts` folder to create a channel
  using the supplied channel name. The script uses the `configtx.yaml` file to
  create the channel creation transaction, as well as two anchor peer update
  transactions. The script uses the peer cli to create the channel, join
  ``peer0.org1.example.com`` and ``peer0.org2.example.com`` to the channel, and
  make both of the peers anchor peers.

- If you issue the `deployCC` command, `./network.sh` runs the ``deployCC.sh``
  script to install the **fabcar** chaincode on both peers and then define then
  chaincode on the channel. Once the chaincode definition is committed to the
  channel, the peer cli initializes the chainocde using the `Init` and invokes
  the chaincode to put initial data on the ledger.

## Troubleshooting

If you have any problems with the tutorial, review the following:

-  You should always start your network fresh. You can use the following command
   to remove the artifacts, crypto material, containers, volumes, and chaincode
   images from previous runs:
   ```
   ./network.sh down
   ```
   You **will** see errors if you do not remove old containers, images, and
   volumes.

-  If you see Docker errors, first check your Docker version ([Prerequisites](prereqs.html)),
   and then try restarting your Docker process. Problems with Docker are
   oftentimes not immediately recognizable. For example, you may see errors
   that are the result of your node not being able to access the crypto material
   mounted within a container.

   If problems persist, you can remove your images and start from scratch:
   ```
   docker rm -f $(docker ps -aq)
   docker rmi -f $(docker images -q)
   ```

-  If you see errors on your create, approve, commit, invoke or query commands,
   make sure you have properly updated the channel name and chaincode name.
   There are placeholder values in the supplied sample commands.

-  If you see the error below:
   ```
   Error: Error endorsing chaincode: rpc error: code = 2 desc = Error installing chaincode code mycc:1.0(chaincode /var/hyperledger/production/chaincodes/mycc.1.0 exits)
   ```

   You likely have chaincode images (e.g. ``dev-peer1.org2.example.com-fabcar-1.0`` or
   ``dev-peer0.org1.example.com-fabcar-1.0``) from prior runs. Remove them and try
   again.
   ```
   docker rmi -f $(docker images | grep peer[0-9]-peer[0-9] | awk '{print $3}')
   ```

-  If you see the below error:

   ```
   [configtx/tool/localconfig] Load -> CRIT 002 Error reading configuration: Unsupported Config Type ""
   panic: Error reading configuration: Unsupported Config Type ""
   ```

   Then you did not set the ``FABRIC_CFG_PATH`` environment variable properly. The
   configtxgen tool needs this variable in order to locate the configtx.yaml. Go
   back and execute an ``export FABRIC_CFG_PATH=$PWD/configtx/configtx.yaml``,
   then recreate your channel artifacts.

-  If you see an error stating that you still have "active endpoints", then prune
   your Docker networks. This will wipe your previous networks and start you with a
   fresh environment:
   ```
   docker network prune
   ```

   You will see the following message:
   ```
   WARNING! This will remove all networks not used by at least one container.
   Are you sure you want to continue? [y/N]
   ```
   Select ``y``.

-  If you see an error similar to the following:
   ```
   /bin/bash: ./scripts/createChannel.sh: /bin/bash^M: bad interpreter: No such file or directory
   ```

   Ensure that the file in question (**createChannel.sh** in this example) is
   encoded in the Unix format. This was most likely caused by not setting
   ``core.autocrlf`` to ``false`` in your Git configuration (see
    [Windows extras](prereqs.html#windows-extras)). There are several ways of fixing this. If you have
   access to the vim editor for instance, open the file:
   ```
   vim ./fabric-samples/test-network/scripts/createChannel.sh
   ```

   Then change its format by executing the following vim command:
   ```
   :set ff=unix
   ```

- If your orderer exits upon creation or if you see that the create channel
  command fails due to an inability to connect to your ordering service, use
  the `docker logs` command to read the logs from the ordering node. You may see
  the following message:
  ```
  PANI 007 [channel system-channel] config requires unsupported orderer capabilities: Orderer capability V2_0 is required but not supported: Orderer capability V2_0 is required but not supported
  ```
  This occurs when you are trying to run the network using Fabric version 1.4.x
  docker images. The test network needs to run using Fabric version 2.x.

If you continue to see errors, share your logs on the **fabric-questions**
channel on [Hyperledger Rocket Chat](https://chat.hyperledger.org/home) or on
[StackOverflow](https://stackoverflow.com/questions/tagged/hyperledger-fabric).

<!--- Licensed under Creative Commons Attribution 4.0 International License
https://creativecommons.org/licenses/by/4.0/ -->
