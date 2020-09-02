---
title: springBoot整合中的一个bug
date: 2019-12-09 16:21:08
comments: "true"
reward: "true"
categories: "坑~"
---
一直想弄一个属于自己的springBoot的项目，然后开始整起来,创建springBoot成功,然后加swagger2和mybatis—plus,这个时候遇到了一个错误：

### java.lang.ClassNotFoundException: org.mybatis.logging.LoggerFactory
<a><img src="http://q1pyga56k.bkt.clouddn.com/springBoot_mybatis-plus1.png" class="animated zoomIn"> </a>



	百度解决错误,网上答案五花八门,不知道具体哪个是对的,只能一个个实验
	我实验了几个方法,错误得已解决，记录一下,也为后来的提供一个参考方向。
	
	这个错误在我这里发生的原因是因为pom.xml的文件中jar的冲突
	我要用mybaits-plus 但是我的pom文件中 有mybatis的jar包 也有mybaits-plus的jar包 
	这两个只能存其一，把mybatis的jar包删掉就行了
