---
layout: post
title: GLSL 内建函数
date: 2014-06-20 12:38
author: ray
comments: true
categories: [Blog]
---
OpenGL ES着色语言为标量和向量操作定义了一套内建便利函数。有些内建函数可以用在多个类型的着色器中，有些是针对固定硬件的，所以这部分只能用在某个特定的着色器上。

内建函数基本上可以分为一下三类：

（1）它们使用一些简便的方式提供必要的硬件功能，如材质贴图。这些函数单独通过着色器是无法模拟出来的。

（2）它们展示了一些可以常简单的写入的繁琐操作（clamp， mix等），但是这些操作非常普遍，并且提供直接对硬件的支持。对于编译器来说，将表达式映射到复杂的装配线指令上是非常困难的。

（3）它们提供了对图形硬件的操作，并且在适当时候进行加速。三角函数就是一个很好的例子。
<!--more-->
有些函数名称和常见的C库函数类似，但是它们支持向量的输入和更多的传统标量输入。

建议应用程序尽量使用内建函数而不是在着色器中实现相同的计算，因为内建函数是经过最大优化的（如，有些内建函数是直接操作硬件的）。

用户定义的代码可以重载内建函数，但最好不要重新定义它们。

内建函数的输入参数（和相应的输出的参数）可以是float, vec2, vec3, vec4。对于函数的任何特定应用，实际的类型必须和所有的参数和返回值相同。就像mat，必须为mat2, mat3, mat4.

参数和返回值的精度修饰符是隐藏的，对于材质函数，返回类型的精度必须和采样器类型匹配。
<blockquote>uniform lowp sampler2D sampler;
highp vec2 coord;
...
lowp vec4 col = texture2D (sampler, coord); // texture2D 返回类型的精度为lowp</blockquote>
其他内建函数形参的精度修饰是没有任何关联的。内建函数的调用将返回输入参数的最高精度。

<!--more-->
<h2>1  角度和三角函数</h2>
标识为angle的函数参数假定以弧度为单位。这些函数不可能出现被0除的情况，如果被除数为0，结果是未定义的。

<img src="http://my.csdn.net/uploads/201208/17/1345172180_4370.png" alt="" />

radian函数是将角度转换为弧度，degrees函数是将弧度转换为角度。sin, cos, tan都是标准的三角函数。asin, acos, atan是反三角函数。genType有点像面向对象中泛型，即如果genType是float型的，那么

genType pow (genType x, genType y)

就变成了：

float pow (float x, float y)

同理，如果genType是int型的，那么就变成了：

int pow (int x, int y)；
<h2>2  指数函数</h2>
<img src="http://my.csdn.net/uploads/201208/17/1345172229_7550.png" alt="" />

（1）genType pow (genType x, genType y)

x的y次方。如果x小于0，结果是未定义的。同样，如果x=0并且y&lt;=0,结果也是未定义的。使用时应特别注意。

（2）genType exp (genType x)

e的x次方

（3）genType log (genType x)

计算满足x等于e的y次方的y的值。如果x的值小于0，结果是未定义的。

（4）genType exp2 (genType x)

计算2的x次方

（5）genType log2 (genType x)

计算满足x等于2的y次方的y的值。如果x的值小于0，结果是未定义的。

（6）genType sqrt (genType x)

计算x的开方。如果x小于0，结果是未定义的。

（7）genType inversesqrt (genType x)

计算x的开方之一的值，如果x小于等于0，结果是未定义的。
<h2>3  常用函数</h2>
<img src="http://my.csdn.net/uploads/201208/17/1345172268_1405.png" alt="" />

<img src="http://my.csdn.net/uploads/201208/17/1345172298_7007.png" alt="" />

（1）genType abs (genType x)
<blockquote>返回x的绝对值</blockquote>
（2）genType sign (genType x)
<blockquote>如果x&gt;0，返回1.0；如果x=0，返回0，如果x&lt;0，返回-1.0</blockquote>
（3）genType floor (genType x)
<blockquote>返回小于等于x的最大整数值</blockquote>
（4）genType ceil (genType x)
<blockquote>返回大于等于x的最小整数值</blockquote>
（5）genType fract (genType x)
<blockquote>返回x-floor(x)，即返回x的小数部分</blockquote>
（6）genType mod (genType x, float y)、genType mod (genType x, genType y)
<blockquote>返回x – y * floor (x/y)，即求模计算%</blockquote>
（7）genType min (genType x, genType y)，genType min (genType x, float y)
<blockquote>返回x和y的值较小的那个值。</blockquote>
（8）genType max (genType x, genType y)，genType max (genType x, float y)
<blockquote>返回x和y的值较大的那个值。</blockquote>
（9）genType clamp (genType x, genType minVal, genType maxVal)、genType clamp (genType x, float minVal, float maxVal)
<blockquote>clamp翻译为夹具，就叫夹具函数吧，这个函数是什么意思呢？看看解释的意思是：获取x和minVal之间较大的那个值，然后再拿较大的那个值和 最后那个最大的值进行比较然后获取较小的那个，意思就明白了，clamp实际上是获得三个参数中大小处在中间的那个值。函数有个说明：如果minVal &gt; minMax的话，函数返回的结果是未定的。也就是说x的值大小没有限制，但是minval的值必须比maxVal小。</blockquote>
（10）genType mix (genType x, genType y, genType a)、genType mix (genType x, genType y, float a)
<blockquote>返回线性混合的x和y，如：x⋅(1−a)+y⋅a</blockquote>
（11）genType step (genType edge, genType x)，genType step (float edge, genType x)
<blockquote>如果x &lt; edge，返回0.0，否则返回1.0</blockquote>
（12）genType smoothstep (genType edge0,genType edge1,genType x)，genType smoothstep (float edge0,float edge1,genType x)
<blockquote>如果x &lt;= edge0，返回0.0 ；如果x &gt;= edge1 返回1.0；如果edge0 &lt; x &lt; edge1，则执行0~1之间的平滑埃尔米特差值。如果edge0 &gt;= edge1，结果是未定义的。</blockquote>
<h2>4  几何函数</h2>
<img src="http://my.csdn.net/uploads/201208/17/1345172341_4872.png" alt="" />

（1）float length (genType x)
<blockquote>返回向量x的长度</blockquote>
（2）float distance (genType p0, genType p1)
<blockquote>计算向量p0，p1之间的距离</blockquote>
（3）float dot (genType x, genType y)
<blockquote>向量x，y之间的点积</blockquote>
（4）vec3 cross (vec3 x, vec3 y)
<blockquote>向量x，y之间的叉积</blockquote>
（5）genType normalize (genType x)
<blockquote>标准化向量，返回一个方向和x相同但长度为1的向量</blockquote>
（6）genType faceforward(genType N, genType I, genType Nref)
<blockquote>如果Nref和I的点积小于0，返回N；否则，返回-N；</blockquote>
（7）genType reflect (genType I, genType N)
<blockquote>返回反射向量</blockquote>
（8）genType refract(genType I, genType N,float eta)
<blockquote>返回折射向量</blockquote>
<h2>5  矩阵函数</h2>
<img src="http://my.csdn.net/uploads/201208/17/1345172404_2445.png" alt="" />

（1）mat matrixCompMult (mat x, mat y)
<blockquote>矩阵x乘以y，result[i][j]是 x[i][j] 和 y[i][j] 的标量积。注意，要获取线性代数矩阵的乘法，使用乘法操作符*。</blockquote>
<h2>6  向量相关函数</h2>
相关或相等的操作符(&lt;, &lt;=, &gt;, &gt;=, ==, !=)被定义（或保留），返回一个标量布尔值。下面, “bvec” 是表示bvec2, bvec3, or bvec4的占位符, “ivec”是ivec2, ivec3, or ivec4的占位符,  “vec” 是vec2, vec3, or vec4的占位符. 在任何情况下，输入和返回值向量的长度必须匹配。

<img src="http://my.csdn.net/uploads/201208/17/1345172441_1312.png" alt="" />

（1）lessThan
<blockquote>比较x &lt; y.</blockquote>
（2）lessThanEqual
<blockquote>比较x&lt;=y</blockquote>
（3）greaterThan
<blockquote>比较x&gt;y</blockquote>
（4）greaterThanEqual
<blockquote>比较x&gt;=y</blockquote>
（5）equal
<blockquote>比较x==y</blockquote>
（6）notEqual
<blockquote>比较x!=y</blockquote>
（7）bool any(bvec x)
<blockquote>如果向量x的任何组件为true，则结果返回true。</blockquote>
（8）bool all(bvec x)

如果向量x的所有组件均为true，则结果返回true。

（9）bvec not(bvec x)
<blockquote>返回向量x的互补矩阵</blockquote>
<h2>7  材质查找函数</h2>
纹理（材质）查找函数对于定点着色器和片元着色器都适用。然而，定点着色器的细节级别并不是通过固定功能计算的，所以顶点着色器和片元着色器纹理查找之间 还是有一些差别的。一下函数是通过采样器访问纹理，和使用OpenGL ES API是一样的。纹理属性如尺寸，像素格式，维数，过滤方法，纹理映射匹配数，深度比较等等在OpenGL ES API中都有定义。

在下面的函数中，bias参数对于片元着色器来说是可选的。但在定点着色器中不可使用。对于片元着色器，如果使用了bias这个参数，它被加到优先细节的 计算级别中来执行纹理访问操作。如果bias没有使用，那么实现将自动选择一个默认级别。对于非纹理映射的纹理，纹理是直接被使用的。如果是纹理映射的， 并且在片元着色器中执行，那么使用LOD来进行纹理查找。如果是纹理映射的，并且在顶点着色器中执行，那么使用的是基本纹理。

以Lod结尾的内建函数只能用在顶点着色器中，在带有Lod的函数中，lod参数直接用来表示细节级别。

<img src="http://my.csdn.net/uploads/201208/17/1345172615_4734.png" alt="" />
<h2>8  片元处理函数</h2>
片元处理函数只有在片元语言中才有，但在GLSL ES中没有片元处理函数。
