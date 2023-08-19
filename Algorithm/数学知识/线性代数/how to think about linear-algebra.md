# 矩阵运算的实质

- 我们先来看
$$AX = b$$

$$

假设A=\begin{equation}
 \left[
 \begin{array}{ccc}
     2 & 5 \\
	   1 & 3 \\
 \end{array}
 \right] 
 ,X=
 \left[
 \begin{array} {ccc}
 x \\
 y
 \end{array}
\right]  
,b=
\left[
 \begin{array} {ccc}
 1 \\
2 
 \end{array}
\right] 
 \end{equation}
$$
此时相当于两个二元方程求解
$$

	\begin{equation}
 \left\{
 \begin{array}{c}
2x+5y=1\\
1x+3y=2
 \end{array}
 \right.
 \end{equation}
$$
也相当于两个二维向量求解

$$
x*
\begin{equation}
\left[
\begin{array}{ccc}
2 \\
1
\end{array}
\right]
+
y*
\left[
\begin{array}{ccc}
5\\
3

\end{array}

\right]
=
\left[
\begin{array}{ccc}
1\\
2
\end{array}
\right]
\end{equation}
$$

- **总结** 矩阵的乘法相当于对于一组或多组方程求解，也相当于对一个或多个向量求解




