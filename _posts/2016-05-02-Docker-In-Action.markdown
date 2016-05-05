---
layout: post
title: "Docker 拾遗 —— 《Docker in Action》读书笔记"
date: 2016-05-02 08:18:29
categories: Docker
---

 > All problems in computer science can be solved by another level of indirection, except of course for the problem of too many indirections."      
 >>-- <cite>David Wheeler</cite>                                

Docker正提供了这样的一种抽象，抽象能大大简化复杂问题。比如对于安装软件，Docker 提供了开箱即用的解决方案。和传统的“绿色版”安装过程类似，你只需要关心要安装什么，而不是怎么装。如果你有处女座的洁癖的话，那对Docker 在卸载应用时的表现再喜欢不过了，因为你不用担心删除时会遗留下任何残留文件。你或许会好奇，为什么Docker的Logo 是一只拖着集装箱的大鲸鱼？我觉得这个Logo 很传神的表现了Docker的特点。集装箱为现代海运带来了革命性的变化，船长不必关心运输的是什么货物，船厂也不需要为货物做特殊的定制，而物流公司能很好的定量计算每次的运输能力。Docker 在软件交付上也展现这样的优越性，你不需要关心软件如何安装，开发者也不需要担心终端的运行环境，运维人员能快速的将产品顺准备上线。