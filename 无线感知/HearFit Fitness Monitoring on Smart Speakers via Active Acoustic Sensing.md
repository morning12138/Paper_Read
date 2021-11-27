# HearFit: Fitness Monitoring on Smart Speakers via Active Acoustic Sensing



## 主要实现的功能：

1. HearFit needs to distinguish fitness from daily activities automatically.(区分日常行动和健身)
2.  HearFit to be anti interference(由于环境的反射信号，需要抗干扰).
3. , HearFit needs to classify and evaluate the fitness actions accurately, and allows users to add new actions.(识别和区分不同的健身动作，且允许增加新的健身动作)



## 具体实现手段：

### 对于任务1：

` We use the autocorrelation of acoustic signal to determine the activity mode of the user.`

通过声学信号的**自相关**来确定用户的活动模式，因为健身动作具有重复性的特点。



### 对于任务2：

`In order to minimize the interference of surrounding people, we use the microphone array to locate the user. `

`So we calculate the accurate direction of the user by Generalized Cross-Correlation with phase transform (GCC-PHAT) [18] and use beamforming to amplify the reflflected signal of the user.`

使用麦克风阵列来减小干扰，使用GCC-PATH方法技术用户的准确位置使用beamforming(波束赋形)来提高反射信号的强度。

* 波束赋形：波束赋形是一种基于天线阵列的信号预处理技术，波束赋形通过调整天线阵列中每个阵元的加权系数产生具有指向性的波束，从而能够获得明显的阵列增益。



### 对于任务3：

` design a Long Short-Term Memory (LSTM) network to identify the type of each action.`

`Combined with incremental learning [20], HearFit can update the network with a small training set of new actions.`

使用LSTM神经网络来区分不同的动作。

结合incremental learning(增量学习)来实现添加新的动作。



## 系统设计

![image-20211127230513387](C:\Users\86150\AppData\Roaming\Typora\typora-user-images\image-20211127230513387.png)

###  Signal Preparation

通过Doppler Shift来检测移动

使用19kHz、19.5kHz、20kHz、20.5kHz和21kHz的信号

原因：

* 人听不见这个高频率的声音
* 大多数扬声器只支持到22kHz
*  which can cause a Doppler shift of 185*.*2*Hz* under a sound of 21*kHz*.Since 185*.*2*Hz* is smaller than the half of intervals between frequency components, their Doppler shifts do not overlap in the frequency domain.
* we find that more than 5 frequency components would make the signal audible even at a low volume due to sub-harmonics



### Motion detection

通过移动导致的Doppler Shift具有的特点来判断

* 经过FFT之后的图像具有每个频率分量中心周围都有几个振幅较大的峰值。

![image-20211127232300834](C:\Users\86150\AppData\Roaming\Typora\typora-user-images\image-20211127232300834.png)

### Repeatability Analysis

`We add a window with length of 13s that slides 1s each time on the reflected signal.`

`HearFit further determines whether the motions are fifitness activities by calculating autocorrelation [35] of the fifiltered signal`

通过前一秒的信号和后一秒的信号进行**自相关**的对比来判断是否是重复的动作。

### Noise Reduction

通过GCC-PATH算法通过Time Difference of Arrival来计算用户的方位角和仰角。

利用Beamforming的方法来对抗噪声。

### Action Segmentation

通过计算STE来判断动作的开始与结束。

![image-20211127235153168](C:\Users\86150\AppData\Roaming\Typora\typora-user-images\image-20211127235153168.png)

通过**斜率**判断。

* 选择STE小于0.03的时刻
* 判断前一个时刻和后一个时刻计算出来的斜率
* 与预设的值比较 得出是否是开始点或者结束点

*s*(*δ*) is the amplitude of the signal at time *δ*

*w* is the hanning window and *l* is the length of the window(定值)

![image-20211127235233799](C:\Users\86150\AppData\Roaming\Typora\typora-user-images\image-20211127235233799.png)

### Fitness Classification

将FFT得到的数据，先切分成不同的频率的块，然后选择19kHz的到21kHz的作为输入。输出结果是八个特征