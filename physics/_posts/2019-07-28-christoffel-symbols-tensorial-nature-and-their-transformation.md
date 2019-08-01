---
layout: post
title: Christoffel Symbols Tensorial Nature and their Transformation
tags: [christoffel, metric, transformation, curvature, tensor, relativity, physics]
categories: [physics]
hidden: true
---
The Christoffel symbols, $\Gamma ^{\mu}_{\nu \sigma}$, are in fact the components of the connection (ie $\nabla$)

$$
\require{cancel}
$$

$$
\begin{equation}
\nabla _{\sigma}\mathrm {e} _{\nu} = \Gamma^{\mu}_{\nu \sigma} \mathrm{e}_ {\mu}
\label{1}
\end{equation}
$$

where $\mathbf{e}_{\nu}$ is a basis for vector fields.

It important to emphasize that the connection itself (ie $\nabla$) is not a tensor for it is not linear in its argument $Y$ as in $\nabla_{X}{Y}$. In order to find out whether the Christoffel symbols are a component of a tensor or not, their transformation law must be derived first:

$$
\begin{equation}
\nabla _{\sigma'}\mathrm {e'} _{\nu} =\Gamma^{\mu'}_{\nu' \sigma'} \mathrm {e} _{\mu'} = \Gamma^{\mu'}_{\nu' \sigma'} \Lambda^{\mu}{}_{\mu'}\mathrm {e} _{\mu} 
\label{2}
\end{equation}
$$

$$
\begin{align*}
\nabla _{\sigma'}\mathrm {e'} _{\nu} =& \nabla_{( \Lambda^{\sigma}{}_{\sigma'} \mathrm{e} _{\sigma} )} {( \Lambda^{\nu}{}_{\nu'} \mathrm {e} _{\nu})} \\
=& \Lambda^{\sigma}{}_{\sigma'} \Lambda^{\nu}{}_{\nu'} \nabla_{\sigma}{\mathrm{e} _{\nu}} + \Lambda^{\sigma}{}_{\sigma'} \nabla_{\sigma}(\Lambda^{\nu}_{\nu'}) \mathrm{e} _{\nu} \\
=& \Lambda^{\sigma}{}_{\sigma'} \Lambda^{\nu}{}_{\nu'} \nabla_{\sigma}{\mathrm{e} _{\nu}} + \Lambda^{\sigma}{}_{\sigma'}\mathrm {e} _{\sigma}(\Lambda^{\nu}_{\nu'}) \mathrm{e} _{\nu}
\label{eq3} \tag{3}
\end{align*}
$$

Both equations are equal, so

$$
\Gamma^{\mu'}_{\nu' \sigma'} \Lambda^{\mu}{}_{\mu'} \mathrm{e} _{\mu} = \Lambda^{\nu}{}_{\nu'} \Lambda^{\sigma}{}_{\sigma'} \Gamma^{\mu}_{\nu \sigma} \mathrm{e} _{\mu} + \Lambda^{\sigma}{}_{\sigma'}  \mathrm{e}_{\sigma} (\Lambda^{\mu}{}_{\nu'}) \mathrm{e}_{\mu}
$$

Multiplying both side with $\Lambda^{\mu'}{}_{\mu}$

$$
\begin{equation}
\Gamma^{\mu'}_{\nu' \sigma'}  =  \Lambda^{\mu'}{}_{\mu} \Lambda^{\nu}{}_{\nu'} \Lambda^{\sigma}{}_{\sigma'} \Gamma^{\mu}_{\nu \sigma} +\Lambda^{\mu'}{}_{\mu} \Lambda^{\sigma}{}_{\sigma'} \mathrm{e}_{\sigma}(\Lambda^{\mu}{}_{\nu'})
\tag{4} \label{4}
\end{equation}
$$

Using a local basis where

$$
\mathbf{e}_{i} = \frac{\partial}{\partial x^{i}}, \quad i=1,2,\dots ,n
$$

$$
\Gamma^{\mu'}_{\nu' \sigma'}  = \frac{\partial x^{\mu'}}{\partial x^{\mu}} \frac{\partial x^{\nu}}{\partial x^{\nu'}} \frac{\partial x^{\sigma}}{\partial x^{\sigma'}} \Gamma^{\mu}_{ \nu \sigma } + \frac{\partial x^{\mu'}}{\partial x^{\mu}} \frac{\partial x^{\sigma}}{\partial x^{\sigma'}} \frac{\partial}{\partial x^\sigma} \left( \frac{\partial x^\mu}{\partial  x^{\nu'}} \right) \Leftrightarrow
$$

$$
\begin{equation}
\tag{5}
\boxed{
\Gamma^{\mu'}_{\nu' \sigma'}  = \frac{\partial x^{\mu'}}{\partial x^{\mu}} \frac{\partial x^{\nu}}{\partial x^{\nu'}} \frac{\partial x^{\sigma}}{\partial x^{\sigma'}} \Gamma^{\mu}_{ \nu \sigma } + \frac{\partial x^{\mu'}}{\partial x^{\mu}} \frac{\partial^2 x^\mu}{\partial x^{\mu'} \partial x^{\nu'}}
}
\label{5}
\end{equation}
$$

Where the last term on the right-hand side was simplified a bit because partial derivatives commute.

It is clear that the first term on the right-hand side transforms like a tensor, but the second inhomogeneous term does not.

Equation $\ref{5}$ has another form which can be derived as follows:

$$
\begin{align*}
\nabla_{\sigma'}V^{\mu'}  =&  \frac{\partial x^\sigma}{\partial x^{\sigma'}}  \frac{\partial x^{\mu'}}{\partial x^{\mu}} \nabla_{\sigma}V^{\mu} \\
=& \frac{\partial x^\sigma}{\partial x^{\sigma'}}  \frac{\partial x^{\mu'}}{\partial x^{\mu}}  \frac{\partial}{\partial x^{\sigma}}V^\mu + \frac{\partial x^\sigma}{\partial x^{\sigma'}}  \frac{\partial x^{\mu'}}{\partial x^{\mu}} \Gamma^{\mu}_{\nu \sigma} V^\nu  \label{6} \tag{6} \\
\end{align*}
$$

Also

$$
\begin{equation}
\tag{7}
\nabla_{\sigma'}V^{\mu'}  =  \frac{\partial x^\sigma}{\partial x^{\sigma'}} \frac{\partial}{\partial x^{\sigma}} \left( \frac{\partial x^{\mu'}}{\partial x^{\mu}} V^\mu \right) + \Gamma^{\mu'}_{ {\nu'} {\sigma'} } \frac{\partial x^{\nu'}}{\partial x^\nu} V^{\nu} 
\label{7}
\end{equation}
$$

Equations \ref{6} and \ref{7} are equal:

$$
\begin{align*}
\frac{\partial x^\sigma}{\partial x^{\sigma'}}  \frac{\partial x^{\mu'}}{\partial x^{\mu}}  \frac{\partial}{\partial x^{\sigma}}V^\mu + \frac{\partial x^\sigma}{\partial x^{\sigma'}}  \frac{\partial x^{\mu'}}{\partial x^{\mu}} \Gamma^{\mu}_{\nu \sigma} V^\nu =& \frac{\partial x^\sigma}{\partial x^{\sigma'}} \frac{\partial}{\partial x^{\sigma}} \left( \frac{\partial x^{\mu'}}{\partial x^{\mu}} V^\mu \right) + \Gamma^{\mu'}_{\nu'\sigma'} \frac{\partial x^{\nu'}}{\partial x^\nu} V^{\nu} \Leftrightarrow  \\
\cancel{\frac{\partial x^\sigma}{\partial x^{\sigma'}}  \frac{\partial x^{\mu'}}{\partial x^{\mu}}  \frac{\partial}{\partial x^{\sigma}}V^\mu} +  \frac{\partial x^\sigma}{\partial x^{\sigma'}}  \frac{\partial x^{\mu'}}{\partial x^{\mu}} \Gamma^{\mu}_{\nu \sigma} V^\nu =&  \frac{\partial x^\sigma}{\partial x^{\sigma'}} \frac{\partial^{2} x^{\mu'}}{\partial x^\sigma \partial x^\mu} V^\mu + \\
& \cancel{\frac{\partial x^\sigma}{\partial x^{\sigma'}} \frac{\partial x^{\mu'}}{\partial x^\mu} \frac{\partial}{\partial x^\sigma} V^\mu} +  \Gamma^{\mu'}_{\nu'\sigma'} \frac{\partial x^{\nu'}}{\partial x^\nu} V^\nu \Leftrightarrow
\end{align*}
$$

$$
\Gamma^{\mu'}_{\nu'\sigma'} \frac{\partial x^{\nu'}}{\partial x^\nu} V^{\nu} = \frac{\partial x^\sigma}{\partial x^{\sigma'}}  \frac{\partial x^{\mu'}}{\partial x^{\mu}} \Gamma^{\mu}_{\nu \sigma} V^\nu - \frac{\partial x^\sigma}{\partial x^{\sigma'}} \frac{\partial^{2} x^{\mu'}}{\partial x^\sigma \partial x^\mu} V^\mu 
$$

Multiplying by $\frac{\partial x^{\nu}}{\partial x^{\nu'}}$ and relabeling the last dummy index on the right-hand side,

$$
\begin{equation}
\tag{8}
\boxed{
\Gamma^{\mu'}_{\nu'\sigma'}  = \frac{\partial x^{\nu}}{\partial x^{\nu'}} \frac{\partial x^\sigma}{\partial x^{\sigma'}}  \frac{\partial x^{\mu'}}{\partial x^{\mu}} \Gamma^{\mu}_{\nu \sigma} - \frac{\partial x^{\nu}}{\partial x^{\nu'}} \frac{\partial x^\sigma}{\partial x^{\sigma'}} \frac{\partial^{2} x^{\mu'}}{\partial x^\sigma \partial x^\nu}
}  
\label{8}
\end{equation}
$$

### Covariant Derivative Transformation

Looking at the definition of the covariant derivative:

$$
\nabla_{\sigma}V^\mu = \frac{\partial V^\mu}{\partial x^\sigma} + \Gamma^{\mu}_{\nu \sigma} V^\nu
$$

Neither one of the terms on the right-hand side above transforms likes the components of a tensor, but their sum does:

Expanding equation $\ref{5}$,

$$
\begin{align*}
\nabla_{\sigma'}V^{\mu'}  =&  \frac{\partial x^\sigma}{\partial x^{\sigma'}} \frac{\partial}{\partial x^{\sigma}} \left( \frac{\partial x^{\mu'}}{\partial x^{\mu}} V^\mu \right) + \Gamma^{\mu'}_{ {\nu'} {\sigma'} } \frac{\partial x^{\nu'}}{\partial x^\nu} V^{\nu} \\
=& \frac{\partial x^\sigma}{\partial x^{\sigma'}} \frac{\partial^{2} x^{\mu'}}{\partial x^\sigma \partial x^\mu} V^\mu + \frac{\partial x^\sigma}{\partial x^{\sigma'}} \frac{\partial x^{\mu'}}{\partial x^\mu} \frac{\partial}{\partial x^\sigma} V^\mu +  \Gamma^{\mu'}_{\nu'\sigma'} \frac{\partial x^{\nu'}}{\partial x^\nu} V^{\nu} 
\end{align*}{}
$$

Using equation $\ref{8}$ to expand $\Gamma^{\mu'}_{\nu' \sigma'}$,

$$
\begin{align*}
\nabla_{\sigma'}V^{\mu'}  =& \cancel{ \frac{\partial x^\sigma}{\partial x^{\sigma'}} \frac{\partial^{2} x^{\mu'}}{\partial x^\sigma \partial x^\mu} V^\mu} + \frac{\partial x^\sigma}{\partial x^{\sigma'}} \frac{\partial x^{\mu'}}{\partial x^\mu} \frac{\partial}{\partial x^\sigma} V^\mu +  \frac{\partial x^{\nu'}}{\partial x^\nu} \frac{\partial x^{\nu}}{\partial x^{\nu'}} \frac{\partial x^\sigma}{\partial x^{\sigma'}}  \frac{\partial x^{\mu'}}{\partial x^{\mu}} \Gamma^{\mu}_{\nu \sigma}V^{\nu}  \\
-& \cancel{ \frac{\partial x^{\nu'}}{\partial x^\nu} \frac{\partial x^{\nu}}{\partial x^{\nu'}} \frac{\partial x^\sigma}{\partial x^{\sigma'}} \frac{\partial^{2} x^{\mu'}}{\partial x^\sigma \partial x^\nu} V^{\nu}} 
\end{align*}
$$

The non-tensorial term in equation $\ref {8}$ cancels out the one from the partials, and the final result indeed transforms like a tensor.

$$
\begin{align*}
\therefore{}    \nabla_{\sigma'}V^{\mu'}  =& \frac{\partial x^\sigma}{\partial x^{\sigma'}} \frac{\partial x^{\mu'}}{\partial x^\mu} \frac{\partial}{\partial x^\sigma} V^\mu +   \frac{\partial x^\sigma}{\partial x^{\sigma'}}  \frac{\partial x^{\mu'}}{\partial x^{\mu}} \Gamma^{\mu}_{\nu \sigma}V^{\nu} \\
 =&  \frac{\partial x^\sigma}{\partial x^{\sigma'}} \frac{\partial x^{\mu'}}{\partial x^\mu} \left( \frac{\partial}{\partial x^\sigma} V^\mu +  \Gamma^{\mu}_{\nu \sigma}V^{\nu} \right)\\
 \frac{\partial}{\partial x^{\sigma'}} V^{\mu'} +  \Gamma^{\mu'}_{\nu \sigma'}V^{\nu} =&  \frac{\partial x^\sigma}{\partial x^{\sigma'}} \frac{\partial x^{\mu'}}{\partial x^\mu} \left( \frac{\partial}{\partial x^\sigma} V^{\mu} +  \Gamma^{\mu}_{\nu \sigma}V^{\nu} \right)  \\
 T_{\sigma'}^{\mu'} =& \frac{\partial x^\sigma}{\partial x^{\sigma'}} \frac{\partial x^{\mu'}}{\partial x^\mu} T_{\sigma}^{\mu} \\
\end{align*}
$$

This resembles the transformation law of a second rank mixed tensor.


In polar coordinates $(r, \theta)$ where $e_r = \frac{\partial}{\partial r}, e_\theta = \frac{\partial}{\partial \theta}$, basic calculation shows that

$$
\begin{equation}
\tag{9}
\Gamma^{\theta}_{r \theta} = \frac{1}{r}
\label{9}
\end{equation}
$$

This is another fact that the symbols $\Gamma ^{\mu}_{\nu \sigma}$ cannot be the components of a tensor for the components vanish in Cartesian coordinates but not in others, such as polar coordinates.

### Is it possible to recover $\ref{9}$ from Cartesian coordinates using the general transformation law?
The answer is yes, in a little complicated way:

Knowing that

$$
\theta(x,y) = \mathrm{arctan}\left(\frac{y}{x}\right) \\
r(x,y) = \sqrt{x^2 + y^2} \\
x(r,\theta) = r \mathrm{cos}(\theta) \\
y(r, \theta) = r \mathrm{sin}(\theta) \\
$$

Using equation $\ref{5}$

$$
\begin{align*}
\Gamma^{\theta}_{r \theta}  =& \cancelto{0}{\frac{\partial x^{\theta}}{\partial x^{\mu}} \frac{\partial x^{\nu}}{\partial x^r} \frac{\partial x^{\sigma}}{\partial x^{\theta}} \Gamma^{\mu}_{ \nu \sigma }} + \frac{\partial x^{\theta}}{\partial x^{\mu}} \frac{\partial^2 x^\mu}{\partial x^{\theta} \partial x^{r}} \\
 =& \frac{\partial \theta}{\partial x}  \frac{\partial^2 x}{\partial \theta \partial r} +  \frac{\partial \theta}{\partial y}  \frac{\partial^2 y}{\partial \theta \partial r} \\
 =& \frac{x  \mathrm{cos}(\theta) + y \mathrm{sin}(\theta)}{x^2 + y^2} \\
 =& \frac{1}{r}
\end{align*}
$$

The first term vanishes since $\Gamma^{\mu}_{ \nu \sigma }$ vanish in Cartesian coordinates. The second term is expanded using Einstein summation convention.
