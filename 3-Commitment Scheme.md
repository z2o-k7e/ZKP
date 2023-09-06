本文尝试以简单的方式介绍常见的commitment scheme, 让初学者对每个commitment scheme过程有一个了解，尽量不涉及群(Group),域(Field), 有限域(Finite Field)等复杂的数学知识。此外，同一个commitment sheme 有不同的表述方法，本文主要参考[1]中的表述方法。
<br>
<br>

# commitment
承诺(Commitment)是零知识证明的一个重要组件，它以安全的方式将一个值隐藏，并在未来的证明这个值的有效性。
所谓承诺，是对消息「锁定」，得到一个锁定值。这个值被称为对象的「承诺」，例如 $C$ 就是消息 $x$的承诺

$$
C=commit(x)
$$

这个值和原对象存在两个关系，即 Hiding 与 Binding。
+ Hiding： $c$ 不暴露任何关于 $x$的信息；
+ Binding：难以找到一个 $x', x'\neq x$，使得 $C=commit(x')$

最简单的承诺操作就是 Hash 运算。请注意这里的 Hash 运算需要具备密码学安全强度，比如 SHA256, Keccak 等。除了 Hash 算法之外，还有 Pedersen 承诺等。承诺通常分成3个阶段：
+ **Setup：** 通常是在椭圆曲线或者有限域挑选一组随机数作为基点作为公共参数。有些commitment scheme 不需要公共参数。
+ **Commit：** Prover 将原始信息和基点进行运算得到一个加密的值，原始的信息隐藏在其中，并将这个加密的值发给Verifier。
+ **Verify：** Prover提供原始信息或者相关的证明给Verifier，Verifer执行验证算法来确认承诺的正确性。
<br>
<br>

# Pedersen Commitment
Pedersen承诺 是1992年在“Non-Interactive and Information-Theoretic Secure Verifiable Secret Sharing”一文中提出。Pedersen承诺可以应用于离散对数和椭圆曲线，本文中将以椭圆曲线为例说明。 Pedersen Commitment 由3个阶段。

+ **Setup**：     
Prover 随机挑选椭圆曲线上的两个点 $G,H$作为公共参数。

+ **Commit**：     
对于消息 $m$, Prover 随机挑选致盲因子 $r$，提交 $C = mG + rH$， $C$ 是椭圆曲线上的一个点。

+ **Verify**：   
Prover 发送 $(m,r)$给Verifier，Verifier计算 $C'=mG+rH$，并验证 $C'\overset{\text{?}}{=} C$。
<br>
<div align=center><img src ="https://github.com/zkp-co-learning/ZKP/assets/78890754/2027ab30-b7a1-4680-8635-2b9b82b7cd17"></div>
<br>

Pedersen Commitment 还可以对向量进行承诺，对于 $\vec a = (a_0,a_1,\cdots,a_{n-1})$。Prover 随机挑选一系列的基点 $(G_0,G_1,\cdots\,G_{n-1})$ ， 计算:

$$
C = a_0G_0+a_1G_1+\cdots+ a_{n-1}G_{n-1}
$$

<br>

### Pedersen承诺具有加法同态性
所谓加法同态，即两数和的密文等于两数的密文的和，假设明文a, b, 加密函数e，满足：

$$
\begin{split}
&c = a + b \\
&e(a) + e (b) = e(a+b) \\
\end{split}
$$

我们称函数e具有加法同态性。

对于消息 $m_1, m_2$，随机数 $r_1，r_2$，它们的承诺分别为

$$
\begin{split}
&C_1=m_1 G+r_1 H \\
&C_2 = m_2G+r_2 H \\
\end{split}
$$

$m_1 +m_2$ 之和的承诺为：

$$
\begin{split}
C &= (m_1+m_2) G+(r_1+r_2) H \\ 
&= (m_1 G + r_1 H) + (m_2 G+r_2 H) \\ 
&= C_1 + C_2 
\end{split}
$$

<br>
<br>

# Vector Commitment
向量承诺用于承诺有 $d$个元素的向量 $\vec u=(u_1,u_2,\cdots,u_d)$，Merkle Commitment 是常用的向量承诺方案， Merkle Commitment只有Commit 和Verify 两个阶段。下面以 $\vec u=[z,k,-,s,n,a,r,k]$ 为例说明Merkle Commitment的流程。

+ **Commit**：     
对于 $\vec u=[z,k,-,s,n,a,r,k]$, Prover 首先将向量的元素作为叶子节点构成一个满二叉树(对于元素个数不是 $2^n$的向量，在后面补0到最近的 $2^n$)，然后两两一组做Hash，得到如下图所示的一棵二叉树，图中每一个非叶子节点都是由它的孩子节点的hash构成：

$$
\begin{split}
&h_1=Hash(h_2,h_3) \\
&h_2 = Hash(h_4,h_5) \\ 
&\vdots 
\end{split}
$$ 

 $h_1$是Merkle Tree的根，也是 $\vec u$ 的commitment。

<br>
<div align=center><img src="https://github.com/zkp-co-learning/ZKP/assets/78890754/51a562b1-37b4-4b06-99c7-e97482b5eb6d"></div>
<br>

+ **Verify**：   
当Prover被要求证明的某元素存在于某个位置时，它将仅提供必要的节点，这些节点构成的集合通常被称为Merkle Path 或者Merkle Proof, 例如为了证明 $n$是 $\vec u$的第4个元素， Prover将提供Merkle Path  $[s, h_4, h_3]$ (图中蓝色的节点) 给Verifier， Verifier将执行以下操作：

$$
\begin{split}
&h_5=Hash(s,n) \\
&h_2=Hash(h_4, h_5) \\ 
&h'_1=Hash(h_2, h_3) \\
\end{split}
$$

Verifier 验证 $h'_1 \overset{?}{=} h_1$，如果相等，那么 $n$就是 $\vec u$中第4个元素。

<br>
<div align=center><img src="https://github.com/zkp-co-learning/ZKP/assets/78890754/d1691b18-96d5-478b-b6ff-7daf8ecec625"></div>
<br>

Merkle commitment 的流程图如下：
<br>
<div align=center><img src="https://github.com/zkp-co-learning/ZKP/assets/78890754/30a24454-c8d5-4e61-9e9d-b418fb9ec526"></div>
<br>
<br>


# KZG10 Commitment
多项式承诺可以理解为「多项式」的「承诺」。如果我们把一个多项式表达成如下的公式，

$$
f(x) = a_0 + a_1x + a_2x^2 + \cdots + a_nx^n
$$

那么我们可以用所有系数构成的向量来唯一标识多项式 $f(x)$。

$$
(a_0, a_1, a_2,\ldots, a_n)
$$

我们可以把「系数向量」进行 Hash 运算，得到一个数值，就能建立与这个多项式之间唯一的绑定关系。

$$
C = \textrm{Hash}(a_0, a_1,a_2, \cdots,a_n)
$$

或者，也可以使用向量版本的 Petersen 承诺。此外，本文后面介绍的IPA commitment，FRI commitment 也可以用于承诺一个多项式。本节我们讨论KZG10[6]承诺。

KZG10 commitment 是基于双线性配对(Bilinear pairings)的，关于双线性配对的原理可以参考其他的资料，这里不做过多的叙述。
<br>
KZG10 承诺分为3个阶段：

+ **Setup**：
1. 产生一个随机数 $\tau$
2. 生成公共参数:

$$
\vec G = (G_0, G_1,\cdots,G_{d-1},H_0, H_1) =(G, \tau G, \tau^2  G, \cdots, \tau^{d-1}G, H,\tau H)
$$
    
其中 $G\in \mathbb{G}_1,H\in\mathbb{G}_2$，并且存在双线性映射 $e\in\mathbb{G}_1 \times \mathbb{G}_2 \rightarrow \mathbb{G}_T$。    
对于双线性群，使用符号 $[1]_1\triangleq G$， $[1]_2\triangleq H$ 表示两个群上的生成元，这样 KZG10 的系统参数（也被称为 SRS, Structured Reference String）可以表示如下： 
    
$$
\mathsf{srs}=([1]_1,[\tau]_1,[\tau^2]_1,[\tau^3]_1,\ldots,[\tau^{n-1}]_1,[1]_2,[\tau]_2)
$$
    
3. 最后也是最重要的，就是删除 $\tau$。如果这个秘密被其他人知道，他们可以伪造证明。在实际执行时，通常有多方参与Setup过程，每个参与方贡献一部分数据组成 $\tau$，只要其中一方是诚实的，删除了自己贡献的数据，那么这个 $\tau$就是可信的，这个过程也被称为Trust Setup Ceremony.     

+ **Commit**：<br>
Prover 将多项式系数 $(a_0,a_1,\cdots,a_{n-1})$和 $(G_0,G_1,\cdots,G_{n-1})$ 逐项相乘，得到commitment $[f(x)]_1$:

$$
\begin{split}
C = &=a_0 G_0 + a_1  G_1 + \cdots + a_{n-1} G_{n-1} \\
 & = a_0  G + a_1 \tau G + \cdots + a_{n-1}\tau^{n-1} G\\
 & = f(\tau) G 
\end{split}
$$


+ **Verify**：
1. Verifier 产生一个随机数 $\zeta$
2. Prover 在 $\zeta$点打开$f(x)$，得到 $f(\zeta)=y$,  因为 $\zeta$ 是 $f(\zeta)-y=0$ 的根，计算商多项式

$$
q(x)=\frac{f(x)-y}{x-\zeta}
$$

的commitment $[q(x)]_1$ ，并将其作为 $f(x)$在 $\zeta$处的求值证明和 $y$ 一起发送给Verifier。

3. Verifier 计算
    
$$
f(x)-y = q(x)(x-\zeta)  
$$
    
将等式右边展开并将 $q(x)\zeta$移到等式左边，得到

$$
f(x)-y+q(x)\zeta = q(x)x
$$

对于上式应用双线性映射, Verifier只需要验证

$$
 e([f(x)]_1- [y]_1 + \zeta[q(x)]_1 , [1]_2)  \overset{?}{=} e([q(x)]_1, [x]_2)
$$

<br>
<div align=center><img src="https://github.com/zkp-co-learning/ZKP/assets/78890754/a079b876-8531-47f5-ac90-1ced1f61cfd9"></div>
<br>

## 单个多项式多点打开

阶为 $d$ 的函数 $f(x)$经过 $f(\zeta_1)=y_1, f(\zeta_2)=y_2，\cdots, f(\zeta_m)=y_m, m < d$ 个点，只需要验证一次就可以了。
首先构造一个经过 $(\zeta_1, y_1),(\zeta_2,y_2)，\cdots,(\zeta_m,y_m)$ 的辅助函数 $h(x)$。
由于 $(\zeta_1, \zeta_2, \cdots, \zeta_m)$同时为 $f(x),h(x)$的根，因此他们也是 $f(x)-h(x)$的根，故存在一个商多项式:

$$
q(x)=\frac{f(x)-h(x)}{\prod \limits_{i=1}^m (x-\zeta_i)}
$$

Prover 生成 $[q(x)]_2$ 并发送给Verifier，Verifier自己计算 $[h(x)]_1$， $[\prod \limits_{i=1}^m(x-\zeta_m)]_1$，最后再验证：


$$
 e([f(x)]_1- [h(x)]_1，[1]_2) \overset{?}{=} e([\prod \limits_{i=1}^m(x-\zeta_m) ]_1,[q(x)]_2)
$$


需要说明 $[q(x)]_2$是 $\mathbb{G_2}$上的承诺，因此需要在Setup阶段产生m个 $\mathbb{G_2}$基 $(H, \tau H, \cdots, \tau^mH)$ $(\color{red}Question(keep)是否需要m个 \mathbb{G}_2上的基点?)$。
<br>
<br>

## 多个多项式同一点打开

假如同时使用多个多项式承诺，那么他们的验证操作可以合并在一起完成。即把多个多项式先合并成一个更大的多项式，然后仅通过验证一点，来完成对原始多项式的批量验证。
假设我们有两个多项式 $f_1(x), f_2(x)$，Prover 要同时向 Verifier 证明 $f_1(\zeta)=y_1$和 $f_2(\zeta)=y_2$ ，那么有

$$
\begin{array}{l}
f_1(x) = q_1(x)\cdot (x-\zeta) + y_1\\
f_2(x) = q_2(x) \cdot (x-\zeta) + y_2 \\
\end{array}
$$

通过一个随机数 $\nu$，Prover 可以把两个多项式 $f_1(x), f_2(x)$) 折叠在一起，得到一个临时的多项式 $g(x)$ ：

$$
g(x) = f_1(x) + \nu\cdot f_2(x)
$$

进而我们可以根据多项式余数定理，推导验证下面的等式：

$$
g(x) - (y_1 + \nu\cdot y_2) = (x-\zeta)\cdot (q_1(x) + \nu\cdot q_2(x))
$$

我们把等号右边的第二项看作为「商多项式」，记为 $q(x)$：

$$
q(x) = q_1(x) + \nu\cdot q_2(x)
$$

假如 $f_1(x)$  在 $x=\zeta$ 处的求值证明为 $\pi_1$，而 $f_2(x)$在 $\zeta$ 处的求值证明为 $\pi_2$，那么根据群加法的同态性，Prover 可以得到商多项式  $q(x)$的承诺：

$$
[q(x)]_1 = \pi = \pi_1 + \nu\cdot\pi_2
$$

因此，只要 Verifier 发给 Prover 一个额外的随机数 $\nu$，双方就可以把两个（甚至多个）多项式承诺折叠成一个多项式承诺 $C$：

$$
C = C_1 + \nu\ast C_2
$$

并用这个折叠后的 $C$ 来验证多个多项式在一个点处的运算取值：

$$
y_g = y_1 + \nu \cdot y_2
$$

从而把多个求值证明相应地折叠成一个，Verifier 可以一次验证完毕：

$$
 e([g(x)]_1- [y_g]_1 + \zeta[q(x)]_1 , [1]_2)  \overset{?}{=} e([q(x)]_1, [x]_2)
$$

<br>
<br>

# Multilinear Commitment
如果一个多项式有 $l$个变量， 并且每个变量的最高次为1，称此多项式为multilinear 多项式 。例如

$$
g(x_1, x_2, x_3, x_4) = (1-x_1)x_2((x_3+x_4)-(x_3x_4))
$$

我们可以使用sum-check protocol[7] 对multilinear 多项式的承诺及其验证。sum-check protocol 只有Commit 和 Verify两个阶段。

+ **Commit**：<br>
Prover 计算multilinear多项式的commitment

$$
C= \sum_{b_0 \in \{0,1\}} \sum_{b_2 \in \{0,1\}} \cdots \sum_{b_l \in \{0,1\}} g(b_1, b_2, b_3, \cdots, b_l)
$$

并将其发给Verifier。仔细观察，不难看出通过枚举 $(b_0,b_1,\cdots,b_l)$ 后就能得到C， 例如对于上面 $g(x_1, x_2, x_3, x_4)$ 这个函数， $C = g(0,0,0,0) + g(0,0,0,1)+\cdots+g(1,1,1,1)$


+ **Verify**：
1. Prover 保留第1个变量 $x_1$为自由变量, 其他变量取值为 $x_i\in \{0,1\}(2\leqslant i \leqslant l)$ 时，得到一个部分和多项式 $g_1(x_1)$。
2. Verifier 检查 $g_0 \overset{?}{=}g_1(0)+g_1(1)$。
3. Verifier 挑选一个随机数 $\zeta_1$ 发送给Prover。
4. Prover 用 $\zeta_1$ 取代 $x_1$。然后保留第2个变量 $x_2$为自由变量，其他变量的取值为 $x_i\in \{0,1\}(3\leqslant i \leqslant l)$时，计算部分和公式 
 $g_2(x_2)$。
5. Verifier 检查 $g_2 \overset{?}{=}g_1(0)+g_1(1)$。
6. 重复3~5直到最后一个变量 $x_l$也变成自由变量。
7. Verifier 挑选一个随机数 $\zeta_l$，本地计算 $g_l(\zeta_l)$，并从一个可信的oracle $(\color{red}Question(keep)实际执行中谁是oracle?)$ 获得 $g(\zeta_1,\zeta_2,\cdots,\zeta_l)$的值，然后验证 $g_l(\zeta_l) \overset{?}{=} g(\zeta_1,\zeta_2,\cdots,\zeta_l)$


下图详细的描述了 $g(x_1, x_2, x_3, x_4) = (1-x_1)x_2((x_3+x_4)-(x_3x_4))$ Multlinear commit的过程.
<br>
<br>
<div align=center><img src="https://github.com/zkp-co-learning/ZKP/assets/78890754/c7c922c8-8018-4d5c-9796-2065d2ead7dc"></div>
<br>
<br>

# IPA Arguments
IPA(Inner Product Arguments) 中文翻译为内积证明，这是一种不要求可信设置的零知识证明算法。门罗币（Monero）就用了这个算法。内积，即计算两个向量中每个分量的乘积和， 例如对于 $\vec a = (a_0, a_1, \ldots, a_{n-1})$， $\vec b = (b_0, b_1, \ldots, b_{n-1})$ 两个向量，他们的内积等于

$$
\vec a \cdot \vec b = a_0 b_0 + a_1 b_1 + a_2 b_2 + \cdots + a_{n-1} b_{n-1} 
$$

我们可以将多项式 $f(x)= a_0+a_1x+a_2x^2+a_3x^3+\cdots+a_{n-1}x^{n-1}$, 看作是 $\vec a = (a_0, a_1, \ldots, a_{n-1})$ 和 $\vec b = (1, x, \ldots, x^{n-1})$ 两个向量的内积。 

IPA arguments 的流程如下：

+ **Setup**：<br>
生成一组随机数，构成公共参数 $gp=(G_0,G_1,G_2,\cdots, G_{n-1})$。

+ **Commit**：<br>
Prover 将系数向量 $\vec a$ 和 $gp$ 做内积，得到commitment C，并发送给Verifier.

$$
C = \vec a \cdot \vec G= a_0 G_0 + a_1  G_1 + \cdots + a_{n-1} G_{n-1}
$$

+ **Verify**：<br>
1. Verifier 发送一个随机数 $u$ 给Prover。
2. Prover 计算 $f(x)$在  $x=u$ 点的取值 $z$：

$$
\begin{split}
z &= (a_0, a_1,a_2,\cdots, a_{n-1}) \cdot (1,u,u^2, \cdots, u^{n-1}) \\
& =(a_0+a_1u+a_2u^2+ \cdots + a_{n-1}u^{n-1}) \\
\end{split}
$$

将 $\vec a, gp$ 从中间分成两半，计算 $z_L, z_R, L, R$（假设 $n=2^k, m=n/2$)

$$
\begin{split}
&z_L=a_0+a_1u+ \cdots a_{m-1}u^{m-1} \\
&z_R=a_{m}+a_{m+1}u+ \cdots + a_{n-1}u^{m-1} \\
&L=a_0 G_m+ a_1 G_{m+1} + \cdots + a_{m-1} G_{n-1} \\
&R=a_{m} G_0 + a_{m+1} G_1 + \cdots + a_{n-1} G_{m_1} \\
\end{split}
$$

$L$ 是左半部分 $\vec a$ 和 右半部分 $gp$ 的内积，  $R$是右半部分 $\vec a$ 和 坐半部分 $gp$ 的内积， 构成了一个交叉项(cross item)，然后将 $z,z_L,z_R,L,R$,发送给Verifier。 

3. Verifier 验证 $z \overset{?}{=}z_{L}+z_{R}u^m$。
4. 如果通过，Verifier 发送一个随机数 $r$给Prover。
5. Prover 使用 $r$ 折叠 $f(x)$ 和 $gp$,  得到 $f'(x)$和 $gp'$，折叠的方式如下：
<br>

$$
\vec{a'} =(a_0', a_1', \cdots, a_{m-1}') = r 
\begin{pmatrix}a_0 \\ 
a_1 \\
\vdots \\ 
a_{m-1} \end{pmatrix} + 
\begin{pmatrix}a_m \\ 
a_{m+1} \\
\vdots \\ 
a_{n-1}\end{pmatrix} = (r a_0+a_{m}，r a_1+a_{m+1}, \cdots, r a_{m-1} + a_{n-1})
$$

<br>

$$
gp' =(G_0', G_1', \cdots, G_{m-1}')= r^{-1} 
\begin{pmatrix}G_0 \\ 
G_1 \\
\vdots \\ 
G_{m-1} \end{pmatrix} + 
\begin{pmatrix}G_m \\ 
G_{m+1} \\
\vdots \\ 
G_{n-1}\end{pmatrix}= (r^{-1}G_0+G_m, r^{-1}G_1+G_{m+1}, \cdots, r^{-1}G_{m-1}+G_{n-1})
$$    

<br>

折叠后，Prover计算新的多项式 $f'(x)$在 $x=u$点的取值 $z'$ 和它的承诺 $C'$:

$$
\begin{split}
z'&= (ra_0+a_{m})+(ra_1+a_{m+1})u+ \cdots + (ra_{m-1}+a_{n-1})u^{m-1} = rz_L+z_R \\
C'&=(a_0',a_1', \cdots, a_{m-1}') \cdot (G_0',G_1',\cdots, G_{m-1}') \\ 
&=(ra_0+a_m，ra_1+a_{m+1}, \cdots, ra_{m-1}+a_{n-1}) \cdot (r^{-1}G_0+G_m, r^{-1}G_1+G_{m+1}, \cdots, r^{-1}G_{m-1}+G_{n-1}) \\
&=(ra_0+a_m)(r^{-1}G_0+G_m) +(ra_1+a_{m+1})(r^{-1}G_1+G_{m+1}) + \cdots + (ra_{m-1}+a_{n-1})(r^{-1}G_{m-1}+G_{n-1}) \\
&=C + rL+r^{-1}R \\
\end{split}
$$


同时将 $\vec a’, gp’$ 从中间分成两半，计算 $z_L’, z_R’, L’, R’$：

$$
\begin{split}
&z_L’=a_0’+a_1’u+ \cdots a_{m/2 -1}’u^{m/2-1} \\
&z_R'=a_{m/2}'+a_{m/2+1}'u+ \cdots + a_{m-1}'u^{m/2-1} \\
&L’= a_0’  G_{m/2}’ + a_1’  G_{m/2+1}’  + \cdots + a_{m/2-1}’  G_{m-1}’ \\
&R’ = a_{m/2}’  G_0’  + a_{m/2+1}’  G_1’  + \cdots + a_{m-1}’ G_{m/2-1}’ \\
\end{split}
$$

然后将 $z',C',z'_L,z'_R,L',R'$ 发送给Verifier 

6. Verifier 知道 $z_L,z_R,L,R,r$ 的值， 自己计算并验证：
    
$$
\begin{split}
&z' \overset{?}{=}rz_L+z_Ru^{m/2} \\
&C' \overset{?}{=}C+rL+r^{-1}R \\
\end{split}
$$
    
7. 令 $z_L=z'_L, z_R=z'_R, L=L', R=R'，m=m/2$，重复4～6步， 直到 $\vec a$ 折叠成1个点。
8. 最后一轮中除了第6步的验证外，还需要验证 $\vec a' \overset{?}{=} z'$  $(\color{red}Question(keep) 这句话是否正确)$。如果上述过程均正确完成，verifier 接受 否则拒绝。
<br>

为了简化，接下来我们以 $f(x)= 1+x+2x^2+3x^3$为例来说明IPA的流程。


<br>
<div align=center><img src="https://github.com/zkp-co-learning/ZKP/assets/78890754/5d19f0ee-2cb7-41f8-8f6b-0851ff1f0de1"></div>
<br>
<br>
IPA argument 的思路是通过随机数 $r$将一个高阶多项式不停的折叠，直到变成一个常数。仔细观察，不难发现IPA argument 的commitment scheme 是pedersen commitment。
<br>
<br>
<br>



# FRI Commitment
FRI 的 <ins>F</ins>ast <ins>R</ins>eed-Solomon <ins>I</ins>nteractive oracle Proofs of Proximity 的缩写。Reed-Solomon 是通信中一种常用的FEC 编码，基本思想是在原始信息的基础上增加冗余信息来对抗信道中的干扰。

例如我们要传输的1个bit原始信息，当原始bit为0b0时，编码后的数据是4个bit的0b0000，当原始bit为1时，编码后的数据是4个bit的0b1111。在传输过程中，因为受到干扰，接收方收可能会收到0b0100的时候，这个时候，接收方将它和0b0000, 0b1111比较，因为0b0100到0b0000的汉明距离时1， 到0b1111的距离时3，所以最后将它译作0b0000，对应的原始信息为0b0。

FRI的思想也是类似，对于 

$$
f(x) = a_0 + a_1x + a_2x^2 + \cdots + a_{n-}x^{n-1}
$$ 

理论上只需要有n个不同的点 

$$
(u_0, f(u_0)), (u_1, f(u_1)), \cdots, (u_{n-1}, f(u_{n-1}))
$$ 

就可以确定该多项式，在FRI中Prover从 $f(x)$上取 $n/\rho$ $(\rho < 1/2)$个点，假设 $\rho=1/8$, 就会有 $8n$个点来表示 $f(x)$，Prover 将这8n个点作为merkle的叶子节点得到merkle root 作为commitment提交。 

一个需要注意的问题是，这个commitment 包含 $8n$个点，我们理论上可以重建出一个 $8n-1$ 阶多项式，为了解决这个问题, Prover 好需要向Verifer 证明这些点对应的多项式 $f(x)$ 是一个比 $8n-1$ 低的多的 $n-1$多项式，这就是FRI 为什么要和LDT(Low Degree Test)配合使用的原因。 

为了简化，接下来我们以 $f(x)= 1+x+2x^2+3x^3$, $\rho^-1=8$ FRI commitment的流程。 
不难看出只需要4个点就能重建 $f(x)$，因为 $\rho^-1=8$，所以我们取 32th root of unity 作为输入集合 $\mathbb{\Omega}$。

<br>
<div align=center><img src="https://github.com/zkp-co-learning/ZKP/assets/78890754/265c039f-24e1-45af-8cbf-b0ebdee6651e"></div>
<br>
<br>


FRI commitment 没有setup，其流程如下：

+ **Commit**：<br>
1. 将 $f(x)$ 其分为偶次项 $g_0(x)$ 和奇次项 $h_0(x)$：

$$ 
\begin{split}
&g_0(x^2)=a_0+a_2x^2 + a_4x^4 + \cdots = 1+2x^2 \\
&h_0(x^2)=a_1+a_3x^2 + a_5x^4 + \cdots = 1+3x^2 \\
&f_0(x) = g_0(x^2) + xh_0(x^2)
\end{split}
$$

2. 计算 $f_0(x)$ 在 $1，-1， w， -w , \cdots$ 的值：

$$
\begin{split}
&f_0(1) = 1 + 1 + 2 + 3=7 \\
&f_0(-1) = 1 - 1 + 2 - 3= -1 \\
&f_0(w) = 1+w+2w^2+3w^3 \\
&f_0(-w) = 1-w+2w^2-3w^3 、、
\vdots
\end{split}
$$

3. 计算 $f_0(x)$ 的merke_root, 并将其发给Verifer。

$$
root_0 = merkle\_root(f_0(1),f_0(-1) f_0(w), f_0(-w) \cdots, f_0(w^{31}), f_o(-w^{31})))
$$

+ **Verify**：<br>
1. Verifer 发送一个随机数 $r$ 给Prover
2. Prover 使用 $r$ 之后做如下操作：
   
2.1 使用 $r_1$ 折叠 $g_0(x), h_0(x)$ 得到 $f_1(x)$,计算偶次项 $g_1(x)$ 和奇次项 $h_1(x)$：

$$
\begin{split}
&f_1(x) = g_0(x)+rh_0(x) = 1+2x + 6(1+3x) = 7+20x \\
&g_1(x^2) = 7 \\
&h_1(x^2) = 20 \\
&f_1(x) = g_1(x) + xh_1(x) \\
\end{split}
$$

2.2 计算 $f_1(x)$ 在 $1, -1，w^2, -w^2,  ..., w^{30}, -w^{30}$的取值：

$$
\begin{split}
&f_1(1) = 27 \\
&f_1(-1) = -13 \\
&f_1(w^2) = g_1(w^2) + w^2h_1(w^2) = 7+20w^2 \\
&f_1(-w^2) = g_1(-w^2) + (-w^2)h_1(-w^2) = 7-20w^2 \\
\vdots
&f_1(w^{30}) = g_1(w^{30}) + w^{30}h_1(w^{30}) = 7+20w^{30} \\
&f_1(-w^{30}) = g_1(-w^{30}) - w^{30}h_1(-w^{30}) = 7-20w^{30} \\
\end{split}
$$

2.2 计算 $f_1(x)$ merkle root,并将其发给Verifer。

$$
root_1 = merkle\_root(f_0(1),f_0(-1), f_0(w^2),  f_0(-w^2), \cdots, f_0(w^{30}), f_0(-w^{30}))
$$

    
3. Verifier 随机挑选1个点，检查：

$$
f_1(x^2) \overset{?}{=}( \frac{f_0(x)+f_0(-x)}{2} + r_1 \frac{f_0(x)-f_0(-x)}{2x})
$$

令 $x=w$, 

$$
\begin{split}
等式左边 &=f_1(w^2) = 7+20w^2 \\
等式右边 &=\frac{f_0(w)+f_0(-w)}{2} + r_1 \frac{f_0(w)-f_0(-w)}{2w} = (1+2w^2) + 6 *\frac{2w+6w^3}{2w} = 7+20w^2 \\
\end{split}
$$

需要说明的是在https://zk-learning.org/assets/lecture8.pdf中提到可以在完成全部折叠之后，再进行query，我感觉也可以让folding 和query 穿插进行不影响整个协议。

4. 检查通过后，请Prover 提供 $f_1(w^2), f_0(w),f_0(-w)$ 的merkle proof，   
5. Prover 提供merkle proof， Verifier验证，
6. 验证通过后，Verifer可以再挑选一个点进行检查，即重复3～5, 或者令 $f_0(x)=f_1(x), g_0(x)=g_1(x), h_0(x)= h_1(x)$ 跳转到1, 进行下一轮折叠。
7. 当 $f(x)$ 被折叠成一个常数时，Prover可以只一个element当作merkle root，Verifer 执行3～5的检查。
8. 当Verifier检查的次数达到安全系数要求且上述过程没有异常，accept否则reject。

<br>
<div align=center><img src="https://github.com/zkp-co-learning/ZKP/assets/78890754/1962dd57-2354-4791-8be4-87f468976bc0"></div>
<br>

### TODO(keep), 当query点不是 $w^i$的处理。

# Reference
[1]: https://zk-learning.org, zk Mooc
[2]: https://people.cs.georgetown.edu/jthaler/ProofsArgsAndZK.pdf
[3]: https://twitter.com/VitalikButerin/status/1371844878968176647
[4]: https://github.com/sec-bit/learning-zkp/blob/develop/plonk-intro-cn/plonk-polycom.md
[5]: https://starkware.co/stark-101
[6]: https://www.iacr.org/archive/asiacrypt2010/6477178/6477178.pdf
[7]: https://semiotic.ai/articles/sumcheck-tutorial/




