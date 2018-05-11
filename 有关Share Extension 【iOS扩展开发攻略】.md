# 有关Share Extension 【iOS扩展开发攻略】
###最近接到的分享需求，要求做原生的分享，就去百度很多资料，坑坑洼洼的完成了任务上线，现在回过来总结一下踩过的坑.
----------------------

* 1、首先给大家介绍一下iOS扩展.

   扩展（ Extension ）是 iOS 8 中引入的一个非常重要的新特性。扩展让 app 之间的数据交互成为可能。用户可以在 app 中使用其他应用提供的功能，而无需离开当前的应用。在 iOS 8 系统之前，每一个 app 在物理上都是彼此独立的， app 之间不能互访彼此的私有数据。而在引入扩展之后，其他 app 可以与扩展进行数据交换。基于安全和性能的考虑，每一个扩展运行在一个单独的进程中，它拥有自己的 bundle ， bundle 后缀名是.appex 。扩展 bundle 必须包含在一个普通应用的 bundle 的内部。

   iOS 8 系统有 6 个支持扩展的系统区域，分别是 Today 、 Share 、 Action 、 Photo Editing 、 Storage Provider 、 Custom keyboard 。支持扩展的系统区域也被称为扩展点。
   接下来我们主要讲一下share Extension具体实现。
* 2、代码实现

  share Extension实现是需要依赖一个工程，如果你没有工程需要重新创建一个(如果你不会创建工程，你可以关闭浏览器了)；创建完工程后我们需要创建一个share Extension的工程，创建方法如图：
<!--(http://github.com/yourname/your-repository/raw/master/images-folder/xxx.png)
-->

![image](http://github.com/wang22290/share-Extension/raw/master/imageFiles/Snip20180510_9.png)

 
  ![](/Users/wagn/Desktop/Snip20180510_9.png)
  
  接着继续点击
  
  ![](/Users/wagn/Desktop/Snip20180510_10.png)
  
  ![](/Users/wagn/Desktop/Snip20180510_12.png)
  
  ![](/Users/wagn/Desktop/Snip20180510_13.png)
  完成后会如最后一张图片，现在我们就可以尝试一下share的功能。
  
  ![](/Users/wagn/Desktop/Snip20180510_15.png)
  share Extension项目运行必须有一个容器，我们先用默认的浏览器尝试一下，
  ![](/Users/wagn/Desktop/Snip20180510_16.png)
  进入手机页面你会发现分享按钮是灰色状态不能点击，我先需要打开一个网页，例如：（www.baidu.com），现在我们点击方向按钮，会在分享栏找到我们的APP
  
  ![](/Users/wagn/Desktop/Snip20180510_17.png)
    

   如果没有发现APP，点击旁边更多按钮，进入下级页面，把APP权限按钮打开
   
  ![img](/Users/wagn/Desktop/Snip20180510_18.png)
    

    
   

