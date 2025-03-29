---
## Front matter
title: "Лабораторная работа №8"
subtitle: "Модель TCP/AQM"
author: "Кадров Виктор Максимович"

## Generic otions
lang: ru-RU
toc-title: "Содержание"

## Bibliography
bibliography: bib/cite.bib
csl: pandoc/csl/gost-r-7-0-5-2008-numeric.csl

## Pdf output format
toc: true # Table of contents
toc-depth: 2
lof: true # List of figures
lot: false # List of tables
fontsize: 12pt
linestretch: 1.5
papersize: a4
documentclass: scrreprt
## I18n polyglossia
polyglossia-lang:
  name: russian
  options:
	- spelling=modern
	- babelshorthands=true
polyglossia-otherlangs:
  name: english
## I18n babel
babel-lang: russian
babel-otherlangs: english
## Fonts
mainfont: IBM Plex Serif
romanfont: IBM Plex Serif
sansfont: IBM Plex Sans
monofont: IBM Plex Mono
mathfont: STIX Two Math
mainfontoptions: Ligatures=Common,Ligatures=TeX,Scale=0.94
romanfontoptions: Ligatures=Common,Ligatures=TeX,Scale=0.94
sansfontoptions: Ligatures=Common,Ligatures=TeX,Scale=MatchLowercase,Scale=0.94
monofontoptions: Scale=MatchLowercase,Scale=0.94,FakeStretch=0.9
mathfontoptions:
## Biblatex
biblatex: true
biblio-style: "gost-numeric"
biblatexoptions:
  - parentracker=true
  - backend=biber
  - hyperref=auto
  - language=auto
  - autolang=other*
  - citestyle=gost-numeric
## Pandoc-crossref LaTeX customization
figureTitle: "Рис."
tableTitle: "Таблица"
listingTitle: "Листинг"
lofTitle: "Список иллюстраций"
lotTitle: "Список таблиц"
lolTitle: "Листинги"
## Misc options
indent: true
header-includes:
  - \usepackage{indentfirst}
  - \usepackage{float} # keep figures where there are in the text
  - \floatplacement{figure}{H} # keep figures where there are in the text
---

# Цель работы

Исследовать модель TCP/AQM с помощью программы *xcos* и OpenModelica[@lab].

# Задание

- реализовать модель TCP/AQM в xcos[@xss];
- реализовать модель TCP/AQM в OpenModelica;
- построить графики динамики изменения размера TCP окна и размера очереди;
- построить фазовые портреты.


# Выполнение лабораторной работы

## Теоретическая часть

Рассмотрим упрощённую модель поведения TCP-подобного трафика с регулируемой некоторым AQM алгоритмом динамической интенсивностью потока. 

$W(t)$ -- средний размер TCP-окна (в пакетах, функция положительна),

$Q(t)$ -- средний размер очереди (в пакетах, функция положительна),

$R(t)$ -- время двойного оборота (Round Trip Time, сек.)

$C$ -- скорость обработки пакетов в очереди (пакетов в секунду)

$N(t)$ -- число TCP-сессий

$p(t-R(t))$ -- вероятностная функция сброса (отметки на сброс) пакета, значения которой лежат на интервале $[0,1]$.

Примем $N(t) \equiv N$, $R(t) \equiv R$, т. е. указанные величины положим постоянными, не изменяющимися во времени. Также положим $p(t-R(t))=KQ(t)$, т.е. функция сброса пакетов пропорциональна длине очереди $Q(t)$. 

Тогда получим систему:

$$
\dot{W}(t) = \frac{1}{R} - \frac{W(t)W(t-R)}{2R} K Q(t-R)
$$

$$
\dot{Q}(t) = 
\begin{cases} 
    \frac{NW(t)}{R} - C, & Q(t) > 0, \\
    \max \left( \frac{NW(t)}{R} - C, 0 \right), & Q(t) = 0.
\end{cases}
$$

## Реализация модели в xcos

В меню Моделирование, Задать переменные окружения зададим значения переменных (рис. [-@fig:001]).

![Ввод переменных окружения](image/1_1.png){#fig:001 width=70%}

Для реализации введем выражение, определяющее $\dot{Q}(t)$, в блок Expression (рис. [-@fig:002]).

![Изменение параметров блока "Expression"](image/2_1.png){#fig:002 width=70%}

Установим начальные значения в блоках интегрирования(рис. [-@fig:003], [-@fig:004]).

![Изменение параметров блоков интегрирования](image/3_1.png){#fig:003 width=70%} 

![Изменение параметров блоков интегрирования](image/4_1.png){#fig:004 width=70%} 

Установим значение задержки блоков "Continuous fix delay" (рис. [-@fig:005]).

![Изменение параметров блока "Continuous fix delay"](image/5_1.png){#fig:005 width=70%}

Укажем параметры моделирования, зададим конечное время интегрирования. (рис. [-@fig:006]).

![Параметры моделирования](image/6_1.png){#fig:006 width=70%}

Изменим параметры генерирующих устройств, изменим цвет графиков, масштаб. Так же у блока CSCOPE ставим параметр `refresh period` = 100(рис. [-@fig:007], [-@fig:008]). 

![Параметры блока "CSCOPE"](image/7_1.png){#fig:007 width=70%}

![Параметры блока "CSCOPXY"](image/8_1.png){#fig:008 width=70%}

Настроим связи между блоками и получим готовую модель TCP/AQM (рис. [-@fig:009]).

![Модель TCP/AQM в xcos](image/9_1.png){#fig:009 width=70%}

Запустим моделирование и получим следующие графики(рис. [-@fig:010], [-@fig:011]). Фазовый портрет показывает наличие автоколебаний параметров системы — фазовая траектория осциллирует вокруг своей стационарной точки.

![Динамика изменения размера TCP окна W(t)(красная) и размера очереди Q(t)(черная) в xcos. C = 1](image/10_1.png){#fig:010 width=70%}

![Фазовый портрет (W, Q) в xcos. C = 1](image/11_1.png){#fig:011 width=70%}

Изменим перменные окружения. Параметр С = 0.9. (рис. [-@fig:012]).

![Измененные переменные окружения](image/12_1.png){#fig:012 width=70%}

Запустим моделирование и получим следующие графики при С = 0.9(рис. [-@fig:013], [-@fig:014]).

![Динамика изменения размера TCP окна W(t)(красная) и размера очереди Q(t)(черная) в xcos. C = 0.9](image/13_1.png){#fig:013 width=70%}

![Фазовый портрет (W, Q) в xcos. C = 0.9](image/14_1.png){#fig:014 width=70%}

## Реализация модели в OpenModelica

Перейдем к реализации модели в OpenModelica. Зададим параметры, начальные значения и систему дифференциальных уравнений (рис. [-@fig:015]).

![Реализация модели TCP/AQM в OpenModelica](image/15_1.png){#fig:015 width=70%}

Установим параметры симуляции. (рис. [-@fig:016]).

![Параметры симуляции в OpenModelica](image/16_1.png){#fig:016 width=70%}

Результаты моделирования в OpenModelica при C = 1.

Запустим моделирование и получим следующие графики в OpenModelica при С = 1.(рис. [-@fig:017], [-@fig:018]).

![Динамика изменения размера TCP окна W(t)(красная) и размера очереди Q(t)(синия) в OpenModelica. C = 1](image/17_1.png){#fig:017 width=70%}

![Фазовый портрет (W, Q) в OpenModelica. C = 1](image/18_1.png){#fig:018 width=70%}

Изменим параметры в OpenModelica. Параметр С = 0.9.(рис. [-@fig:019]).

![Измененные параметры симуляции в OpenModelica. С = 0.9](image/19_1.png){#fig:019 width=70%}

И получим следующие графики.  (рис. [-@fig:020], [-@fig:021]).

![Динамика изменения размера TCP окна W(t)(красная) и размера очереди Q(t)(синия) в OpenModelica. C = 0.9](image/20_1.png){#fig:020 width=70%}

![Фазовый портрет (W, Q) в OpenModelica. C = 0.9](image/21_1.png){#fig:021 width=70%}


# Выводы

Мы исследовали модель TCP/AQM с помощью программы *xcos* и OpenModelica.

# Список литературы{.unnumbered}

::: {#refs}
:::