Cloud Native
=====
[迁移到云原生]: https://jimmysong.io/migrating-to-cloud-native-application-architectures/

到了2017年, 云原生应用的提出者之一的Pivotal在其官网上将云原生的定义概况为DevOps、持续交付、微服务、容器这四大特征，这也成了很多人对 Cloud Native的基础印象。

>Cloud native technologies empower organizations to build and run scalable applications in modern, dynamic environments such as public, private, and hybrid clouds. Containers, service meshes, microservices, immutable infrastructure, and declarative APIs exemplify this approach.
>These techniques enable loosely coupled systems that are resilient, manageable, and observable. Combined with robust automation, they allow engineers to make high-impact changes frequently and predictably with minimal toil.
>The Cloud Native Computing Foundation seeks to drive adoption of this paradigm by fostering and sustaining an ecosystem of open source, vendor-neutral projects. We democratize state-of-the-art patterns to make these innovations accessible for everyone.

**为云而生**

总结一下就是：

- （1）基于容器、服务网格、微服务、不可变基础设施和声明式API构建的可弹性扩展的应用；
- （2）基于自动化技术构建具备高容错性、易管理和便于观察的松耦合系统；
- （3）构建一个统一的开源云技术生态，能和云厂商提供的服务解耦，

ME:基于云而生，为云而生



#### CN=微服务+DevOps+CD/ID+容器化



**微服务** (Microservices) 是一种[软件架构风格](https://zh.wikipedia.org/wiki/软件架构)，它是以专注于单一责任与功能的小型功能区块 (Small Building Blocks) 为基础，利用模块化的方式组合出复杂的大型应用程序，各功能区块使用与语言无关 (Language-Independent/Language agnostic) 的 API 集相互通信。

**DevOps**（**Dev**elopment和**Op**erations的组合词）是一种重视“软件开发人员（Dev）”和“IT运维技术人员（Ops）”之间沟通合作的文化、运动或惯例。透过自动化“软件交付”和“架构变更”的流程，来使得构建、测试、发布软件能够更加地快捷、频繁和可靠。

**CD**（英语：Continuous delivery，缩写为 CD），是一种[软件工程](https://zh.wikipedia.org/wiki/軟體工程)手法，让软件产品的产出过程在一个短周期内完成，以保证软件可以稳定、持续的保持在随时可以释出的状况。它的目标在于让软件的建置、测试与释出变得更快以及更频繁。这种方式可以减少软件开发的成本与时间，减少风险。*要做到这点非常非常难*

**containerized**
 容器化的好处在于运维的时候不需要再关心每个服务所使用的技术栈了，每个服务都被无差别地封装在容器里，可以被无差别地管理和维护，现在比较流行的工具是docker和k8s。

