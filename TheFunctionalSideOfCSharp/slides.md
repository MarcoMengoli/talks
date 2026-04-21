---
theme: default
colorSchema: light
title: The Functional Side of C#
info: |
  ## The Functional Side of C#
  Functors and Monads in everyday code
class: text-center
drawings:
  persist: false
transition: slide-left
comark: true
duration: 35min
---

# THE FUNCTIONAL SIDE OF C#

FUNCTORS AND MONADS IN EVERYDAY CODE

<div class="pt-12 text-gray-600">
  Marco Mengoli
</div>

---
layout: center
class: text-center
transition: slide-up
---

# Monad

<div class="text-lg mt-8 p-3 rounded">

> "A monad in X is just a monoid in the category of endofunctors of X"

</div>

<div class="text-sm text-gray-600">Saunders Mac Lane - Categories for the Working Mathematician</div>

<v-clicks>

<div class="text-left text-sm mt-4 p-6 bg-gray-100 rounded">

A **monad** on a category $\mathcal{C}$ is an endofunctor $T: \mathcal{C} \to \mathcal{C}$ together with two natural transformations:

$$
\eta: \mathrm{Id}_{\mathcal{C}} \Rightarrow T \qquad \mu: T \circ T \Rightarrow T
$$

satisfying the following coherence laws:

$$
\mu \circ T\mu = \mu \circ \mu T \qquad \mu \circ T\eta = \mu \circ \eta T = \mathrm{Id}_T
$$

i.e. a *monoid* in the category of endofunctors of $\mathcal{C}$.

</div>

</v-clicks>

<!-- 
IDENTITA e MOLTIPLICAZIONE

ASSOCIATIVITA e UNITA DESTRA E SINISTRA
-->

---
layout: center
class: text-center
---

<img src="/global/thisisfine.jpg" class="h-4/5 mx-auto" />

---
layout: center
transition: slide-up
---

# A ~~few~~ lot of steps back. Let's observe a pattern

Three snippets of **everyday** code

<v-click>

```csharp
IEnumerable<int> numbers = new[] { 1, 2, 3, 4, 5 };
IEnumerable<int> doubled = numbers.Select(n => n * 2);
```

</v-click>
<br>
<v-click>

```csharp
Task<User> userTask = FetchUserAsync(id);
Task<string> nameTask = userTask.ContinueWith(t => t.Result.Name.ToUpper());
```

</v-click>
<br>
<v-click>

```csharp
int? age = GetAge();
int? nextYear = age is not null ? age.Value + 1 : null;
```

</v-click>
<!-- 
I never pull the value out. **I apply a function inside the context.**

I don't wait. **I apply a function inside the async context.**

I check for absence. **I apply a function inside the context of "maybe".**

-->

---
layout: center
transition: slide-up
---

# What do they have in common?

<v-clicks>

### They're all **boxes** 📦

- `IEnumerable<T>` — a box with **many** values
- `Task<T>`        — a box with a value that **will be computed**
- `Nullable<T>`    — a box that **might or might not** have a value

</v-clicks>

<div v-click class="mt-8 text-xl text-orange-600">
We don't care about the value itself.<br>
We care about <b>how the value moves inside the box</b>.
</div>

---
layout: center
class: text-center
transition: slide-up
---

# 📦

<div class="text-3xl mt-4">Let's talk about boxes.</div>

---
layout: center
transition: slide-up
---

# The Box 📦 concept

Let's define a generic type `Box<T>`

<div class="flex items-center justify-center gap-12 mt-10">

  <div class="flex flex-col items-center gap-4">
    <div class="w-28 h-28 flex items-center justify-center border-4 border-slate-400 rounded-xl bg-gradient-to-br from-emerald-100 from-50% to-slate-300 to-50% shadow-md relative">
      <div class="absolute top-2 right-2 p-1 rounded-full bg-white border border-slate-200 shadow-sm z-10">
        <div class="i-carbon-help text-lg text-slate-600" />
      </div>
      <span class="text-5xl font-mono font-bold text-slate-700 z-0">2</span>
    </div>
    <div class="flex flex-col items-center leading-tight">
      <span class="text-[10px] font-mono opacity-60 uppercase text-center">might NOT<br>EXIST</span>
      <code class="mt-2 text-xs text-slate-600 font-bold px-2 py-0.5 bg-slate-100 rounded">Nullable&lt;int&gt;</code>
    </div>
  </div>

  <div class="flex flex-col items-center gap-4">
    <div class="w-28 h-28 flex items-center justify-center border-4 border-sky-400 rounded-xl bg-sky-100 shadow-md relative">
      <div class="absolute top-2 right-2 p-1 rounded-full bg-white border border-sky-200 shadow-sm z-10">
        <div class="i-carbon-time text-lg text-sky-600" />
      </div>
      <span class="text-5xl font-mono font-bold text-sky-700">2</span>
    </div>
    <div class="flex flex-col items-center leading-tight">
      <span class="text-[10px] font-mono opacity-60 uppercase text-center">arrives<br>LATER</span>
      <code class="mt-2 text-xs text-sky-700 font-bold px-2 py-0.5 bg-sky-100 rounded">Task&lt;int&gt;</code>
    </div>
  </div>

  <div class="flex flex-col items-center gap-4">
    <div class="w-28 h-28 flex items-center justify-center border-4 border-rose-400 rounded-xl bg-gradient-to-br from-emerald-100 from-50% to-rose-300 to-50% shadow-md relative text-rose-700">
      <div class="absolute top-2 right-2 p-1 rounded-full bg-white border border-rose-200 shadow-sm z-10">
        <div class="i-carbon-warning-alt-filled text-lg text-rose-500" />
      </div>
      <span class="text-5xl font-mono font-bold text-rose-800">2</span>
    </div>
    <div class="flex flex-col items-center leading-tight">
      <span class="text-[10px] font-mono opacity-60 uppercase text-center">might have<br>FAILED</span>
      <code class="mt-2 text-xs text-rose-700 font-bold px-2 py-0.5 bg-rose-50 rounded">Result&lt;int&gt;</code>
    </div>
  </div>

</div>

<div class="mt-16 space-y-2 text-gray-500">
  <p class="italic text-sm">You cannot touch the value <b>directly</b> when it's inside a box.</p>
  <p class="text-xl font-light">
    A value with <span class="text-sky-600 font-bold">superpower</span>. 
    A value inside a <span class="text-emerald-600 font-bold">context</span>.
  </p>
</div>

---
layout: center
transition: slide-up
---

# Box: Option

Define a new type <code>Option&lt;T&gt;</code>: a special <code>Box&lt;T&gt;</code> that can contain a value or can be empty

<div class="flex items-center justify-center gap-16 mt-4">

  <div class="flex flex-col items-center gap-4 text-emerald-700/70">
    <div class="w-32 h-32 flex items-center justify-center border-4 border-emerald-300 rounded-xl bg-emerald-50/50 shadow-sm">
      <span class="text-5xl font-mono font-bold text-emerald-600/80">2</span>
    </div>
    <div class="flex flex-col items-center">
      <span class="text-sm font-bold uppercase tracking-widest">Some(2)</span>
    </div>
  </div>

  <div class="flex flex-col items-center gap-4 text-rose-700/70">
    <div class="w-32 h-32 flex items-center justify-center border-4 border-rose-200 rounded-xl bg-rose-50/30">
      <div class="i-carbon-circle-dash text-4xl opacity-30" />
    </div>
    <div class="flex flex-col items-center">
      <span class="text-sm font-bold uppercase tracking-widest">None</span>
    </div>
  </div>

</div>

Definition in a generic typed functional language:
```haskell
type Option v = None | Some v
```


Implementation in C#:
```csharp
interface Option<T> {}
record Some<T>(T Value) : Option<T>;
record None<T> : Option<T>;
```
---
layout: center
---

# Box: other examples

Real-life boxes:

<div class="text-base mt-4 space-y-2">

- <code>Nullable&lt;T&gt;</code> — a <code>T</code> that can be <code>null</code>
- <code>Func&lt;T&gt;</code> — a <code>T</code> that can be computed on demand
- <code>Lazy&lt;T&gt;</code> — a <code>T</code> that can be computed on demand once, then cached
- <code>Task&lt;T&gt;</code> — a <code>T</code> that is being computed asynchronously and will be available in the future, if it isn't already
- <code>IEnumerable&lt;T&gt;</code> — a sequence of zero or more <code>T</code>s

</div>


---

# A normal value and a plain function

<div class="flex items-center justify-center gap-4 mt-8">

  <div class="flex flex-col items-center gap-3">
    <div class="w-24 h-24 flex items-center justify-center">
      <span class="text-4xl font-mono font-bold text-blue-600">2</span>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">plain value</span>
  </div>

  <div class="text-3xl text-gray-300 i-carbon-arrow-right mx-1" />

  <div class="flex flex-col items-center gap-3">
    <div class="h-24 flex items-center justify-center px-2">
      <span class="text-4xl font-mono font-bold text-pink-500 whitespace-nowrap">
        n=>n*3
      </span>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">plain function</span>
  </div>

  <div class="text-3xl font-light text-gray-200 mx-2">=</div>

  <div class="flex flex-col items-center gap-3">
    <div class="w-24 h-24 flex items-center justify-center">
      <span class="text-4xl font-mono font-bold text-green-600">6</span>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">plain result</span>
  </div>

</div>


We have a value

We have a function

We apply the function to the value — `2 × 3`

---
layout: center
class: text-center
transition: slide-up
---

<div class="text-8xl">
Functor
</div>

<div class="text-2xl mt-6 text-gray-700">
The Transformer
</div>

<div class="text-lg mt-2 text-gray-600"><code>map</code></div>


---
layout: center
transition: slide-up
---

# Box: applying a plain function

How can we apply our function to the value when it's wrapped in a context?

<div class="flex items-center justify-center gap-4 mt-8">

  <div class="flex flex-col items-center gap-3">
    <div class="w-24 h-24 flex items-center justify-center border-4 border-blue-500 rounded-xl bg-blue-50 shadow-md">
      <span class="text-4xl font-mono font-bold text-blue-600">2</span>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">boxed value</span>
  </div>

  <div class="text-3xl text-gray-300 i-carbon-arrow-right mx-1" />

  <div class="flex flex-col items-center gap-3">
    <div class="h-24 flex items-center justify-center px-2">
      <span class="text-4xl font-mono font-bold text-pink-500 whitespace-nowrap">
        n=>n*3
      </span>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">plain function</span>
  </div>

  <div class="text-3xl font-light text-gray-200 mx-2">=</div>

  <div class="flex flex-col items-center gap-3">
    <span class="text-4xl font-mono font-bold text-green-600">???</span>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter mx-10">.</span>
  </div>

</div>


<div class="mt-8 text-lg">
The function is a <b>plain</b> function. It doesn't know what a box is.
</div>


---
transition: slide-up
---

# `map` function

<div class="flex items-center justify-center gap-4 mt-8">

  <div class="flex flex-col items-center gap-3">
    <div class="w-24 h-24 flex items-center justify-center border-4 border-blue-500 rounded-xl bg-blue-50 shadow-md">
      <span class="text-4xl font-mono font-bold text-blue-600">2</span>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">boxed value</span>
  </div>

  <div class="text-3xl text-gray-300 i-carbon-arrow-right mx-1" />

  <div class="flex flex-col items-center gap-3">
    <div class="h-24 flex items-center justify-center px-2">
      <span class="text-4xl font-mono font-bold text-pink-500 whitespace-nowrap">
        n=>n*3
      </span>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">plain function</span>
  </div>

  <div class="text-3xl font-light text-gray-200 mx-2">=</div>

  <div class="flex flex-col items-center gap-3" v-click=2>
    <div class="w-24 h-24 flex items-center justify-center border-4 border-green-500 rounded-xl bg-green-50 shadow-md">
      <span class="text-4xl font-mono font-bold text-green-600">6</span>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">boxed result</span>
  </div>

</div>


<div class="mt-1" v-click=1>

The <b>map</b> is the function that:
1. Opens the box
2. Takes the value out of the boox
3. Applies the function to the value <code>2 × 3</code>
4. Re-wraps the result <code>6</code> in a box of the same kind

</div>


<div class="mt-8 text-center text-xl text-gray-700" v-click=2>
<code>map</code> takes a <b>plain function</b> and applies it to a <b>boxed value</b>
</div>

---
transition: slide-up
---

# Functor

A <code>Functor</code> is any data type that defines how <code>Map</code> applies to it.

Here's a generic <code>Map</code> signature:

```haskell
map::   (a->b)     ->        fa        ->        fb

         takes         and a functor        and returns
       a function                          a new functor
```

And here is the <code>Map</code> signature in C#:

```csharp
public static Box<TOut> Map<TIn, TOut>(this
    Box<TIn> val, 
    Func<TIn, TOut> f);
```

So we can take a plain function and apply it to <code>Box</code>ed value, because <code>Box</code> is a <code>Functor</code>

---

# Implementation

Implementing <code>Map</code> function for both <code>IEnumerable</code> and <code>Option</code> 


```csharp
public static IEnumerable<TOut> Map<TIn, TOut>(this
    IEnumerable<TIn> val, 
    Func<TIn, TOut> f)
{
    foreach (var item in val)
            yield return f(item);
}
    
```
<br>

```csharp
public static Option<TOut> Map<TIn, TOut>(this
    Option<TIn> val, 
    Func<TIn, TOut> f)
    =>
    val switch
    {
        None<TIn> _ => new None<TOut>(),
        Some<TIn>(var x) => new Some<TOut>(f(x)),
        _ => throw new InvalidOperationException("Option can be only Some or None")
    };
```

---
layout: center
class: text-center
transition: slide-up
---

<div class="text-8xl">
Applicative
</div>

<div class="text-2xl mt-6 text-gray-700">
The Combiner
</div>

<div class="text-lg mt-2 text-gray-600"><code>apply</code></div>

---
transition: slide-up
---

# Raise the bar

<div class="text-xl mt-4">
What if <b>both</b> are in a box?
</div>

<div class="flex items-center justify-center gap-4 mt-8">

  <div class="flex flex-col items-center gap-3">
    <div class="w-24 h-24 flex items-center justify-center border-4 border-blue-500 rounded-xl bg-blue-50 shadow-md">
      <span class="text-4xl font-mono font-bold text-blue-600">2</span>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">boxed value</span>
  </div>

  <div class="text-3xl text-gray-300 i-carbon-arrow-right mx-1" />

  <div class="flex flex-col items-center gap-3">
    <div class="h-24 flex items-center justify-center px-6 border-4 border-pink-500 rounded-xl bg-pink-50 shadow-md">
      <span class="text-4xl font-mono font-bold text-pink-500 whitespace-nowrap">
        n=>n*3
      </span>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">boxed function</span>
  </div>

  <div class="text-3xl font-light text-gray-200 mx-2">=</div>

  <div class="flex flex-col items-center gap-3">
      <span class="text-4xl font-mono font-bold text-green-600">???</span>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">.</span>
  </div>

</div>

<v-click>

<div class="mt-6 text-lg">
Neither one is plain — also the <b>function itself</b> is now wrapped.
</div>

<div class="mt-2 text-lg text-gray-700">
<code>map</code> can't help: it wants a plain function.
</div>

</v-click>

---
transition: slide-up
---

# `apply` function

<div class="flex items-center justify-center gap-4 mt-8">

  <div class="flex flex-col items-center gap-3">
    <div class="w-24 h-24 flex items-center justify-center border-4 border-blue-500 rounded-xl bg-blue-50 shadow-md">
      <span class="text-4xl font-mono font-bold text-blue-600">2</span>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">boxed value</span>
  </div>

  <div class="text-3xl text-gray-300 i-carbon-arrow-right mx-1" />

  <div class="flex flex-col items-center gap-3">
    <div class="h-24 flex items-center justify-center px-6 border-4 border-pink-500 rounded-xl bg-pink-50 shadow-md">
      <span class="text-4xl font-mono font-bold text-pink-500 whitespace-nowrap">
        n=>n*3
      </span>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">boxed function</span>
  </div>

  <div class="text-3xl font-light text-gray-200 mx-2">=</div>

  <div class="flex flex-col items-center gap-3" v-click=2>
    <div class="w-24 h-24 flex items-center justify-center border-4 border-green-500 rounded-xl bg-green-50 shadow-md">
      <span class="text-4xl font-mono font-bold text-green-600">6</span>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">boxed result</span>
  </div>

</div>



<div class="mt-1" v-click=1>

The <code>apply</code> is the function that:
1. Opens the boxes
2. Takes the value and the function out of the box
3. Applies the function to the value <code>2 × 3</code>
4. Re-wraps the result <code>6</code> in a box of the same kind

</div>

<div class="mt-3 text-center text-xl text-gray-700" v-click="3">
<code>apply</code> takes a <b>boxed function</b> and applies it to a <b>boxed value</b>
</div>

<div class="mt-2 text-center text-xl text-gray-700" v-click="4">
It makes two boxes <b>interact</b>
</div>

<div class="mt-2 text-center text-lg text-gray-700" v-click="5">
<code>apply</code> is the bridge that lets a <b>boxed function</b> consume <b>boxed arguments</b>, one at a time
</div>

---
transition: slide-up
---

# Applicative

An <code>Applicative</code> is any data type that defines how <code>Pure</code> and <code>Apply</code> apply to it:
1. <code>Pure</code> (or <code>Return</code>): put a plain value into a default box.
2. <code>Apply</code> (or <code>&lt;*&gt;</code>): apply a boxed function to a boxed value.

```haskell
pure::     a       ->       fa
         takes          and puts it 
        a value           in a box

apply::  f (a->b)  ->        fa        ->        fb
          takes         and a functor        and returns
    a boxed function                        a new functor
```


```csharp
public static Box<T> Return<T>(T val);

public static Box<TOut> Apply<TIn, TOut>(this 
    Box<Func<TIn, TOut>> f, 
    Box<TIn> val);
```

So we can take a <code>Box</code>ed function and apply it to a <code>Box</code>ed value, because <code>Box</code> is an <code>Applicative</code>

---
transition: slide-up
---

# Implementation: IEnumerable

Implementing `Return` and `Apply` for `IEnumerable` (Cartesian Product)

```csharp
public static IEnumerable<T> Return<T>(T val)
{
    yield return val;
}
```
<br>
```csharp
public static IEnumerable<TOut> Apply<TIn, TOut>(this
    IEnumerable<Func<TIn, TOut>> functions, 
    IEnumerable<TIn> values)
{
    foreach (var f in functions)
        foreach (var val in values)
            yield return f(val);
}
```

---

# Implementation: Option

Implementing `Return` and `Apply` for the `Option` type.

```csharp
public static Option<T> Return<T>(T val) => new Some<T>(val);
```
<br>
```csharp
public static Option<TOut> Apply<TIn, TOut>(this
    Option<Func<TIn, TOut>> f, 
    Option<TIn> val) 
    =>
    (f, val) switch
    {
        (Some<Func<TIn, TOut>>(var func), Some<TIn>(var v)) => new Some<TOut>(func(v)),
        _ => new None<TOut>()
    };
```

---
layout: center
class: text-center
transition: slide-up
---

<div class="text-8xl">
Monad
</div>

<div class="text-2xl mt-6 text-gray-700">
The Chainer
</div>

<div class="text-lg mt-2 text-gray-600"><code>bind</code></div>

---
transition: slide-up
---

# A function that returns a box

<div class="flex items-center justify-center gap-4 mt-8">

  <div class="flex flex-col items-center gap-3" v-click=1>
    <div class="w-24 h-24 flex items-center justify-center border-4 border-blue-500 rounded-xl bg-blue-50 shadow-md">
      <span class="text-4xl font-mono font-bold text-blue-600">2</span>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">boxed value</span>
  </div>

  <div class="text-3xl text-gray-300 i-carbon-arrow-right mx-1" v-click=1 />

  <div class="flex flex-col items-center gap-3">
    <div class="h-24 flex items-center justify-center px-2 gap-3">
      <span class="text-4xl font-mono font-bold text-pink-500 whitespace-nowrap">n =></span>
      <div class="w-28 h-20 flex items-center justify-center border-4 border-pink-500 rounded-xl bg-pink-50">
        <span class="text-3xl font-mono font-bold text-pink-500">n * 3</span>
      </div>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">function returning a box</span>
  </div>

  <div class="text-3xl font-light text-gray-200 mx-2" v-click=3>=</div>

  <div class="flex flex-col items-center gap-3" v-click=3>
    <div class="w-32 h-32 flex items-center justify-center border-4 border-green-600 rounded-2xl bg-green-100/50 shadow-lg">
      <div class="w-20 h-20 flex items-center justify-center border-4 border-green-500 rounded-xl bg-green-50 shadow-sm">
        <span class="text-4xl font-mono font-bold text-green-600">6</span>
      </div>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">double boxed result</span>
  </div>

</div>

<div class="mt-8 text-lg">
The function takes a <b>plain</b> value, but it hands back a <b>boxed</b> one.
</div>

<div class="mt-8 text-lg" v-click=2>
What if we use <code>map</code>?

It always wraps the result in a box, but the function already returns a box…
</div>

<div class="mt-4 text-center text-xl text-red-700" v-click=3>
A box inside a box. The Matrioska effect. 🪆
</div>


---
transition: slide-up
---

# `bind` function

<div class="flex items-center justify-center gap-4 mt-8">

  <div class="flex flex-col items-center gap-3">
    <div class="w-24 h-24 flex items-center justify-center border-4 border-blue-500 rounded-xl bg-blue-50 shadow-md">
      <span class="text-4xl font-mono font-bold text-blue-600">2</span>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">boxed value</span>
  </div>

  <div class="text-3xl text-gray-300 i-carbon-arrow-right mx-1" />

  <div class="flex flex-col items-center gap-3">
    <div class="h-24 flex items-center justify-center px-2 gap-3">
      <span class="text-4xl font-mono font-bold text-pink-500 whitespace-nowrap">n =></span>
      <div class="w-28 h-20 flex items-center justify-center border-4 border-pink-500 rounded-xl bg-pink-50">
        <span class="text-3xl font-mono font-bold text-pink-500">n * 3</span>
      </div>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">function returning a box</span>
  </div>

  <div class="text-3xl font-light text-gray-200 mx-2" v-click=2>=</div>

  <div class="flex flex-col items-center gap-3" v-click=2>
    <div class="w-24 h-24 flex items-center justify-center border-4 border-green-500 rounded-xl bg-green-50 shadow-md">
      <span class="text-4xl font-mono font-bold text-green-600">6</span>
    </div>
    <span class="text-[10px] font-mono opacity-40 uppercase tracking-tighter">flat result</span>
  </div>

</div>


<div class="mt-6" v-click=1>

1. Opens the first box
2. Applies the box-returning function
3. **Flattens** the two boxes into one

</div>

<div class="mt-3 text-center text-xl text-gray-700" v-click=3>
<code>bind</code> takes a <b>plain function</b>, applies it to a <b>boxed value</b> and <b>flattens</b> 
</div>

<div class="mt-6 text-center text-lg text-gray-700" v-click=4>
box of box &nbsp;→&nbsp; single box
</div>


---
transition: slide-up
---

# Monad

A <code>Monad</code> is any data type that defines how <code>Return</code> and <code>Bind</code> apply to it:
1. <code>Return</code>: exactly like the Applicative, put a plain value into a box.
2. <code>Bind</code> (or <code>&gt;&gt;=</code>, <code>flatMap</code>): apply a box-returning function to a boxed value and flatten the result.


```haskell
return::     a     ->        fa
           takes         and puts it 
          a value          in a box

bind::      fa      ->     (a -> fb)     ->       fb
          takes        and a box-returning    and returns
        a functor          function         a flattened box
```

<br>

```csharp
public static Box<T> Return<T>(T val);

public static Box<TOut> Bind<TIn, TOut>(this 
  Box<TIn> val, 
  Func<TIn, Box<TOut>> f);
```

---
transition: slide-up
---

# Implementation: IEnumerable

Implementing `Return` and `Bind` for `IEnumerable`

```csharp
public static IEnumerable<T> Return<T>(T val)
{
    yield return val;
}
```
<br>
```csharp
public static IEnumerable<TOut> Bind<TIn, TOut>(this
    IEnumerable<TIn> values, 
    Func<TIn, IEnumerable<TOut>> f)
{
    foreach (var val in values)
    {
        IEnumerable<TOut> innerValues = f(val);
        foreach (var innerVal in innerValues)
        {
            yield return innerVal;
        }
    }
}
```

---

# Implementation: Option

Implementing `Return` and `Bind` for the `Option` type.

```csharp
public static Option<T> Return<T>(T val) => new Some<T>(val);
```
<br>

```csharp
public static Option<TOut> Bind<TIn, TOut>(this
    Option<TIn> val, 
    Func<TIn, Option<TOut>> f) =>
    
    val switch
    {
        Some<TIn>(var x) => f(x),
        _ => new None<TOut>()
    };
```

---
layout: center
class: text-center
---

<h1 class="text-7xl font-normal tracking-tighter mt-4">
  Breathe
</h1>

---
transition: slide-up
---

# Four tools for the box

```text
 return :   A                           →   Box<A>     put a value in a box
 map    :   Box<A>     ×  A → B         →   Box<B>     transform what's inside
 apply  :   Box<A → B> ×  Box<A>        →   Box<B>     combine two boxes
 bind   :   Box<A>     ×  A → Box<B>    →   Box<B>     chain without nesting
```

---
layout: center
transition: slide-up
---

# The Power Hierarchy

<div class="mt-8 text-3xl text-center text-gray-700">
  <b>Monad</b> <span class="mx-4">&gt;</span> <b>Applicative</b> <span class="mx-4">&gt;</span> <b>Functor</b>
</div>

<div v-click class="mt-12 space-y-6 text-lg">

If you have an **Applicative**, you get a **Functor** for free:
```csharp
Map(val, f) == Apply(Return(f), val)
```

If you have a **Monad**, you get an **Applicative** for free:
```csharp
Apply(f, val) == Bind(f, func => Bind(val, v => Return(func(v))))
```

</div>

<div v-click class="mt-12 text-xl text-center text-orange-600 font-bold">
Monads are the most powerful.<br>So why not use them everywhere?
</div>

---
layout: center
transition: slide-up
---

# Applicative vs Monad 

Look at the arguments

<div class="grid grid-cols-2 gap-12 mt-12">

<div class="p-8 bg-blue-50 rounded-xl shadow-sm border border-blue-100">

### 1. Applicative (Combine)

<div class="text-2xl mt-8 mb-8 font-mono font-bold text-center text-blue-700">
  Box&lt;A&gt;, Box&lt;B&gt;
</div>

<ul class="space-y-4 text-lg text-gray-700 list-disc ml-6">
  <li>Both boxes <b>already exist</b>.</li>
  <li>Neither box needs the other to be created.</li>
</ul>

<div class="mt-12 text-center text-blue-600 font-black text-2xl uppercase tracking-widest">
  Parallel
</div>
    
</div>

<div class="p-8 bg-green-50 rounded-xl shadow-sm border border-green-100">

### 2. Monad (Bind)

<div class="text-2xl mt-8 mb-8 font-mono font-bold text-center text-green-700">
  Box&lt;A&gt;, A &rarr; Box&lt;B&gt;
</div>

<ul class="space-y-4 text-lg text-gray-700 list-disc ml-6">
  <li>Only the <b>first box</b> exists.</li>
  <li>To create the second box, you <b>must</b> extract the inner <code>A</code> first.</li>
</ul>

<div class="mt-12 text-center text-green-600 font-black text-2xl uppercase tracking-widest">
  Sequential
</div>

</div>

</div>

---
layout: center
transition: slide-up
---

# Applicative vs Monad

The effects

<div class="grid grid-cols-2 gap-8 mt-8">

<div v-click class="p-6 bg-blue-50 rounded-lg">

### Applicative (Combine)
<div class="text-orange-600 font-bold mb-4">Independent contexts</div>

- Step A and Step B don't know about each other.
- They can be computed in **parallel**.
- If one fails, the other can still run (we can collect *all* errors).

</div>

<div v-click class="p-6 bg-green-50 rounded-lg">

### Monad (Chain)
<div class="text-orange-600 font-bold mb-4">Sequential contexts</div>

- Step B **depends** on the result of Step A.
- They **must** be computed in order.
- If Step A fails, Step B is **never executed** (fail-fast).

</div>

</div>

---
layout: center
class: text-center
---

<h2 class="opacity-60 font-light italic italic">And after all this effort...</h2>

<v-click>
<h1 class="text-7xl font-semibold tracking-tighter mt-4">
  the dessert!
</h1>

<div class="text-5xl mt-8 animate-bounce">
  🍰 🍩 🍮
</div>
</v-click>

---
layout: center
transition: slide-up
---

# You already know them
And you already use them!

<div class="mt-6 text-sm">

<table class="w-full border-collapse" v-click="1">
<thead>
<tr class="border-b-2 border-gray-400">
<th class="text-left p-2"><b>Concept</b></th>
<th class="text-left p-2"><b>FP operation</b></th>
<th class="text-left p-2"><b>What it does</b></th>
<th class="text-left p-2" v-click="2"><b>C# counterpart</b></th>
</tr>
</thead>
<tbody>
<tr class="border-b border-gray-200">
<td class="p-2"><b>Context</b></td>
<td class="p-2"><code>return</code> / <code>unit</code></td>
<td class="p-2">Put a value in a box</td>
<td class="p-2" v-click="3"><code>Task.FromResult</code>, <code>new[] { x }</code>, <code>Result.Success(x)</code></td>
</tr>
<tr class="border-b border-gray-200">
<td class="p-2"><b>Functor</b></td>
<td class="p-2"><code>map</code> / <code>fMap</code></td>
<td class="p-2">Transform the inner value</td>
<td class="p-2" v-click="4"><code>Select(x =&gt; ...)</code></td>
</tr>
<tr class="border-b border-gray-200">
<td class="p-2"><b>Applicative</b></td>
<td class="p-2"><code>apply</code> / <code>pure</code></td>
<td class="p-2">Combine multiple contexts</td>
<td class="p-2" v-click="5"><code>Task.WhenAll</code>, <code>Zip</code>, <code>Result.Combine</code></td>
</tr>
<tr>
<td class="p-2"><b>Monad</b></td>
<td class="p-2"><code>bind</code> / <code>flatMap</code></td>
<td class="p-2">Chain and flatten</td>
<td class="p-2" v-click="6"><code>SelectMany(x =&gt; ...)</code>, <code>await</code>, <code>?.</code></td>
</tr>
</tbody>
</table>

</div>

<div v-click="7" class="mt-10 text-center text-2xl text-orange-600">
You were just missing the <b>names</b>.
</div>

---
layout: center
transition: slide-up
---

# The main monads in C#

<div class="text-base mt-4 space-y-2">

- <code>Nullable&lt;T&gt;</code> — a <code>T</code> that could be <code>null</code>
- <code>Func&lt;T&gt;</code> — a <code>T</code> that can be computed on demand
- <code>Lazy&lt;T&gt;</code> — a <code>T</code> that can be computed on demand once, then cached
- <code>Task&lt;T&gt;</code> — a <code>T</code> that is being computed asynchronously and will be available in the future, if it isn't already
- <code>IEnumerable&lt;T&gt;</code> — an ordered, read-only sequence of zero or more <code>T</code>s

</div>


From [<code>CSharpFunctionalExtensions</code>](https://github.com/vkhorikov/CSharpFunctionalExtensions):

- <code>Maybe&lt;T&gt;</code> — a <code>T</code> that may or may not exist (null-safe alternative)
- <code>Result&lt;T&gt;</code> — a <code>T</code> that may have failed, with an error attached
<div v-click class="text-base mt-2 pt-2 border-t border-gray-300 space-y-2">

**Other popular C# libraries featuring Monads:**
- [LanguageExt](https://github.com/louthy/language-ext) — The most robust functional programming library for C#, featuring a wide array of Monads (Option, Either, Seq, etc.) inspired by Haskell.
- [ErrorOr](https://github.com/amantinband/error-or) — A simple, fluent Result monad implementation focused strictly on error handling.
- [Optional](https://github.com/nlkl/Optional) — A robust Option/Maybe monad implementation for C# to deal with missing values without nulls.

</div>

---
transition: slide-up
---

# Functors in action

```csharp
// 1. Transform each order into its total        (IEnumerable)
IEnumerable<decimal> totals = orders.Select(o => o.Total);

// 2. Uppercase a name, if it exists             (Maybe)
Maybe<string> upperName = maybeName.Map(n => n.ToUpper());

// 3. Format a date once the async call completes (Task)
Task<string> formatted = fetchDateAsync.Map(d => d.ToString("yyyy-MM-dd"));
```

<div v-click class="mt-8 text-center text-lg text-gray-700">
A <b>plain function</b> applied <b>inside</b> a box.
</div>

---
transition: slide-up
---

# Applicatives in action

```csharp
// 1. Two independent async calls, run in parallel
await Task.WhenAll(SaveUser(u), SendWelcomeEmail(u));

// 2. Zip two sequences element-by-element
IEnumerable<decimal> subtotals = prices.Zip(quantities, (p, q) => p * q);

// 3. Validate independent fields, collecting ALL errors
Result form = Result.Combine(
    ValidateEmail(req.Email),
    ValidateAge(req.Age),
    ValidateName(req.Name));
```

<div v-click class="mt-6 text-center text-lg text-gray-700">
<b>Independent</b> boxed values, combined into one.
</div>

---

# Monads in action

```csharp
// 1. Chained async — each step depends on the previous
var user    = await FetchUser(id);
var orders  = await FetchOrders(user.Id);
var invoice = await BuildInvoice(orders);

// 2. Null-safe property chain
string? city = user?.Address?.City;

// 3. Flatten a list of lists
IEnumerable<Employee> all = departments.SelectMany(d => d.Employees);
```

<div v-click class="mt-6 text-center text-lg text-gray-700">
Each step <b>returns a box</b>. The chain stays <b>flat</b>.
</div>


---
layout: center
transition: slide-up
---

# The limits of `Bind`

---
transition: slide-up
---

# Chaining too many binds...

```csharp
public Task<string?> GetCityOfFirstOrder(int userId) =>
    FetchUser(userId)
        .Bind(user => FetchFirstOrder(user.Id)
            .Bind(order => FetchAddress(order.AddressId)
                .Map(address => address.City)));
```

<v-click>

<div class="text-red-700 mt-8 text-xl text-center">
Nesting. Indentation. Arrows. Parentheses.<br>
As the chain grows, it becomes harder to read.
</div>

</v-click>
<v-click>
<div class="text-gray-700 mt-4 text-lg text-center">
We need a help.
</div>

</v-click>

---
transition: slide-up
transition: slide-up
---

# In other languages
How it is handled 

Haskell - <code>do</code>notation

```haskell
getCityOfFirstOrder :: Int -> IO (Maybe String)
getCityOfFirstOrder userId = do
  user    <- fetchUser userId
  order   <- fetchFirstOrder (userIdOf user)
  address <- fetchAddress (addressIdOf order)
  return (cityOf address)
```

Scala - <code>for</code>comprehension

```scala
def getCityOfFirstOrder(userId: Int): Future[Option[String]] =
  for {
    user    <- fetchUser(userId)
    order   <- fetchFirstOrder(user.id)
    address <- fetchAddress(order.addressId)
  } yield address.city
```

---
transition: slide-up
layout: center
class: text-center
---

<h1 class="text-7xl font-normal tracking-tighter mt-4">
  Who will be our C# hero?
</h1>
---
transition: slide-up
---

# The LINQ QUERY EXPRESSION SYNTAX

<v-click>

```csharp
public Task<string?> GetCityOfFirstOrder(int userId)
{
    return
        from user in FetchUser(userId)
        from order in FetchFirstOrder(user.Id)
        from address in FetchAddress(order.AddressId)
        select address.City;
}
```

</v-click>

<v-click>

<div class="text-gray-700 mt-4 text-lg text-center">
The hero we need, but don't deserve
</div>

</v-click>


<v-click>
<div class="text-gray-700 mt-4 text-lg text-center">
It's a <b>Monad Comprehension</b>: a universal language for <b>chaining contexts</b>.

</div>
</v-click>

<v-click>
<div class="text-gray-700 mt-4 text-lg text-center">
Every <code>from</code> is a <code>SelectMany</code> (<code>bind</code>)

</div>
</v-click>


<v-click>
<div class="text-gray-700 mt-4 text-lg text-center">
Every <code>select</code> is a <code>Select</code> (<code>map</code>)
</div>

</v-click>


<v-click>
<div class="text-gray-700 mt-4 text-lg text-center">
That's it

</div>
</v-click>

---
transition: slide-up
---

# Beautify

<div class="ext-gray-700 mt-4 text-lg">We have the following methods that we want to chain:</div>

```csharp
public static Result<User> GetUser(int id) => Result<User>.Success(new User(id, "Marco"));
public static Result<Address> GetAddress(User u) => Result<Address>.Success(new Address("Mengoli"));
public static Result<double> GetShippingCost(Address a) => Result<double>.Success(100.00);
```

<v-click>

<div class="ext-gray-700 mt-4 text-lg">From chained <code>Bind</code>s:</div>

```csharp
var result = GetUser(1)
    .SelectMany(user => GetAddress(user)
        .SelectMany(address => GetShippingCost(address)
            .SelectMany(cost => Result<string>.Success($"User: {user.Name}, Cost: {cost}"))));
```


</v-click>
<v-click>

<div class="ext-gray-700 mt-4 text-lg">To monadic comprehension:</div>

```csharp
Result<string> result = 
    from user in GetUser(1)
    from address in GetAddress(user)
    from cost in GetShippingCost(address)
    select $"User: {user.Name}, Cost: {cost}";
```
</v-click>

---
transition: slide-up
---

# This show was kindly sponsored by

The implementation of the <code>SelectMany</code>s

```csharp
public static Result<U> SelectMany<T, U>(
    this Result<T> source, 
    Func<T, Result<U>> bind) 
    => source.IsSuccess 
        ? bind(source.Value) 
        : Result<U>.Failure(source.Error);

public static Result<V> SelectMany<T, U, V>(
    this Result<T> source,
    Func<T, Result<U>> bind,
    Func<T, U, V> project)
    => !source.IsSuccess 
        ? Result<V>.Failure(source.Error) 
        : bind(source.Value) is var next && !next.IsSuccess 
            ? Result<V>.Failure(next.Error) 
            : Result<V>.Success(project(source.Value, next.Value));
```


---
transition: slide-up
---

# Another example: Task

From repeated blocks

```csharp
public async Task<string?> GetCityOfFirstOrder(int userId)
{
    var user = await FetchUser(userId);
    if (user is null) return null;

    var order = await FetchFirstOrder(user.Id);
    if (order is null) return null;

    var address = await FetchAddress(order.AddressId);
    if (address is null) return null;

    return address.City;
}
```

<div class="text-red-700 mt-2">Nesting, null-checks, noise</div>


---
transition: slide-up
---

# Or monadic Bind


```csharp
public Task<string?> GetCityOfFirstOrder(int userId)
{
    return FetchUser(userId)
        .ContinueWith(userTask =>
            userTask.Result is null
                ? Task.FromResult<string?>(null)
                : FetchFirstOrder(userTask.Result.Id)
        )
        .Unwrap()
        .ContinueWith(orderTask =>
            orderTask.Result is null
                ? Task.FromResult<string?>(null)
                : FetchAddress(orderTask.Result.AddressId)
        )
        .Unwrap()
        .ContinueWith(addressTask =>
            addressTask.Result?.City
        );
}

```

<div class="text-red-700 mt-2">Even worse</div>

---
transition: slide-up
---

# …to Monad Comprehension

```csharp
public Task<string?> GetCityOfFirstOrder(int userId) =>
    from user    in FetchUser(userId)
    from order   in FetchFirstOrder(user.Id)
    from address in FetchAddress(order.AddressId)
    select address.City;
```

<v-click>

The flow is **linear**.

The `Bind`s are **implicit**.

Business logic **emerges** from the code.

</v-click>

---
layout: center
---

# Railway Oriented Programming

Code as a **railway** 🛤️

---

# Two tracks

<img src="/global/railway-opaque.png" class="mx-auto"/>

<v-click>

If **any** step fails, the train **automatically jumps** onto the failure track.

No `try/catch`. No cascades of `if (success)`.

</v-click>

---

# Before: nested `if`s

```csharp
public HttpResponse CreateUser(string email, string name)
{
    if (string.IsNullOrEmpty(email))
        return BadRequest("Empty email");

    if (!email.Contains("@"))
        return BadRequest("Invalid email");

    var existing = _repo.FindByEmail(email);
    if (existing is not null)
        return Conflict("Email already registered");

    try
    {
        var user = _repo.Save(new User(email, name));
        return Ok(user);
    }
    catch (DbException ex)
    {
        return ServerError(ex.Message);
    }
}
```

---

# After: the railway

```csharp {all|2|3|4|5|6|all}
public HttpResponse CreateUser(string email, string name) =>
    Email.Create(email)
        .Ensure(e => !_repo.Exists(e), "Email already registered")
        .Map(e => new User(e, name))
        .Bind(u => _repo.TrySave(u))
        .Match(
            onSuccess: user => Ok(user),
            onFailure: err  => BadRequest(err));
```

<v-click>

Every step returns `Result<T>`. One fails → the rest is **skipped**.

Library: [`CSharpFunctionalExtensions`](https://github.com/vkhorikov/CSharpFunctionalExtensions)

</v-click>

---
layout: center
---

# Solid foundations

The railway works only if the tracks are **straight**.

---

# Immutability — `record`

```csharp
// ❌ Mutable: who changed it? when? why?
public class User
{
    public string Email { get; set; }
    public string Name  { get; set; }
}

user.Email = "other@domain.com"; // 🤷
```

<v-click>

```csharp
// ✅ Immutable: data doesn't change, it EVOLVES
public record User(string Email, string Name);

var updated = user with { Email = "other@domain.com" };
// `user` is untouched. `updated` is a new instance.
```

</v-click>

---

# Pure functions

<v-clicks>

A function is **pure** if:

- Same input → **same output**, always.
- **No side effects** (no I/O, no globals, no exceptions).

</v-clicks>

<v-click>

```csharp
// ❌ Impure
public int Next() => _counter++;

// ✅ Pure
public int Next(int counter) => counter + 1;
```

</v-click>

<v-click>

Pure functions are **trivial to test** and **trivial to reason about**.

</v-click>

---

# Smart Constructors

> An object must never be allowed to exist in an **invalid** state.

<v-click>

```csharp
// ❌ Any string goes. Boom at runtime.
public record Email(string Value);

new Email("not-an-email"); // 🙃 compiles happily
```

</v-click>

<v-click>

```csharp
// ✅ Centralized validation, error as a VALUE
public record Email
{
    public string Value { get; }
    private Email(string value) => Value = value;

    public static Result<Email> Create(string input) =>
        string.IsNullOrWhiteSpace(input) ? Result.Failure<Email>("empty")
        : !input.Contains('@')           ? Result.Failure<Email>("invalid")
        : Result.Success(new Email(input));
}
```

</v-click>

---
layout: center
---

# The big practical example

Let's put it all together.

---

# The requirement

<v-clicks>

1. We receive a **dirty string** (user input).
2. We **clean** it (trim, lowercase).
3. We **validate** it with a Smart Constructor.
4. We make an **async call** to an external API.
5. We **log** the step.
6. We return an **HTTP response**.

</v-clicks>

<div v-click class="mt-8 text-orange-600">
Zero try/catch. Zero nested ifs. Type-safe end-to-end.
</div>

---

# The implementation

````md magic-move {lines: true}
```csharp
// Step 1 — just the cleanup
public Task<IResult> Handle(string rawEmail) =>
    rawEmail.Trim().ToLowerInvariant();
    // 🤔 where do we go from here?
```

```csharp
// Step 2 — validation via Smart Constructor
public Task<IResult> Handle(string rawEmail) =>
    Email.Create(rawEmail.Trim().ToLowerInvariant());
    // Result<Email>, but we still need async...
```

```csharp
// Step 3 — Map to clean, Bind to validate
public Task<IResult> Handle(string rawEmail) =>
    Result.Success(rawEmail)
        .Map(s => s.Trim().ToLowerInvariant())
        .Bind(Email.Create);
```

```csharp
// Step 4 — async call to the API
public Task<IResult> Handle(string rawEmail) =>
    Result.Success(rawEmail)
        .Map(s => s.Trim().ToLowerInvariant())
        .Bind(Email.Create)
        .Bind(email => _userApi.FetchProfileAsync(email));
```

```csharp
// Step 5 — Tap for logging (side-effect, ISOLATED)
public Task<IResult> Handle(string rawEmail) =>
    Result.Success(rawEmail)
        .Map(s => s.Trim().ToLowerInvariant())
        .Bind(Email.Create)
        .Bind(email => _userApi.FetchProfileAsync(email))
        .Tap(profile => _logger.Info($"Fetched {profile.Id}"));
```

```csharp
// Step 6 — Match to close the railway into an HTTP response
public Task<IResult> Handle(string rawEmail) =>
    Result.Success(rawEmail)
        .Map(s => s.Trim().ToLowerInvariant())
        .Bind(Email.Create)
        .Bind(email => _userApi.FetchProfileAsync(email))
        .Tap(profile => _logger.Info($"Fetched {profile.Id}"))
        .Match(
            onSuccess: Results.Ok,
            onFailure: Results.BadRequest);
```
````

---
layout: center
---

# Look at it again

```csharp
public Task<IResult> Handle(string rawEmail) =>
    Result.Success(rawEmail)
        .Map(s => s.Trim().ToLowerInvariant())
        .Bind(Email.Create)
        .Bind(email => _userApi.FetchProfileAsync(email))
        .Tap(profile => _logger.Info($"Fetched {profile.Id}"))
        .Match(
            onSuccess: Results.Ok,
            onFailure: Results.BadRequest);
```

<div v-click class="text-xl mt-8 text-cyan-600">
Pure masterclass
</div>

<div v-click class="text-lg mt-4 text-gray-700">
Readable. Type-safe. Testable. Zero ceremony.
</div>

---
layout: center
class: text-center
---

# Conclusion

<v-clicks>

<div class="text-xl mt-8">
FP in C# is <b>feasible</b>, and you are already using it.
</div>

<div class="text-xl mt-4">
It's a tool to write code that's<br>
<b>more robust, more testable</b>, and paradoxically<br>
<b>simpler</b>.
</div>

<div class="text-lg mt-12 text-gray-600">
You don't need categories, endofunctors, or monoids.
</div>

<div class="text-lg mt-2 text-gray-600">
You just need to look at your <code>Select</code> and <code>SelectMany</code> with new eyes.
</div>

</v-clicks>

---
layout: default
---

# ⚠️ But also... With great power comes great... risks

Overusing these patterns can lead to pitfalls:

* **The "Stacked Monads" Problem**: 
    C# **doesn't** handle **nested** contexts well. 
    `Task<Option<IEnumerable<User>>>` quickly becomes a nightmare to unwrap without native Monad Transformers.

* **Team Mindset & Culture**: 
    They work beautifully in a team that with an **open mind** towards change and a **willingness** to learn new paradigms. Without this shared mindset, "elegant" code becomes "alien" code for your colleagues.
* **Performance**: 
    Every "Box" 📦 is an object allocation. In high-throughput paths, too many wrappers can impact the GC.

> **Rule of thumb:** Use functional patterns to simplify logic and reduce errors, not to show off how much Category Theory you know.

> **Keep it "C#-idiomatic" where possible.**

---
layout: center
class: text-center
---

# Thank you. 🙏

<div class="mt-8 text-gray-600">
Questions?
</div>

<div class="mt-8 text-gray-600">
(or let's go grab that dessert!)
<div class="text-5xl mt-8 animate-bounce">
  🍰 🍩 🍮
</div>
</div>


