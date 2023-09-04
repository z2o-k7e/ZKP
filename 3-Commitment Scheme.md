本文尝试以尽可能简单的方式介绍常见的commitment scheme, 让读者了解每个commitment scheme 的过程，尽量不涉及复杂的数学知识。
<br>
<br>

# commitment
承诺(Commitment)是零知识证明领域的一个重要组件，它以安全的方式将一个值隐藏，并在未来的证明这个值的有效性。
所谓承诺，是对消息「锁定」，得到一个锁定值。这个值被称为对象的「承诺」，例如 $C$ 就是消息 $x$的承诺

$$
C=commit(x)
$$

这个值和原对象存在两个关系，即 Hiding 与 Binding。
+ Hiding： $c$ 不暴露任何关于 $x$的信息；
+ Binding：难以找到一个 $x', x'\neq x$，使得 $C=commit(x')$

最简单的承诺操作就是 Hash 运算。请注意这里的 Hash 运算需要具备密码学安全强度，比如 SHA256, Keccak 等。除了 Hash 算法之外，还有 Pedersen 承诺等。承诺通常分成3个阶段：
+ **Setup：** 选择一组公共参数，通常是在椭圆曲线或者有限域挑选一组随机数作为基点。有些commitment scheme 没有Setup阶段。
+ **Commit：** Prover 将原始信息和基点进行运算得到一个加密的值，原始的信息隐藏在其中，并将这个加密的值发给Verifier。
+ **Verify：** Prover提供原始信息或者相关的证明给Verifier，Verifer执行验证算法来确认承诺的正确性。
<br>
<br>

# Pedersen Commitment
Pedersen承诺 是1992年在“Non-Interactive and Information-Theoretic Secure Verifiable Secret Sharing”一文中提出。Pedersen承诺既可以应用于离散对数和椭圆曲线，本文中将以椭圆曲线为例说明。 Pedersen Commitment

+ **Setup**：     
Prover 随机挑选椭圆曲线上的两个点 $G,H$，并公开。

+ **Commit**：     
对于消息 $m$, Prover 随机挑选致盲因子 $r$，提交 $C = mG + rH$，$C$是椭圆曲线上的一个点。

+ **Verify**：   
Prover 发送 $(m,r)$给Verifier，Verifier计算 $C'=mG+rH$，并验证 $C'\overset{\text{?}}{=} C$。
<br>
<div align=center><img src ="https://github.com/zkp-co-learning/ZKP/assets/78890754/2027ab30-b7a1-4680-8635-2b9b82b7cd17"></div>
<br>

Pedersen Commitment 还可以对多个值进行承诺，对于 $\vec a = (a_0,a_1,\cdots,a_n)$。Prover 随机挑选一系列的基点 $(G_0,G_1,\cdots\,vGn)$ ， 计算:

$$
C = a_0G_0+a_1G_1+\cdots+ a_nG_n
$$

<br>
<br>

# Vector Commitment
向量承诺用于承诺有 $d$个元素的向量 $\vec u=(u_1,u_2,\cdots,u_d)$，Merkle Commitment 是常用的向量承诺方案， Merkle Commitment只有Commit 和Verify 两个阶段。下面以 $\vec u=[z,k,-,s,n,a,r,k]$ 为例说明Merkle Commitment的流程。

+ **Commit**：     
对于 $\vec u=[z,k,-,s,n,a,r,k]$, Prover 首先将向量的元素作为叶子节点构成一个满二叉树(当元素个数不是 $2^n$的向量，在后面补0到最近的 $2^n$)，然后两两一组做Hash，得到如下图所示的一棵二叉树，图中每一个非叶子节点都是由它的孩子节点的hash构成： $h_1=Hash(h_2,h_3)$， $h_2 = Hash(h_4,h_5)$， $\cdots$。 $h_1$是Merkle Tree的根，也是 $\vec u$ 的commitment。

<br>
<div align=center><img src="https://github.com/zkp-co-learning/ZKP/assets/78890754/51a562b1-37b4-4b06-99c7-e97482b5eb6d"></div>
<br>

+ **Verify**：   
当Prover被要求证明的某元素确实存在于某个位置时，它将仅提供必要的节点，这些节点构成的集合通常被称为Merkle Path, 例如为了证明 $n$是 $\vec u$的第4个元素， Prover将提供Merkle Path  $[s, h_4, h_3]$ (图中蓝色的节点) 给Verifier， Verifier将执行以下操作： $h_5=Hash(s,n)$， $h_2=Hash(h_4, h_5)$， $h'_1=Hash(h_2, h_3)$。Verifier 验证 $h'_1 \overset{?}{=} h_1$，如果它们相等，那么 $n$确实是 $\vec u$中第4个元素。

<br>
<div align=center><img src="https://github.com/zkp-co-learning/ZKP/assets/78890754/d1691b18-96d5-478b-b6ff-7daf8ecec625"></div>
<br>


<br>
<div align=center><img src="https://github.com/zkp-co-learning/ZKP/assets/78890754/30a24454-c8d5-4e61-9e9d-b418fb9ec526"></div>
<br>
<br>


# KZG10 Commitment
多项式承诺可以理解为「多项式」的「承诺」。如果我们把一个多项式表达成如下的公式，

$$
f(X) = a_0 + a_1X + a_2X^2 + \cdots + a_nX^n
$$

那么我们可以用所有系数构成的向量来唯一标识多项式 $f(x)$。

$$
(a_0, a_1, a_2,\ldots, a_n)
$$

我们可以把「系数向量」进行 Hash 运算，得到一个数值，就能建立与这个多项式之间唯一的绑定关系。

$$
C = \textrm{Hash}(a_0, a_1,a_2, \cdots,a_n)
$$

或者，也可以使用 向量版本的 Petersen 承诺。此外，本文后面介绍的IPA commitment，FRI commitment 也可以用于承诺一个多项式。本节我们讨论KZG10 [Kate-Zaverucha-Goldberg'10] 承诺。

KZG10 commitment 是基于双线性配对(Bilinear pairings)的，关于双线性配对的原理可以参考其他的资料，这里不做过多的叙述。<br>
KZG10 承诺分为3个阶段：

+ **Setup**：
1. 产生一个随机数 $\tau$
2. 生成公共参数:

$$
\vec G = (G_0, G_1,\cdots,G_{d-1},H_0, H_1) =(G, \tau G, \tau^2  G, \cdots, \tau^{d-1}G, H,\tau H)
$$
    
其中 $G\in \mathbb{G}_1,H\in\mathbb{G}_2$，并且存在双线性映射 $e\in\mathbb{G}_1 \times \mathbb{G}_2 \rightarrow \mathbb{G}_T$。    
对于双线性群，使用符号 $[1]_1\triangleq G$，$[1]_2\triangleq H$ 表示两个群上的生成元，这样 KZG10 的系统参数（也被称为 SRS, Structured Reference String）可以表示如下： 
    
$$
\mathsf{srs}=([1]_1,[\tau]_1,[\tau^2]_1,[\tau^3]_1,\ldots,[\tau^{n-1}]_1,[1]_2,[\tau]_2)
$$
    
3. 最后也是最重要的，就是删除 $\tau$。如果这个秘密被其他人知道，他们可以伪造证明。在实际执行时，通常有多方参与Setup过程，每个参与方贡献一部分数据组成 $\tau$，只要其中一方是诚实的，删除了自己贡献的数据，那么这个 $\tau$就是可信的，这个过程也被称为Trust Setup Ceremony.     

+ **Commit**：
Prover 将多项式系数 $(a_0,a_1,\cdots,a_{n-1})$和 $(G_0,G_1,\cdots,G_{n-1})$ 逐项相乘，得到commitment $[f(x)]_1$:

$$
\begin{split}
C &=a_0 G_0 + a_1  G_1 + \cdots + a_{n-1} G_{n-1} \\
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
\begin{split}
f(x)-y = q(x)(x-\zeta)  
\end{split}
$$
    
将等式右边展开并将 $q(x)\zeta$移到等式左边，得到

$$
\begin{split}
f(x)-y+q(x)\zeta = q(x)x
\end{split}
$$

对于上式应用双线性映射, Verifier只需要验证

$$
 e([f(x)]_1- [y]_1 + \zeta[q(x)]_1 , [1]_2)  \overset{?}{=} e([q(x)]_1, [x]_2)
$$

<br>
<div align=center><img src="https://github.com/zkp-co-learning/ZKP/assets/78890754/a079b876-8531-47f5-ac90-1ced1f61cfd9"></div>
<br>

## 单个多项式多点打开

Prover 明阶为 $d$ 的函数 $f(x)$经过 $f(\zeta_1)=y_1, f(\zeta_2)=y_2，\cdots, f(\zeta_m)=y_m, m < d$ 个点，只需要验证一次就可以了。
首先构造一个经过 $(\zeta_1, y_1),(\zeta_2,y_2)，\cdots,(\zeta_m,y_m)$ 的辅助函数 $h(x)$。
由于 $(\zeta_1, \zeta_2, \cdots, \zeta_m)$同时为 $f(x),h(x)$的根，因此他们也是 $f(x)-h(x)$的根，故存在一个商多项式:

$$
q(x)=\frac{f(x)-h(x)}{\prod \limits_{i=1}^m (x-\zeta_i)}
$$

Prover 生成$[q(x)]_2$ 并发送给Verifier，Verifier 自己计算 $[h(x)]_1$和 $[\prod \limits_{i=1}^m(x-\zeta_m)]_1$ ，最后再验证

$$
e([f(x)]_1- [h(x)]_1，[1]_2) \overset{?}{=} e([\prod \limits_{i=1}^m(x-\zeta_m) ]_1,[q(x)]_2)
$$

需要说明 $[q(x)]_2$是 $\mathbb{G_2}$上的承诺，因此需要在Setup阶段产生m个 $\mathbb{G_2}$基 $(H, \tau H, \cdots, \tau^mH)$ 。
<br>
<br>

## 多个多项式同一点打开

假如同时使用多个多项式承诺，那么他们的验证操作可以合并在一起完成。即把多个多项式先合并成一个更大的多项式，然后仅通过验证一点，来完成对原始多项式的批量验证。
假设我们有两个多项式 $f_1(x), f_2(x)$，Prover 要同时向 Verifier 证明 $f_1(\zeta)=y_1$和 $f_2(\zeta)=y_2$ ，那么有

$$
\begin{array}{l}
f_1(X) = q_1(X)\cdot (X-\zeta) + y_1\\
f_2(X) = q_2(X) \cdot (X-\zeta) + y_2 \\
\end{array}
$$

通过一个随机数 $\nu$，Prover 可以把两个多项式 $f_1(x), f_2(x)$) 折叠在一起，得到一个临时的多项式 $g(x)$ ：

$$
g(X) = f_1(X) + \nu\cdot f_2(X)
$$

进而我们可以根据多项式余数定理，推导验证下面的等式：

$$
g(X) - (y_1 + \nu\cdot y_2) = (X-\zeta)\cdot (q_1(X) + \nu\cdot q_2(X))
$$

我们把等号右边的第二项看作为「商多项式」，记为 $q(x)$：

$$
q(X) = q_1(X) + \nu\cdot q_2(X)
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
y_g = y_1 + \nu\cdot y_2
$$

从而把多个求值证明相应地折叠成一个，Verifier 可以一次验证完毕：

$$
 e([g(x)]_1- [y_g]_1 + \zeta[q(x)]_1 , [1]_2)  \overset{?}{=} e([q(x)]_1, [x]_2)
$$
