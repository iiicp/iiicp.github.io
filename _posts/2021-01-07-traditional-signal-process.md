---
layout: post
title: 语音增强的传统算法
tags: audio process
excerpt: about audio process algorithm
---  

### 总结  

接触到的一些传统语音信号处理算法  

1. 线性卷积和循环卷积     
2. 分帧加窗和频谱泄漏     
3. 单通道降噪    	
	
	平稳噪声(噪声的能量不突变)

	噪声的估计   

	最小值跟踪法获得带噪语音的最小值，即噪声的初步估计。    
	根据噪声的初步估计，计算语音存在的概率。    
	根据语音存在的概率，计算噪声估计得平滑因子。  
	用递归平均法的到噪声估计。  

	增益因子的确定  

	谱减法 	(后验信噪比)
	维纳滤波法（纯净谱-带噪谱xG）-> 让误差最小  (先验信噪比)   
	mmse 

 
4. 单通道回声消除    
	时延估计   
	双讲检测    
	线性滤波
		频域分块自适应滤波（实时性要求）
	非线性滤波 
		看论文做算法吧...
5. vad
6. agc