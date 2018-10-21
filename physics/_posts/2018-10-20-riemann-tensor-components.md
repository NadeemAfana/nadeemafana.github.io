---
layout: post
title: Riemann Curvature Tensor Components
tags: [riemann, curvature, tensor, relativity, physics]
categories: [physics]
hidden: true
---
 To calculate the number of unique components of the Riemann curvature tensor, we first exploit the symmetry and antisymmetry of the first and last two indices. Before that, a quick review of combinatorics is essential in the process.
$\def\multiset#1#2{\left(\kern-.3em\left(\genfrac{}{}{0pt}{}{#1}{#2}\right)\kern-.3em\right)}$

$$
\binom{n}{k} = \text{Number of independent components for an }\textbf{antisymmetric} \text{ matrix.}
$$

$$
\multiset{n}{k} = \binom{n + k -1}{k} = \text{Number of independent components for a } \textbf{symmetric} \text{ matrix.}
$$

where $n$ is the number of dimensions (eg $4$), and $k$ is the group rank.
 For example, the metric tensor $g_{\mu \nu}$ has $\multiset{4}{2} = 10$ unique components in $4$ dimensions.

$R_{\alpha \beta \mu \nu}$ has $4^4 = 256$ total components without considering any symmetry or antisymmetry. However, $R_{\alpha \beta \mu \nu}$ is antisymmteric on the first and second pair of indices (eg $R_{\alpha \beta \mu \nu} = - R_{\beta \alpha \mu \nu}$) and is symmetric on the two pairs (eg $R_{\alpha \beta \mu \nu} =  R_{\mu \nu \alpha \beta}$) and thus
$$
\begin{equation}
\tag{1}
R_{\alpha \beta \mu \nu} = -R_{\beta \alpha  \mu \nu} = -R_{\alpha \beta \nu \mu} = R_{\mu \nu \alpha \beta} \label{1}
\end{equation}
$$

$$
\begin{equation}
\tag{2}
R_{\alpha \beta \mu \nu} + R_{\alpha \nu \beta \nu} + R_{\alpha \mu \nu \beta} = 0 \label{2}
\end{equation}
$$

Each of the antisymmetries in $\alpha \beta$ and $\mu \nu$ has $\binom{n}{2}=6$ unique components in $4$ dimensions, and they are $10$, $12$, $13$, $23$, $03$ and $02$.
The symmetry in the two pairs, $R_{\alpha \beta \mu \nu} = R_{\mu \nu \alpha \beta}$ (eg $R_{1012} = R_{1210}$) combined with the antisymmetry reduce the number of components from $256$ to

$$
\multiset{\binom{n}{2}}{2} = \multiset{\frac{n(n-1)}{2}}{2} = \frac{1}{8}(n^4-2n^3 +3n^2 -2n)
$$ 
which is $21$ in $4$ dimensions.


Equation \ref{2} can be re-written as 
$$
\begin{equation}
\tag{2*}
\label{eq:2*}
R_{\alpha [\beta \mu \nu]} = \frac{1}{3}(R_{\alpha \beta \mu \nu} + R_{\alpha \nu \beta \nu} + R_{\alpha \mu \nu \beta}) = 0
\end{equation}
$$

I used symmetry and antisymmetry to reduce $\frac{1}{6}(...)$ to \eqref{eq:2*}.
Now $[\beta \mu \nu]$ has $\binom{n}{3} = 4$ equations in $4$ dimensions, but $\alpha[\beta \mu \nu]$ are not $4 \cdot4$ because if $\alpha$ is equal to any of $\{\beta, \mu, \nu\}$ then $\alpha[\beta \mu \nu]$ becomes equation \ref{1}. For example, if $\alpha=\mu$,

$$
R_{\mu[\beta \mu \nu]} = R_{\mu \beta \mu \nu} + R_{\mu \mu \nu \beta} + R_{\mu \nu \beta \mu} = 0   \quad \text{(Used symmetry and antisymmetry of \ref{1})}
$$
The only cases missing are when $\alpha \ne \{ \beta, \mu, \nu \}$, but these equations are not independent either. For example, in 4 dimensions, $R_{\alpha[\beta \mu \nu]}$ is

$$
\begin{align*} 
R_{0[123]} =& R_{0123} + R_{0231} + R_{0312} \nonumber \label{eq3} \tag{3} \\
R_{1[032]} =& R_{1032} + R_{1320} + R_{1203} \nonumber \label{eq4} \tag{4} \\
R_{2[013]} =& R_{2013} + R_{2130} + R_{2301} \nonumber \label{eq5} \tag{5} \\
R_{3[012]} =& R_{3012} + R_{3120} + R_{3201} \nonumber \label{eq6} \tag{6}
 \end{align*}
$$
These 4 equations are all linearly dependent, for example, for $\eqref{eq3}$ and $\eqref{eq6}$:

$$
\begin{align*} 
R_{3012} =& -R_{0312} \\
R_{3120} =& R_{2031} = -R_{0231} \\
R_{3201} =& R_{0132} = -R_{0123}
 \end{align*}
$$

All the values of $\alpha, \beta, \mu$ and $\nu$ must be different. In $4$ dimensions, these equations are only $1$. 

Also, $\alpha$ is redundant in $R_{\alpha[\beta \mu \nu]}$. In fact,  $R_{\alpha[\beta \mu \nu]} = R_{[\alpha \beta \mu \nu]} = 0$.

With the aid of Mathematica:
```
Array[Subscript[R, #1, #2, #3, #4] &, {4, 4, 4, 4}] /. {1 -> \[Alpha],
   2 -> \[Beta], 3 -> \[Mu], 4 -> \[Nu]}

proj = Symmetrize[%, 
  Antisymmetric[{1, 2, 3, 4}]]; SymmetrizedArrayRules[proj]
```

$$
\begin{equation*}
    \begin{split}
     R_{[\alpha \beta \mu \nu]} = \frac{1}{4!} \left(\right. &R_{\alpha \beta \mu \nu } -  R_{\alpha \beta \nu \mu } - R_{\alpha \mu \beta \nu } + R_{\alpha \mu \nu \beta } + R_{\alpha \nu \beta \mu } - R_{\alpha \nu \mu \beta } - R_{\beta \alpha \mu \nu }  + \\  &R_{\beta \alpha \nu \mu } + R_{\beta \mu \alpha \nu } - R_{\beta \mu \nu \alpha }-R_{\beta \nu \alpha \mu } + R_{\beta \nu \mu \alpha } + R_{\mu \alpha \beta \nu } -  R_{\mu \alpha \nu \beta } - \\
     &R_{\mu \beta \alpha \nu } + R_{\mu \beta \nu \alpha } + R_{\mu \nu \alpha \beta } - R_{\mu \nu \beta \alpha } - R_{\nu \alpha \beta \mu } + R_{\nu \alpha \mu \beta } +  R_{\nu \beta \alpha \mu } - \\ 
    &R_{\nu \beta \mu \alpha } - R_{\nu \mu \alpha \beta } + R_{\nu \mu \beta \alpha}\left. \right)
 \end{split}
\end{equation*}
$$

Using symmetry and antisymmetry of the first and last two indices:
$$
  R_{[\alpha \beta \mu \nu]} =\frac{1}{3}(R_{\alpha \beta \mu \nu} + R_{\alpha \nu \beta \nu} + R_{\alpha \mu \nu \beta}) =  R_{\alpha [\beta \mu \nu]} = 0 \quad  \text{eq. \eqref{eq:2*}}
$$

The total number of unique components in $n$ dimensions is:
$$
\frac{1}{8}(n^4-2n^3 +3n^2 -2n) - \binom{n}{4} = \boxed{ \frac{n^2(n^2-1)}{12} }
$$