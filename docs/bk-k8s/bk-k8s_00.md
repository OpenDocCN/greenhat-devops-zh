## 前言

![image](img/common01.jpg)

容器和 Kubernetes 一起正在改变应用程序的架构、开发和部署方式。容器确保软件在任何部署地点都能可靠运行，而 Kubernetes 让你可以从单一控制平面管理所有容器。

本书旨在帮助你充分利用这些重要的新技术，采用实践示例，不仅尝试主要功能，还探索每个功能的工作原理。通过这种方式，除了能够准备好将应用程序部署到 Kubernetes 外，你还将获得设计高效且可靠的 Kubernetes 集群中应用程序架构的技能，并能够在问题出现时快速诊断。

### 方法论

Kubernetes 集群的最大优势在于，它通过抽象层隐藏了在多个主机上运行容器的工作。Kubernetes 集群是一个“黑盒”，我们告诉它运行什么，它就运行什么，具备自动扩展、故障切换和应用程序的新版本升级等功能。

尽管这种抽象使得部署和管理应用程序变得更加容易，但它也使得理解集群正在做什么变得困难。因此，本书从“调试”视角呈现每个容器运行时和 Kubernetes 集群的功能。每一次好的调试会话都从将应用程序当作黑盒并观察其行为开始，但它不会仅止步于此。经验丰富的问题解决者知道如何打开黑盒，深入当前的抽象层以下，查看程序是如何运行的，数据是如何存储的，以及流量是如何在网络中流动的。熟练的架构师利用对系统的深刻理解，避免性能和可靠性问题。本书提供了对容器和 Kubernetes 的详细理解，这种理解来源于不仅探索这些技术做了什么，还要了解它们是如何工作的。

在第一部分中，我们将从运行一个容器开始，然后深入容器运行时，理解什么是容器以及如何使用普通操作系统命令模拟容器。在第二部分中，我们将安装一个 Kubernetes 集群并将容器部署到集群中。我们还将看到集群如何工作，包括它如何与容器运行时交互，以及数据包如何在主机网络中从一个容器流向另一个容器。本书的目的是不是为了重复参考文档，列出每个功能提供的所有选项，而是演示每个功能如何实现，从而使得所有文档内容都能理解且有用。

Kubernetes 集群非常复杂，因此本书包含了大量的实践示例，并提供了足够的自动化工具，使你能够独立探索每一章。这些自动化工具可以在 *[`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)* 上找到，并以宽松的开源许可证发布，因此你可以在自己的项目中进行探索、实验和使用。

### 运行示例

在本书的许多示例练习中，你将把多个主机组合在一起以构建一个集群，或者操作 Linux 内核的低级功能。基于这个原因，并且为了帮助你在实验过程中感到更舒适，你将完全在临时虚拟机上运行示例。这样，如果你犯了错误，可以迅速删除虚拟机并重新开始。

本书的示例代码库可以在 *[`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)* 上找到。所有设置示例运行的说明都在示例代码库的 *setup* 文件夹中的 *README.md* 文件里。

#### 你需要的东西

即使你在虚拟机中工作，你仍然需要一台控制机器作为起始点，可以运行 Windows、macOS 或 Linux。它甚至可以是一台支持 Linux 的 Chromebook。如果你使用 Windows，你需要使用 Windows Subsystem for Linux (WSL) 来使 Ansible 正常工作。有关详细说明，请参见 *setup* 文件夹中的 *README.md* 文件。

#### 在云端或本地运行

为了尽可能让这些示例易于访问，我提供了自动化工具，可以通过 Vagrant 或 Amazon Web Services (AWS) 运行它们。如果你有一台至少具有八核和 8GB 内存的 Windows、macOS 或 Linux 计算机，可以尝试安装 VirtualBox 和 Vagrant，并使用本地虚拟机。如果没有，你可以设置自己在 AWS 上工作。

我们使用 Ansible 来执行 AWS 设置并自动化一些繁琐的步骤。每个章节都包含一个单独的 Ansible 剧本，利用了常见的角色和集合。这意味着你可以逐章工作，每次从全新安装开始。在某些情况下，我还提供了一个“额外的”配置剧本，你可以选择性使用它跳过一些详细的安装步骤，直接进入学习内容。有关更多信息，请参阅每个章节目录中的 *README.md* 文件。

#### 终端窗口

在你使用 Ansible 配置好虚拟机后，你需要至少一个终端窗口来运行命令。每个章节中的 *README.md* 文件会告诉你如何做到这一点。在运行任何示例之前，你首先需要成为 root 用户，如下所示：

```
sudo su -
```

这将为你提供一个 root shell，并设置你的环境和主目录以匹配。

**以 ROOT 用户身份运行**

如果你以前使用过 Linux，可能会对以 root 用户身份频繁工作感到不安，因此你可能会感到惊讶，本书中的所有示例都是以 root 用户身份运行的。这是使用临时虚拟机和容器的一大优势；当我们以 root 用户身份操作时，我们是在一个临时的、受限的空间内进行的，这个空间无法影响到其他任何东西。

当你从学习容器和 Kubernetes 转向在生产环境中运行应用时，你将会为集群应用安全控制措施，这些措施将限制管理员访问权限，并确保容器无法突破其隔离环境。这通常包括配置容器使其以非 root 用户身份运行。

在某些示例中，你可能需要打开多个终端窗口，以便在检查一个进程时保持另一个进程在运行。如何操作取决于你；大多数终端应用都支持多个标签页或多个窗口。如果你需要一种在单个标签页中打开多个终端的方法，可以尝试使用终端复用器应用。所有示例中使用的临时虚拟机都预装了 `screen` 和 `tmux`，可以随时使用。
