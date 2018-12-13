System Chaincode Plugins-系统链码插件
========================

System chaincodes are specialized chaincodes that run as part of the peer process
as opposed to user chaincodes that run in separate docker containers. As
such they have more access to resources in the peer and can be used for
implementing features that are difficult or impossible to be implemented through
user chaincodes. Examples of System Chaincodes include QSCC (Query System Chaincode)
for ledger and other Fabric-related queries, CSCC (Configuration System Chaincode)
which helps regulate access control, and LSCC (Lifecycle System Chaincode).

系统链码是专用链码，作为peer节点进程的一部分运行，而不是在单独的docker容器中运行的用户代码。 因此，他们可以更多地访问peer中的资源，并且可以用于实现难以或不可能通过用户链码实现的特征。 系统链码的示例包括用于超级账本和其他与Fabric相关的查询的QSCC（查询系统链接），用于帮助调节访问控制的CSCC（配置系统链码）和LSCC（生命周期系统链码）。

Unlike a user chaincode, a system chaincode is not installed and instantiated
using proposals from SDKs or CLI. It is registered and deployed by the peer at
start-up.

与用户链码不同，系统链码未使用SDK或CLI中的提议进行安装和实例化。 它在启动时由peer注册和部署。

System chaincodes can be linked to a peer in two ways: statically, and dynamically
using Go plugins. This tutorial will outline how to develop and load system chaincodes
as plugins.

系统链码可以通过两种方式链接到peer：静态和动态使用Go插件。 本教程将概述如何开发系统链码插件和加载系统链码插件。

Developing Plugins
------------------

A system chaincode is a program written in `Go <https://golang.org>`_ and loaded
using the Go `plugin <https://golang.org/pkg/plugin>`_ package.

系统链代码是用Go <https://golang.org>`_编写的程序，使用Go`插件<https://golang.org/pkg/plugin>`_ package加载。

A plugin includes a main package with exported symbols and is built with the command
``go build -buildmode=plugin``.

插件包含一个带有导出符号的主包，并使用命令``go build -buildmode = plugin``构建。

Every system chaincode must implement the `Chaincode Interface <https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#Chaincode>`_
and export a constructor method that matches the signature ``func New() shim.Chaincode``
in the main package. An example can be found in the repository at ``examples/plugin/scc``.

每个系统链码都必须实现`Chaincode接口<https://godoc.org/github.com/hyperledger/fabric/core/chain ode / shim #Chaincode>`_并在主包中导出一个与签名``func New() shim.Chaincode``匹配的构造方法。 可以在``examples / plugin / scc``的存储库中找到一个示例。

Existing chaincodes such as the QSCC can also serve as templates for certain
features, such as access control, that are typically implemented through
system chaincodes. The existing system chaincodes also serve as a reference for
best-practices on things like logging and testing.

诸如QSCC的现有链码还可以用作通常通过系统链码实现的某些特征的模板，例如访问控制。 现有的系统链码也可作为日志记录和测试等最佳实践的参考。

.. note:: On imported packages: the Go standard library requires that a plugin must
​          include the same version of imported packages as the host application
​          (Fabric, in this case).

注意：在导入的包上：Go标准库要求插件必须包含与主机应用程序相同的导入包版本（在本例中为Fabric）。

Configuring Plugins
-------------------

Plugins are configured in the ``chaincode.systemPlugin`` section in ``core.yaml``:

插件在``core.yaml``的``chaincode.systemPlugin``部分配置：

.. code-block:: bash

  chaincode:
​    systemPlugins:
​      - enabled: true
​        name: mysyscc
​        path: /opt/lib/syscc.so
​        invokableExternal: true
​        invokableCC2CC: true

A system chaincode must also be whitelisted in the ``chaincode.system`` section
in ``core.yaml``:

系统链码也必须列在``core.yaml``的``chaincode.system``部分中：

.. code-block:: bash

  chaincode:
​    system:
​      mysyscc: enable


.. Licensed under Creative Commons Attribution 4.0 International License
   https://creativecommons.org/licenses/by/4.0/
