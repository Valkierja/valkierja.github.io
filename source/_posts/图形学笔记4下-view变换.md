---
title: 图形学笔记4下 view变换
poc: true
categories:
  - - 图形学笔记
tags:
  - 图形学
date: 2023-04-09 19:47:15
---

承接自上半笔记

### view transformation

**viewing transformation**包括以下内容

<img src="https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304092156821.png" alt="image-20230409215625787" style="zoom:50%;" />

也就是相机的变换

考虑现实生活中的拍照片

首先需要让被拍摄对象摆好位置(进行model transformation) 然后让摄影师找个好位置(view transformation)然后拍照(投影 projection) 合起来简称 MVP变换



<img src="https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304091953234.png" alt="image-20230409195313168" style="zoom:50%;" />



### 相机定义

相机的定义如下

![image-20230409195532703](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304091955730.png)

位置, 视方向, 向上方向(因为相机可以绕自身中轴线旋转)



### 相对运动

当相机和其他东西一起以相同方式运动时, 得到的图片不会变化

<img src="https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304092000326.png" alt="image-20230409200004274" style="zoom:67%;" />

也就是说, 坐标系是一个可以相对的概念, 因此为了讨论方便, 在讨论视点变换时, 我们认为:

**相机永远处于原点, 相机永远向-z轴看, 相机永远以y为向上方向**

原因是: 相机所在的地方我们视为一个平面, 由x和y轴确立

相机看向的位置视作纵深位置也就是z轴

但是为了满足右手坐标系, 纵深位置视作-z轴

这样就可以让z方向的矩阵的数值为正数, -z轴上的 1 单位长度向量相当于z轴上的 -1 单位长度向量

<img src="https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304092004987.png" alt="image-20230409200405956" style="zoom:50%;" />

如何把相机设置为上述状态? 方法如下

![image-20230409200618485](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304092006514.png)

首先位置变换到原点, 然后旋转g和t到-Z和Y

这样就对齐了具体操作请看下一小节

### 相机归位操作

设最终归位所需矩阵为 $$M = RT$$ 也就说先做平移变换 然后旋转

其中旋转矩阵 $$ R = R_g \times R_t$$

首先是平移变换


$$
\left[
\begin{matrix}
1 & 0 & 0 & -x_e\\
0 & 1 & 0 & -y_e\\
0 & 0 & 1 & -z_e\\
0 & 0 & 0 & 1\\
\end{matrix}
\right]
$$



可以把相机放在原点



旋转矩阵并不好直接求, 因为我们前面学的公式都是从轴到目标地点的旋转, 但是我们现在需要的是任意位置旋转到轴

因此我们可以先求逆矩阵

<img src="https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304092100421.png" alt="image-20230409210031389" style="zoom:50%;" />





在此之前插入小插曲

#### 正交矩阵与旋转矩阵

正交矩阵判定方法:

每一列向量的长度等于单位长度, 且各个列向量之间正交(互相垂直), 由此易得, **旋转矩阵都是正交矩阵**

因为旋转之前的坐标系三个轴之间是正交的, 旋转之后也必然正交

另外正交矩阵有一个性质, **正交矩阵的逆矩阵等于正交矩阵的转置矩阵**

因此, 求顺时针旋转, 或者是求旋转的逆旋转操作, 都只需要求得原始旋转的逆矩阵, 也就是转置矩阵

#### 另一种旋转公式

为了确定一个物体的当前旋转状态, 我们需要三个量, 这样还可以构建一个该物体的本地坐标系

因此我们可以得出另一种旋转公式

该公式起始位置为世界坐标, 结束位置为目标位置

在此期间, 该物体最初的本地坐标与世界坐标重合, 最终状态下, 本地坐标不再与世界坐标系重合

<img src="C:/Users/valkie/AppData/Roaming/Typora/typora-user-images/image-20230409212953343.png" alt="image-20230409212953343" style="zoom:50%;" />

我们只需要写出目标位置的本地坐标系的三个轴

前面的笔记说过, 为了新建一个坐标系我们可以用叉乘

所以得出上图的坐标系

利用上面的公式可以不需要任何**角度**量的介入而完成旋转



回到最初的相机归位操作

我们只需要找到当前相机的本地坐标系, 就可以轻松找到让他旋转回到世界坐标系的旋转矩阵(求逆, 也就是求转置即可)

**这样旋转矩阵R和平移矩阵T都有了, 乘起来就可以得到单一的变换矩阵M 方便后续计算**

<img src="https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304092140226.png" alt="image-20230409214029206" style="zoom:67%;" />



得到矩阵M 之后, 世界中所有物件都需要做变换M, 这样就可以进行相对运动, 类似于相对静止

也就是说优先保证相机移到世界坐标系, 然后让其他物件绑定到相机上面



对所有世界物体做出M变换后就可以开始准备下一阶段了



## 投影变换

分为两种投影变换 正交投影(orthographic)和透视投影(perspective)

![image-20230410082203555](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304100822603.png)

投影变换后还需要进行视口变换

在投影变换过程中, 物体不会等比例拉伸, 会变形, 这点很正常,视口变换会将其还原



### 正交投影和透视投影分别是什么含义

正交投影意味着光线是平行光, 也就是说摄像头位于无限远处

透视投影就是日常情况, 近大远小, 平行线相交于消失点

![image-20230411192722289](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304111927370.png)

由于正交投影摄像机处于无限远, 对于摄像机而言, 距离他x单位的物体和x+t单位的物体看起来就一样大了(前提是他们的体积确实是一样的), 这样就不存在近大远小了,平行线也就不会相较于消失点



![image-20230411192750747](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304111927799.png)



### 正交投影流程

首先完成了摄像机的相机归位操作

![image-20230411193135497](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304111931542.png)

此时我们可以认为有一个幕布, 放在了原点处, 幕布大小是$[-1,1]^3$

也就是说, 把相机这个对象, 抽象成了一块显影底片

这样的话z轴就消失了, 空间中只需要关心xy轴(也就是说有图层和遮罩了, 这也符合我们对相机拍照的认知



![image-20230411193435278](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304111934326.png)

这里有一个小疑问点

**为什么摄像机看向-z方向, 但是z方向的物体还可以成像**

暂时按下不表, 先看正交投影的具体操作



首先把物体的"中心"平移到xy平面内,然后缩放

更具体的来说:

设长方体(这个长方体是视锥, 具体解释在下面的小结)为 $ [left,right] \times [bottom,top] \times [far, near]$, 将这个长方体缩放为 $[-1,1]^3$  的标准立方体(这里和投影不完全一样, 先这样做)

![image-20230411193750391](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304111937432.png)

需要注意的是

由于是右手系, 看向-z方向, 所以l<r,  b<t **注意f<n**(正确)

**由于看向-z, 远点的z值会更小**

**但是如果是一根正坐标轴上, 离我们远的值应该是更大** 

所以有的api采用了左手系来规避这个不符合直觉的地方

变换矩阵如下, 先做负的平移操作,然后缩放变成$[-1,1]^3$ 总大小是2×2

注意, 这里的压缩不会导致数据丢失(除非超出浮点数表示范围), 因为实数是稠密的, 像素点才是离散的

![image-20230411200707911](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304112007952.png)

### 上一小节的长方形是什么

这里课程主讲老师没太讲清楚,这个长方体不是空间中的物体本身,而是视锥

也就是说, 这个长方体规定了拍照的区域, 这个区域包括了从背景板到前景的范围, 如果我们把前景(本质上是前景到背景之间的 物体的映射像素)位移, 他对应的实际物体像素就会跟着位移,看下图, 长方体中间的每一根线对应着一个需要被成像的像素, 以及这个像素在前景上被映射的位置

![image-20230411204033734](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304112040758.png)

### 透视投影

![image-20230411205649146](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304112056188.png)

透视投影的视锥是一个四棱台, 底面**压缩**为相同长方形后就可以按照正交投影来了

这里需要定义一下这个压缩过程:

1.  $near$ 面永远不变, 因为这个面意味着胶片位置(正交投影过程时还是会变的, 但是压缩过程不会变)
2.  $far$ 面的z值仍然保持不变, 也就是说整个四棱台的纵深不变
3. 上述两个面的中心点 压缩后仍然为中心点

根据上述要求 求解变换矩阵 $M_{persp->ortho}$ 该矩阵将透视投影转换为正交投影



我们先来看yz视图

![image-20230411210221139](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304112102184.png)

可以看到一个相似三角形

所以挤压之后的y有
$$
y' = \frac{n}{z}y
$$


再求挤压之后的x, 同理根据相似三角形

![image-20230411211732446](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304112117480.png)

所以有:


$$
\begin{equation}
\left\{
\begin{array}{lr}
x' = \frac{n}{z}x &  \\
y' = \frac{n}{z}y &  
\end{array}
\right.
\end{equation}
$$


根据上述内容可以得到 $M_{persp->ortho}$  矩阵的部分元素(把除以z全部约掉, 让w值从1变为z)



![image-20230412223629294](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304122236366.png)

![image-20230412223823784](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304122238818.png)

如下所示
$$
\left[
\begin{matrix}
n & 0 & 0 & 0\\
0 & n & 0 & 0\\
? & ? & ? & ?\\
0 & 0 & 1 & 0\\
\end{matrix}
\right]
$$


然后我们研究上面的问号都要填写哪些元素

首先观测 $near$ 平面, 发现其变换后, 平面内每个点的z值(也就是纵深距离) 都是不变的

同理 $far$ 平面的纵深值也是不变的, 写出下面两个方程组

![近平面](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304132205727.png)

![原平面中心点0 0 f变换后仍然是0 0 f](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304132207034.png)



这样就可以首先观察出,从左往右第一个和第二个问号是0

并且需要求解第三个问号和第四个问号, 分别设他们为A和B

得到如下方程

![image-20230413220849874](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304132208911.png)

解得:

![image-20230413220907984](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304132209010.png)

综上, 透视变换转为正交变换的矩阵$M_{persp->ortho}$为

公式
				
					
				
				
						
				
			
$$
\left[
\begin{matrix}
n & 0 & 0 & 0\\
0 & n & 0 & 0\\
0 & 0 & n+f & -nf\\
0 & 0 & 1 & 0\\
\end{matrix}
\right]
$$

非常的简洁！

### 其他的透视变换方法

上述方法先转换成正交投影然后再做其他操作

其实也有一步到位的公式

首先定义视角(笔记5内容)和视图比率

![image-20230414190228185](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304141902223.png)

可以写出一个一步到位的变换矩阵(直接从视锥变换到标准正方体)
$$
M_{\text {frustum }}=\left[\begin{array}{cccc}
\frac{\cot \frac{f o v}{2}}{\text { Aspect }} & 0 & 0 & 0 \\
0 & \cot \frac{f o v}{2} & 0 & 0 \\
0 & 0 & -\frac{f+n}{f-n} & -\frac{2 n f}{f-n} \\
0 & 0 & -1 & 0
\end{array}\right]
$$
化简一下就是
$$
M_{\text {frustum }}=
\left[
\begin{array}{cccc}
\frac{|n|}{r} & 0 & 0 & 0 \\
0 & \frac{|n|}{t} & 0 & 0 \\
0 & 0 & -\frac{f+n}{f-n} & -\frac{2 n f}{f-n} \\
0 & 0 & -1 & 0
\end{array}
\right]
$$


还有一种形式(形式二)
$$
\left[\begin{array}{cccc}
\frac{2 n}{r-l} & 0 & \frac{r+l}{r-l} & 0 \\
0 & \frac{2 n}{t-b} & \frac{t+b}{t-b} & 0 \\
0 & 0 & -\frac{f+n}{f-n} & -\frac{2 n f}{f-n} \\
0 & 0 & -1 & 0
\end{array}\right]
$$
更常用的是形式一








### 课后习题

请问 $near$ 和 $far$ 平面之间的点的z值如何变化?

由于老师没有给出标准答案, 我这边也有点困惑

过程如下

设点A坐标 $\left( 0,0,n+f/2,1 \right)$ 设 $n=5 , f=3$ (因为f<n)

则A点坐标矩阵为
$$
\left[
\begin{matrix}
0 \\ 
0 \\
4 \\
1 
\end{matrix}
\right]
$$


则变换矩阵为
$$
\left[
\begin{matrix}
5 & 0 & 0 & 0\\
0 & 5 & 0 & 0\\
0 & 0 & 8 & -15\\
0 & 0 & 1 & 0\\
\end{matrix}
\right]
$$
相乘得到
$$
\left[
\begin{matrix}
0 \\ 
0 \\
4.25 \\
1 
\end{matrix}
\right]
$$
**可以发现变大了**



再取A点靠近n平面的点B
$$
\left[
\begin{matrix}
0 \\ 
0 \\
3.5 \\
1 
\end{matrix}
\right]
$$
相乘得到
$$
\left[
\begin{matrix}
0 \\ 
0 \\
3.71 \\
1 
\end{matrix}
\right]
$$
**也是变大的**



再取点C靠近f平面
$$
\left[
\begin{matrix}
0 \\ 
0 \\
4.5 \\
1 
\end{matrix}
\right]
$$
相乘得到
$$
\left[
\begin{matrix}
0 \\ 
0 \\
4.67 \\
1 
\end{matrix}
\right]
$$
**还是变大的**

**但是**

如果n和f是**负数**分别是 $n=-3 , f=-5$ (f<n)

得到变换矩阵为
$$
\left[
\begin{matrix}
-3 & 0 & 0 & 0\\
0 & -3 & 0 & 0\\
0 & 0 & -8 & -15\\
0 & 0 & 1 & 0\\
\end{matrix}
\right]
$$
点A为


$$
\left[
\begin{matrix}
0 \\ 
0 \\
-4 \\
1 
\end{matrix}
\right]
$$
得到
$$
\left[
\begin{matrix}
0 \\ 
0 \\
-4.25 \\
1 
\end{matrix}
\right]
$$
可以发现是变小的,其他几个点结论相同

总的来说是**远离原点方向 推向f平面, 但是变大变小不确定** 得看立方体在哪里, 总的来说绝对值变大

**数据集有限, 而且想不出来严格证明, 可能会有出错的地方**



图示如下

![image-20230413225631806](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304132256856.png)

![image-20230413225843371](https://raw.githubusercontent.com/Valkierja/ALLPIC/main/img/202304132258413.png)

我这里把线段 $I_nI_f$ 想象成一根绳子, 绳子转动之后可以形成一个三角形, 由于三角形斜边大于直角边, 所以长度变长, 坐标的绝对值变大

图二中点 $J$ 变换到点 $I$ 的位置了

这个也可以间接说明实数的稠密性和不可数性, 因为总长度变短了, 但是每个点的z坐标绝对值反而变长了

