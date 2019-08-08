---
title: "My Article Name"
date: 2019-08-08T10:44:03+09:00
markup: mmark
---

### Unity的View矩阵

- Unity的矩阵存储：Matrices in unity are column major ，Data is accessed as: row + (column*4) 

  即$$\begin{pmatrix}1&5&9&13\\ 2&6&10&14\\3&7&11&15\\4&8&12&16\end{pmatrix}$$

- 向量也是column major

  $\begin{bmatrix}X\\Y\\Z\\W\end{bmatrix}$

- Unity的世界坐标系是左手坐标系

- Unity的坐标变换是左乘，即矩阵在点的左边$M \cdot \vec p $，点是列向量。

- D3D的坐标变换是右乘，点是行向量。

- LookAt矩阵的推导

  - LookAt矩阵把点变换到Camera Space，由于CameraSpace的正方向符号，左右手规则都不同，导致了这个LookAt矩阵对Unity和D3D也有不一样

  - 先介绍Unity的LookAt矩阵，

    规则是：

    1. 左手坐标系

    2. 最终Camera正方向是Z的负轴

    3. 左乘，点是列向量

    4. 组织方式如下，有的教程是xaxis，yaxis，zaxis，分别对应这里的right，up，forward，这里没有使用xyz的形式是为了区别开xyz世界坐标轴

       forward=normal(at-eye)

       right=normal(cross(up,forward))

       up=cross(forward, right)

       position=(-dot(right,eye), -dot(up,eye), -dot(forward,eye))

    $\begin{pmatrix}right_x&right_y&right_z&position_x\\up_y&up_y&up_z&position_y\\forward_x&forward_y&forward_z&position_z\\0&0&0&1\end{pmatrix}\begin{bmatrix}X\\Y\\Z\\W\end{bmatrix}$

    ​	使用这个矩阵和计算方法，得到的点是以Camera的中心为原点，camera的forward方向为正方向的点，由于Unity需要同Opengl一样把正方向变成z的负轴，所以这个点最后要把z的坐标变成负。这里可以理解为Unity都是在左手坐标系下进行的计算，但是最后要和OpenGL统一，则需要把z变成负，即把z轴反转变成右手坐标系

  - 接下来是D3D的LookAt矩阵，由于D3D也是左手坐标系，所以right，up，forward和Unity的计算方法是一样的，但是D3D的矩阵是row-major，同时是右乘，点是行向量，矩阵的排列方式有所不同，组织方式如下

    $$\begin{bmatrix}X&Y&Z&W\end{bmatrix}\begin{pmatrix}right_x&up_x&forward_x&0\\right_y&up_y&forward_y&0\\right_z&up_z&forward_z&0\\position_x&position_y&position_z&1\end{pmatrix}$$

  - 同时也存在右手坐标系的LookAt矩阵，也是分为column-major和row-major的两种组织方式，矩阵的元素排列规则和上面的一样，但是forward，right，position的计算方法有所不同，注意这里forward是指向z的负方向，这是OpenGL的要求，而不是右手坐标系的要求，若Camera指向正面，得到的也是指向z的正方向的坐标值

    forward=normal(eye-at)

    right=normal(cross(up,forward))

    up=cross(forward, right)

    position=(-dot(right,eye), -dot(up,eye), -dot(forward,eye))

  - D3D在camera space，camera的forward是z轴正方向，而OpenGL的forward是z轴负方向。

- Camera的方向以及Camera（View）Space的稍有不同

  - Camera的transform.forward是其Camera的朝向，即Camera的正方向，Unity's convention, where forward is the positive Z 
  - 但是点在View Space,Camera的正反向是Z的负轴， camera's forward is the negative Z axis 

- 

### Unity的投影矩阵

- 这里使用大写字母表示空间，下标表示x，y，z分量，

  - View Space，Eye Space和Camera Space都是同一个space，统一写作$，，V_x，V_y，V_z$
  - Clip Space是指View Space的坐标乘以Projection Matrix之后，w没有normalized时候的空间，表示为$，，C_x，C_y，C_z$
  - NDC Space是指除以w之后的空间，表示为$，，NDC_x，NDC_y，NDC_z$。

- 而Frustum的几个变量表示为$l,r,t,b,n,f$，分别对应左右上下近远六个平面的大小，注意这几个变量都是大小，且为正。这个Frustum一般是没有坐标的，但是可以和坐标原点重合并指定朝向，就可以根据变量的值推出坐标的值。这个坐标也是和OpenGL或者D3D的规定有关系

  - Unity/OpenGL的Frustum是在朝向z轴的负方向，即左下是$(-l,-b,-n)$右上是$(r,t,-f)$，Near Plane的坐标是$N=-n$, Far Plane的坐标是$F=-f$
  - D3D的Frustum是在朝向z轴的正方向，即左下是$(-l,-b,n)$右上是$(r,t,f)$，Near Plane的坐标是$N=n$, Far Plane的坐标是$F=f$

- 投影矩阵的意义，是把view space的坐标，转化为normalized的ndc space坐标。ndc space的值的范围，根据OpenGL和D3D有所不同，

  - Unity/OpenGL的View Space的z是负值，NDC Space范围是$NDC_x\in\{-1,1\}, NDC_y\in\{-1,1\},NDC_z\in\{-1,1\}$
  - D3D的View Space的z是正值，NDC Space范围是$NDC_x\in\{-1,1\}, NDC_y\in\{-1,1\},NDC_z\in\{0,1\}$
  - View Space的Camera朝向的正负，已经在View Matrix指定了，这里并不需要关心，但是由于涉及到Frustum的坐标，Frustum定义的时候只有大小，需要把大小转换成OpenGL和D3D的View Space坐标，之后再进行计算。
  - 同时需要注意NDC Space的值的范围与View Space的值的范围的对应关系
    - 在OpenGL的投影下near plane的值，会变成NDC Space的-1，而far plane的值，会变成NDC Space的1；
    - 而在D3D的投影下，near plane的值，会变成NDC Space的0，而far plane的值，会变成NDC Space的1；

- 首先是View Space到NDC Space的坐标范围

  - Unity OpenGL和D3D在XY方向都是把值映射到
    - $V_x\in \{-l,r\}\Rightarrow NDC_x\in\{-1,1\}$
    - $V_y\in \{-b,t\}\Rightarrow NDC_y\in\{-1,1\}$
  - 在Z方向
    - Unity的投影矩阵和OpenGL是同一个Convention，并且Camera的朝向是负z轴，即view position的z的坐标从 $V_z\in\{ -n,-f\} \Rightarrow NDC_z\in\{-1,1\}$，
    - D3D则是 $V_z\in\{n,f\} \Rightarrow NDC_z\in\{0 ,1\}$，这里假设D3D的View Space的值是正的

- 透视投影的推导如下

  - 先推导x，y方向

    - 由于view space的点，先把x，y方向映射到clip space，即把这个点按照离camera的距离（z的值）进行缩放，所以

      $\frac{V_z}{N}=\frac{V_x}{C_x}\\ \frac{V_z}{N}=\frac{V_y}{C_y}$

      这里的N是near plane的坐标，F是Far Plane的坐标

      这个式子的意思是view space的点的x值和clip space的点的x值的比等于这个点view space的z值和near plane的z值的比。即表示对x，y根据远近进行缩放

      注意这个N根据OpenGL和D3D不同，值得大小和正负也不同

    - 上面式子移项，推出$C_x$和$C_y$的表达式

      $C_x=\frac{N}{V_z}\times V_x$

      $C_y=\frac{N}{V_z}\times V_y$

    - 由于$NDC_x$和$NDC_y$是把clip space的值再映射到{-1,1}，假设$NDC$和$C$有如下关系

      $NDC=A\times C+B$，由左下角的值和右上角的值得到X的方程组和Y的方程组

      $\begin{cases}-1=A\times-l+B\\1=A\times r+B\end{cases}$

      $\begin{cases}-1=A\times-b+B\\1=A\times t+B\end{cases}$

      分别得到解，注意这里的解和[这个教程](http://www.songho.ca/opengl/gl_projectionmatrix.html )的解符号不一样，这里的$l,r,t,b$和上面定义的一样，是大小，而教程里面的包含了方向，最后的结果是一样的，但是要注意区分。

      $A=\frac{2}{l+r}, B=\frac{l-r}{l+r}$

      $A=\frac{2}{b+t}, B=\frac{b-t}{b+t}$

      所以把A，B带回式子，得到Clip Space到NDC Space的推导

      $NDC_x=\frac{2}{l+r}\times C_x+\frac{l-r}{l+r}$

      $NDC_y=\frac{2}{b+t}\times C_y+\frac{b-t}{b+t}$

    - 再把上面推导的由$V_x,V_y$到$C_x, C_y$的公式代到替换$C_x, C_y$这里得到

      $NDC_x = \frac{2}{l+r}\times \frac{N}{Vz}\times V_x+\frac{l-r}{l+r}$

      $NDC_y=\frac{2}{b+t}\times \frac{N}{V_z}\times V_y +\frac{b-t}{b+t}$

      整理一下把$Vz$的项移出来，得到

      $NDC_x = \frac{1}{Vz}\times (\frac{2N}{l+r}\times V_x+\frac{l-r}{l+r}\times V_z)\qquad(1)$

      $NDC_y=\frac{1}{V_z}\times (\frac{2N}{b+t}\times V_y +\frac{b-t}{b+t}\times V_z)\qquad(2)$

  - 再推导Z的方向

    - 假设: $NDC_z = \frac{1}{V_z}(AV_z +B)$

      由Z的映射关系 $V_z\in\{ -n,-f\} \Rightarrow NDC_z\in\{-1,1\}$写出方程组，

      即在$Vz$为-n时$NDC_z=-1$，在$Vz$为-f时$NDC_z=1$

      注意这里的大小和符号在Unity/OpenGL和D3D有不同，这里以Unity/OpenGL为例

      $\begin{cases}-1=\frac{1}{-n}(A\times -n+B)\\1=\frac{1}{-f}(A\times -f+B)\end{cases}$

      得到解:

      $\begin{cases}A=-\frac{n+f}{n-f}\\B=\frac{-2fn}{n-f}\end{cases}$

      And

      $NDC_z=\frac{1}{V_z}(-\frac{n+f}{n-f}V_z + \frac{-2fn}{n-f})\qquad(3)$

- 至此NDC的xyz坐标都可由V的xyz表示成$NDC = \frac{1}{V_z}(AV +B)$的形式了，之所以要表示成这样的形式，是因为把$V_z$保存到了Clip Space的w分量，这样变化之后就可以把xyz除以w分量来得到最终的结果

- Unity/OpenGL表示成矩阵的形式就是
  $$
  P_{clip} = \begin{pmatrix}
  \frac{2N}{l+r}&0&\frac{l-r}{l+r}&0\\0
  &\frac{2N}{b+t}&\frac{b-t}{b+t}&0\\
  0&0&-\frac{n+f}{n-f}&\frac{2fn}{n-f}\\
  0&0&1&0
  \end{pmatrix}\times
  \begin{bmatrix}V_x\\V_y\\V_z\\1
  \end{bmatrix}
  $$
  注意如果Frustum的中心位于Camera朝向的中心，则Frustum的大小$l=r,t=b$，这个矩阵可以化简为

$$
  \begin{aligned}
  P_{clip} &= \begin{pmatrix}
  \frac{N}{r}&0&0&0\\0
  &\frac{N}{t}&0&0\\
  0&0&-\frac{n+f}{n-f}&\frac{2fn}{n-f}\\
  0&0&1&0
  \end{pmatrix}\times
  \begin{bmatrix}V_x\\V_y\\V_z\\1
  \end{bmatrix}\\&=
  (\frac{N}{r}V_x,\frac{N}{t}V_y,-\frac{n+f}{n-f}V_z + \frac{2fn}{n-f},V_z)
  \end{aligned} in\ ClipSpace
$$

  注意这里的矩阵对于Unity/OpenGL：$N=-n$，对于D3D：$N=n$

  [教程](http://www.songho.ca/opengl/gl_projectionmatrix.html)的推导，是把m33的1变成了-1，理由是View space的点的z值是负的，这样的话根据上面推导的NDC的式子(1)(2)(3)，代入$N=-n$，需要把负号放到$\frac 1 V$之前可以得到(再次注意这里的rltb是大小的值)

  $NDC_x = \frac{1}{-Vz}\times (\frac{2n}{l+r}\times V_x+\frac{r-l}{r+l}\times V_z)$

  $NDC_y=\frac{1}{-V_z}\times (\frac{2n}{b+t}\times V_y +\frac{t-b}{t+b}\times V_z)$

  $NDC_z=\frac{1}{-V_z}(-\frac{f+n}{f-n}V_z + \frac{-2fn}{f-n})$

  只是符号的变化，结果是一样的，

这里也可以理解成使用了这个z-reverse矩阵来使z值求反，因为ndc space的正方向和view space的相反

$\mathbf{v}^{\prime}=\left(\begin{array}{cccc}{1} & {0} & {0} & {0} \\ {0} & {1} & {0} & {0} \\ {0} & {0} & {-1} & {1} \\ {0} & {0} & {0} & {1}\end{array}\right)$

所以上面的矩阵也就可以化为同[教程](http://www.songho.ca/opengl/gl_projectionmatrix.html)一样的形式，这个就是在Unity/OpenGL的表达形式即
$$
\begin{aligned}
  P_{clip} &= \begin{pmatrix}
  \frac{n}{r}&0&0&0\\0
  &\frac{n}{t}&0&0\\
  0&0&-\frac{f+n}{f-n}&\frac{-2fn}{f-n}\\
  0&0&-1&0
  \end{pmatrix}\times
  \begin{bmatrix}V_x\\V_y\\V_z\\1
  \end{bmatrix}\\&=
  (\frac{n}{r}V_x,\frac{n}{t}V_y,-\frac{f+n}{f-n}V_z + \frac{-2fn}{f-n},-V_z)
  \end{aligned} in\ ClipSpace
$$

- 同时可以得出在D3D的表达形式，这里的xy方向是和式子(1)(2)是一样的，最后代入$N=n$，得到

  $NDC_x = \frac{1}{Vz}\times (\frac{2n}{l+r}\times V_x+\frac{l-r}{l+r}\times V_z)$

  $NDC_y=\frac{1}{V_z}\times (\frac{2n}{b+t}\times V_y +\frac{b-t}{b+t}\times V_z)$

  但是D3D在z的时候，映射关系是$V_z\in\{ n,f\} \Rightarrow NDC_z\in\{0,1\}$，方程组是

  $\begin{cases}0=\frac{1}{n}(A\times n+B)\\1=\frac{1}{f}(A\times f+B)\end{cases}$

  得到解:

  $\begin{cases}A=\frac{f}{f-n}\\B=-\frac{fn}{f-n}\end{cases}$

  And

  $NDC_z=\frac{1}{V_z}(\frac{f}{f-n}V_z - \frac{fn}{f-n})$

  加之条件Frustum的大小$l=r,t=b$，表示成矩阵就是，注意D3D是右乘，行向量
  $$
  \begin{aligned}
  P_{clip} &=
  \begin{bmatrix}V_x, V_y,V_z, 1
  \end{bmatrix}\times
  \begin{pmatrix}
  \frac{2n}{l+r}&0&0&0\\0
  &\frac{2n}{b+t}&0&0\\
  0&0&\frac{f}{f-n}&1\\
  0&0&\frac{-fn}{f-n}&0
  \end{pmatrix}
  \\&=
  (\frac{2n}{l+r}V_x,\frac{2n}{b+t}V_y,\frac{f}{f-n}V_z + \frac{-fn}{f-n},V_z)
  \end{aligned} in\ ClipSpace
  $$
  这个结果和之前的文章是一样的结果，同时[整篇文章](https://docs.microsoft.com/en-us/windows/win32/direct3d9/projection-transform)的Q，w，h也能和这里的矩阵对应位置的值所对应。

  注意这里的矩阵是和D3D文档描述的矩阵相同，Unity当中对平台相关的矩阵也有一些变化。

- 不管是那个表达方法，在xy的分量也可以用field of view（fov）来表示，所以矩阵中的XY分量部分$\frac n r$和$\frac n t$可以表示为cot的形式，这个角度就是fov，即$cot(\frac{fov_x}{2})=\frac n r$和$cot(\frac{fov_y}{2})=\frac n t$

- 在Unity中使用的是vertical fov，垂直部分的值由aspect=width/height来决定，关系是

  $\frac n t = cot(\frac{fov_y}{2})$

  $\frac n r = cot(\frac{fov_y}{2})/aspect$

- 平行投影的推导会简单一些

  - 平行投影式通过平行的Frustum把所有的x，y，z值都线性的映射到NDC space

  - 即xyz都有这样的关系$NDC=A\times C+B$

  - 和投影推导一样，对于xy，取左下和右上的两个值

    $\begin{cases}-1=A\times-l+B\\1=A\times r+B\end{cases}$

    $\begin{cases}-1=A\times-b+B\\1=A\times t+B\end{cases}$

    得到解为

    $A=\frac{2}{l+r}, B=\frac{l-r}{l+r}$

    $A=\frac{2}{b+t}, B=\frac{b-t}{b+t}$

    所以把A，B带回式子，得到Clip Space到NDC Space的推导

    $NDC_x=\frac{2}{l+r}\times C_x+\frac{l-r}{l+r}$

    $NDC_y=\frac{2}{b+t}\times C_y+\frac{b-t}{b+t}$

  - 对于z值，也是一样，带入near，far的值

    - 在OpenGL里，z的范围是[-1,1]，注意这里的near，far是z的负方向，n和f只是大小，所以带上负号

      $\begin{cases}-1=A\times-n+B\\1=A\times -f+B\end{cases}$

      解是

      $A=\frac{2}{n-f}, B=\frac{n+f}{n-f}$

      得到z的推导 $NDC_z=\frac{2}{n-f}V_z + \frac{n+f}{n-f}$

  - 由以上的xyz的推导，由于不需要把xy的值除z，得到一下的矩阵
    $$
    P_{clip} = \begin{pmatrix}
    \frac{2}{l+r}&0&0&\frac{l-r}{l+r}\\0
    &\frac{2}{b+t}&0&\frac{b-t}{b+t}\\
    0&0&\frac{2}{n-f}&\frac{n+f}{n-f}\\
    0&0&1&1
    \end{pmatrix}\times
    \begin{bmatrix}V_x\\V_y\\V_z\\1
    \end{bmatrix}
    $$

  - 大多数情况下l和r，b和t都是相等的，可以化简为
    $$
    P_{clip} = \begin{pmatrix}
    \frac{2}{r}&0&0&0\\0
    &\frac{2}{t}&0&0\\
    0&0&\frac{2}{n-f}&\frac{n+f}{n-f}\\
    0&0&1&1
    \end{pmatrix}\times
    \begin{bmatrix}V_x\\V_y\\V_z\\1
    \end{bmatrix}
    $$

### Depth相关的问题

- 由于投影的关系，在Z的处理上，Unity/OpenGL的$NDC_z=\frac{1}{-V_z}(-\frac{f+n}{f-n}V_z + \frac{-2fn}{f-n})(1)$或者D3D的$NDC_z=\frac{1}{V_z}(\frac{f}{f-n}V_z - \frac{fn}{f-n})(2)$式子中，z的变化不是线性的，这样会造成z-fighting，即很多的值集中在far plane的附近，例如，D3D的情况下在View Space，near=1，p=50，far=100,则在NDC Space，near=0，p=0.989898，far=1，可以看出很多点都集中到了靠近1的部分；OpenGL/Unity也是，集中到far plane的值。这样会造成精度的损失。

- D3D的Depth Buffer值的范围是$[0,1]$，是non-linear的；

  OpenGL的Depth Buffer值的范围，虽然在变换到NDC Space的时候除以w，范围是[-1,1]，但是之后会被[映射](https://www.khronos.org/registry/OpenGL-Refpages/gl2.1/xhtml/glDepthRange.xml )到[0,1]，也是non-linear的；

  Unity的Depth Buffer也是[0,1]，是non-linear的，[文档](https://docs.unity3d.com/Manual/SL-DepthTextures.html )

- Unity的Clip Space，和上面定义的一样，是没有除过w的值，在vertex shader，通过UnityObjectToClipPos变换的点，是w=Vz的homogeneous coordinates。之后硬件会做除w的操作，最后在depth buffer得到[0,1]范围的值，这个值就是除以w之后的值，同时也是没有linearize的值。

- Linearize的步骤，就是把Depth还原为view space的z值，所以做的是除w的逆运算，这里为了表示统一，把线性的depth写成$V_z$，参考文章(1)[这里](http://www.humus.name/temp/Linearize%20depth.txt )

  - Unity/OpenGL需要把改写(2)式子，得到$V_z=\frac{-2fn}{(f+n)-NDC_z(f-n)}$，注意这里的$V_z$是负的值，所以要需要把$V_z$反向，得到正式的$d_{linear}=-V_z=\frac{2fn}{(f+n)-NDC_z(f-n)}$，和参考文章(1)当中的GL推导一样。
  - 在Unity的D3D， Z buffer range is 1–0  ，从_CameraDepthTexture得到的值也是[1,0]，

### 一些Unity内置的ShaderVariables的说明

- CPU端的Variables：如camera的projectionMatrix，worldToCameraMatrix，都是在OpenGL Convention下得到的，符合上面的说明。

- For post process because it has two camera: main and post process camera Projection matrix is different between CPU and GPU in Unity

- Unity的Shader当中有一个UNITY_MATRIX_V内置的变量，得到的值是Camera.worldToCameraMatrix

- 其他的一些值在GPU端，根据设置的不同，内置的[Built-in shader variables](https://docs.unity3d.com/Manual/SL-UnityShaderVariables.html)会有所不同

  - projectionMatrix会在DX11的时候把值映射到[1,0]，即$V_z\in\{ -n,-f\} \Rightarrow NDC_z\in\{1,0\}$

    所以得到的矩阵是
    $$
    \begin{aligned}
      P_{clip} &= \begin{pmatrix}
      \frac{n}{r}&0&0&0\\0
      &\frac{n}{t}&0&0\\
      0&0&\frac{n}{f-n}&\frac{fn}{f-n}\\
      0&0&-1&0
      \end{pmatrix}\times
      \begin{bmatrix}V_x\\V_y\\V_z\\1
      \end{bmatrix}\\&=
      (\frac{n}{r}V_x,\frac{n}{t}V_y,\frac{n}{f-n}V_z + \frac{fn}{f-n},-V_z)
      \end{aligned} in\ ClipSpace
    $$

  - 这个矩阵在DX11的Unity版本对应UNITY_MATRIX_P 的值，在CPU端可以通过[GetGPUProjectionMatrix](https://docs.unity3d.com/ja/current/ScriptReference/GL.GetGPUProjectionMatrix.html) 来得到一致的CPU的值

  - [这篇文章](http://logicalbeat.jp/blog/929/)也有比较，得出的结论是一致的

  - 从_CameraDepthTexture得到的值也是[1,0]

  - GetGPUProjectionMatrix的方法，若设置了rendertexture，则会reverse y轴

  - Linear01Depth的方法，是把ndc space的z值，还原为view space， 同时在除以f来normalize到[0,1]，注意这里变换会view space之后，得到的z值是负的，需要再reverse到正向，之后再normalize到[0,1]。

    对应的解法在这里：<http://www.humus.name/temp/Linearize%20depth.txt> 

    注意区分OpenGL和D3D，注意在Unity的Shader部分，这些相关的值都是根据当前的平台来转换到对应的OpenGL或者D3D的值，同时最终的depth范围也有所不同。

  - Unity Shader几个相关的定义和函数

    - UNITY_MATRIX_V得到Camera.worldToCameraMatrix，和OpenGL的convention一样，forward是z轴负方向；

      但是Unity在CPU端的计算是基于左手坐标系的，所以若需要左手的View matrix，可以使用Camera的Transform.worldToLocalMatrix

    - UNITY_MATRIX_P根据平台的不同由不同的映射，即$V_z\in\{ -n,-f\}$映射到：

      - OpenGL映射到[-1,1]

      - D3D9映射到[0,1]

      - D3D11映射到[1,0]

        相对于不同的映射，对应的UNITY_MATRIX_P也不同

    - Depth Buffer的范围

      - OpenGL是[0,1]
      - D3D9是[0,1]
      - D3D11是[1,0]

    - Depth相关函数的定义：<https://docs.unity3d.com/Manual/SL-DepthTextures.html> 

      - Linear01Depth是把depth换原到view space并使用far plane来normalize到[0,1]

      - LinearEyeDepth是把depth换原到view space但是不normalize

      - 由于View space的z值，和OpenGL的Convention一样，是负的值，所以这里换原回去的值也是负的，需要变成正号，_ZBufferParams的xy都是反向之后的参数，这里的Linear01Depth和LinearEyeDepth都是正的值

      - 详细的推导在这里http://www.humus.name/temp/Linearize%20depth.txt

      - 相关函数变量的源码

        ```c++
        // x = 1 or -1 (-1 if projection is flipped)
        // y = near plane
        // z = far plane
        // w = 1/far plane
        float4 _ProjectionParams;
        
        // x = width
        // y = height
        // z = 1 + 1.0/width
        // w = 1 + 1.0/height
        float4 _ScreenParams;
        
        // Values used to linearize the Z buffer (http://www.humus.name/temp/Linearize%20depth.txt)
        // x = 1-far/near
        // y = far/near
        // z = x/far
        // w = y/far
        // or in case of a reversed depth buffer (UNITY_REVERSED_Z is 1)
        // D3D11
        // x = -1+far/near
        // y = 1
        // z = x/far
        // w = 1/far
        float4 _ZBufferParams;
        
        // Z buffer to linear 0..1 depth
        inline float Linear01Depth( float z )
        {
            return 1.0 / (_ZBufferParams.x * z + _ZBufferParams.y);
        }
        // Z buffer to linear depth
        inline float LinearEyeDepth( float z )
        {
            return 1.0 / (_ZBufferParams.z * z + _ZBufferParams.w);
        }
        ```