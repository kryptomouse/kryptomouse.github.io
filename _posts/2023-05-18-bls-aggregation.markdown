---
layout: post
title:  "BLS Signatures (history)"
math: true
categories: [cryptography,signatures]
tags: [cryptography,math,signatures]

---

BLS signatures were introduces in [[BLS01]](https://www.iacr.org/archive/asiacrypt2001/22480516.pdf) by the authors Boneh, Lynn and Shacham. In fact, the name used in their paper is GDH-signatures where GDH stands for _Gap-Diffie-Hellman_. This name describes a property of the underlying group which is assumed to hold for the signature scheme to be proved secure. Put simply, the security of BLS-scheme is based on the GDH assumption.

## Gap Diffie Hellman
Consider a cyclic group $G_1=\langle g_1 \rangle$ of prime order $p$. We say that the adversary solves the [Computational Diffie-Hellman](https://en.wikipedia.org/wiki/Computational_Diffie–Hellman_assumption) (CDH) if on input $(g_1,g_1^a,g_1^b)$, it can output $g^{ab}$ (with non-negligible probability) for random $a,b \in Z_q^*$. Furthermore, upon receiving a tuple $(g_1,g_1^a,g_1^b,g_1^c)$, the adversary solves the so-called [Decisional Diffie-Hellman](https://en.wikipedia.org/wiki/Decisional_Diffie–Hellman_assumption) (DDH) problem if it can correctly decide whether $c=ab$. The only known way to solve CDH is by computing the $log_{g_1}(g_1^a)=a$ (discrete log) and then do $(g_1^b)^a$. And we can solve the DDH problem easily with access to an algorithm solving CDH. 

So a natural question is then; __are there any groups where CDH is hard[^1] but DDH is easy?__

This was the topic of [[JN01]](https://link.springer.com/content/pdf/10.1007/s00145-003-0052-4.pdf), by Joux and Nguyen, who presented a family of groups where CDH (and DL) were hard but DDH was easy to compute and thereby separating CDH and DDH. In fact, [[OP01]](https://www.di.ens.fr/david.pointcheval/Documents/Papers/2001_pkc.pdf) - Okamoto and Pointcheval had already identified the Gap-DH (GDH) among other (gap-problems) and showed that this tractable/intractable asymmetry can be used to build signature schemes with specific properties. So the main contribution of BLS was not so much the idea a signature scheme on top of GDH, but rather the fact that (using the right parameters) these signatures can be made _extremely_ short.

## co-GDH and Bilinear Maps
Before talking about BLS signatures we need to introduce bilinear maps. First, let's define a few other groups $G_2=\langle g_2 \rangle$ and $G_T=\langle g_T \rangle$ to the mix. We call $G_1$ and $G_2$ source groups and $G_T$ the target group and we assume that $|G_1|=|G_2|=|G_3|$.
The bilinear map is then defined as $e:G_1\times G_2 \rightarrow G_T$ with properties 
1. (_Bilinearity_) $\forall a,b\in \mathbb{Z},e(g_1^a,g_2^b)=e(g_1,g_2)^{ab}$
2. (_Non-degeneracy_)   $e(g_1,g_2)\neq 1$.
3. (_Computability_) The mapping $e$ is efficiently computable.

And now that we work over both $G_1$ and $G_2$ we need to introduce natural extensions[^2] of DDH, CDH and GDH.
- Computational Co-Diffie-Hellman: Given $g_2,g_2^a$ and $g_1\in G_1$, compute $g_1^a\in G_1$.
- Decisional Co-Diffie Hellman: Given $(g_1,g_2^a,g_2,g_2^b)$ output __yes__ if $a=b$.

Then, basically co-GDH groups are pairs of groups $(G_1,G_2)$ where computational co-DH is hard but decisional co-DH is easy.

### The Weil Pairing as a DDH Oracle
You probably know where this is going by now :) In BLS signatures the Weil pairing is used as the bilinear map $e$ and it can be used to solve co-DDH efficiently (if $e$ is efficiently computable). Consider the tuple $(g_1,g_2^a,g_2,g_2^b)$. We just check if $e(g_1^a,g_2)=e(g_1,g_2^b)$ and that is our oracle for DDH. Notice that, to establish that this is a pair of gap groups $(G_1,G_2)$, we still have to make sure that the co-CDH problem is hard.

## BLS Signatures
Now we can finally present the BLS signature scheme.

__Key Generation__ A user picks $x\leftarrow \mathbb{Z}_p$ as private key and computes the public key $v\leftarrow g_2^x$.

__Sign__ A user signs a message $M\in {0,1}^*$ by computing $h\leftarrow H(M)$ such that $h\in G_1$ and computes the signature $\sigma\leftarrow h^x\in G_1$.

__Verify__ A verifier can use the public key $v$ of the user and verify that $\sigma$ is a valid signature on $M$ by checking $e(\sigma,g_2)=e(h,v)$.

## Summary
We looked briefly at the history of BLS signatures. BLS is a signature scheme with security based on the co-Gap DH assumption. Due to the structure of the curves involved, the scheme can allow for very short signatures [(~48 bytes)](https://datatracker.ietf.org/doc/html/draft-irtf-cfrg-bls-signature-00#section-1.1). There are plenty of additional things to explore in the realm of BLS signature. Perhaps the most compelling feature is that the scheme is so __aggregation-friendly__. But that is a topic for another post.


[^1]: We are being very informal about the words "easy" and "hard". To be slightly more precise. Easy means computable in polynomial time while hard means not computable in polynomial time.

[^2]: If the two source groups are the same $G_1=G_2$ (aka. Type-1) then the generalization is not necessary. 



<!-- Yay - first blog post of the year 2023!  
We start with the basics of secret sharing before diving into more exotic stuff. The aim is cover the basics in this post and dive into [[CDI05]](https://iacr.org/archive/tcc2005/3378_342/3378_342.pdf) or [[Maurer02]](https://crypto.ethz.ch/publications/files/Maurer02b.pdf) in the coming weeks.  

## Motivation
Inspired by Oded Goldreich's [My Choices](https://www.wisdom.weizmann.ac.il/~oded/my-choice.html "Oded's blog")[^1], I have decided to kick the year off by researching some papers that I have never fully dug into but I have seen referenced countless of times during my research. However, to fully appreciate the specific techniques employed, we will in this post revisit some basic concepts.

## Basics
**:warning: Disclaimer: You should not be reading the following; you should first/instead visit Beimel's timeless [survey](https://www.cs.bgu.ac.il/~beimel/Papers/Survey.pdf) on Secret Sharing  Schemes.**

Before looking at LSSS we need to talk about <u>access structures</u>. These are important because they define which sets of parties in the system that together can access the secret. 
### Access Structures
Let $\mathcal{P}=\{\mathsf{P}_1,\ldots,\mathsf{P}_n\}$ be the set of parties.
An access structure $\Gamma$ has the following properties
- $\Gamma\subseteq 2^{\mathcal{P}}$ *i.e.* it is a subset of the power-set over all parties.
- $\Gamma$ is monotone meaning if $B\in\Gamma$ and $B\subseteq C$ then $C\in\Gamma$ 

We call sets in $\Gamma$ *qualified* sets while the members of $\Sigma=2^{\mathcal{P}}\setminus \Gamma$ (secrecy structure) are *unqualified*.
Indeed, the monotonicity property essentially means that if set of parties are qualified then any super-set is also qualified.
Conversely, if a set of parties is unqualified then any subset should also be unqualified.

#### Threshold Access Structures
A special case is the simple $t$-out-of-$n$ access structure $\Gamma_{t,n}$ where any subset of size greater than $t$ collectively have access to the secret.
$\Gamma_{t,n}=\\{A\subseteq 2^{\mathcal{P}}:|A|>t\\}$.

### Secret Sharing Schemes
A secret sharing scheme $\Pi$ consist of a sharing algorithm $\mathsf{Share}$ which takes as input a secret $s$ in some secret space $\mathcal{K}$ and some random string $r$ sampled from domain $R$ according to some prescribed distribution and outputs shares in $\mathcal{K}_1\times\dots\times\mathcal{K}_n$. Conversely, the reconstruction algorithm for a qualified set $A\in\Gamma$ can be written as a function $\mathsf{Recon}_A:\mathcal{K}_1\times\dots\times\mathcal{K}_n\rightarrow \mathcal{K}$. Very informally, *correctness* of $\Pi$ guarantees that any qualified set can collectively, by using the shares dealt by $\mathsf{Share}$, obtain $s$ by applying $\mathsf{Recon}$. While, *security* claims that to any unqualified set, all values of $s$ are equiprobable in $\mathcal{K}$.   

#### Linear Secret Sharing
LSSS is a special class of secret sharing schemes based on linear algebra. In short, these schemes are based on an underlying finite field $\mathbb{F}$ such that both the secret space, share space and the randomness are related to $\mathbb{F}$. Importantly, we let the random string $r$ be a vector where each coordinate is an independent and uniformly chosen element in $\mathbb{F}$ such that $\mathsf{Share}$ is now a fixed *linear* function combining the secret $s$ and $r$ and outputting the shares.   

<details markdown="1"><summary>LSSS and Minimal Span Programs...</summary>
The study of minimum share size necessary while keeping efficient (time polynomial in the number of parties $n$) $\mathsf{Share}$ and $\mathsf{Recon}$ algorithms is related to a concept in complexity theory called Monotone Span Programs (MSP). In fact, an MSP for specific access structure $\Gamma$ implies a linear secret sharing scheme for the same access structure [[KW93]](https://www.math.ias.edu/~avi/PUBLICATIONS/MYPAPERS/KW93/proc.pdf). This relationship is used to show relatively strong lower bounds on the information ratio (share size) for LSSS where the corresponding lower bounds proved for general access structures are seemingly weaker.
</details>

### Shamir's Secret Sharing
The most famous secret sharing scheme is arguably Shamir's secret sharing. The concept was introduced independently by both [[Shamir79]](https://dl.acm.org/doi/pdf/10.1145/359168.359176) and [[Blakley79]](https://services10.ieee.org/idp/SSO.saml2). The scheme is limited to threshold access structures $\Gamma_{t,n}$. This is a fairly well-known scheme so we will describe it briefly: 
- $\mathsf{Share}$. Sample a polynomial $P\in \mathbb{F}[X]$ uniformly random subject to $P(0)=s$ and deg$(P)=t$ the resulting share vector is $(P(\alpha_1),\ldots,P(\alpha_n))\in\mathbb{F}^n$ where $\alpha_1,\ldots,\alpha_n$ are public predefined values.
- $\mathsf{Recon}$. Simply, do [Lagrange's interpolation](https://en.wikipedia.org/wiki/Lagrange_polynomial) on shares from the qualified set $A$.

*Security* comes from the fact that any $t$ unique points on the polynomomial (from an unqualified set) will render all values of $s$ equiprobable, while *correctness* ensures that any set of $t+1$ shares (qualified set) uniquely defines the polynomial $P$ and hence also $P(0)=s$.  
> **Cheesy Tip.** Shamir's secret sharing is a testament to the fact that the threshold access structures $\Gamma_{t,n}$ are <u>ideal</u>. Essentially, this means that the individual share size is optimal and equal to size of the secret.
{: .prompt-warning }

### CNF Secret Sharing
The concept of secret sharing for general access structures was first put forth by [[ISN89]](https://archiv.infsec.ethz.ch/education/as09/secsem/papers/ItSaNi87.pdf) and generalized to access structured described by [monotone formulae](https://en.wikipedia.org/wiki/Monotonic_function#In_Boolean_functions) by [[BL90]](https://viterbi-web.usc.edu/~shanghua/teaching/Fall2014-476/BenalohLeichter.pdf) who also showed that access structures described by small monotone formulae admits an efficient perfect secret sharing scheme.

The scheme for realizing secret sharing for general access structures are often referred to as CNF secret sharing[^2] or replicated secret sharing because the same values appears in multiple shares. First consider the maximal sets of the unqualified sets $\Sigma$. We can enumerate them as $\mathcal{T}=\{T_1,\ldots,T_k\}$. 

- $\mathsf{Share}$. To share a secret $s\in \mathbb{F}$, first sample $\lbrace r_T\rbrace\_{T\in \mathcal{T}}$ uniformly at random from $\mathbb{F}$ subject to $s=\sum_{T\in\mathcal{T}} r_T$ (obtain a $k$-out-of-$k$ additive secret sharing of $s$). Let $s_i$ be the vector $s_i=\( r_T \)\_{T\not\owns i}$.
- $\mathsf{Recon}$. Let $A\in \Gamma$ be a qualified set. Together, the parties in $A$ will hold all shares $\lbrace r_T\rbrace\_{T\in \mathcal{T}}$ and can reconstruct the secret by $s=\sum_{T\in\mathcal{T}} r_T$. 

*Security* is given since any maximally unqualified set $T$ will miss exactly one additive share, namely $r_T$. *Correctness* is ensured because any qualified set $A\in\Gamma$ cannot be contained in any unqualified set (due to the monotonicity of $\Gamma$) so parties in $A$ must jointly have all shares $r_T$.  
![Dark mode only](/assets/img/pblack.png){: .dark }
![Light mode only](/assets/img/pwhite.png){: .light }
_CNF-Secret Sharing for access structure $\Gamma\_{1,4}$_

## Conclusion
So, we have covered the basics of secret sharing. We got to know the most common secret sharing schemes; Shamir's and CNF secret sharing. We got a feeling of how a question such as: 

>"Does a *cruel* access structure $\Gamma_{\mathsf{evil}}$ for $n$ parties exist, such that *every* secret sharing scheme has to distribute shares of exponential size in $n$?"

is deeply related to complexity theoretic concepts like Monotone Span Programs. However, we only scratched the surface. We did not even get into the relation between Shamir's secret sharing and [Reed-Solomon codes](https://en.wikipedia.org/wiki/Reed–Solomon_error_correction) or the relation between ideal access structures with efficient LSSS and mathematical objects called matroids (read more [here](https://web.mat.upc.edu/carles.padro/arc02v03.pdf)).  

[^1]: Oded Goldreich has chosen to focus on current and impactful (although) lesser known works within cryptography/complexity theory. My intention is to focus on techniques/papers that are close to my field of research and where I can extract small nuggets of wisdom as opposed to doing a full paper review.

[^2]: This is somewhat of a misnomer since CNF circuits contain **NOT**-gates which are explicitly forbidden in monotone formulae since they compute a monotone function that decides if a set is in the access structure (qualified) or not (unqualified).  -->

