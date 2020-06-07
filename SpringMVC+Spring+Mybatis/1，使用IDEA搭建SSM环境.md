# 使用IDEA搭建SSM记录
## 创建Maven项目
> 点击File -> New -> Project -> Maven -> Create from archetype -> archetype-webapp  
> 然后命名，配置Maven等，项目就创建完成了  
> 在main文件夹下创建java文件夹，并标注为Source Root  
> 在java文件夹下创建package com.***.controller/entity/mapper/service  
> 跟java文件夹并列创建resources文件夹，并标注为 Resources Root  
> 在resources文件夹中创建mapper文件夹和spring文件夹  
> 跟java文件夹并列创建Test文件夹，并标注为Test Root   
> 现在我们的文件夹目录已经创建完毕了  
## 配置依赖
以下部分目前不熟悉，只能照着别人抄，就先不写了，先把能搭成功的博客记录一下，等研究明白了再写自己的版本。  
https://www.cnblogs.com/shaoyu/p/11701189.html  
> 这部分目前的疑问：  
> 1, 如何找到pom中都需要哪些核心包？
> 2, 如何确定核心包中的兼容性？ 是否有官方文档支持？
> 3, 配置文件的各个部分分管什么作用？ 原理？ 是如何共同工作的？ 