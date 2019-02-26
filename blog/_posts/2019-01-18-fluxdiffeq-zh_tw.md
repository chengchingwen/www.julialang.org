---
layout: post
title:  DiffEqFlux.jl – 一個 Julia 的神經微分方程套件
author: Chris Rackauckas, Mike Innes, Yingbo Ma, Jesse Bettencourt, Lyndon White, Vaibhav Dixit, 譯者：杜岳華（Yueh-Hua Tu）Dboy Liao (Yin-Chen Liao)
---

在這篇部落格文章中，我們將會展示在 Julia 中使用微分方程解法（DiffEq）在神經網路上有多麼簡單、有效而且穩定。

<!-- In this blog post we will show you how to easily, efficiently, and
robustly use differential equation (DiffEq) solvers with neural networks in Julia. -->

![Flux ODE 訓練動畫](https://user-images.githubusercontent.com/1814174/51399500-1f4dd080-1b14-11e9-8c9d-144f93b6eac2.gif)

<!-- ![Flux ODE Training Animation](https://user-images.githubusercontent.com/1814174/51399500-1f4dd080-1b14-11e9-8c9d-144f93b6eac2.gif) -->

[Neural Ordinary Differential Equations](https://arxiv.org/abs/1806.07366)，
在這篇文章得到 NeurIPS 2018 的最佳論文獎的殊榮之前，早已成為熱門話題。
這篇論文給出了許多令人讚賞的結果，他結合了兩個不相干的領域，但這只不過是個開始而已：
神經網路與微分方程簡直天生絕配。這篇部落格文章來自 [Flux](https://github.com/FluxML/Flux.jl)
套件的作者與 [DifferentialEquations.jl](https://github.com/JuliaDiffEq/DifferentialEquations.jl)
套件作者的合作，實作 Neural ODEs 論文, 將會解釋為什麼這個專案會誕生，以及這個專案現在和未來的走向，
也會開始描繪極致的工具會有怎樣的可能性。

<!-- The [Neural Ordinary Differential Equations](https://arxiv.org/abs/1806.07366)
paper has been a hot topic even before it made a splash as Best Paper of NeurIPS 2018.
The paper already gives many exciting results combining these two
disparate fields, but this is only the beginning: neural networks and
differential equations were born to be together. This blog post, a collaboration
between authors of [Flux](https://github.com/FluxML/Flux.jl),
[DifferentialEquations.jl](https://github.com/JuliaDiffEq/DifferentialEquations.jl)
and the Neural ODEs paper, will explain why, outline current and future
directions for this work, and start to give a sense of what's possible with
state-of-the-art tools. -->

 Julia 中運用數值方法來解微分方程的 [DifferentialEquations.jl](https://github.com/JuliaDiffEq/DifferentialEquations.jl) 函式庫
的眾多優勢已經在[其他文章中被詳細討論](http://www.stochasticlifestyle.com/comparison-differential-equation-solver-suites-matlab-r-julia-python-c-fortran/)。
除了[經典 Fortran 方法的眾多效能評測](https://github.com/JuliaDiffEq/DiffEqBenchmarks.jl)，
他包含了其他現代功能，像是 [GPU 加速](http://www.stochasticlifestyle.com/solving-systems-stochastic-pdes-using-gpus-julia/)、
[分散式（多節點）平行運算](http://docs.juliadiffeq.org/latest/features/monte_carlo.html)
以及[精密的事件處理](http://docs.juliadiffeq.org/latest/features/callback_functions.html)。
最近，這些 Julia 土生土長的微分方程方法已經成功地整合進 [Flux](https://github.com/FluxML/Flux.jl/) 深度學習套件，
並允許在神經網路中使用整套完整測試、優化的 DiffEq 方法。
使用新套件 [DiffEqFlux.jl](https://github.com/JuliaDiffEq/DiffEqFlux.jl/)，我們將會展示給讀者
在神經網路中增加微分方程層有多麼簡單，並可以使用一系列微分方程方法，
包含剛性（stiff）常微分方程、隨機微分方程、延遲微分方程，以及混合（非連續）微分方程。

<!-- The advantages of the Julia
[DifferentialEquations.jl](https://github.com/JuliaDiffEq/DifferentialEquations.jl) library for numerically solving differential equations have been
[discussed in detail in other posts](http://www.stochasticlifestyle.com/comparison-differential-equation-solver-suites-matlab-r-julia-python-c-fortran/).
Along with its
[extensive benchmarking against classic Fortran methods](https://github.com/JuliaDiffEq/DiffEqBenchmarks.jl),
it includes other modern features such as
[GPU acceleration](http://www.stochasticlifestyle.com/solving-systems-stochastic-pdes-using-gpus-julia/),
[distributed (multi-node) parallelism](http://docs.juliadiffeq.org/latest/features/monte_carlo.html),
and [sophisticated event handling](http://docs.juliadiffeq.org/latest/features/callback_functions.html).
Recently, these native Julia differential equation solvers have successfully been embedded
into the [Flux](https://github.com/FluxML/Flux.jl/) deep learning package, to allow the use of a full suite of
highly tested and optimized DiffEq methods within neural networks. Using the new package
[DiffEqFlux.jl](https://github.com/JuliaDiffEq/DiffEqFlux.jl/),
we will show the reader how to easily add differential equation
layers to neural networks using a range of differential equations models, including stiff ordinary
differential equations, stochastic differential equations, delay differential
equations, and hybrid (discontinuous) differential equations. -->

這是第一個完美結合完整微分方程方法及神經網路模型的工具組。這個部落格文章將會說明為什麼完整微分方程
方法套組的彈性如此重要。能夠融合神經網路及 ODEs、SDEs、DAEs、DDEs、剛性方程，
以及像伴隨敏感度運算（adjoint sensitivity calculations）這樣不同的方法，
這是一個神經微分方程重大的廣義化工作，將來提供更好的工具讓研究者去探索問題領域。

<!-- This is the first toolbox to combine a fully-featured differential equations
solver library and neural networks seamlessly together. The blog post will also
show why the flexibility of a full differential equation solver suite is
necessary. With the ability to fuse neural networks with ODEs, SDEs, DAEs, DDEs,
stiff equations, and different methods for adjoint sensitivity calculations,
this is a large generalization of the neural ODEs work and will allow
researchers to better explore the problem domain. -->

（註：如果你對這個工作有興趣，同時是大學或是研究所學生，
我們有 [提供 Google Summer of Code 專案](https://julialang.org/soc)。
並且 [暑假過後有豐厚的津貼補助](https://developers.google.com/open-source/gsoc/help/student-stipends)。
請加入 [Julia Slack](https://slackinvite.julialang.org/) 的 #jsoc 頻道，
歡迎更進一步的細節討論。）

<!-- (Note: If you are interested in this work and are an undergraduate or graduate
student, we have [Google Summer of Code projects available in this area](https://julialang.org/soc). This
[pays quite well over the summer](https://developers.google.com/open-source/gsoc/help/student-stipends).
Please join the [Julia Slack](https://slackinvite.julialang.org/) and the #jsoc channel to discuss in more detail.) -->


## 微分方程究竟與神經網絡有何關聯？

<!-- ## What do differential equations have to do with machine learning? -->

對於不熟悉相關領域的人來說，想必第一個問題自然是：為什麼微分方程在神經網絡這個脈絡下，會有舉足輕重的關聯？簡而言之，微分方程可以藉由數學模型來敘述、編碼 (encoding) 先驗的結構化假設，來表示任何一種非線性系統。

讓我們稍稍解釋一下最後這句話在說什麼。一般來說，主要有三種方法來定義一個非線性轉換: 直接數學建模、機器學習與微分方程式。直接數學建模可以直接寫下輸入與輸出間的非線性轉換，但只有在輸入與輸出間的函數關係形式為已知時可用，然而大部分的狀況，兩者間的確切關係並不是事先知道的。所以大多數的問題是，你如何在輸入輸出間的關係未知的情況下，來對其做非線性數學建模？

其中一種解決方法是使用機器學習演算法。典型的機器學習處理的問題裡，會給定一些輸入資料 [[x]] 和你想預測的輸出 [[y]]。而由給定 [[x]] 產生預測值 [[y]] 就是一個機器學習模型 (以下稱作 [[ML]])。在訓練階段，我們想辦法調整 [[ML]] 的參數讓它得以產生更正確的預測值。接下來，我們即可用 [[ML]] 進行推論 (即針對事前沒見過的 `x` 值去產生相對應的 [[y]])。同時，這也不過是一個非線性轉換而已 [[y=ML(x)]]。但是 [[ML]] 有趣的地方在於他本身數學模型的形式可以非常基本但卻可以調整適應至各種資料。舉例來說，一個簡單的以 sigmoid 函數作為激活函數的神經網絡模型 (以設計矩陣的形式，design matrix)，本質上來說就是簡單的矩陣運算複合帶入 sigmoid 函數裡。舉例來說，[[ML(x)=σ(W3⋅σ(W2⋅σ(W1⋅x)))]] 即是一個簡單的三層神經網絡模型，其中 [[W=(W1, W2, W3)]] 為可以被調整的模型參數。接下來即是選擇適當的 [[W]] 使得 [[ML(x)=y]] 可以合理的逼近收集到的資料。相關機器學習理論已經保證了這是一個估計非線性系統的一個好方法。舉例來說，Universal Approximation Theorem 說明了只要有足夠的層數或參數 (即夠大的 [[W]] 矩陣)，[[ML(x)]] 可以逼近任何非線性函數 (在常見的限制條件下)。

這太好了，它總是有解！然而有幾個必須注意的地方，主要在於這模型需直接從資料裡學習非線性轉換。但在大多數的狀況，我們並不知曉有興趣的非線性方程整體，但我們卻可以知道它的*結構細節*。舉例來說，這個非線性轉換可以是關於森林裡的兔子的數量，而我們可能知道兔子群體的出生率正比於其數量。因此，與其從無到有去學習兔子群體數量的非線性模型，我們或許希望能夠套用這個數量與出生率的已知*先驗（a priori）*關係，和一組參數來描寫它。對於我們的兔子群體模型來說，可以寫成

$[[\text{rabbits(明日)} = \text{Model}(\text{rabbits(今日)}).]]

在這個例子裡，我們得知群體出生率正比於群體數量這個先驗知識。而如果用數學的方式去描述這個關於兔子群體大小結構的假設，即是微分方程。在這裡，我們想描寫的事準確地說，是在給定的某一時間點的兔子群體的出生率將會隨著兔子群體大小的增加而增加。簡單地寫的話，可以寫成以下的式子

$[[\text{rabbits}'(t) = \alpha\cdot \text{rabbits}(t)]]

其中，[[α]] 是可以學習調整的參數。如果你還記得以前學過的微積分，這個方程的解即為成長率為 `α` 的指數成長函數:

[[\text{rabbits}(t_\text{start})e^{(\alpha t)}]]

其中 [[rabbits(start)]] 為初始的兔子數量。但值得注意的是，其實我們並不需要知道這個微分方程的解才能驗證以下想法：我們只需描寫模型的結構條件，數學即可幫助我們求解出這個解應該有的樣子。基於這個理由，使得微分方程成為許多科學領域的工具。例如物理學的基本定律明述了電荷的作用力 ([馬克士威方程組](https://en.wikipedia.org/wiki/Maxwell%27s_equations))。這些方程組對於物體如何改變是重要的方程組，因此這些方程組的解即是物體*將會*在哪裡的預測結果。

<!-- ## What do differential equations have to do with machine learning?

The first question someone not familiar with the field might ask is, why are
differential equations important in this context? The simple answer is that a
differential equation is a way to specify an arbitrary nonlinear transform by
mathematically encoding prior structural assumptions.

Let's unpack that statement a bit. There are three
common ways to define a nonlinear transform: direct
modeling, machine learning, and differential equations.
Directly writing down the nonlinear function only works
if you know the exact functional form that relates the input to the output.
However, in many cases, such exact relations are not known *a priori*.
So how do you do nonlinear modeling if you don't know the nonlinearity?

One way to address this is to use machine
learning. In a typical machine learning problem, you are given some input [[x]] and
you want to predict an output [[y]]. This generation of a prediction [[y]] from [[x]]
is a machine learning model (let's call it [[ML]]).  During training, we attempt to
adjust the parameters of [[ML]] so that it generates accurate predictions.  We
can then use [[ML]] for inference (i.e., produce [[y]]s for novel inputs [[x]]).
This is just a nonlinear transformation [[y=ML(x)]].
The reason [[ML]] is interesting is because its form is basic but adapts to the
data itself. For example, a simple neural network (in design matrix form) with
sigmoid activation functions is simply matrix multiplications followed
by application of sigmoid functions. Specifically,  [[ML(x)=\sigma(W_{3}\cdot\sigma(W_{2}\cdot\sigma(W_{1}\cdot x)))]] is a three-layer deep
neural network, where [[W=(W_1,W_2,W_3)]] are learnable parameters.
You then choose [[W]] such that [[ML(x)=y]] reasonably fits the function you wanted it to fit.
The theory and practice of machine learning confirms that this is a good way to learn nonlinearities.
For example, the Universal Approximation Theorem states that, for
enough layers or enough parameters (i.e. sufficiently large [[W_{i}]] matrices), [[ML(x)]]
can approximate any nonlinear function sufficiently close (subject to common constraints).

So great, this always works! But it has some caveats, the main being
that it has to learn everything about the nonlinear transform directly from the data.
In many cases we do not know the full nonlinear equation, but we may know details about its structure.
For example, the nonlinear function could be the population of rabbits in the forest,
and we might know that their rate of births is dependent on the current population.
Thus instead of starting from nothing, we may want to use this known _a priori_ relation and a set of parameters that defines it.
For the rabbits, let's say that we want to learn

$[[\text{rabbits tomorrow} = \text{Model}(\text{rabbits today}).]]

In this case, we have prior knowledge of the rate of births being dependent on
the current population. The way to mathematically state this
structural assumption is via a differential equation. Here, what we are saying
is that the birth rate of the rabbit population at a given time point increases
when we have more rabbits. The simplest way of encoding that is

$[[\text{rabbits}'(t) = \alpha\cdot \text{rabbits}(t)]]

where [[\alpha]] is some learnable constant. If you know your calculus, the solution
here is exponential growth from the starting point with a growth rate [[\alpha]]:
[[\text{rabbits}(t_\text{start})e^{(\alpha t)}]]. But notice that we didn't need to know the
solution to the differential equation to validate the idea: we encoded the
structure of the model and mathematics itself then outputs what the solution
should be. Because of this, differential equations have been the tool of choice
in most science. For example, physical laws tell you how electrical quantities
emit forces ([Maxwell's Equations](https://en.wikipedia.org/wiki/Maxwell%27s_equations)).
These are essentially equations of how things change and thus
"where things will be" is the solution to a differential equation. -->

但在近十年這些應用已經有長遠的發展，隨著像是系統生物學（systems biology）領域的發展，
整合已知的生物結構以及數學上列舉的假設，以學習到關於細胞間的交互作用，
或是系統藥理學（systems pharmacology）中藉由對一些特定藥物劑量 PK/PD 的建模。

<!-- But in recent
decades this application has gone much further, with fields like systems biology
learning about cellular interactions by encoding known biological structures and
mathematically enumerating our assumptions or in targeted drug dosage through
PK/PD modelling in systems pharmacology. -->

所以隨著我們的機器學習模型成長，會渴求更多更大量的資料，
微分方程因此成為一個很有吸引力的選項，用以指定一個可學習（透過參數）但又有限制條件的非線性轉換。
他們會是在整合既有結構關係的領域知識，以及輸入輸出之間很重要的一個方式。
有這樣的方法跟觀點來看待兩者，兩個方法都有其需要取捨的優缺點，
可以讓彼此成為建模上互補的方法。
這看起來是一條開始將科學實踐與機器學習兩相結合的明顯道路，期待未來會有嶄新而令人興奮的未來！

## 什麼是神經微分方程（ODE）？

<!-- ## What is the Neural Ordinary Differential Equation (ODE)? -->

神經微分方程只是眾多結合這兩個領域的方法之一。
最簡單的解釋方法就是，並不是直接去學非線性轉換，我們希望去學到非線性轉換的結構。
如此一來，不用去計算 $[[y=ML(x)]]，我們將機器學習模型放在導數項上 $[[y'(x) = ML(x)]]，然後我們解微分方程。
為什麼要這麼做？這是因為，一個動機就是這樣定義的模型，然後用最簡單、最容易出錯的方式，尤拉法（Euler method），
解微分方程，你會得到跟[殘差神經網路（residual neural network）](https://arxiv.org/abs/1512.03385)等價的結果。
尤拉法的工作原理是基於 $[[y'(x) = \frac{dy}{dx}]] 這個事實，因此，

<!-- The neural ordinary differential equation is one of many ways to put these two
subjects together. The simplest way of explaining it is that, instead of
learning the nonlinear transformation directly, we wish to learn the structures
of the nonlinear transformation. Thus instead of doing $[[y=ML(x)]], we put the
machine learning model on the derivative, $[[y'(x) = ML(x)]], and now solve the ODE.
Why would you ever do this? Well, one motivation is that defining the model in this way
and then solving the ODE using the simplest and most error prone method, the
Euler method, what you get is equivalent to a [residual neural network](https://arxiv.org/abs/1512.03385).
The way the Euler method works is based on the fact that $[[y'(x) = \frac{dy}{dx}]], thus -->

$[[\Delta y = (y_\text{next} - y_\text{prev}) = \Delta x\cdot ML(x)]]，
則會導出
$[[y_{i+1} = y_{i} + \Delta x\cdot ML(x_{i})]]。

<!-- $[[\Delta y = (y_\text{next} - y_\text{prev}) = \Delta x\cdot ML(x)]]
which implies that
$[[y_{i+1} = y_{i} + \Delta x\cdot ML(x_{i}).]] -->

這在結構上相似於 ResNet，最為成功的影像處理模型之一。
Neural ODEs 論文的洞見就是，更加深、更加強大的類 ResNet 的模型可以有效地逼近類似於「無限深」，
如同每一層趨近於零的模型。
我們可以只是直接建構微分方程，並沒有多增加層數，然後用特製的微分方程方法求解。
數值微分方程方法是個科學方法，可以追溯回到第一台電腦時期，而現代方法可以視情況調整步長 [[\Delta x]]，
以及使用高階逼近的方法來大幅減少實際需要的步數。並且事實證明，他實務上也運作得很好。

<!-- This looks similar in structure to a ResNet, one of the most successful image
processing models. The insight of the the Neural ODEs paper was that
increasingly deep and powerful ResNet-like models effectively approximate a kind
of "infinitely deep" model as each layer tends to zero. Rather than adding more
layers, we can just model the differential equation directly and then solve it
using a purpose-built ODE solver. Numerical ODE solvers are a science that goes
all the way back to the first computers, and modern ones can adaptively choose
step sizes [[\Delta x]] and use high order approximations to dratically reduce the
number of actual steps required. And as it turns out, this works well in
practice, too. -->

## 那要怎麼解微分方程呢？

<!-- ## How do you solve an ODE? -->

首先，要如何解出微分方程的數值解呢？如果你是解微分方程的新手，
你可能想要參考我們的[用 Julia 解微分方程影片教學](https://www.youtube.com/watch?v=KPEqYtEd-zY)，
以及參考我們的[DifferentialEquations.jl 微分方程教學手冊](http://docs.juliadiffeq.org/latest/tutorials/ode_example.html)。
概念是這樣的，如果你透過導函數 `u'=f(u,p,t)` 定義一個 `ODEProblem`，
接著提供一個初始條件 `u0`，一個需要解的時間區段 `tspan`，以及相關的參數 `p`。

<!-- First, how do you numerically specify and solve an ODE? If you're new to solving
ODEs, you may want to watch our
[video tutorial on solving ODEs in Julia](https://www.youtube.com/watch?v=KPEqYtEd-zY)
and look through the
[ODE tutorial of the DifferentialEquations.jl documentation](http://docs.juliadiffeq.org/latest/tutorials/ode_example.html).
The idea is that you define an `ODEProblem` via a derivative equation `u'=f(u,p,t)`,
and provide an initial condition `u0`, and a timespan `tspan` to solve over, and
specify the parameters `p`. -->

舉例來說，[洛特卡－沃爾泰拉方程（Lotka-Volterra equations）描述了野兔與狼族群的動態關係](https://en.wikipedia.org/wiki/Lotka%E2%80%93Volterra_equations)。
他們可以被寫成：

<!-- For example, the
[Lotka-Volterra equations describe the dynamics of the population of rabbits and wolves](https://en.wikipedia.org/wiki/Lotka%E2%80%93Volterra_equations).
They can be written as: -->

$[[x^\prime = \alpha x + \beta x y]]
$[[y^\prime = -\gamma y + \gamma x y]]

進一步轉成 Julia 會像：

<!-- and encoded in Julia like: -->

```julia
using DifferentialEquations
function lotka_volterra(du,u,p,t)
  x, y = u
  α, β, δ, γ = p
  du[1] = dx = α*x - β*x*y
  du[2] = dy = -δ*y + γ*x*y
end
u0 = [1.0,1.0]
tspan = (0.0,10.0)
p = [1.5,1.0,3.0,1.0]
prob = ODEProblem(lotka_volterra,u0,tspan,p)
```

然後要解微分方程，你可以簡單地呼叫 `solve` 來處理 `prob`：

<!-- Then to solve the differential equations, you can simply call `solve` on the
`prob`: -->

```julia
sol = solve(prob)
using Plots
plot(sol)
```

![LV Solution Plot](https://user-images.githubusercontent.com/1814174/51388169-9a07f300-1af6-11e9-8c6c-83c41e81d11c.png)

最後一件要說的事情就是我們可以讓我們的初始條件（`u0`）以及時間區段（`tspans`）
成為參數（`p` 的元素）的函式。舉例來說，我們可以這樣定義 `ODEProblem`：

<!-- One last thing to note is that we can make our initial condition (`u0`) and time spans (`tspans`)
to be functions of the parameters (the elements of `p`). For example, we can define the `ODEProblem`: -->

```julia
u0_f(p,t0) = [p[2],p[4]]
tspan_f(p) = (0.0,10*p[4])
p = [1.5,1.0,3.0,1.0]
prob = ODEProblem(lotka_volterra,u0_f,tspan_f,p)
```

如此一來，關於這個問題的所有東西都由參數向量決定（`p`，或是文獻中的 `θ`）。
這東西的用途會在後續彰顯出來。

<!-- In this form, everything about the problem is determined by the parameter vector (`p`, referred to
as `θ` in associated literature). The utility of this will be seen later. -->

DifferentialEquations.jl 提供非常多強大的選項可以客製化一些指標，
像是準確度（accuracy）、容忍度（tolerances）、微分方程方法、事件等等；可以參考
[手冊](http://docs.juliadiffeq.org/latest/)以獲得更多進階的使用方式。

<!-- DifferentialEquations.jl has many powerful options for customising things like
accuracy, tolerances, solver methods, events and more; check out [the
docs](http://docs.juliadiffeq.org/latest/) for more details on how to use it in
more advanced ways. -->

## 讓我們把微分方程放到神經網路架構裡吧！

<!-- ## Let's Put an ODE Into a Neural Net Framework! -->

要理解一個微分方程是怎麼被嵌入到一個神經網路中，那我們就要看看一個神經網路層實際上是什麼。
一個層實際上就是一個*可微分函數*，他會吃進一個大小為 `n` 的向量，然後吐出一個大小為 `m` 的新向量。
就這樣！網路層傳統上是使用簡單的函數，像是矩陣相乘，但有了[可微分程式設計](https://julialang.org/blog/2017/12/ml&pl-zh_tw)的精神，
人們越來越傾向實驗複雜的函數，像是光線追蹤以及物理引擎。


<!-- To understand embedding an ODE into a neural network, let's look at what a
neural network layer actually is. A layer is really just a *differentiable
function* which takes in a vector of size `n` and spits out a new vector of size
`m`. That's it! Layers have traditionally been simple functions like matrix
multiply, but in the spirit of [differentiable
programming](https://julialang.org/blog/2018/12/ml-language-compiler) people are
increasingly experimenting with much more complex functions, such as ray tracers and
physics engines. -->

恰巧微分方程方法也符合這樣的架構：一個方法會吃進某個向量 `p`
（它有可能包含一些參數像是初始起點），然後輸出某個新向量，也就是解。
而且它還是可微分的，這代表我們可以直接把他推進大型可微分程式內。
這個大型程式可以開心地容納神經網路，以及我們可以繼續使用標準最佳化技巧，
像是 ADAM 來最佳化那些權重。

<!-- Turns out that differential equations solvers fit this framework, too: A solve
takes in some vector `p` (which might include parameters like the initial
starting point), and outputs some new vector, the solution. Moreover it's
differentiable, which means we can put it straight into a larger differentiable
program. This larger program can happily include neural networks, and we can keep
using standard optimisation techniques like ADAM to optimise their weights. -->

DiffEqFlux.jl 讓這件事做起來很簡單；我們一起動手做！
我們就一如往常地開始解這個方程式，不需要計算梯度。

<!-- DiffEqFlux.jl makes it convenient to do just this; let's take it for a spin.
We'll start by solving an equation as before, without gradients. -->

```julia
p = [1.5,1.0,3.0,1.0]
prob = ODEProblem(lotka_volterra,u0,tspan,p)
sol = solve(prob,Tsit5(),saveat=0.1)
A = sol[1,:] # length 101 vector
```

我們一起將微分方程的解畫在 `(t,A)` 座標軸上，一起看看我們得到什麼：

<!-- Let's plot `(t,A)` over the ODE's solution to see what we got: -->

```julia
plot(sol)
t = 0:0.1:10.0
scatter!(t,A)
```

![Data points plot](https://user-images.githubusercontent.com/1814174/51388173-9c6a4d00-1af6-11e9-9878-3c585d3cfffe.png)

最基礎的微分方程層是 `diffeq_rd`，它會做相同的事，只有一點語法上的改變。
`diffeq_rd` 會接受被積函數的參數 `p`，並且把它放進由 `prob` 定義好的微分方程中，
然後根據挑選好的程式參數（解法、容忍度...等等）解方程式。
範例如下：

<!-- The most basic differential equation layer is `diffeq_rd`, which does the same
thing with a slightly altered syntax. `diffeq_rd` takes in parameters `p` for
the integrand, puts it in the differential equation defined by `prob`, and
solves it with the chosen arguments (solver, tolerance, etc). For example: -->

```julia
using Flux, DiffEqFlux
diffeq_rd(p,prob,Tsit5(),saveat=0.1)
```

在 `diffeq_rd` 中的一個好的設計是，它會處理型別的相容性，讓它可以相容於神經網路框架（Flux）。
要證明這個，我們來用函數定義一層神經網路，然後還有一個損失函數，是輸出值相對 `1` 距離的平方。
在 Flux 中，他看起來像這樣：

<!-- The nice thing about `diffeq_rd` is that it takes care of the type handling
necessary to make it compatible with the neural network framework (here Flux). To show this,
let's define a neural network with the function as our single layer, and then a loss
function that is the squared distance of the output values from `1`. In Flux, this looks like: -->

```julia
p = param([2.2, 1.0, 2.0, 0.4]) # 初始參數向量
params = Flux.Params([p])

function predict_rd() # 我們的單層神經網路
  diffeq_rd(p,prob,Tsit5(),saveat=0.1)[1,:]
end

loss_rd() = sum(abs2,x-1 for x in predict_rd()) # 損失函數
```

現在我們會叫 Flux 來訓練神經網路，藉由跑 100 epoch 來最小化我們的損失函數（`loss_rd()`），
因此，可以得到最佳化的參數：

<!-- Now we tell Flux to train the neural network by running a 100 epoch
to minimise our loss function (`loss_rd()`) and thus obtain the optimized parameters: -->

```julia
data = Iterators.repeated((), 100)
opt = ADAM(0.1)
cb = function () # 用 callback function 來觀察訓練情況
  display(loss_rd())
  # 利用 `remake` 來再造我們的 `prob` 並放入目前的參數 `p`
  display(plot(solve(remake(prob,p=Flux.data(p)),Tsit5(),saveat=0.1),ylim=(0,6)))
end

# 顯示初始參數的微分方程
cb()

Flux.train!(loss_rd, params, data, opt, cb = cb)
```

結果會以動畫顯示在上面。
[這些程式碼會被放在 model-zoo](https://github.com/FluxML/model-zoo/blob/da4156b4a9fb0d5907dcb6e21d0e78c72b6122e0/other/diffeq/ode.jl)

<!-- The result of this is the animation shown at the top.
[This code can be found in the model-zoo](https://github.com/FluxML/model-zoo/blob/da4156b4a9fb0d5907dcb6e21d0e78c72b6122e0/other/diffeq/ode.jl) -->

Flux 在尋找可以最小化損失函數的嗔經網路參數（`p`），也就是，他會訓練神經網路：
整個過程是這樣的，在神經網路中向前傳遞（forward pass）的過程也包含了解微分方程的過程。
我們的損失函數會懲罰當兔子數量遠離 1 的時候，
所以我們的神經網路會找到兔子以及狼的族群都是常數 1 的時候的參數。

<!-- Flux finds the parameters of the neural network (`p`) which minimize
the cost function, i.e. it trains the neural network: it just so happens that
the forward pass of the neural network includes solving an ODE.
Since our cost function put a penalty whenever the number of
rabbits was far from 1, our neural network found parameters where our population
of rabbits and wolves are both constant 1. -->

現在，我們已經把微分方程作為一層網路解完了，我們可以隨意將他加到任何地方。
舉例來說，多層感知器（multilayer perceptron）可以用 Flux 寫成像這樣

<!-- Now that we have solving ODEs as just a layer, we can add it anywhere. For example,
the multilayer perceptron is written in Flux as -->

```julia
m = Chain(
  Dense(28^2, 32, relu),
  Dense(32, 10),
  softmax)
```

而且，假如我們有個帶有合適大小的參數向量的 ODE ，我們可以像這樣把它代入我們的模型中：
<!-- 
and if we had an appropriate ODE which took a parameter vector of the right size,
we can stick it right in there: -->

```julia
m = Chain(
  Dense(28^2, 32, relu),
  # this would require an ODE of 32 parameters
  p -> diffeq_rd(p,prob,Tsit5(),saveat=0.1)[1,:],
  Dense(32, 10),
  softmax)
```

抑或是，我們也可以把它代入到卷積神經網路中，使用前一層卷積層的輸出當作 ODE 的初始條件使用：
<!-- or we can stick it into a convolutional neural network, where the previous
layers define the initial condition for the ODE: -->

```julia
m = Chain(
  Conv((2,2), 1=>16, relu),
  x -> maxpool(x, (2,2)),
  Conv((2,2), 16=>8, relu),
  x -> maxpool(x, (2,2)),
  x -> reshape(x, :, size(x, 4)),
  x -> diffeq_rd(p,prob,Tsit5(),saveat=0.1,u0=x)[1,:],
  Dense(288, 10), softmax) |> gpu
```

只要你可以寫下向前傳播 (forward pass)，我們就可以處理任何參數化可微分的程序並優化它。一個新世界即為你的囊中之物啊。
<!-- As long as you can write down the forward pass, we can take any parameterised,
differentiable program and optimise it. The world is your oyster. -->

## 完整的 ODE 求解工具對於這個應用為什麼這麼必須呢?
<!-- ## Why is a full ODE solver suite necessary for doing this well? -->
前文中，我們把現有的求解工具和深度學習結合在一起。反觀另一個傑出的實作 [torchdiffeq](https://github.com/rtqichen/torchdiffeq) 採取了另一種實作方式，直接使用 pytorch 實作了許多求解演算法，包含一個適應性的 Runge Kutta 4-5 (`dopri5`) 和一個 Adams-Bashforth-Moulton 方法 (`adams`)。然而，其中的實作對於特定的模型來說，雖可算是非常有效率地，但無法完整整合所有可行的求解工具卻帶來了一些限制。
<!-- Where we have combined an existing solver suite and deep learning library, the
excellent [torchdiffeq](https://github.com/rtqichen/torchdiffeq) project takes
an alternative approach, instead implementing solver methods directly in
PyTorch, including an adaptive Runge Kutta 4-5 (`dopri5`) and an
Adams-Bashforth-Moulton method (`adams`). However, while their approach is very
effective for certain kinds of models, not having access to a full solver suite
is limiting. -->

我們考慮以下這個例子：[ROBER
ODE](https://www.radford.edu/~thompson/vodef90web/problems/demosnodislin/Single/DemoRobertson/demorobertson.pdf)。最被廣泛測試過 (且最佳化) 過的 Adams-Bashforth-Moulton 方法的實作是著名的 [C++ 套件 SUNDIALS 中的 CVODE 積分器](https://computation.llnl.gov/projects/sundials) (傳統的 LSODE 的一個分支)。讓我們用 DifferentialEquations.jl 去使用 CVODE 中的 Admas 方法來解這個 ODE 吧：
<!-- 
Consider the following example, the [ROBER
ODE](https://www.radford.edu/~thompson/vodef90web/problems/demosnodislin/Single/DemoRobertson/demorobertson.pdf).
The most well-tested (and optimized) implementation of an
Adams-Bashforth-Moulton method is the [CVODE integrator in the C++ package
SUNDIALS](https://computation.llnl.gov/projects/sundials) (a derivative of the
classic LSODE). Let's use DifferentialEquations.jl to call CVODE with its Adams
method and have it solve the ODE for us: -->

```julia
rober = @ode_def Rober begin
  dy₁ = -k₁*y₁+k₃*y₂*y₃
  dy₂ =  k₁*y₁-k₂*y₂^2-k₃*y₂*y₃
  dy₃ =  k₂*y₂^2
end k₁ k₂ k₃
prob = ODEProblem(rober,[1.0;0.0;0.0],(0.0,1e11),(0.04,3e7,1e4))
solve(prob,CVODE_Adams())
```

(For those familiar with solving ODEs in MATLAB, this is similar to `ode113`)

Both this and the `dopri` method from [Ernst Hairer's Fortran
Suite](https://www.unige.ch/~hairer/software.html) stall and fail to solve the
equation. This happens because the ODE is
[stiff](https://en.wikipedia.org/wiki/Stiff_equation), and thus methods with
"smaller stability regions" will not be able to solve it appropriately (for more
details, I suggest reading Hairer's Solving Ordinary Differential Equations II).
On the other hand `KenCarp4()` to this problem, the equation is solved in a
blink of an eye:

```julia
sol = solve(prob,KenCarp4())
using Plots
plot(sol,xscale=:log10,tspan=(0.1,1e11))
```

![ROBER Plot](https://user-images.githubusercontent.com/1814174/51388944-eb18e680-1af8-11e9-874f-09478759596e.png)

This is just one example of subtlety in integration: Stabilizing explicit
methods via PI-adaptive controllers, step prediction in implicit solvers, etc.
are all intricate details that take a lot of time and testing to become
efficient and robust. Different problems require different methods: [Symplectic
integrators](http://docs.juliadiffeq.org/latest/solvers/dynamical_solve.html#Symplectic-Integrators-1)
are required to [adequately handle physical many problems without
drift](https://scicomp.stackexchange.com/questions/29149/what-does-symplectic-mean-in-reference-to-numerical-integrators-and-does-scip/29154#29154),
and tools like [IMEX
integrators](http://docs.juliadiffeq.org/latest/solvers/split_ode_solve.html#Implicit-Explicit-(IMEX)-ODE-1)
are required to handle ODEs which [come from partial differential
equations](https://www.youtube.com/watch?v=okGybBmihOE). Building a
production-quality solver is thus an enormous undertaking and relatively few
exist.

Rather than building an ML-specific solver suite in parallel to one suitable for
scientific computing, in Julia they are one and the same, meaning you can take
advantage of all of these methods today.

## What kinds of differential equations are there?

Ordinary differential equations are only one kind of differential equation. There
are many additional features you can add to the structure of a differential equation.
For example, the amount of bunnies in the future isn't dependent on the number
of bunnies right now because it takes a non-zero amount of time for a parent
to come to term after a child is incepted. Thus the birth rate of bunnies is
actually due to the amount of bunnies in the past. Using a lag term in a
differential equation's derivative makes this equation known as a delay
differential equation (DDE). Since
[DifferentialEquations.jl handles DDEs](http://docs.juliadiffeq.org/latest/tutorials/dde_example.html)
through the same interface as ODEs, it can be used as a layer in
Flux as well. Here's an example:

```julia
function delay_lotka_volterra(du,u,h,p,t)
  x, y = u
  α, β, δ, γ = p
  du[1] = dx = (α - β*y)*h(p,t-0.1)[1]
  du[2] = dy = (δ*x - γ)*y
end
h(p,t) = ones(eltype(p),2)
prob = DDEProblem(delay_lotka_volterra,[1.0,1.0],h,(0.0,10.0),constant_lags=[0.1])

p = param([2.2, 1.0, 2.0, 0.4])
params = Flux.Params([p])
function predict_rd_dde()
  diffeq_rd(p,prob,MethodOfSteps(Tsit5()),saveat=0.1)[1,:]
end
loss_rd_dde() = sum(abs2,x-1 for x in predict_rd_dde())
loss_rd_dde()
```

The full code for this example, including generating an
animation,
[can be found in the model-zoo](https://github.com/FluxML/model-zoo/blob/da4156b4a9fb0d5907dcb6e21d0e78c72b6122e0/other/diffeq/dde.jl)

Additionally we can add randomness to our differential equation to simulate
how random events can cause extra births or more deaths than expected. This
kind of equation is known as a stochastic differential equation (SDE).
Since [DifferentialEquations.jl handles SDEs](http://docs.juliadiffeq.org/latest/tutorials/sde_example.html)
(and is currently the only library with adaptive stiff and non-stiff SDE integrators),
these can be handled as a layer in Flux similarly. Here's a neural net layer
with an SDE:

```julia
function lotka_volterra_noise(du,u,p,t)
  du[1] = 0.1u[1]
  du[2] = 0.1u[2]
end
prob = SDEProblem(lotka_volterra,lotka_volterra_noise,[1.0,1.0],(0.0,10.0))

p = param([2.2, 1.0, 2.0, 0.4])
params = Flux.Params([p])
function predict_fd_sde()
  diffeq_fd(p,sol->sol[1,:],101,prob,SOSRI(),saveat=0.1)
end
loss_fd_sde() = sum(abs2,x-1 for x in predict_fd_sde())
loss_fd_sde()
```

And we can train the neural net to watch it in action and find parameters to make
the amount of bunnies close to constant:

```julia
data = Iterators.repeated((), 100)
opt = ADAM(0.1)
cb = function ()
  display(loss_fd_sde())
  display(plot(solve(remake(prob,p=Flux.data(p)),SOSRI(),saveat=0.1),ylim=(0,6)))
end

# Display the ODE with the current parameter values.
cb()

Flux.train!(loss_fd_sde, params, data, opt, cb = cb)
```

![SDE NN Animation](https://user-images.githubusercontent.com/1814174/51399524-2c6abf80-1b14-11e9-96ae-0192f7debd03.gif)

[This code can be found in the model-zoo](https://github.com/FluxML/model-zoo/blob/da4156b4a9fb0d5907dcb6e21d0e78c72b6122e0/other/diffeq/sde.jl)

And we can keep going. There are differential equations
[which are piecewise constant](http://docs.juliadiffeq.org/latest/tutorials/discrete_stochastic_example.html)
used in biological simulations, or
[jump diffusion equations from financial models](http://docs.juliadiffeq.org/latest/tutorials/jump_diffusion.html),
and the solvers map right over to the Flux neural network frame work through FluxDiffEq.jl
FluxDiffEq.jl uses only around ~100 lines of code to pull this all off.

## Implementing the Neural ODE layer in Julia

Let's go all the way back for a second and now implement the neural ODE layer
in Julia. Remember that this is simply an ODE where the derivative function
is defined by a neural network itself. To do this, let's first define the
neural net for the derivative. In Flux, we can define a multilayer perceptron
with 1 hidden layer and a `tanh` activation function like:

```julia
dudt = Chain(Dense(2,50,tanh),Dense(50,2))
```

To define a `neural_ode` layer, we then just need to give
it a timespan and use the `neural_ode` function:

```julia
tspan = (0.0f0,25.0f0)
x->neural_ode(dudt,x,tspan,Tsit5(),saveat=0.1)
```

As a side note, to run this on the GPU, it is sufficient to make the initial
condition and neural network be on the GPU. This will cause the entire ODE
solver's internal operations to take place on the GPU without extra data
transfers in the integration scheme. This looks like[^gpu]:

```julia
x->neural_ode(gpu(dudt),gpu(x),tspan,BS3(),saveat=0.1)
```

[^gpu]: We have changed the solver from Tsit5 to BS3, as Tsit5 is [currently not GPU compatible](https://github.com/JuliaDiffEq/OrdinaryDiffEq.jl/issues/106).

## Understanding the Neural ODE layer behavior by example

Now let's use the neural ODE layer in an example to find out what it means.
First, let's generate a time series of an ODE at evenly spaced time points.
We'll use the test equation from the Neural ODE paper.

```julia
u0 = Float32[2.; 0.]
datasize = 30
tspan = (0.0f0,1.5f0)

function trueODEfunc(du,u,p,t)
    true_A = [-0.1 2.0; -2.0 -0.1]
    du .= ((u.^3)'true_A)'
end
t = range(tspan[1],tspan[2],length=datasize)
prob = ODEProblem(trueODEfunc,u0,tspan)
ode_data = Array(solve(prob,Tsit5(),saveat=t))
```

Now let's pit a neural ODE against this data. To do so, we
will define a single layer neural network which just has the same neural ODE
as before (but lower the tolerances to help it converge closer, makes for a
better animation!):

```julia
dudt = Chain(x -> x.^3,
             Dense(2,50,tanh),
             Dense(50,2))
ps = Flux.params(dudt)
n_ode = x->neural_ode(dudt,x,tspan,Tsit5(),saveat=t,reltol=1e-7,abstol=1e-9)
```

Notice that the `neural_ode` has the same timespan and `saveat` as the solution
that generated the data. This means that given an `x` (and initial value), it
will generate a guess for what it things the time series will be where the
dynamics (the structure) is predicted by the internal neural network. Let's see
what time series it gives before we train the network. Since the ODE
has two-dependent variables, we will simplify the plot by only showing the first.
The code for the plot is:

```julia
pred = n_ode(u0) # Get the prediction using the correct initial condition
scatter(t,ode_data[1,:],label="data")
scatter!(t,Flux.data(pred[1,:]),label="prediction")
```

![Neural ODE Start](https://user-images.githubusercontent.com/1814174/51585822-d9449400-1ea8-11e9-8665-956a16e95207.png)

But now let's train our neural network. To do so, define a prediction function like before, and then
define a loss between our prediction and data:

```julia
function predict_n_ode()
  n_ode(u0)
end
loss_n_ode() = sum(abs2,ode_data .- predict_n_ode())
```

And now we train the neural network and watch as it learns how to
predict our time series:

```julia
data = Iterators.repeated((), 1000)
opt = ADAM(0.1)
cb = function () #callback function to observe training
  display(loss_n_ode())
  # plot current prediction against data
  cur_pred = Flux.data(predict_n_ode())
  pl = scatter(t,ode_data[1,:],label="data")
  scatter!(pl,t,cur_pred[1,:],label="prediction")
  display(plot(pl))
end

# Display the ODE with the initial parameter values.
cb()

Flux.train!(loss_n_ode, ps, data, opt, cb = cb)
```

![Neural ODE Train](https://user-images.githubusercontent.com/1814174/51585825-dc3f8480-1ea8-11e9-8498-18cf55fba3e6.gif)

[This code can be found in the model-zoo](https://github.com/FluxML/model-zoo/blob/da4156b4a9fb0d5907dcb6e21d0e78c72b6122e0/other/diffeq/neural_ode.jl)

Notice that we are not learning a solution to the ODE.
Instead, what we are learning is the tiny ODE system from which the ODE
solution is generated. I.e., the neural network inside the neural_ode
layer learns this function:

Thus **it learned a compact representation of how the
time series works**, and it can easily extrapolate to what would happen with
different starting conditions. Not only that, it's a very flexible
method for learning such representations. For example, if your data is
unevenly spaced at time points `t`, just pass in `saveat=t` and the
ODE solver takes care of it.

As you could probably guess by now, the DiffEqFlux.jl has all kinds of
extra related goodies like Neural SDEs (`neural_msde`) for you to explore in your
applications.

## The core technical challenge: backpropagation through differential equation solvers

Let's end by explaining the technical issue that needed a solution to make this
all possible. The core to any neural network framework is the ability to
backpropagate derivatives in order to calculate the gradient of the loss function
with respect to the network's parameters. Thus if we stick an ODE solver as a
layer in a neural network, we need to backpropagate through it.

There are multiple ways to do this. The most common is known as (adjoint) sensitivity
analysis. Sensitivity analysis defines a new ODE whose solution gives the
gradients to the cost function w.r.t. the parameters, and solves this secondary
ODE. This is the method discussed in the neural ordinary differential equations
paper, but actually dates back much further, and popular ODE solver frameworks
like [FATODE](http://people.cs.vt.edu/~asandu/Software/FATODE/index.html),
[CASADI](https://web.casadi.org/), and
[CVODES](https://computation.llnl.gov/projects/sundials/cvodes)
have been available with this adjoint method for a long time (CVODES came out
in 2005!). [DifferentialEquations.jl has sensitivity analysis implemented too](http://docs.juliadiffeq.org/latest/analysis/sensitivity.html)

The efficiency problem with adjoint sensitivity analysis methods is that they require
multiple forward solutions of the ODE. As you would expect, this is very costly.
Methods like the checkpointing scheme in CVODES reduce the cost by saving closer
time points to make the forward solutions shorter at the cost of using more
memory. The method in the neural ordinary differential equations paper tries to
eliminate the need for these forward solutions by doing a backwards solution
of the ODE itself along with the adjoints. The issue with this is that this
method implicitly makes the assumption that the ODE integrator is
[reversible](https://www.physics.drexel.edu/~valliere/PHYS305/Diff_Eq_Integrators/time_reversal/).
Sadly, there are no reversible adaptive integrators for first-order ODEs, so
with no ODE solver method is this guaranteed to work. For example, here's a quick
equation where a backwards solution to the ODE using the Adams method from the
paper has >1700% error in its final point, even with solver tolerances of 1e-12:

```julia
using Sundials, DiffEqBase
function lorenz(du,u,p,t)
 du[1] = 10.0*(u[2]-u[1])
 du[2] = u[1]*(28.0-u[3]) - u[2]
 du[3] = u[1]*u[2] - (8/3)*u[3]
end
u0 = [1.0;0.0;0.0]
tspan = (0.0,100.0)
prob = ODEProblem(lorenz,u0,tspan)
sol = solve(prob,CVODE_Adams(),reltol=1e-12,abstol=1e-12)
prob2 = ODEProblem(lorenz,sol[end],(100.0,0.0))
sol = solve(prob,CVODE_Adams(),reltol=1e-12,abstol=1e-12)
@show sol[end]-u0 #[-17.5445, -14.7706, 39.7985]
```

(Here we once again use the CVODE C++ solvers from SUNDIALS since they are a close
match to the SciPy integrators used in the neural ODE paper.)

This inaccuracy is the reason why the method from the neural ODE paper is not
implemented in software suites, but it once again highlights a detail. Not
all ODEs will have a large error due to this issue. And for ODEs where it's not
a problem, this will be the most efficient way to do adjoint sensitivity
analysis. And this method only applies to ODEs. Not only that, it doesn't even
apply to all ODEs. For example, ODEs with discontinuities ([events](http://docs.juliadiffeq.org/latest/features/callback_functions.html)) are excluded by the assumptions of the derivation.
Thus once again we arrive at the conclusion that one method is not enough.

In DifferentialEquations.jl have implemented many different methods for
computing the derivatives of differential equations with respect to parameters.
We have a [recent preprint](https://arxiv.org/abs/1812.01892) detailing
some of these results. One of the things we have found is that direct use of
automatic differentiation can be one of the most efficient and flexible methods.
Julia's ForwardDiff.jl, Flux, and ReverseDiff.jl can directly be applied to
perform automatic differentiation on the native Julia differential equation
solvers themselves, and this can increase performance while giving new features.
Our findings show that forward-mode automatic differentiation is fastest when
there are less than 100 parameters in the differential
equations, and that for >100 number of parameters adjoint
sensitivity analysis is the most efficient. Even
then, we have good reason to believe that
[the next generation reverse-mode automatic differentiation via source-to-source AD, Zygote.jl](https://julialang.org/blog/2018/12/ml-language-compiler),
will be more efficient than all of the adjoint sensitivity implementations for
large numbers of parameters.

Altogether, being able to switch between different gradient methods without changing
the rest of your code is crucial for having a scalable, optimized, and
maintainable framework for integrating differential equations and neural networks.
And this is precisely what FluxDiffEq.jl gives the user direct access to. There
are three functions with a similar API:

- `diffeq_rd` uses Flux's reverse-mode AD through the differential equation
  solver.
- `diffeq_fd` uses ForwardDiff.jl's forward-mode AD through the differential
  equation solver.
- `diffeq_adjoint` uses adjoint sensitivity analysis to "backprop the ODE solver"

Therefore, to switch from a reverse-mode AD layer to a forward-mode
AD layer, one simply has to change a single character. Since Julia-based automatic
differentiation works on Julia code, the native Julia differential equation
solvers will continue to benefit from advances in this field.

## Conclusion

Machine learning and differential equations are destined to come together due to
their complementary ways of describing a nonlinear world. In the Julia ecosystem
we have merged the differential equation and deep learning packages in such a
way that new independent developments in the two domains can directly be used together.
We are only beginning to understand the possibilities that have opened up with
this software. We hope that future blog posts will detail some of the cool
applications which mix the two disciplines, such as embedding our coming
pharmacometric simulation engine [PuMaS.jl](https://doi.org/10.1007/s10928-018-9606-9)
into the deep learning framework. With access to the full range of solvers for ODEs,
SDEs, DAEs, DDEs, PDEs, discrete stochastic equations, and more, we are
interested to see what kinds of next generation neural networks you will build with Julia.

Note: a citable version of this post will be published on
Arxiv soon.
