# Лабораторная работа №3: "Метание тел сжатым газом"


## Часть 1 — Решение задачи Лагранжа в Эйлеровых координатах

Задача Лагранжа описывается одномерными нестационарными уравнениями газовой динамики:

$$
\frac{\partial \rho}{\partial t} + \frac{\partial \rho v}{\partial x} = 0,
$$

$$
\frac{\partial \rho v}{\partial t} + \frac{\partial}{\partial x} (\rho v^2 + p) = 0,
$$

$$
\frac{\partial \rho E}{\partial t} + \frac{\partial}{\partial x} (\rho v H) = 0,
$$

$$
E = \varepsilon + v^2/2
$$

с подвижной границей, описываемой уравнением движения 

$$
m \frac{\mathrm{d} v_\mathrm{п}}{\mathrm{d} t} = p_\mathrm{п} S,
$$

где $p_\mathrm{п}$ — давление на дно поршня.

Начальные условия при $t = 0$:

$$\rho = \rho_0, \quad   p= p_0, \quad v = 0, \quad v_\mathrm{п} = 0.$$

В векторном виде эти уравнения можно записать как:

$$
\frac{\partial \vec{q}}{\partial t} + \frac{\partial \vec{f}}{\partial x} = 0,
$$

где

$$
\vec{q} = \left[\begin{matrix}
    \rho \\ \rho v \\ \rho E
\end{matrix}\right];
\quad
\vec{f} = \left[\begin{matrix}
    \rho v  \\ \rho v^2 + p \\ \rho v H
\end{matrix}\right].
$$

Обозначения:
узлы по времени $n$, узлы по координате $k$.
Значение функции в узле $(t_n, x_k)$:

$$
\vec q^n_k = \vec q(x_k,t_n).
$$

Вводится подвижная пространственная сетка.
Положения узлов в каждый момент времени определяются из соотношения
$x^n_k = x^n_p \cdot k/N$.

Явная разностная схема по методу С.К. Годунова: 

$$
\vec q^{n+1}_{k+1/2} =
\frac{\Delta x^n}{\Delta x^{n+1}} \left[
    \vec q^n_{k+1/2} -
    \frac{\tau^n}{\Delta x^n} \left(
        \vec f^n_{k+1} - \vec f^n_{k}
    \right)
\right].
$$

Потоки на границах ячеек $\vec f^n_k$ определяются по схеме В.В. Русанова:

$$
\vec f^n_k =
\frac 1 2 \left[
    \left(
        (1 + \mathrm{M}^n_k) \vec f^n_{k+1/2}
        + (1 - \mathrm{M}^n_k) \vec f^n_{k-1/2}
    \right)
    - c_\mathrm{max} \left(
        (1 + \mathrm{M}^n_k) \vec q^n_{k+1/2}
        - (1 - \mathrm{M}^n_k) \vec q^n_{k-1/2}
    \right)
\right],
$$

где
$\mathrm{M}^n_k = u^n_k/(c_\mathrm{max})^n_k$.


## Часть 2 — Решение задачи Лагранжа в массовых Лагранжевых координатах

Уравнения газовой динамики в массовых Лагранжевых координатах
записываются следующим образом:

$$
\frac{\mathrm{d}}{\mathrm{d} t} \left( \frac{1}{\rho} \right)
- \frac{\partial v}{\partial \xi}
= 0,
$$

$$
\frac{\mathrm{d} v}{\mathrm{d} t}
+ \frac {\partial p}{\partial \xi}
= 0,
$$

$$
\frac{\mathrm{d} \varepsilon}{\mathrm{d} t}
+ p \frac {\partial v}{\partial \xi}
= 0,
$$

где массовая Лагранжева координата $\xi$ представляет собой интеграл

$$
q = \int\limits_{x_1}^{x_2} \rho(x) \, \mathrm{d} x.
$$

Численный метод Неймана:

$$
v^{n+1}_i =
v^n_i
- \tau^n \frac{p^n_{i+1/2}
- p^n_{i-1/2}} {0.5 (\xi_{i+1/2}
+ \xi_{i-1/2}) + \xi_i},
$$

$$
x^{n+1}_i = x^n_i + \tau^n v^{n+1}_i,
$$

$$
\rho^{n+1}_{i+1/2} =
\frac{\xi_{i+1/2}}{x^{n+1}_{i+1} - x^{n+1}_i},
$$

$$
\varepsilon^{n+1}_{i+1/2} =
\varepsilon^n_{i+1/2}
- \tau^n \frac{p^{n+1}_{i+1/2}
+ p^n_{i+1/2}}{2} \frac{v^{n+1}_{i+1}
- v^{n+1}_i}{\xi_{i+1/2}},
$$

$$
p^{n+1}_{i+1/2} =
(k-1) \rho^{n+1}_{i+1/2} \varepsilon^{n+1}_{i+1/2}.
$$
