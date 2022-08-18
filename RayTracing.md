直线（line）公式：

m为斜率，b为y轴上下偏移量

$$
y = mx + b
$$

射线（ray）公式：

o点到p点/p‘的距离为：o点坐标 + 方向（d）*距离（t）

$$
p_{xy} = a_{xy} + b_{xy}t
$$

***

圆公式 :

(a,b)-->圆心

$$
(x-a)^2 +(y-b)^2 = r^2
$$

****

将ray公式（a+bt）带入圆公式 ($x^2+y^2-r^2=0$)：

（$a_x+b_xt$）带入x，（a_y+b_yt）带入y

得到：$(a_x+b_xt)^2+(a_y+b_yt)^2-r^2=0$

展开：${a_x}^2 + 2a_xb_x+{b_x}^2t^2 +{a_y}^2 + 2a_yb_y+{b_y}^2t^2-r^2=0$

一元二次方程式：

$$
ax^2+bx+c=0
$$

合并同类项得到：

$$
({b_x}^2 + {b_y}^2)t^2 + 2(a_xb_x+a_yb_y)t + (a_x^2+a_y^2-r^2)=0

$$

- [x] 已知 a = ray origin

- [x] 已知 b = ray direction

- [x] 已知 r = (circle/sphere) radius

- [ ] 求解 t = hit distance

****

求根公式：

$$
x = \frac{-b\pm \sqrt{b^2 - 4ac}}{2a}
$$

根据：

$({b_x}^2 + {b_y}^2)t^2 + 2(a_xb_x+a_yb_y)t + (a_x^2+a_y^2-r^2)=0$

得出：

a = ${b_x}^2 + {b_y}^2$  == `dot(b)` == `dot(rayDirection);`

b = $2(a_xb_x+a_yb_y)$ == `2 * dot(a,b)` == `2.0f * dot(rayOrigin,rayDirection);`

c = $a_x^2+a_y^2-r^2$ == `dot(a)-r*r` == `dot(rayOrigin)-r*r;`



判别式（discriminant）:$ {b^2 - 4ac}$

当结果 >0时，有两个解（有两个交点）

            =0时，有一个解（一个交点）

            <0时，没有解（没有交点）
