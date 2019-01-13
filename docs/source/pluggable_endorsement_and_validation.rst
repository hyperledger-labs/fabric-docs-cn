Pluggable transaction endorsement and validation - 交易背书和验证的插件化
================================================

Motivation - 动机
----------

When a transaction is validated at time of commit, the peer performs various
checks before applying the state changes that come with the transaction itself:

当一个交易提交后，在验证的时候，节点会在应用这些状态变更之前进行交易自带的多种检查：

- Validating the identities that signed the transaction.
- 验证签名该交易的身份信息。

- Verifying the signatures of the endorsers on the transaction.
- 验证交易背书者的签名。

- Ensuring the transaction satisfies the endorsement policies of the namespaces
  of the corresponding chaincodes.
- 确定交易满足相关链码命名空间的背书规则。

There are use cases which demand custom transaction validation rules different
from the default Fabric validation rules, such as:

有一些场景需要不同于 Fbaric 默认验证规则的自定义验证规则，比如：

- **UTXO (Unspent Transaction Output):** When the validation takes into account
  whether the transaction doesn't double spend its inputs.
- **UTXO （未消费交易输出）：** 当验证账户中的输入是否被双花的时候。

- **Anonymous transactions:** When the endorsement doesn't contain the identity
  of the peer, but a signature and a public key are shared that can't be linked
  to the peer's identity.
- **匿名交易：** 当背书中不包含节点身份信息，但是共享的签名和公钥没有和节点的身份信息相关联的时候。

Pluggable endorsement and validation logic - 背书和验证插件化的逻辑
------------------------------------------

Fabric allows for the implementation and deployment of custom endorsement and
validation logic into the peer to be associated with chaincode handling in a
pluggable manner. This logic can be either compiled into the peer as built in
selectable logic, or compiled and deployed alongside the peer as a
`Golang plugin <https://golang.org/pkg/plugin/>`_.

Fabric 允许以插件的方式在节点上实施和部署自定义的相关链码的背书和验证逻辑。这个逻辑可以是编译进节点的内置可选逻辑，也可以是编译后作为 `Golang 插件 <https://golang.org/pkg/plugin/>`_ 和节点部署在一起的。


Recall that every chaincode is associated with its own endorsement and validation
logic at the time of chaincode instantiation. If the user doesn't select one, the
default built-in logic is selected implicitly. A peer administrator may alter the
endorsement/validation logic that is selected by extending the peer's local
configuration with the customization of the endorsement/validation logic which is
loaded and applied at peer startup.

重申一下，每一个链码都是在初始化的时候关联它的背书和验证逻辑的。如果用户没有选择，就会选择内置的逻辑。节点管理员可以在节点启动的时候改变背书/验证逻辑，根据扩展节点的本地配置选择自定义的背书/验证逻辑来加载和应用。

Configuration - 配置
-------------

Each peer has a local configuration (``core.yaml``) that declares a mapping
between the endorsement/validation logic name and the implementation that is to
be run.

每一个节点都有一个本地配置文件（``core.yaml``）声明了一个背书/验证逻辑的名字和启用映射。

The default logic are called ``ESCC`` (with the "E" standing for endorsement) and
``VSCC`` (validation), and they can be found in the peer local configuration in
the ``handlers`` section:

默认逻辑被成为 ``ESCC`` （“E”代表 endorsement ）和 ``VSCC`` （validation），你可以在节点本地配置文件中的 ``handlers`` 部分找到他们： 

.. code-block:: YAML

    handlers:
        endorsers:
          escc:
            name: DefaultEndorsement
        validators:
          vscc:
            name: DefaultValidation

When the endorsement or validation implementation is compiled into the peer, the
``name`` property represents the initialization function that is to be run in order
to obtain the factory that creates instances of the endorsement/validation logic.

当背书和验证被编译到节点中的时候， ``name`` 属性代表要运行的为了包含创建背书/验证逻辑实例的工厂函数的初始化函数。

The function is an instance method of the ``HandlerLibrary`` construct under
``core/handlers/library/library.go`` and in order for custom endorsement or
validation logic to be added, this construct needs to be extended with any
additional methods.

这个函数是 ``core/handlers/library/library.go`` 中 ``HandlerLibrary`` 结构的一个实例方法，为了增加自定义背书和验证逻辑，这个结构需要被其他任何方法扩展。

Since this is cumbersome and poses a deployment challenge, one can also deploy
custom endorsement and validation as a Golang plugin by adding another property
under the ``name`` called ``library``.

因为这很麻烦，而且对部署构成了挑战，所以通过在 ``name`` 下增加额外的属性 ``library`` 作为一个 Golang 插件来部署自定义的背书和验证。

For example, if we have custom endorsement and validation logic which is
implemented as a plugin, we would have the following entries in the configuration
in ``core.yaml``:

例如，如果我们实现了一个插件自定义背书和验证逻辑，我们可以在配置文件 ``core.yaml`` 中增加如下入口：

.. code-block:: YAML

    handlers:
        endorsers:
          escc:
            name: DefaultEndorsement
          custom:
            name: customEndorsement
            library: /etc/hyperledger/fabric/plugins/customEndorsement.so
        validators:
          vscc:
            name: DefaultValidation
          custom:
            name: customValidation
            library: /etc/hyperledger/fabric/plugins/customValidation.so

And we'd have to place the ``.so`` plugin files in the peer's local file system.

而且我们必须把 ``.so`` 插件文件放在节点的本地文件系统中。

.. note:: Hereafter, custom endorsement or validation logic implementation is
          going to be referred to as "plugins", even if they are compiled into
          the peer.

Endorsement plugin implementation - 背书插件的实现
---------------------------------

To implement an endorsement plugin, one must implement the ``Plugin`` interface
found in ``core/handlers/endorsement/api/endorsement.go``:

为了实现背书插件，必须实现 ``core/handlers/endorsement/api/endorsement.go`` 中的 ``Plugin`` 接口：

.. code-block:: Go

    // Plugin endorses a proposal response
    type Plugin interface {
    	// Endorse signs the given payload(ProposalResponsePayload bytes), and optionally mutates it.
    	// Returns:
    	// The Endorsement: A signature over the payload, and an identity that is used to verify the signature
    	// The payload that was given as input (could be modified within this function)
    	// Or error on failure
    	Endorse(payload []byte, sp *peer.SignedProposal) (*peer.Endorsement, []byte, error)

    	// Init injects dependencies into the instance of the Plugin
    	Init(dependencies ...Dependency) error
    }

An endorsement plugin instance of a given plugin type (identified either by the
method name as an instance method of the ``HandlerLibrary`` or by the plugin ``.so``
file path) is created for each channel by having the peer invoke the ``New``
method in the ``PluginFactory`` interface which is also expected to be implemented
by the plugin developer:

一个给定插件类型（通过识别方法名是否为 ``HandlerLibrary`` 的实例方法或者 ``.so`` 插件的路径）的背书插件实例，是通过让节点执行 ``PluginFactory`` 接口中的 ``New`` 方法了来让每一个通道创建的，这个方法需要插件的开发者来实现。

.. code-block:: Go

    // PluginFactory creates a new instance of a Plugin
    type PluginFactory interface {
    	New() Plugin
    }


The ``Init`` method is expected to receive as input all the dependencies declared
under ``core/handlers/endorsement/api/``, identified as embedding the ``Dependency``
interface.

``Init`` 方法接收 ``core/handlers/endorsement/api/`` 声明的所有依赖项，他们被表示为嵌入 ``Dependency`` 接口。

After the creation of the ``Plugin`` instance, the ``Init`` method is invoked on
it by the peer with the ``dependencies`` passed as parameters.

在 ``Plugin`` 实例被创建之后，节点将调用 ``Init`` 方法，并将依赖项作为参数传递。

Currently Fabric comes with the following dependencies for endorsement plugins:

目前 Fabric 的背书插件有如下依赖项：

- ``SigningIdentityFetcher``: Returns an instance of ``SigningIdentity`` based
  on a given signed proposal:

- ``SigningIdentityFetcher`` ： 返回一个基于给定签名提案的 ``SigningIdentity`` 实例。

.. code-block:: Go

    // SigningIdentity signs messages and serializes its public identity to bytes
    type SigningIdentity interface {
    	// Serialize returns a byte representation of this identity which is used to verify
    	// messages signed by this SigningIdentity
    	Serialize() ([]byte, error)

    	// Sign signs the given payload and returns a signature
    	Sign([]byte) ([]byte, error)
    }

- ``StateFetcher``: Fetches a **State** object which interacts with the world
  state:

- ``StateFetcher`` ：获取一个和世界状态交互的 **State** 对象。

.. code-block:: Go

    // State defines interaction with the world state
    type State interface {
    	// GetPrivateDataMultipleKeys gets the values for the multiple private data items in a single call
    	GetPrivateDataMultipleKeys(namespace, collection string, keys []string) ([][]byte, error)

    	// GetStateMultipleKeys gets the values for multiple keys in a single call
    	GetStateMultipleKeys(namespace string, keys []string) ([][]byte, error)

    	// GetTransientByTXID gets the values private data associated with the given txID
    	GetTransientByTXID(txID string) ([]*rwset.TxPvtReadWriteSet, error)

    	// Done releases resources occupied by the State
    	Done()
     }

Validation plugin implementation - 验证插件的实现
--------------------------------

To implement a validation plugin, one must implement the ``Plugin`` interface
found in ``core/handlers/validation/api/validation.go``:

实现验证插件，必须实现 ``core/handlers/validation/api/validation.go`` 中的 ``Plugin`` 接口：

.. code-block:: Go

    // Plugin validates transactions
    type Plugin interface {
    	// Validate returns nil if the action at the given position inside the transaction
    	// at the given position in the given block is valid, or an error if not.
    	Validate(block *common.Block, namespace string, txPosition int, actionPosition int, contextData ...ContextDatum) error

    	// Init injects dependencies into the instance of the Plugin
    	Init(dependencies ...Dependency) error
    }

Each ``ContextDatum`` is additional runtime-derived metadata that is passed by
the peer to the validation plugin. Currently, the only ``ContextDatum`` that is
passed is one that represents the endorsement policy of the chaincode:

每一个 ``ContextDatum`` 都是节点传递给验证插件的附加的运行时导出的元数据。现在，传递的唯一 ``ContextDatum`` 表示链码的背书规则。

.. code-block:: Go

   // SerializedPolicy defines a serialized policy
  type SerializedPolicy interface {
	validation.ContextDatum

	// Bytes returns the bytes of the SerializedPolicy
	Bytes() []byte
   }

A validation plugin instance of a given plugin type (identified either by the
method name as an instance method of the ``HandlerLibrary`` or by the plugin ``.so``
file path) is created for each channel by having the peer invoke the ``New``
method in the ``PluginFactory`` interface which is also expected to be implemented
by the plugin developer:


一个给定插件类型（通过识别方法名是否为 ``HandlerLibrary`` 的实例方法或者 ``.so`` 插件的路径）的验证插件实例，是通过让节点执行 ``PluginFactory`` 接口中的 ``New`` 方法了来让每一个通道创建的，这个方法需要插件的开发者来实现。

.. code-block:: Go

    // PluginFactory creates a new instance of a Plugin
    type PluginFactory interface {
    	New() Plugin
    }

The ``Init`` method is expected to receive as input all the dependencies declared
under ``core/handlers/validation/api/``, identified as embedding the ``Dependency``
interface.

``Init`` 方法接收 ``core/handlers/endorsement/api/`` 声明的所有依赖项，他们被表示为嵌入 ``Dependency`` 接口。

After the creation of the ``Plugin`` instance, the **Init** method is invoked on
it by the peer with the dependencies passed as parameters.

在 ``Plugin`` 实例被创建之后，节点将调用 **Init** 方法，并将依赖项作为参数传递。

Currently Fabric comes with the following dependencies for validation plugins:

目前 Fabric 的验证插件有如下依赖项：

- ``IdentityDeserializer``: Converts byte representation of identities into
  ``Identity`` objects that can be used to verify signatures signed by them, be
  validated themselves against their corresponding MSP, and see whether they
  satisfy a given **MSP Principal**. The full specification can be found in
  ``core/handlers/validation/api/identities/identities.go``.

- ``IdentityDeserializer`` ：将表示身份的字节码转换为 ``Identity`` 对象，以便通过和他们相关的 MSP 验证他们的签名，和判断是否满足 **MSP 规则** 。完整的定义在 ``core/handlers/validation/api/identities/identities.go`` 。

- ``PolicyEvaluator``: Evaluates whether a given policy is satisfied:

- ``PolicyEvaluator`` ：判断给定的策略是否合适：

.. code-block:: Go

    // PolicyEvaluator evaluates policies
    type PolicyEvaluator interface {
    	validation.Dependency

    	// Evaluate takes a set of SignedData and evaluates whether this set of signatures satisfies
    	// the policy with the given bytes
    	Evaluate(policyBytes []byte, signatureSet []*common.SignedData) error
    }

- ``StateFetcher``: Fetches a ``State`` object which interacts with the world state:

- ``StateFetcher`` ：获取一个和世界状态交互的 **State** 对象。

.. code-block:: Go

    // State defines interaction with the world state
    type State interface {
        // GetStateMultipleKeys gets the values for multiple keys in a single call
        GetStateMultipleKeys(namespace string, keys []string) ([][]byte, error)

        // GetStateRangeScanIterator returns an iterator that contains all the key-values between given key ranges.
        // startKey is included in the results and endKey is excluded. An empty startKey refers to the first available key
        // and an empty endKey refers to the last available key. For scanning all the keys, both the startKey and the endKey
        // can be supplied as empty strings. However, a full scan should be used judiciously for performance reasons.
        // The returned ResultsIterator contains results of type *KV which is defined in protos/ledger/queryresult.
        GetStateRangeScanIterator(namespace string, startKey string, endKey string) (ResultsIterator, error)

        // GetStateMetadata returns the metadata for given namespace and key
        GetStateMetadata(namespace, key string) (map[string][]byte, error)

        // GetPrivateDataMetadata gets the metadata of a private data item identified by a tuple <namespace, collection, key>
        GetPrivateDataMetadata(namespace, collection, key string) (map[string][]byte, error)

        // Done releases resources occupied by the State
        Done()
    }

Important notes - 重要提醒
---------------

- **Validation plugin consistency across peers:** In future releases, the Fabric
  channel infrastructure would guarantee that the same validation logic is used
  for a given chaincode by all peers in the channel at any given blockchain
  height in order to eliminate the chance of mis-configuration which would might
  lead to state divergence among peers that accidentally run different
  implementations. However, for now it is the sole responsibility of the system
  operators and administrators to ensure this doesn't happen.

- **验证插件的跨节点一致性：** 未来的发布版本中，为了消除可能导致节点突然运行不同实现的状态差异的错误配置的可能性， Fabric 通道基础设施将确保通道中所有节点的给定链码使用同样的验证逻辑。但是，现在系统操作员和管理员唯一的职责就是确保它不会发生。

- **Validation plugin error handling:** Whenever a validation plugin can't
  determine whether a given transaction is valid or not, because of some transient
  execution problem like inability to access the database, it should return an
  error of type **ExecutionFailureError** that is defined in ``core/handlers/validation/api/validation.go``.
  Any other error that is returned, is treated as an endorsement policy error
  and marks the transaction as invalidated by the validation logic. However,
  if an ``ExecutionFailureError`` is returned, the chain processing halts instead
  of marking the transaction as invalid. This is to prevent state divergence
  between different peers.

- **验证插件错误处理：** 任何时候验证插件不能判定一个交易是否合法，由于某些临时执行问题，比如无数据库权限，它应该返回一个 **ExecutionFailureError** 类型的错误，该错误定义在 ``core/handlers/validation/api/validation.go`` 。其他返回的错误，被当做背书策略错误并且验证逻辑把交易标记为无效。另外，如果返回一个 ``ExecutionFailureError`` ，链处理将停止而不是标记交易为无效。这是防止不同节点之间的状态差异。

- **Error handling for private metadata retrieval**: In case a plugin retrieves
  metadata for private data by making use of the ``StateFetcher`` interface,
  it is important that errors are handled as follows: ``CollConfigNotDefinedError''
  and ``InvalidCollNameError'', signalling that the specified collection does
  not exist, should be handled as deterministic errors and should not lead the
  plugin to return an ``ExecutionFailureError``.
  
- **私有元数据检索的错误处理：** 如果插件使用 ``StateFetcher`` 接口来检索私有数据的元数据，必须按一下方式处理错误： ``CollConfigNotDefinedError`` 和 ``InvalidCollNameError`` 表示指定集合不存在，应该按确定性错误处理而不应该让插件返回 ``ExecutionFailureError`` 。

- **Importing Fabric code into the plugin**: Importing code that belongs to Fabric
  other than protobufs as part of the plugin is highly discouraged, and can lead
  to issues when the Fabric code changes between releases, or can cause inoperability
  issues when running mixed peer versions. Ideally, the plugin code should only
  use the dependencies given to it, and should import the bare minimum other
  than protobufs.

- **将 Fabric 代码导入插件：** 将属于 Fabric 而不是 protobufs 的代码作为插件的一部分是不鼓励的，当不同发布版本的 Fabric 代码不同时会导致问题，或者在运行不同节点版本时导致不可操作的问题。理想情况下，插件代码应该值使用给定的依赖项，最小化导入 protobufs 以外的值。

  .. Licensed under Creative Commons Attribution 4.0 International License
     https://creativecommons.org/licenses/by/4.0/
