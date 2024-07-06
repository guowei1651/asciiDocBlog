---
title: "NAS缓存用SSD的选型，规格M.2 2280 NVMe PCIE3.0"
lead: "随笔"
date: 2023-04-22T12:52:56+08:00
lastmod: 2023-04-22T12:52:56+08:00
draft: false
images: []
menu:
  essay:
    parent: "essay"
weight: 4000
toc: true
---


NAS缓存用SSD的选型，规格M.2 2280 NVMe PCIE3.0

## 品牌对比

|品牌|型号|容量|读速度|MTTF|TBW|JD价格|TBW价格比(TBW/元)|TB价格|TBW价格比(TBW/元)|备注|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|[西部数据](https://item.jd.com/100030458284.html)|Red SN700|500GB|3430MB/s|175万|1000|899|1.11|558|1.79|[官网](https://www.westerndigital.com/zh-cn/products/internal-drives/wd-red-sn700-nvme-ssd?sku=WDS500G1R0C)|
|[三星](https://item.jd.com/100002183459.html)|970 EVO Plus|500G|3200MB/s|X|300|479|0.63|325(工包)|0.92|[官网](https://www.samsung.com/tw/memory-storage/nvme-ssd/970-evo-plus-nvme-m-2-ssd-500gb-mz-v7s500bw/)|
|[致态长江存储](https://item.jd.com/100018551169.html)|TiPlus7100|512GB|7000MB/s|150万|300|389|0.77|540(1T)|1.11(600TBW)|没官网|
|[金士顿](https://item.jd.com/100036659931.html)|NV2|512GB|3500MB/s|X|160|319|0.50|319|0.50|[官网](https://www.kingston.com/tw/ssd/nv2-nvme-pcie-ssd?capacity=500gb)|
|[宏碁掠夺者](https://item.jd.com/100011580275.html)|GM3500系列|1TB|3400MB/s|X|600|459|1.31|X|X|[官网](https://www.predatorstorage.cn/products/predator-gm3500-m2-pcie-gen3x4-nvme13.html)|
|[希捷](https://item.jd.com/100021413570.html)|希捷酷鱼Q5|500GB|2300MB/s|180万|531|309|1.72|||官网没有|
|[铠侠](https://item.jd.com/100032601800.html)|EXCERIA G2 RC20系列|500GB|2100MB/s|150万|200|319|0.63|336|0.60|[官网](https://tw.kioxia.com/zh-tw/personal/ssd/exceria-g2.html)|
|[闪迪](https://item.jd.com/100011213834.html)|游戏高速版|250GB|3500MB/s|X|300|369|0.81|299|1.00|官网没有|
|[SK HYNIX海力士](https://item.jd.com/100069532463.html)|P31高性能版|500GB|3500MB/s|X|500|399|1.25|399|1.25|官网没有|

选择同样的价格下买到的TBW最多的，并有足够的公司能力支撑。

## 西部数据 Red SN700 的具体规格对比

|容量(G)|总写入量(TBW)|价格(元)|TBW价格比(TBW/元)|擦写次数(次)|单位存储价格(G/元)|积分|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|250|500|418|1.196172249|2000|0.598086124|75.19|
|500|1000|558|1.792114695|2000|0.896057348|81.7|
|1000|2000|1058|1.890359168|2000|0.945179584|77.75|
|2000|2500|2099|1.191043354|1250|0.952834683|52.82|
|4000|5100|4499|1.133585241|1275|0.889086464|47.46|

> 积分计算公式：积分 = TBW价格比 * 32.5 +	擦写次数 * 0.001 + 单位存储价格 * 7.2 + 15000 / 价格 

> 在NAS场景下TBW是最主要考虑的点所以权重最重。擦写次数在TBW中已经有所体现这里简单的加一点。单位存储价格也体现在用量上。价格越低分数越高，价格以反比关系加入到积分中。

> 在算法中容量以一阶方式体现，TBW以二阶方式体现，价格以反比的三次体现。

**积分越大代表着越值得购买。**