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



