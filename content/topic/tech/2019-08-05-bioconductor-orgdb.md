---
title: Bioconductor org.Xx.eg.db 系列包
author: 沈维燕
type: post
topic/tags: ["others", "生信"]
date: 2019-08-05T06:17:11.000Z
category: 生信
published: true
---

在 bioconductor 的官网里面可以查找到 "**OrgDb**" 的包大约有 20 个，基本上跨越了我们生物信息分析中最常用的物种啦！

![Bioconductor-BiocViews.png](https://note.bioitee.com/yuque/0/2019/png/126032/1574738466531-b38e6d4d-03c3-449f-bdf2-a2660e28c373.png#align=left&display=inline&height=926&name=Bioconductor-BiocViews.png&originHeight=926&originWidth=621&size=93293&status=done&width=621)

完整链接：[`http://bioconductor.org/packages/release/BiocViews.html#___OrgDb`](http://bioconductor.org/packages/release/BiocViews.html#___OrgDb)

R 安装示例：
```r
BiocManager::install(c("org.Hs.eg.db", "org.Mm.eg.db", "org.Mmu.eg.db", 
	  "org.Pf.plasmo.db", "org.Pt.eg.db", "org.Rn.eg.db", 
    "org.Sc.sgd.db", "org.Ss.eg.db", "org.Xl.eg.db", 
    "org.Gg.eg.db", "org.EcSakai.eg.db", "org.EcK12.eg.db", 
    "org.Dr.eg.db", "org.Dm.eg.db", "org.Cf.eg.db", 
    "org.Ce.eg.db", "org.Bt.eg.db", "org.At.tair.db", 
    "org.Ag.eg.db"))
```

1. 人类：[org.Hs.eg.db](http://www.bioconductor.org/packages/release/data/annotation/html/org.Hs.eg.db.html)，Homo_sapiens
1. 小鼠：[Bioconductor - org.Mm.eg.db](http://www.bioconductor.org/packages/release/data/annotation/html/org.Mm.eg.db.html)，Mus_musculus 
1. 大鼠：[org.Rn.eg.db](http://bioconductor.org/packages/release/data/annotation/html/org.Rn.eg.db.html)，Rat
1. 酵母：[org.Sc.sgd.db](http://bioconductor.org/packages/release/data/annotation/html/org.Sc.sgd.db.html)，Yeast
1. 飞行类：[org.Dm.eg.db](http://bioconductor.org/packages/release/data/annotation/html/org.Dm.eg.db.html)，Fly
1. 拟南芥：[org.At.tair.db](http://www.bioconductor.org/packages/release/data/annotation/html/org.At.tair.db.html) , Arabidopsis（Arabidopsis_thaliana）
1. 斑马鱼：[org.Dr.eg.db](http://www.bioconductor.org/packages/release/data/annotation/html/org.Dr.eg.db.html)，Zebrafish（Danio_rerio）
1. 蠕虫：[org.Ce.eg.db](http://bioconductor.org/packages/release/data/annotation/html/org.Ce.eg.db.html)，Worm
1. 牛：[org.Bt.eg.db](http://bioconductor.org/packages/release/data/annotation/html/org.Bt.eg.db.html)，Bovine
1. 鸡：[org.Gg.eg.db](http://bioconductor.org/packages/release/data/annotation/html/org.Gg.eg.db.html)，Chicken
1. 犬类：[org.Cf.eg.db](http://bioconductor.org/packages/release/data/annotation/html/org.Cf.eg.db.html)，Canine
1. 猪：[org.Ss.eg.db](http://bioconductor.org/packages/release/data/annotation/html/org.Ss.eg.db.html)，Pig
1. 恒河猴：[org.Mmu.eg.db](http://bioconductor.org/packages/release/data/annotation/html/org.Mmu.eg.db.html)，Rhesus
1. 大肠杆菌 K12 株：[org.EcK12.eg.db](http://bioconductor.org/packages/release/data/annotation/html/org.EcK12.eg.db.html)，E coli strain K12
1. 非洲爪蟾：[org.Xl.eg.db](http://bioconductor.org/packages/release/data/annotation/html/org.Xl.eg.db.html)，Xenopus
1. 按蚊：[org.Ag.eg.db](http://bioconductor.org/packages/release/data/annotation/html/org.Ag.eg.db.html)，Anopheles
1. 黑猩猩：[org.Pt.eg.db](http://bioconductor.org/packages/release/data/annotation/html/org.Pt.eg.db.html)，Chimp
1. 疟原虫：[org.Pf.plasmo.db](http://bioconductor.org/packages/release/data/annotation/html/org.Pf.plasmo.db.html)，Malaria
1. 酒井大肠杆菌：[org.EcSakai.eg.db](http://bioconductor.org/packages/release/data/annotation/html/org.EcSakai.eg.db.html)，E coli strain Sakai
1. 粘球菌 xanthus DK 1622：[org.Mxanthus.db](http://bioconductor.org/packages/release/data/annotation/html/org.Mxanthus.db.html)，Myxococcus xanthus DK 1622
