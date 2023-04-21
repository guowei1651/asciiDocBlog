# 警告

本文中涉及到国家通信安全。请不要随便尝试。如若尝试后造成任何后果与本文作者无任何关系。

# 整体介绍

下面是从[HackRF One官网](https://greatscottgadgets.com/hackrf/)上抄录的一段：
>来自Great Scott Gadgets的Hackrf One是一款软件定义的无线电（SDR）外围设备，能够传输或接收1MHz到6GHz的无线电信号。Hackrf One旨在测试和开发现代和下一代无线电技术，是一个开源硬件平台，可作为USB外设使用或编程用于独立操作。
HackrfOne有一个注塑塑料外壳，附带一根微型USB电缆。不包括天线。建议将ANT500作为Hackrf One的启动天线。Hackrf One是射频系统的测试设备。它还没有被测试是否符合无线电信号传输的规定。您有责任合法使用您的hackrf。

![HackRF One](https://upload-images.jianshu.io/upload_images/2454595-d659ec61a5c07842.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

HackRF和树莓派一样是一款全开源的硬件项目。不过HackRF的主要是为了提供廉价的SDR方案，作者[Mike Ossmann](https://github.com/mossmann)，这个开源项目的代码和文档都在[github](https://github.com/mossmann/hackrf)上。HackRF跟其他的SDR方案相比是一款覆盖频率最宽，而且价格最低廉的SDR板卡。并且它几乎所有的信息都是开源的，Github仓库中还包括KiCad文件。缺点是它没有FPGA，使用的低速的USB2接口，ADC/DAC的精度比较低。总的来说，HackRF非常适合那些对开放性要求很高的黑客，和那些那些对价格敏感的用户。

[Taylor Killian](http://www.taylorkillian.com)在13年8月发表了一篇关于SDR硬件对比的博文，翻译版： [三款SDR平台对比：HackRF，bladeRF和USRP](https://www.cnblogs.com/k1two2/p/4611003.html)这篇文章对比了当时的三种SDR方案，并且对HackRF的评价也不低。再根据另外一遍[2019年10个流行的软件定义无线电（SDR）](https://blog.bliley.com/10-popular-sdrs-software-defined-radios-2018)

下面是HackRF参数：
>LPC4320/4330： ARM Cortex M4处理器, 主频204MHz
XC2C64A：Xilinx，CoolRunner-II系列CPLD，1500门
MAX2837：2.3GHz到2.7GHz无线宽带射频收发器
RFFC5072：混频器提供80MHz到4200MHz的本振
MAX5864：ADC/DAC, 8-bit，22MHz采样率
Si5351C：I2C可编程任意CMOS时钟生成器，有800MHz分频提供40MHz 50MHz级采样时钟
MGA-81563：0.1–6GHz 3V, 14 dBm 放大器
SKY13317：20 MHz-6.0 GHz 射频单刀三掷(SP3T)开关
SKY13350：0.01-6.0 GHz 射频单刀双掷(SPDT)开关

# 查找频段

因为使用工程机成本较高，并且不单独出售。所以，就找个手机作为私网的终端使用。这个手机的频段如下：
>**FDD LTE**: B1,2,3,4,5,7,8,12,13,17,18,19,20,25,26,28,29,32,66
**TDD-LTE**: B34,38,39,40,41
**TDS**: B34,39
**UMTS**: B1,2,4,5,8,9,19
**CDMA**: BC0,BC1
**GSM**: B2,3,5,8

3GPPP规定的LTE协议的频段划分：
![Band划分](https://upload-images.jianshu.io/upload_images/2454595-271e2d114fd29464.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由于LTE频谱各国不一致，导致3GPP组织在定义LTE频谱时无法如同GSM一样较为规整，所以只能定义LTE支持的频段列表供运营商、通信设备制造商等参照。所以，每个国家都会根据各自国家的情况选用频段。而国内三大运营商所使用频段：
![国内频段使用情况](https://upload-images.jianshu.io/upload_images/2454595-c1d5cd8d6b1c991c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以，可以选择不被国内运营商使用的频段进行通信使用。例如说：Band22，Band42，Band43。然后就对不上上面的手机频段了。那就只能用HackRF把LTE频段扫描出来，找一个离他们很远的频点进行配置。并且把发射功率调到最低。

# 准备

选定好不干扰正常通信的频段后，下一步就是要开始准备各项物料。并开始测试系统是否能正常运行。下面是要准备的内容：

1. 空白/虚拟SIM卡
    淘宝有卖，可以买几张。
2. SIM卡开卡器
    淘宝上有卖，可以买一个。
3. 测试手机
    支持多频段的手机，或者lte网卡。
4. 调试机
    连接HackRF的机器，编译环境运行的机器。
5. 编译环境
    交叉编译工具链。
6. 测试软件
    - 联通测试软件
    - 频段检测软件
    - lte代码
7. HackRF
    - 调试连接线
    - ANT500天线
    - HackRF
    - GPS时钟源/GPS蘑菇头
    - 电源

# 调试HackRF One

这里要做的就是让手机可以在私网LTE基站下拨通电话，所以，不考虑太多其他的内容。例如：核心网，计费等。

我们这里使用开源的LTE源代码，可以选用：[openlte](https://www.openhub.net/p/openlte)或者[srsLTE](https://github.com/srsLTE/srsLTE)进行相关的lte支持。

可以参照 [构建ARM Linux交叉编译工具链 详解](http://www.embeddedlinux.org.cn/emb-linux/system-development/201708/12-7114.html#)进行交叉编译环境的搭建，参照[“HackRF+电池”独立中继1090MHz ADS-B信息经BTLE链路至手机](https://www.hackrf.net/2015/08/hackrf%E7%94%B5%E6%B1%A0%E7%8B%AC%E7%AB%8B%E4%B8%AD%E7%BB%A71090mhz-ads-b%E4%BF%A1%E6%81%AF%E7%BB%8Fbtle%E9%93%BE%E8%B7%AF%E8%87%B3%E6%89%8B%E6%9C%BA/)做固件打包刷如。可以参考[Software Defined Radio with HackRF, Lesson 2:DSP](https://greatscottgadgets.com/sdr/2/)整一下DSP，参考[Software Defined Radio with HackRF, Lesson 5:HackRF One](https://greatscottgadgets.com/sdr/5/)中的相关调试工具。

具体过程我就不写了。因为我这怕这篇过不了审。

# 总结

最重要的是国家通信安全的问题。不要触犯国家法律。也不要在法律的边缘试探。

# 其他更多用法可以参见：
https://cn0xroot.com/tag/hackrf/
https://www.hackrf.net

# 参考：
[HackRF，一款简易的开源软件无线电平台](http://www.witimes.com/hackrf/)
[HackRF - SDR的技术原理介绍及案例分析](http://www.elecfans.com/tongxin/rf/20171121583283_2.html)
[使用HackRF解调TDD-LTE信号](https://cloud.tencent.com/developer/article/1035315)
