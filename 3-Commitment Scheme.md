本文尝试以尽可能简单的方式介绍常见的commitment scheme, 让读者了解每个commitment scheme 的过程，尽量不涉及复杂的数学知识。
承诺(Commitment)是零知识证明领域的一个重要组件，它以安全的方式将一个值隐藏，并在未来的证明这个值的有效性。
所谓承诺，是对消息「锁定」，得到一个锁定值。这个值被称为对象的「承诺」，例如 $C$ 就是消息 $x$的承诺

$$
C=commit(x)
$$

这个值和原对象存在两个关系，即 Hiding 与 Binding。
+ Hiding： $c$ 不暴露任何关于 $x$的信息；
+ Binding：难以找到一个 $x', x'\neq x$，使得 $C=commit(x')$

最简单的承诺操作就是 Hash 运算。请注意这里的 Hash 运算需要具备密码学安全强度，比如 SHA256, Keccak 等。除了 Hash 算法之外，还有 Pedersen 承诺等。通常承诺有两个阶段组成：

+ **承诺：** Prover 选择一个方案生成承诺，生成的承诺是一个加密的值，原始的信息隐藏在其中。承诺生成阶段的目标是确保隐藏性，即使攻击者无法从承诺中推断出真实的值。
+ **验证：** Prover提供原始信息或者相关的证明给Verifier，Verifer执行验证算法来确认承诺的正确性。

# Pedersen Commitment
Pedersen承诺 是1992年在“Non-Interactive and Information-Theoretic Secure Verifiable Secret Sharing”一文中提出。Pedersen承诺既可以应用于离散对数和椭圆曲线，本文中将以椭圆曲线为例说明。    

**Setup**：     
Prover 随机挑选椭圆曲线上的两个点 $G,H$(称为基点)，并公开。

**Commit**：     
对于消息 $m$, Prover 随机挑选致盲因子 $r$，提交 $C = mG + rH$，$C$是椭圆曲线上的一个点。

**Verify**：   
Prover 发送 $(m,r)$给Verifier，Verifier计算 $C'=mG+rH$，并验证 $C'\overset{\text{?}}{=} C$。

Pedersen Commitment 还可以对多个值进行承诺，对于 $\vec a = (a_0,a_1,\cdots,a_n)$。Prover 随机挑选一系列的基点 $(G_0,G_1,\cdots\,Gn)$, 计算:

$$
 C = a_0G_0+a_1G_1+\cdots+ a_nG_n
$$

<div align=center><img src ="https://github.com/zkp-co-learning/ZKP/assets/78890754/2027ab30-b7a1-4680-8635-2b9b82b7cd17"></div>


# Vector Commitment
向量承诺用于承诺有 $d$个元素的向量 $\vec u=(u_1,u_2,\cdots,u_d)$，Merkle Tree 是常用的矢量承诺方案，对于 $\vec u=[z,k,-,s,n,a,r,k]$ 向量， 它的Merkle承诺如下图中的二叉树所示，图中每一个节点都是由它的孩子节点的hash构成:
- $h_1=H(h_2,h_3)$
- $h_2 = H(h_4,h_5)$
- and so on

<div align=center><img src="https://github.com/zkp-co-learning/ZKP/assets/78890754/51a562b1-37b4-4b06-99c7-e97482b5eb6d"></div>

$h_1$是Merkle Tree的根，也是 $\vec u$ 的commitment。 当Prover被要求证明的某元素确实存在于某个位置时，它将仅提供必要的节点。这些节点构成的集合通常被称为Merkle Path, 例如，为了证明 $n$是 $\vec u$的第4个元素， Prover将提供Merkle Path  $[s, h_4, h_3]$ 给Verifier， Verifier将执行以下操作：

- $h_5=H(s,n)$
- $h_2=H(h_4, h_5)$
- $h'_1 = H(h_2, h_3)$

<div align=center><img src="https://github.com/zkp-co-learning/ZKP/assets/78890754/d1691b18-96d5-478b-b6ff-7daf8ecec625"></div>

Verifier 验证 $h'_1 \overset{?}{=} h_1$，如果它们相等，那么 $n$确实是 $\vec u$中第4个元素。

<div align=center><img src="https://github.com/zkp-co-learning/ZKP/assets/78890754/30a24454-c8d5-4e61-9e9d-b418fb9ec526"></div>







 
