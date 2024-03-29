# Лабораторная работа №2: "Методы численного расчёта течений в ударной трубе"

В данной лабораторной работе рассматривается решение задачи об ударной трубе.
Имеется бесконечной длины труба, в некотором месте которой расположена непроницаемая перегородка.
В левой части трубы газ находится в одном состоянии, в правой — в другом.
Либо в двух частях трубы находятся два разных газа, что сути не меняет.
В некоторый (начальный) момент времени перегородка мгновенно исчезает.
Что при этом произойдёт?
Что за процесс начнётся и как он будет развиваться?
В данной работе вам предстоит разобраться с этими вопросами.

Однако начнём несколько издалека — с численного решения обыкновенных дифференциальных уравнений, коими живёт газовая динамика да и физика в целом.


## Численное интегрирование обыкновенного дифференциального уравнения: явный метод Эйлера

Рассмотрим обыкновенное дифференциальное уравнение (ОДУ) вида

$$
\frac{\mathrm{d} s}{\mathrm{d} t} = \dot s(t) = f(s, t).
$$

Ничего экстраординарного.
Пусть требуется проинтегрировать его по времени $t$ в пределах от $t_0$ до $t_\mathrm{к}$.
При этом для физической величины $s$ в начальный момент времени $t_0$ нам известно начальное условие (или её распределение):
$s(t_0) = s^0$ (почему индекс проставлен сверху, будет сказано далее).
Правая часть $f(s, t)$ — это некоторая известная функция, зависящая в общем случае как от времени, так и от искомой величины $s$.

Найдём приближённое решение в момент $t_1 = t_0 + \Delta t$ (вспоминаем разложение функций в ряд), отбросив члены второго и большего порядка малости:

$$
s(t_0 + \Delta t) =
s(t_0)
+ \Delta t \cdot \dot s(t_0) + O(\Delta t^2).
$$

Заменяя $\dot s$ на $f(s, t)$, получим

$$
s(t_1) =
s(t_0) +
\Delta t \cdot f\left( s(t_0), t_0 \right).
$$

Выполняя этот процесс итерационно, найдём общее соотношение:

$$
s(t_{n+1}) =
s(t_n)
+ \Delta t \cdot f\left( s(t_n), t_n \right),
$$

$$
t_{n+1} = t_n + \Delta t.
$$

Полученное уравнение можно записать в несколько более ёмком виде:

$$
s^{n+1} = s^n + \Delta t \cdot f(s^n, t_n).
$$

Верхний индекс отвечает за номер шага по времени.

```{note}
Нижние индексы мы будем использовать для номеров шагов по пространству (координатам).
То, что в обозначении времени $t_n$ индекс стоит внизу, никак с принятыми обозначениями не связано: время никак не связано с координатами, поэтому двойной индексации наподобие $t^n_i$ у него быть не может.
```

Здесь $s^{n+1}$ явно зависит от $s^n$, следовательно, и метод легко назвать **явным**.
Данный метод легко реализовать в коде.


### Пример: остывание чашки кофе

Скорость изменения температуры кофе в чашке определяется таким ОДУ:

$$
\frac{\mathrm{d} T}{\mathrm{d} t} = -r (T - T_\mathrm{окр}),
$$ (eq_T_coffee)

где
$T_\mathrm{окр}$ — температура окружающей среды;
$r$ — коэффициент "остывания" (теплоотдачи).

Начальную температуру кофе обозначим как
$T^0 = T(t=0)$.

Численное решение этого уравнение явным методом Эйлера
выглядит следующим образом:

$$
T^{n+1} = T^n - \Delta t \cdot r \cdot (T^n - T_\mathrm{окр}).
$$ (eq_T_coffee_euler)

Проинтегрировав {eq}`eq_T_coffee` по схеме {eq}`eq_T_coffee_euler`, получим решение $T(t)$, графически изображённое на рисунке ниже.

![Остывание чашки кофе](pics/T_coffee.png)

Стоит отметить, что точность решения зависит от числа разбиений отрезка интегрирования $N$ (в данном случае отрезка времени $t$).
В пределе при $N \rightarrow \infty$ или (что то же самое) при $\Delta t \rightarrow 0$ численное решение совпадает с точным.


## Уравнение переноса: физический и математический смысл

Уравнение переноса для некоторой физической величины $s(t, \vec x)$, зависящей от времени $t$ и координаты $\vec x = (x, y, z)$ имеет следующий вид:

$$
\frac{\mathrm{d} s(t, \vec x)}{\mathrm{d} t} = 0.
$$ (eq_s_material_diff)

Такое уравнение может описывать, например,
концентрацию пыли в потоке воздуха.

Поскольку величина $s$ зависит как от времени, так и от пространства, то производная в {eq}`eq_s_material_diff` является субстанциональной.
То есть уравнение на данный момент записано в постановке Лагранжа.
Раскрыв её, получим:

$$
\frac{\partial s}{\partial t}
+ \vec u \cdot \nabla s =
0.
$$

Дифференциальный оператор $\nabla$ должен быть вам знаком.
Будучи применённым к скалярной функции $s$, он означает вычисление её градиента.

$$
\nabla s =
\frac{\partial s}{\partial x} \vec i
+ \frac{\partial s}{\partial y} \vec j
+ \frac{\partial s}{\partial z} \vec k,
$$

где
$\vec i$, $\vec j$, $\vec k$ — орты системы отсчёта.

Скалярное произведение поля скорости $\vec u = (u, v, w)$ и градиента $s$ даёт следующее:

$$
\vec u \cdot \nabla s =
u \frac{\partial s}{\partial x}
+ v \frac{\partial s}{\partial y}
+ w \frac{\partial s}{\partial z}.
$$

Тогда в частном одномерном случае при потоке, например, вдоль оси $x$ имеем:

$$
\frac{\partial s}{\partial t}
+ u \frac{\partial s}{\partial x} =
0.
$$ (eq_advection_euler)

Это уравнение переноса некоторой величины $s$ вдоль оси $x$ со скоростью $u$, записанное в постановке Эйлера.

Никого не удивишь тем, что для решения этого уравнения требуется задать начальное условие для $s$.
Пусть начальное условие будет следующим: $s(t=0) = s^0 = q(x)$.
Здесь начальное распределение $q(x)$ может описываться, к примеру, ступенчатой функцией:

$$
q(x) = \begin{cases}
    0, & x < 0, \\
    1, & 0 \le x \le 1, \\
    0, & x > 1.
\end{cases}
$$ (eq_q_step)

Граничных условий нет — пусть прямая $x$ будет бесконечна.
Пыль будет лететь в бесконечность вместе с потоком воздуха.

Одномерное уравнение переноса {eq}`eq_advection_euler` имеет известное точное решение:

$$
s(t, x) = q(x - u t).
$$

Это просто параллельный перенос начального распределения $q(x)$ вдоль оси $x$ со скоростью $u$.
Проще говоря, представьте, что наша "ступенька" {eq}`eq_q_step` с течением времени просто перемещается по оси $x$ со скоростью $u$.
Только и всего.

<!-- TODO -->
<!-- На таком простом примере с известным точным решением рассмотрим очень важный аспект численных методов — _численная (или искусственная) вязкость_. -->


### Дискретизация уравнения переноса

Введём равномерную разностную сетку по координате
$x_0, x_1, \ldots, x_N$, причём

$$
x_1 - x_0 = x_2 - x_1 = \ldots = x_N - x_{N-1} = \Delta x.
$$

Количество узлов сетки $N = (x_N - x_0) / \Delta x$.

Искомая сеточная функция определена в каждой точке
дискретной разностной сетки:
$s_k = s(x_k)$ (зависимость от времени здесь опущена, но подразумевается).
Нижний индекс отвечает за номер шага по координате (пространству).

Теперь полноценное обозначение выглядит так: $s_k^n$ — $n$-ый шаг по времени и $k$-ый шаг по координате.


### Аппроксимация производных конечными разностями

Для простоты рассмотрим зависимость функции $s$ только от координаты.
Все последующие рассуждения будут верны и для зависимости от времени.

Разложим функцию $s(x)$ в ряд Тейлора:

$$
s(x + \Delta x) =
    s(x)
    + \Delta x \cdot s'(x)
    + \frac{\Delta x^2}{2!} s''(x)
    + \frac{\Delta x^3}{3!} s'''(x)
    + \ldots,
$$

$$
s(x - \Delta x) =
    s(x)
    - \Delta x \cdot s'(x)
    + \frac{\Delta x^2}{2!} s''(x)
    - \frac{\Delta x^3}{3!} s'''(x)
    + \ldots.
$$

С математической точки зрения при $\Delta x \rightarrow 0$ эти два разложения одинаковы.
Но с точки зрения конечно-разностной схемы аппроксимации производных — нет.

Из первой формулы можно получить так называемую _разность вперёд_:

$$
s'(x) =
\frac{s(x + \Delta x) - s(x)}{\Delta x}
- \frac{1}{2} \Delta x \cdot s''(x)
- \ldots.
$$ (eq_diff_forward)

Из второй формулы – _разность назад_:

$$
s'(x) =
\frac{s(x) - s(x - \Delta x)}{\Delta x}
+ \frac{1}{2} \Delta x \cdot s''(x)
- \ldots.
$$ (eq_diff_backward)

Вычитая эти формулы, получим _центральную разность_:

$$
s'(x) =
\frac{s(x + \Delta x)
- s(x - \Delta x)}{2 \Delta x}
- \frac{1}{6} \Delta x^2 s'''(x)
+ \ldots.
$$

Как было сказано выше, конечно-разностные схемы совсем не то же самое, что математическая производная.
Посмотрим, как это выражается на примере уравнения переноса.


### Разностные схемы уравнения переноса

Без деталей рассмотрим некоторые конечно-разностные схемы для уравнения переноса.

Разность вперёд и по времени и по координате:

$$
\frac{{\color{purple} s_i^{n+1}} - s_i^n}{\Delta t}
+ u \frac{s_{i+1}^n - s_i^n}{\Delta x}
= 0.
$$

```{warning}
Эта схема оказывается всегда неустойчивой.
```

Разность "вперёд" по времени и "назад" по координате:

$$
\frac{{\color{purple} s_i^{n+1}} - s_i^n}{\Delta t}
+ u \frac{s_i^n - s_{i-1}^n}{\Delta x}
= 0.
$$

Данная схема может быть устойчивой при соблюдении условия Куранта-Фридрихса-Леви (о нём ниже).

Неявная схема:

$$
\frac{{\color{purple} s_i^{n+1}} - s_i^n}{\Delta t}
+ u \frac{{\color{purple} s_i^{n+1}} - {\color{blue} s_{i-1}^{n+1}}}{\Delta x}
= 0.
$$

Неявная схема сложнее в реализации, но она безусловно устойчива.


### Об устойчивости конечно-разностных схем

Основной вопрос, связанный с любым численным методом, заключается в том, стабилен ли он: не накапливаются ли ошибки лавинообразно?
Если метод безусловно устойчив, то это значит следующее: независимо от того, насколько велик шаг $\Delta t$, решение не "взрывается".
Безусловно устойчивые схемы очень привлекательны.
Мы можем выбрать временной шаг исключительно на основе компромисса между точностью и скоростью вычислений.

На практике можно получать странные результаты, если выбрать слишком большой шаг $\Delta t$.
Существуют различные _эмпирические_ правила.
Например, можно ограничить $\Delta t$ так, чтобы переносимая частица (пыли, жидкости и т.п.) преодолевала за это время не более заданного числа ячеек сетки (пяти, к примеру):

$$
\Delta t \le \frac{5 \Delta x}{u_\mathrm{max}},
$$ (eq_time_step)

где
$u_\mathrm{max}$ — максимальная скорость среди всех узлов сетки.

При таком временном шаге в некоторых случаях артефакты численного расчёта всё равно будут присутствовать.
Можно снизить вероятность возникновения заметных ошибок, например, рассмотрением не 5 ячеек, а всего одной, то есть

$$
\|u(x)\| \Delta t < \Delta x.
$$


### Условие Куранта-Фридрихса-Леви (CFL)

Прежде чем оставить тему размеров временных шагов, более подробно рассмотрим то, что называется условием Куранта-Фридрихса-Леви (CFL).
В литературе существует некоторая путаница относительно того, что именно оно собой представляет.

Условие CFL, названное в честь математиков-прикладников Р. Куранта, К. Фридрихса и Х. Леви, представляет собой простое и интуитивно понятное необходимое условие _сходимости_.
_Сходимость_ означает, что если вы повторяете расчёт (моделирование) со все меньшими и меньшими шагами $\Delta t$ и $\Delta x$, в пределе, стремящимися к нулю, то численное решение должно приближаться к точному решению.

Решение $s(t^*, \vec x^*)$ зависящего от времени уравнения в частных производных, такого как уравнение переноса, в конкретной точке пространства $\vec x^*$ и времени $t^*$ зависит от части или всех начальных условий.
То есть, если мы изменим начальные условия в некоторых местах и снова решим задачу, она изменится.
В других местах изменения могут не иметь эффекта.
В случае уравнения _адвекции_ (переноса) с постоянной скоростью $u$ значение $s(t^*, \vec x^*)$ в точности равно $s(0, \vec x^* - u t^*)$, поэтому оно зависит только от одной точки в начальных условиях (от вектора $\vec x$).
Для других уравнений в частных производных, таких как уравнение теплодиффузии $\partial s / \partial t = \nabla \cdot \nabla s$, каждая точка решения зависит от всех точек начальных условий.
_Область зависимости_ точки (расчётной сетки) — это именно набор точек, которые влияют на значение решения в этой точке.

Каждая точка численного решения также имеет область зависимости: опять же, набор мест в начальных условиях, которые влияют на значение решения в рассматриваемой точке.
Вполне очевидно, что область зависимости при численном расчёте, по крайней мере в пределе, должна содержать истинную (математическую) область зависимости, чтобы численное решение вообще могло стремиться к точному решению.
Это, по сути, и есть условие CFL: сходимость в общем случае возможна только в том случае, если в пределе $\Delta x \rightarrow 0$ и $\Delta t \rightarrow 0$ численная область зависимости для каждой точки содержит истинную область зависимости.

При расчёте потенциального течения, например, условие CFL выполняется, если в пределе траектории жидких частиц достаточно близки к истинным траекториям — достаточно близко, чтобы они не выходили за границы ячеек, через которые проходит реальные траектории.
Тогда, если мы не сделаем что-то сильно неправильное, в пределе траектории частиц должны сходиться к истинным.

Тем не менее, для стандартных явных методов конечных разностей для уравнения переноса, где новое значение точки сетки $s_k^{n+1}$ вычисляется на основе одного или нескольких предыдущих значений в соседних точках сетки, то есть из точек, удаленных только на $C \Delta x$ при небольшой целочисленной константе $C$, существует гораздо более очевидное выражение CFL.
В частности, точное решение движется со скоростью $\vec u$, поэтому скорость передачи информации при численном расчёте, то есть $C \Delta x / \Delta t$, должна быть как минимум такой же по величине.
То есть для сходимости таких численных схем требуется соблюдение условия (в одномерном случае)

$$
\frac{C \Delta x}{\Delta t} \ge |u|.
$$

Отсюда

$$
\Delta t \le \frac{C \Delta x}{|u|}.
$$ (eq_CFL_time_step)

Вот тут-то и возникает больше всего путаницы.
Часто с точностью до небольшого постоянного множителя {eq}`eq_CFL_time_step` то же самое, что и максимальный устойчивый шаг по времени для численного метода и, в частности, в оригинальной статье Куранта так и было.
Таким образом, иногда условие CFL путают с условием устойчивости конечно-разностной схемы.
Но на деле существуют методы, которые нестабильны независимо от размера временного шага, такие как прямой метод Эйлера и схема центральных разностей.
Существуют также явные методы, которые стабильны для произвольных размеров временного шага, однако при этом могут привести к неправильному ответу, если не будет выполнено условие CFL.

Чтобы ещё больше запутать всех, кого только можно, существует связанная с условием CFL величина, называемая числом CFL, часто обозначаемая как $\alpha$.
Под $|u|$ подразумевается некоторая характерная скорость потока.
Как правило, в качестве $|u|$ выбирается какое-то характерное значение.
К примеру, при расчёте волнового процесса выбирают скорость звука как максимальную скорость распространения возмущений (информации) в среде.
Тогда число CFL $\alpha$ определяется так:

$$
\Delta t = \alpha \frac{\Delta x}{c}.
$$

Таким образом, временной шаг, о котором говорилось выше (неравенство {eq}`eq_time_step`), можно выразить как задание числа CFL, равного пяти.
Число же CFL само по себе является просто полезным параметром, а не условием чего-либо.

В лабораторной работе используйте $\alpha < 1$.


<!-- TODO -->
<!-- ### Численная диффузия -->


## Лабораторная работа

### Часть 1

Оттачиваем навыки Python и численного решения дифференциальных
уравнений в частных производных, а именно, уравнения переноса.

Само уравнение выглядит так:

$$
\frac{\partial s}{\partial t}
+ u \frac{\partial s}{\partial x}
= 0,
$$ (eq_trans)

где
$s$ — некоторая физическая величина: концентрация вещества, плотность и т.д.;
$u$ — скорость переноса этой величины в данном случае в направлении оси $x$.

Запишем это уравнение в виде разностной схемы Эйлера:

$$
\frac{s_k^{n+1} - s_k^n}{\Delta t}
+ u \frac{s_{k+1}^n - s_k^n}{\Delta x}
= 0,
$$

или

$$
s_k^{n+1} = s_k^n - u \frac{\Delta t}{\Delta x}
(s_{k+1}^n - s_k^n).
$$ (num_scheme)

При решении задачи Коши для уравнения {eq}`eq_trans` необходимо задать сетку $X$ в пределах от $x_\mathrm{нач}$ до $x_\mathrm{к}$ с $N$ узлами и начальное распределение функции $s$ по этой сетке, то есть функцию:

$$
q(x) = s(t=0, x).
$$

Требуется написать программу решения уравнения {eq}`eq_trans`, используя схему {eq}`num_scheme` и обеспечивая её устойчивость.


### Часть 2 — задача Сода

Имеется бесконечной длины труба с непроницаемой перегородкой (мембраной) с координатой $x_а = 1/2$.
Слева от перегородки находится идеальный газ с параметрами $p_\mathrm{л}$, $\rho_\mathrm{л}$ и $u_\mathrm{л}$, справа — газ с параметрами $p_\mathrm{п}$, $\rho_\mathrm{п}$ и $u_\mathrm{п}$.
В момент времени $t=0$ мембрана мгновенно исчезает.

**Требуется** определить параметры смеси газов на участке трубы $x \in [0; 1]$ в заданный момент времени $t_\mathrm{к}$.
Иными словами в момент времени $t_\mathrm{к}$ найти распределение по длине трубы давления смеси $p(x)$, плотности $\rho(x)$, полной энергии $E(x)$, а также скорости $u(x)$.


#### Теория

Математически процессы, происходящие в рассматриваемой ударной трубе, описываются системой дифференциальных уравнений в частных производных — нестационарными уравнениями ГД — вида:

$$
\begin{cases}
    \cfrac{\partial \rho}{\partial t} +
    \cfrac{\partial \rho u}{\partial x}
    = 0, \\
    \cfrac{\partial \rho u}{\partial t} +
    \cfrac{\partial}{\partial x} (\rho u^2 + p)
    = 0, \\
    \cfrac{\partial \rho E}{\partial t} +
    \cfrac{\partial}{\partial x} \left[
        \rho u \left( E + \cfrac{p}{\rho} \right)
    \right]
    = 0, \\
    E = \varepsilon + \cfrac{u^2}{2}.
\end{cases}
$$ (sod_system)

Систему {eq}`sod_system` можно переписать в векторном виде:

$$
\frac{\partial \vec q}{\partial t}
+ \frac{\partial \vec f}{\partial x} =
0,
$$

где
$\vec q$ — вектор канонических переменных:

$$
\vec q = \left[\begin{matrix}
    \rho \\ \rho u \\ \rho E
\end{matrix}\right];
$$

$\vec f$ — вектор потоков:

$$
\vec f = \left[\begin{matrix}
    \rho u \\ \rho u^2 + p \\ \rho u H
\end{matrix}\right];
$$

$E$ — полная энергия газа;
$\varepsilon$ — внутренняя энергия газа;
$H$ — энтальпия газа, $H = E + p/\rho$;
$p$ — статическое давление газа;
$\rho$ — плотность газа;
$u$ — скорость газа.

Полученное уравнение может быть решено численно методом Годунова:

$$
\vec q_{k+1/2}^{n+1} =
\vec q_{k+1/2}^{n} - \frac{\Delta t}{\Delta x}
(\vec f_{k+1}^n - \vec f_k^n),
$$ (eq_godunov)

где поток на левой границе ячейки $k+1/2$ рассчитывается по методу Русанова:

$$
\vec f_k^n =
\frac{1}{2}
\left[
    \vec f_{k-1/2}^n + f_{k+1/2}^n -
    c_\mathrm{max} (\vec q_{k+1/2}^n - \vec q_{k-1/2}^n)
\right].
$$ (eq_rusanov)

В {eq}`eq_godunov` и {eq}`eq_rusanov` дробные индексы соответствуют центрам расчётных ячеек, целочисленные индексы — границам этих ячеек.
Так, левая и правая граница ячейки $k+1/2$ имеют индексы $k$ и $k+1$ соответственно.

Таким образом, требуется решить систему уравнений {eq}`sod_system` методом Годунова {eq}`eq_godunov`, рассчитывая потоки на границах ячеек по формуле {eq}`eq_rusanov`.
