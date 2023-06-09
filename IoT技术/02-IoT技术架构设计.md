# 背景

因作者在工作的前几年在做跟硬件相关的内容。所以，对于IoT技术进行了一些理解。在2018年的时候总结出来，并进行了一些通用型IoT项目的思考。在这里公开出来。

本文原先为PPT，这里将PPT内容粘贴出来并对其中内容进行一些简要的文字描述。PPT以五个部分为基础进行说明：技术方向，技术选型，架构设计，概要设计，环境准备。

# 技术方向

![Gartner分析结论](https://upload-images.jianshu.io/upload_images/2454595-b81aa124b1401bb6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)
世界著名的IT咨询公司对于IoT的评价以及分析。Gartner对于IoT的前景还是比较看好的。

![物联网产业四层价值链](https://upload-images.jianshu.io/upload_images/2454595-c10c526f501f2986.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)
物联网的技术体系主要包括四个层次：感知与控制层、网络层、平台服务层、应用服务层。物联网平台是物联网产业链的枢纽，向下接入分散的物联网传感层，汇集传感数据，向上面向应用服务提供商提供应用开发的基础性平台和面向底层网络的统一数据接口，支持具体的基于传感数据的物联网应用。 目前物联网平台可以部署在企业私有云和物联网厂商的公有云上。

感知与控制层： 通过从传感器、计量器等器件获取环境、资产或者运营状态信息，在进行适当的处理之后，通过传感器传输网关将数据传递出去；同时通过传感器接收网关接收控制指令信息，在本地传递给控制器件达到控制资产、设备及运营的目的。 在此层次中，感知及控制器件的管理，传输与接收网关，本地数据及信号处理是重要的技术领域。

通信网络层： 通过公网或者专网以无线或者有线的通讯方式将信息、数据与指令在感知控制层与平台及应用层之间传递。 主要由运营商提供的各种广域 IP 通信网络组成，包括 ATM、xDSL、光纤等有线网络，以及 GPRS、 3G、 4G、 NB-IoT 等移动通信网络。

平台服务层： 物联网平台是物联网网络架构和产业链条中的关键环节，通过它不仅实现对终端设备和资产的“管、控、营”一体化， 向下连接感知层， 向上面向应用服务提供商提供应用开发能力和统一接口。并为各行各业提供通用的服务能力，如数据路由、数据处理与挖掘、仿真与优化、 业务流程和应用整合、通信管理、 应用开发、设备维护服务等。

应用服务层： 丰富的应用是物联网的最终目标，未来基于政府、企业、消费者三类群体将衍生出多样化物联网应用，创造巨大社会价值。 根据企业业务需要,在平台服务层之上建立相关的物联网应用,例如: 城市交通情况的分析与预测；城市资产状态监控与分析；环境状态监控、分析与预警(例如：风力、雨量、滑坡)；健康状况监测与医疗方案建议等等。

![技术前提](https://upload-images.jianshu.io/upload_images/2454595-6870a84864fc5604.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)
左图为开放组体系结构框架（TOGAF）中的架构开发方法（ADM）。参照这个方法做架构设计方法需要在设计之前做需求的了解。整体业务架构，业务方向等了解。

从业务架构与业务方向初步判断，右侧箭头中的：硬件设备，设备通信，IOT平台，业务场景。其中橙色部分为我们会涉及到的整体解决方案。设备通信部分借助通信运营商或者自建网络完成。

# IOT技术选型

![技术需求分析](https://upload-images.jianshu.io/upload_images/2454595-9e63d2cea82c2138.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)

因此物联网平台从底层到高层可分为四大平台类型：设备管理平台 DMP、 连接管理平台 CMP、应用使能平台 AEP、业务分析平台 BAP 

DMP 平台对物联网终端进行远程监控、 设臵调整、 软件升级、 系统升级、 故障排查、 生命周期管理等功能。 同时可实时提供网关和应用状态监控告警反馈， 为预先处理故障提供支撑，提高客户服务满意度；开放的 API调用接口则能帮助客户轻松地进行系统集成和增值功能开发；所有设备的数据可以存储在云端。

CMP 平台一般应用于运营商网络上， 实现对物联网连接配臵和故障管理、 保证终端联网通道稳定、 网络资源用量管理、 连接资费管理、账单管理、 套餐变更、 号码/IP 地址/Mac 资源管理， 更好的帮助移动运营商做好物联网 SIM 的管理，运营商客户还可以自主进行 SIM 卡管控，自主查看账单。

AEP 平台是提供应用开发和统一数据存储两大功能的 PaaS 平台，架构在 CMP 平台之上。 AEP 平台具体功能有提供成套应用开发工具（大部分能提供图形化开发工具， 甚至不需要开发者编写代码）、中间件、 数据存储功能、业务逻辑引擎、 对接第三方系统 API 等。

BAP平台包含基础大数据分析服务和机器学习两大功能。 大数据服务：平台在集合各类相关数据后，进行分类处理、分析并提供视觉化数据分析结果；通过实时动态分析，监控设备状态并予以预警。 平台的机器学习： 通过对历史数据进行训练生成预测模型或者客户根据平台提供工具自己开发模型，满足预测性的、认知的或复杂的分析业务逻辑。 未来 IoT 平台上的机器学习将向人工智能过度。

![技术需求--整体架构分层](https://upload-images.jianshu.io/upload_images/2454595-916acdd31011701b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)
 - 企业层
每个企业的商业模式都不相同，系统平台需要能够适应企业层中特异化的业务所造成的系统的不同。

- 平台层
上下层的一个连接功能。这一层中包罗万象。需要可以管理设备，并提供设备数据的分析能力。这样才能连接上下层。

- 通信层
为设备与服务平台之间提供通信机制。从物理承载层到业务协议层都需要有相关的解决方案。

- 边缘层
主要是设备，资产，上位机等等组成。为我们提供真正连接资产，检查资产状态的能力。

![技术需求--设备层](https://upload-images.jianshu.io/upload_images/2454595-dce68eca95001672.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)
![技术需求--通信层0](https://upload-images.jianshu.io/upload_images/2454595-e74c1b4dedc927a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)

根据IOT场景的不同可以使用不同的设备通信方式完成。例如：城市环卫，车载设备使用NB-IoT，因为范围较大，设备还可能在移动。家用设备可能使用Wifi，一个家庭范围比较小，网络带宽要求高等等

上图中说明设备通信中的解决方案。此图中没有说明通信功率，可以简单的说明传输带宽越高，通信功耗越高。功耗说明设备在用电池供电时所能支撑的最长时间。

![技术需求--通信层1](https://upload-images.jianshu.io/upload_images/2454595-3fb1456287799142.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)
在物联网协议中，我们一般分为两大类，一类是传输协议，一类是通信协议。传输协议一般负责子网内设备间的组网及通信；通信协议则主要是运行在传统互联网TCP/IP协议之上的设备通讯协议，负责设备通过互联网进行数据交换及通信。

![技术需求--通信层2](https://upload-images.jianshu.io/upload_images/2454595-f2e36b4ff7b5d6ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)
DDS(Data Distribution Service for Real-Time Systems)，面向实时系统的数据分布服务，适用范围：分布式高可靠性、实时传输设备数据通信。目前DDS已经广泛应用于国防、民航、工业控制等领域。应用于国防数据广播性下发的能力。
MQTT是一种基于发布 / 订阅 (publish/subscribe) 模式的 “轻量级” 通讯协议，该协议构建于 TCP/IP 协议上。与DDS相比实时性与服务发现方面的内容。
一般情况下会使用MQTT协议，配合动态发现用来完成整体通信。
右图为业界在IOT协议的使用情况，之后可以参考张图完成IOT协议选择。

![技术需求--信息层0--协议](https://upload-images.jianshu.io/upload_images/2454595-e752610f6fefda50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)


![技术需求--信息层1--存储](https://upload-images.jianshu.io/upload_images/2454595-b31023e90ab58c22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)

![技术需求--功能层](https://upload-images.jianshu.io/upload_images/2454595-abe973a905b23112.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)

![技术需求--业务层](https://upload-images.jianshu.io/upload_images/2454595-8cde37608d8aa52f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
软件即服务有时被作为“即需即用软件”提及，它是一种软件交付模式。在这种交付模式中云端集中式托管软件及其相关的数据，软件仅需透过互联网，而不须透过安装即可使用。用户通常使用精简客户端经由一个网页浏览器来访问软件即服务。 对于许多商业应用来说，软件即服务已经成为一种常见的交付模式。 

# IoT系统架构设计
![业界IoT架构](https://upload-images.jianshu.io/upload_images/2454595-93f1d657474c0dc7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)

![AWS IoT解决方案](https://upload-images.jianshu.io/upload_images/2454595-e9be537cb0d08949.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)

![AWS IIoT解决方案](https://upload-images.jianshu.io/upload_images/2454595-4c63a763923212e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)

![公司IOT解决方案](https://upload-images.jianshu.io/upload_images/2454595-a38221759f02a10c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)

![IoT业务蓝图](https://upload-images.jianshu.io/upload_images/2454595-33c0fc17e364f2c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)
1. 左图中为IoT架构。其中包含很多内容，主要有IaaS和Paas&SaaS组成。IaaS为云服务提供基础的运行环境。PaaS&SaaS为客户提供IoT管理服务。
2. PaaS&SaaS为提供不同的终端使用界面。PC端和移动端为IoT提供界面化的服务，第三方为第三方数据使用的接口服务，支付为系统支付接口为一些特定IoT设备提供支付服务。可能会有其他内容，所以之后扩展。
3. 设备接入和事件模型是系统对接设备的必要内容，可以在不进行代码修改的情况下支持第三方设备，最新设备加入的平台。
4. 业务服务中是系统中主要业务，用来给IoT平台提供所有的服务。这里可以为IoT提供设备、资产、告警、事件、用户、系统、租户地域、对时、第三方整合等等服务。其中搜索服务，数据可视化都与底层的分析平台有密不可分的关系。
5. 接口管理和安全管理都是为第三方提供服务的内容。可以为第三方提供数据访问的权限，可以帮助客户建立更好的上层应用。
6. 分析平台包括存储，计算，学习的内容。这个平台上主要承载数据并记录数据，以及对数据的分析工作。

![IoT系统部署](https://upload-images.jianshu.io/upload_images/2454595-eca966512f7bd1c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)
左图为IoT Cloud整体框架，在上层框中为PaaS和SaaS部分，下层为IaaS的支撑。在IaaS层可以使用公有云或者私有云。公有云和私有云都有一个问题，在于全国性的设备部署时，需要尽快处理时就会有问题。所以，一般会在设备当地建设一台聚合服务器。用于设备现场处理，另一点可以分散云平台压力。
聚合服务器上可以实现PaaS&SaaS相关的功能为不同的客户提供服务。
PaaS&SaaS相关的服务需要人工完成实现。具体内容会在后面进行详细的说明。设备接入为IoT云接入设备时使用，在使用过程中可能需要。接口管理将数据服务提供给应用前端。也会提供给第三方应用。这两层还包裹了业务层和底层，业务层主要处理系统业务。底层主要提供模型能力，计算能力，安全，可靠等等等。

![系统技术架构](https://upload-images.jianshu.io/upload_images/2454595-44d7cd22141b70e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)

![性能设计](https://upload-images.jianshu.io/upload_images/2454595-98666b53ebbccf4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)

![安全设计](https://upload-images.jianshu.io/upload_images/2454595-872fc965d79611e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)

![系统安全设计](https://upload-images.jianshu.io/upload_images/2454595-5a1aea7c7a866356.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)

![可靠性&可扩展性设计](https://upload-images.jianshu.io/upload_images/2454595-84b2d52173678636.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)

![运维&DevOps设计](https://upload-images.jianshu.io/upload_images/2454595-92c2b0e3d798a2c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/740)

# IoT技术概要设计
因为每个IoT都有它需要关注与重点，这里就不进行通用型考虑了。

# 环境准备
![环境准备](https://upload-images.jianshu.io/upload_images/2454595-bbb8776f9f7dabad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 总结
这里汇总了网上对于IoT的一些资料，并加入了作者的一些理解。可以尽量为各位提供一种IoT的设计思路。
