---
title: A Fistful of Monads
---




<center>
<a href="functors-applicative-functors-and-monoids"
  class="prevlink">Functors, Applicative Functors and Monoids</a>
<a href="/pages/chapters">Table of contents</a>
 <a href="for-a-few-monads-more" class="nxtlink">For a Few Monads More</a>
</center>


When we first talked about functors, we saw that they were a useful
concept for values that can be mapped over. Then, we took that concept
one step further by introducing applicative functors, which allow us to
view values of certain data types as values with contexts and use normal
functions on those values while preserving the meaning of those
contexts.

In this chapter, we'll learn about monads, which are just beefed up
applicative functors, much like applicative functors are only beefed up
functors.

<img src="/assets/smugpig.png" class="right"
width="307" height="186" alt="more cool than u" />

When we started off with functors, we saw that it's possible to map
functions over various data types. We saw that for this purpose, the
<span class="fixed">Functor</span> type class was introduced and it had
us asking the question: when we have a function of type
<span class="fixed">a -&gt; b</span> and some data type
<span class="fixed">f a</span>, how do we map that function over the
data type to end up with <span class="fixed">f b</span>? We saw how to
map something over a <span class="fixed">Maybe a</span>, a list
<span class="fixed">\[a\]</span>, an <span class="fixed">IO a</span>
etc. We even saw how to map a function <span class="fixed">a -&gt;
b</span> over other functions of type <span class="fixed">r -&gt; a</span>
to get functions of type <span class="fixed">r -&gt; b</span>. To answer
this question of how to map a function over some data type, all we had
to do was look at the type of <span class="fixed">fmap</span>:

``` haskell:hs
fmap :: (Functor f) => (a -> b) -> f a -> f b
```

And then make it work for our data type by writing the appropriate
<span class="fixed">Functor</span> instance.

Then we saw a possible improvement of functors and said, hey, what if
that function <span class="fixed">a -&gt; b</span> is already wrapped
inside a functor value? Like, what if we have <span class="fixed">Just
(\*3)</span>, how do we apply that to <span class="fixed">Just 5</span>?
What if we don't want to apply it to <span class="fixed">Just 5</span>
but to a <span class="fixed">Nothing</span> instead? Or if we have
<span class="fixed">\[(\*2),(+4)\]</span>, how would we apply that to
<span class="fixed">\[1,2,3\]</span>? How would that work even? For
this, the <span class="fixed">Applicative</span> type class was
introduced, in which we wanted the answer to the following type:

``` haskell:hs
(<*>) :: (Applicative f) => f (a -> b) -> f a -> f b
```

We also saw that we can take a normal value and wrap it inside a data
type. For instance, we can take a <span class="fixed">1</span> and wrap
it so that it becomes a <span class="fixed">Just 1</span>. Or we can
make it into a <span class="fixed">\[1\]</span>. Or an I/O action that
does nothing and just yields <span class="fixed">1</span>. The function
that does this is called <span class="fixed">pure</span>.

Like we said, an applicative value can be seen as a value with an added
context. A *fancy* value, to put it in technical terms. For instance,
the character <span class="fixed">'a'</span> is just a normal character,
whereas <span class="fixed">Just 'a'</span> has some added context.
Instead of a <span class="fixed">Char</span>, we have a
<span class="fixed">Maybe Char</span>, which tells us that its value
might be a character, but it could also be an absence of a character.

It was neat to see how the <span class="fixed">Applicative</span> type
class allowed us to use normal functions on these values with context
and how that context was preserved. Observe:

``` haskell:hs
ghci> (*) <$> Just 2 <*> Just 8
Just 16
ghci> (++) <$> Just "klingon" <*> Nothing
Nothing
ghci> (-) <$> [3,4] <*> [1,2,3]
[2,1,0,3,2,1]
```

Ah, cool, so now that we treat them as applicative values,
<span class="fixed">Maybe a</span> values represent computations that
might have failed, <span class="fixed">\[a\]</span> values represent
computations that have several results (non-deterministic computations),
<span class="fixed">IO a</span> values represent values that have
side-effects, etc.

Monads are a natural extension of applicative functors and with them
we're concerned with this: if you have a value with a context,
<span class="fixed">m a</span>, how do you apply to it a function that
takes a normal <span class="fixed">a</span> and returns a value with a
context? That is, how do you apply a function of type
<span class="fixed">a -&gt; m b</span> to a value of type
<span class="fixed">m a</span>? So essentially, we will want this
function:

``` haskell:hs
(>>=) :: (Monad m) => m a -> (a -> m b) -> m b
```

**If we have a fancy value and a function that takes a normal value but
returns a fancy value, how do we feed that fancy value into the
function?** This is the main question that we will concern ourselves
when dealing with monads. We write <span class="fixed">m a</span>
instead of <span class="fixed">f a</span> because the
<span class="fixed">m</span> stands for
<span class="fixed">Monad</span>, but monads are just applicative
functors that support <span class="fixed">&gt;&gt;=</span>. The
<span class="fixed">&gt;&gt;=</span> function is pronounced as *bind*.

When we have a normal value <span class="fixed">a</span> and a normal
function <span class="fixed">a -&gt; b</span> it's really easy to feed the
value to the function — you just apply the function to the value
normally and that's it. But when we're dealing with values that come
with certain contexts, it takes a bit of thinking to see how these fancy
values are fed to functions and how to take into account their behavior,
but you'll see that it's easy as one two three.

<span id="getting-our-feet-wet-with-maybe"></span>

## Getting our feet wet with Maybe

<img src="/assets/buddha.png" class="left"
width="302" height="387" alt="monads, grasshoppa" />

Now that we have a vague idea of what monads are about, let's see if we
can make that idea a bit less vague.

Much to no one's surprise, <span class="fixed">Maybe</span> is a monad,
so let's explore it a bit more and see if we can combine it with what we
know about monads.



Make sure you understand
[applicatives](functors-applicative-functors-and-monoids#applicative-functors)
at this point. It's good if you have a feel for how the various
<span class="fixed">Applicative</span> instances work and what kind of
computations they represent, because monads are nothing more than taking
our existing applicative knowledge and upgrading it.



A value of type <span class="fixed">Maybe a</span> represents a value of
type <span class="fixed">a</span> with the context of possible failure
attached. A value of <span class="fixed">Just "dharma"</span> means that
the string <span class="fixed">"dharma"</span> is there whereas a value
of <span class="fixed">Nothing</span> represents its absence, or if you
look at the string as the result of a computation, it means that the
computation has failed.

When we looked at <span class="fixed">Maybe</span> as a functor, we saw
that if we want to <span class="fixed">fmap</span> a function over it,
it gets mapped over the insides if it's a
<span class="fixed">Just</span> value, otherwise the
<span class="fixed">Nothing</span> is kept because there's nothing to
map it over!

Like this:

``` haskell:hs
ghci> fmap (++"!") (Just "wisdom")
Just "wisdom!"
ghci> fmap (++"!") Nothing
Nothing
```

As an applicative functor, it functions similarly. However, applicatives
also have the function wrapped. <span class="fixed">Maybe</span> is an
applicative functor in such a way that when we use
<span class="fixed">&lt;\*&gt;</span> to apply a function inside a
<span class="fixed">Maybe</span> to a value that's inside a
<span class="fixed">Maybe</span>, they both have to be
<span class="fixed">Just</span> values for the result to be a
<span class="fixed">Just</span> value, otherwise the result is
<span class="fixed">Nothing</span>. It makes sense because if you're
missing either the function or the thing you're applying it to, you
can't make something up out of thin air, so you have to propagate the
failure:

``` haskell:hs
ghci> Just (+3) <*> Just 3
Just 6
ghci> Nothing <*> Just "greed"
Nothing
ghci> Just ord <*> Nothing
Nothing
```

When we use the applicative style to have normal functions act on
<span class="fixed">Maybe</span> values, it's similar. All the values
have to be <span class="fixed">Just</span> values, otherwise it's all
for <span class="fixed">Nothing</span>!

``` haskell:hs
ghci> max <$> Just 3 <*> Just 6
Just 6
ghci> max <$> Just 3 <*> Nothing
Nothing
```

And now, let's think about how we would do
<span class="fixed">&gt;&gt;=</span> for <span class="fixed">Maybe</span>.
Like we said, <span class="fixed">&gt;&gt;=</span> takes a monadic value,
and a function that takes a normal value and returns a monadic value and
manages to apply that function to the monadic value. How does it do
that, if the function takes a normal value? Well, to do that, it has to
take into account the context of that monadic value.

In this case, <span class="fixed">&gt;&gt;=</span> would take a
<span class="fixed">Maybe a</span> value and a function of type
<span class="fixed">a -&gt; Maybe b</span> and somehow apply the function
to the <span class="fixed">Maybe a</span>. To figure out how it does
that, we can use the intuition that we have from
<span class="fixed">Maybe</span> being an applicative functor. Let's say
that we have a function <span class="fixed">\x -&gt; Just (x+1)</span>. It
takes a number, adds <span class="fixed">1</span> to it and wraps it in
a <span class="fixed">Just</span>:

``` haskell:hs
ghci> (\x -> Just (x+1)) 1
Just 2
ghci> (\x -> Just (x+1)) 100
Just 101
```

If we feed it <span class="fixed">1</span>, it evaluates to
<span class="fixed">Just 2</span>. If we give it the number
<span class="fixed">100</span>, the result is <span class="fixed">Just
101</span>. Very straightforward. Now here's the kicker: how do we feed
a <span class="fixed">Maybe</span> value to this function? If we think
about how <span class="fixed">Maybe</span> acts as an applicative
functor, answering this is pretty easy. If we feed it a
<span class="fixed">Just</span> value, take what's inside the
<span class="fixed">Just</span> and apply the function to it. If give it
a <span class="fixed">Nothing</span>, hmm, well, then we're left with a
function but <span class="fixed">Nothing</span> to apply it to. In that
case, let's just do what we did before and say that the result is
<span class="fixed">Nothing</span>.

Instead of calling it <span class="fixed">&gt;&gt;=</span>, let's call it
<span class="fixed">applyMaybe</span> for now. It takes a
<span class="fixed">Maybe a</span> and a function that returns a
<span class="fixed">Maybe b</span> and manages to apply that function to
the <span class="fixed">Maybe a</span>. Here it is in code:

``` haskell:hs
applyMaybe :: Maybe a -> (a -> Maybe b) -> Maybe b
applyMaybe Nothing f  = Nothing
applyMaybe (Just x) f = f x
```

Okay, now let's play with it for a bit. We'll use it as an infix
function so that the <span class="fixed">Maybe</span> value is on the
left side and the function on the right:

``` haskell:hs
ghci> Just 3 `applyMaybe` \x -> Just (x+1)
Just 4
ghci> Just "smile" `applyMaybe` \x -> Just (x ++ " :)")
Just "smile :)"
ghci> Nothing `applyMaybe` \x -> Just (x+1)
Nothing
ghci> Nothing `applyMaybe` \x -> Just (x ++ " :)")
Nothing
```

In the above example, we see that when we used
<span class="fixed">applyMaybe</span> with a
<span class="fixed">Just</span> value and a function, the function
simply got applied to the value inside the
<span class="fixed">Just</span>. When we tried to use it with a
<span class="fixed">Nothing</span>, the whole result was
<span class="fixed">Nothing</span>. What about if the function returns a
<span class="fixed">Nothing</span>? Let's see:

``` haskell:hs
ghci> Just 3 `applyMaybe` \x -> if x > 2 then Just x else Nothing
Just 3
ghci> Just 1 `applyMaybe` \x -> if x > 2 then Just x else Nothing
Nothing
```

Just what we expected. If the monadic value on the left is a
<span class="fixed">Nothing</span>, the whole thing is
<span class="fixed">Nothing</span>. And if the function on the right
returns a <span class="fixed">Nothing</span>, the result is
<span class="fixed">Nothing</span> again. This is very similar to when
we used <span class="fixed">Maybe</span> as an applicative and we got a
<span class="fixed">Nothing</span> result if somewhere in there was a
<span class="fixed">Nothing</span>.

It looks like that for <span class="fixed">Maybe</span>, we've figured
out how to take a fancy value and feed it to a function that takes a
normal value and returns a fancy one. We did this by keeping in mind
that a <span class="fixed">Maybe</span> value represents a computation
that might have failed.

You might be asking yourself, how is this useful? It may seem like
applicative functors are stronger than monads, since applicative
functors allow us to take a normal function and make it operate on
values with contexts. We'll see that monads can do that as well because
they're an upgrade of applicative functors, and that they can also do
some cool stuff that applicative functors can't.

We'll come back to <span class="fixed">Maybe</span> in a minute, but
first, let's check out the type class that belongs to monads.

<span id="the-monad-type-class"></span>

## The Monad type class

Just like functors have the <span class="fixed">Functor</span> type
class and applicative functors have the
<span class="fixed">Applicative</span> type class, monads come with
their own type class: <span class="fixed">Monad</span>! Wow, who would
have thought? This is what the type class looks like:

``` haskell:hs
class Monad m where
    return :: a -> m a

    (>>=) :: m a -> (a -> m b) -> m b

    (>>) :: m a -> m b -> m b
    x >> y = x >>= \_ -> y

    fail :: String -> m a
    fail msg = error msg
```

<img src="/assets/kid.png" class="right"
width="363" height="451" alt="this is you on monads" />

Let's start with the first line. It says <span class="fixed">class Monad
m where</span>. But wait, didn't we say that monads are just beefed up
applicative functors? Shouldn't there be a class constraint in there
along the lines of <span class="fixed">class (Applicative m) = &gt; Monad
m where</span> so that a type has to be an applicative functor first
before it can be made a monad? Well, there should, but when Haskell was
made, it hadn't occured to people that applicative functors are a good
fit for Haskell so they weren't in there. But rest assured, every monad
is an applicative functor, even if the <span class="fixed">Monad</span>
class declaration doesn't say so.

The first function that the <span class="fixed">Monad</span> type class
defines is <span class="fixed">return</span>. It's the same as
<span class="fixed">pure</span>, only with a different name. Its type is
<span class="fixed">(Monad m) =&gt; a -&gt; m a</span>. It takes a value and
puts it in a minimal default context that still holds that value. In
other words, it takes something and wraps it in a monad. It always does
the same thing as the <span class="fixed">pure</span> function from the
<span class="fixed">Applicative</span> type class, which means we're
already acquainted with <span class="fixed">return</span>. We already
used <span class="fixed">return</span> when doing I/O. We used it to
take a value and make a bogus I/O action that does nothing but yield
that value. For <span class="fixed">Maybe</span> it takes a value and
wraps it in a <span class="fixed">Just</span>.



Just a reminder: <span class="fixed">return</span> is nothing like the
<span class="fixed">return</span> that's in most other languages. It
doesn't end function execution or anything, it just takes a normal value
and puts it in a context.



<img src="/assets/tur2.png" class="left"
width="169" height="145" alt="hmmm yaes" />

The next function is <span class="fixed">&gt;&gt;=</span>, or bind. It's
like function application, only instead of taking a normal value and
feeding it to a normal function, it takes a monadic value (that is, a
value with a context) and feeds it to a function that takes a normal
value but returns a monadic value.

Next up, we have <span class="fixed">&gt;&gt;</span>. We won't pay too much
attention to it for now because it comes with a default implementation
and we pretty much never implement it when making
<span class="fixed">Monad</span> instances.

The final function of the <span class="fixed">Monad</span> type class is
<span class="fixed">fail</span>. We never use it explicitly in our code.
Instead, it's used by Haskell to enable failure in a special syntactic
construct for monads that we'll meet later. We don't need to concern
ourselves with <span class="fixed">fail</span> too much for now.

Now that we know what the <span class="fixed">Monad</span> type class
looks like, let's take a look at how <span class="fixed">Maybe</span> is
an instance of <span class="fixed">Monad</span>!

``` haskell:hs
instance Monad Maybe where
    return x = Just x
    Nothing >>= f = Nothing
    Just x >>= f  = f x
    fail _ = Nothing
```

<span class="fixed">return</span> is the same as
<span class="fixed">pure</span>, so that one's a no-brainer. We do what
we did in the <span class="fixed">Applicative</span> type class and wrap
it in a <span class="fixed">Just</span>.

The <span class="fixed">&gt;&gt;=</span> function is the same as our
<span class="fixed">applyMaybe</span>. When feeding the
<span class="fixed">Maybe a</span> to our function, we keep in mind the
context and return a <span class="fixed">Nothing</span> if the value on
the left is <span class="fixed">Nothing</span> because if there's no
value then there's no way to apply our function to it. If it's a
<span class="fixed">Just</span> we take what's inside and apply
<span class="fixed">f</span> to it.

We can play around with <span class="fixed">Maybe</span> as a monad:

``` haskell:hs
ghci> return "WHAT" :: Maybe String
Just "WHAT"
ghci> Just 9 >>= \x -> return (x*10)
Just 90
ghci> Nothing >>= \x -> return (x*10)
Nothing
```

Nothing new or exciting on the first line since we already used
<span class="fixed">pure</span> with <span class="fixed">Maybe</span>
and we know that <span class="fixed">return</span> is just
<span class="fixed">pure</span> with a different name. The next two
lines showcase <span class="fixed">&gt;&gt;=</span> a bit more.

Notice how when we fed <span class="fixed">Just 9</span> to the function
<span class="fixed">\x -&gt; return (x\*10)</span>, the
<span class="fixed">x</span> took on the value
<span class="fixed">9</span> inside the function. It seems as though we
were able to extract the value from a <span class="fixed">Maybe</span>
without pattern-matching. And we still didn't lose the context of our
<span class="fixed">Maybe</span> value, because when it's
<span class="fixed">Nothing</span>, the result of using
<span class="fixed">&gt;&gt;=</span> will be
<span class="fixed">Nothing</span> as well.

<span id="walk-the-line"></span>

## Walk the line

<img src="/assets/pierre.png" class="left"
width="374" height="405" alt="pierre" />

Now that we know how to feed a <span class="fixed">Maybe a</span> value
to a function of type <span class="fixed">a -&gt; Maybe b</span> while
taking into account the context of possible failure, let's see how we
can use <span class="fixed">&gt;&gt;=</span> repeatedly to handle
computations of several <span class="fixed">Maybe a</span> values.

Pierre has decided to take a break from his job at the fish farm and try
tightrope walking. He's not that bad at it, but he does have one
problem: birds keep landing on his balancing pole! They come and they
take a short rest, chat with their avian friends and then take off in
search of breadcrumbs. This wouldn't bother him so much if the number of
birds on the left side of the pole was always equal to the number of
birds on the right side. But sometimes, all the birds decide that they
like one side better and so they throw him off balance, which results in
an embarrassing tumble for Pierre (he's using a safety net).

Let's say that he keeps his balance if the number of birds on the left
side of the pole and on the right side of the pole is within three. So
if there's one bird on the right side and four birds on the left side,
he's okay. But if a fifth bird lands on the left side, then he loses his
balance and takes a dive.

We're going to simulate birds landing on and flying away from the pole
and see if Pierre is still at it after a certain number of birdy
arrivals and departures. For instance, we want to see what happens to
Pierre if first one bird arrives on the left side, then four birds
occupy the right side and then the bird that was on the left side
decides to fly away.

We can represent the pole with a simple pair of integers. The first
component will signify the number of birds on the left side and the
second component the number of birds on the right side:

``` haskell:hs
type Birds = Int
type Pole = (Birds,Birds)
```

First we made a type synonym for <span class="fixed">Int</span>, called
<span class="fixed">Birds</span>, because we're using integers to
represent how many birds there are. And then we made a type synonym
<span class="fixed">(Birds,Birds)</span> and we called it
<span class="fixed">Pole</span> (not to be confused with a person of
Polish descent).

Next up, how about we make a function that takes a number of birds and
lands them on one side of the pole. Here are the functions:

``` haskell:hs
landLeft :: Birds -> Pole -> Pole
landLeft n (left,right) = (left + n,right)

landRight :: Birds -> Pole -> Pole
landRight n (left,right) = (left,right + n)
```

Pretty straightforward stuff. Let's try them out:

``` haskell:hs
ghci> landLeft 2 (0,0)
(2,0)
ghci> landRight 1 (1,2)
(1,3)
ghci> landRight (-1) (1,2)
(1,1)
```

To make birds fly away we just had a negative number of birds land on
one side. Because landing a bird on the <span class="fixed">Pole</span>
returns a <span class="fixed">Pole</span>, we can chain applications of
<span class="fixed">landLeft</span> and
<span class="fixed">landRight</span>:

``` haskell:hs
ghci> landLeft 2 (landRight 1 (landLeft 1 (0,0)))
(3,1)
```

When we apply the function <span class="fixed">landLeft 1</span> to
<span class="fixed">(0,0)</span> we get
<span class="fixed">(1,0)</span>. Then, we land a bird on the right
side, resulting in <span class="fixed">(1,1)</span>. Finally two birds
land on the left side, resulting in <span class="fixed">(3,1)</span>. We
apply a function to something by first writing the function and then
writing its parameter, but here it would be better if the pole went
first and then the landing function. If we make a function like this:

``` haskell:hs
x -: f = f x
```

We can apply functions by first writing the parameter and then the
function:

``` haskell:hs
ghci> 100 -: (*3)
300
ghci> True -: not
False
ghci> (0,0) -: landLeft 2
(2,0)
```

By using this, we can repeatedly land birds on the pole in a more
readable manner:

``` haskell:hs
ghci> (0,0) -: landLeft 1 -: landRight 1 -: landLeft 2
(3,1)
```

Pretty cool! This example is equivalent to the one before where we
repeatedly landed birds on the pole, only it looks neater. Here, it's
more obvious that we start off with <span class="fixed">(0,0)</span> and
then land one bird one the left, then one on the right and finally two
on the left.

So far so good, but what happens if 10 birds land on one side?

``` haskell:hs
ghci> landLeft 10 (0,3)
(10,3)
```

10 birds on the left side and only 3 on the right? That's sure to send
poor Pierre falling through the air! This is pretty obvious here but
what if we had a sequence of landings like this:

``` haskell:hs
ghci> (0,0) -: landLeft 1 -: landRight 4 -: landLeft (-1) -: landRight (-2)
(0,2)
```

It might seem like everything is okay but if you follow the steps here,
you'll see that at one time there are 4 birds on the right side and no
birds on the left! To fix this, we have to take another look at our
<span class="fixed">landLeft</span> and
<span class="fixed">landRight</span> functions. From what we've seen, we
want these functions to be able to fail. That is, we want them to return
a new pole if the balance is okay but fail if the birds land in a
lopsided manner. And what better way to add a context of failure to
value than by using <span class="fixed">Maybe</span>! Let's rework these
functions:

``` haskell:hs
landLeft :: Birds -> Pole -> Maybe Pole
landLeft n (left,right)
    | abs ((left + n) - right) < 4 = Just (left + n, right)
    | otherwise                    = Nothing

landRight :: Birds -> Pole -> Maybe Pole
landRight n (left,right)
    | abs (left - (right + n)) < 4 = Just (left, right + n)
    | otherwise                    = Nothing
```

Instead of returning a <span class="fixed">Pole</span> these functions
now return a <span class="fixed">Maybe Pole</span>. They still take the
number of birds and the old pole as before, but then they check if
landing that many birds on the pole would throw Pierre off balance. We
use guards to check if the difference between the number of birds on the
new pole is less than 4. If it is, we wrap the new pole in a
<span class="fixed">Just</span> and return that. If it isn't, we return
a <span class="fixed">Nothing</span>, indicating failure.

Let's give these babies a go:

``` haskell:hs
ghci> landLeft 2 (0,0)
Just (2,0)
ghci> landLeft 10 (0,3)
Nothing
```

Nice! When we land birds without throwing Pierre off balance, we get a
new pole wrapped in a <span class="fixed">Just</span>. But when many
more birds end up on one side of the pole, we get a
<span class="fixed">Nothing</span>. This is cool, but we seem to have
lost the ability to repeatedly land birds on the pole. We can't do
<span class="fixed">landLeft 1 (landRight 1 (0,0))</span> anymore
because when we apply <span class="fixed">landRight 1</span> to
<span class="fixed">(0,0)</span>, we don't get a
<span class="fixed">Pole</span>, but a <span class="fixed">Maybe
Pole</span>. <span class="fixed">landLeft 1</span> takes a
<span class="fixed">Pole</span> and not a <span class="fixed">Maybe
Pole</span>.

We need a way of taking a <span class="fixed">Maybe Pole</span> and
feeding it to a function that takes a <span class="fixed">Pole</span>
and returns a <span class="fixed">Maybe Pole</span>. Luckily, we have
<span class="fixed">&gt;&gt;=</span>, which does just that for
<span class="fixed">Maybe</span>. Let's give it a go:

``` haskell:hs
ghci> landRight 1 (0,0) >>= landLeft 2
Just (2,1)
```

Remember, <span class="fixed">landLeft 2</span> has a type of
<span class="fixed">Pole -&gt; Maybe Pole</span>. We couldn't just feed it
the <span class="fixed">Maybe Pole</span> that is the result of
<span class="fixed">landRight 1 (0,0)</span>, so we use
<span class="fixed">&gt;&gt;=</span> to take that value with a context and
give it to <span class="fixed">landLeft 2</span>.
<span class="fixed">&gt;&gt;=</span> does indeed allow us to treat the
<span class="fixed">Maybe</span> value as a value with context because
if we feed a <span class="fixed">Nothing</span> into
<span class="fixed">landLeft 2</span>, the result is
<span class="fixed">Nothing</span> and the failure is propagated:

``` haskell:hs
ghci> Nothing >>= landLeft 2
Nothing
```

With this, we can now chain landings that may fail because
<span class="fixed">&gt;&gt;=</span> allows us to feed a monadic value to a
function that takes a normal one.

Here's a sequence of birdy landings:

``` haskell:hs
ghci> return (0,0) >>= landRight 2 >>= landLeft 2 >>= landRight 2
Just (2,4)
```

At the beginning, we used <span class="fixed">return</span> to take a
pole and wrap it in a <span class="fixed">Just</span>. We could have
just applied <span class="fixed">landRight 2</span> to
<span class="fixed">(0,0)</span>, it would have been the same, but this
way we can be more consistent by using <span class="fixed">&gt;&gt;=</span>
for every function. <span class="fixed">Just (0,0)</span> gets fed to
<span class="fixed">landRight 2</span>, resulting in
<span class="fixed">Just (0,2)</span>. This, in turn, gets fed to
<span class="fixed">landLeft 2</span>, resulting in
<span class="fixed">Just (2,2)</span>, and so on.

Remember this example from before we introduced failure into Pierre's
routine:

``` haskell:hs
ghci> (0,0) -: landLeft 1 -: landRight 4 -: landLeft (-1) -: landRight (-2)
(0,2)
```

It didn't simulate his interaction with birds very well because in the
middle there his balance was off but the result didn't reflect that. But
let's give that a go now by using monadic application
(<span class="fixed">&gt;&gt;=</span>) instead of normal application:

``` haskell:hs
ghci> return (0,0) >>= landLeft 1 >>= landRight 4 >>= landLeft (-1) >>= landRight (-2)
Nothing
```

<img src="/assets/banana.png" class="right"
width="262" height="130" alt="iama banana" />

Awesome. The final result represents failure, which is what we expected.
Let's see how this result was obtained. First,
<span class="fixed">return</span> puts <span class="fixed">(0,0)</span>
into a default context, making it a <span class="fixed">Just
(0,0)</span>. Then, <span class="fixed">Just (0,0) &gt;&gt;= landLeft
1</span> happens. Since the <span class="fixed">Just (0,0)</span> is a
<span class="fixed">Just</span> value, <span class="fixed">landLeft
1</span> gets applied to <span class="fixed">(0,0)</span>, resulting in
a <span class="fixed">Just (1,0)</span>, because the birds are still
relatively balanced. Next, <span class="fixed">Just (1,0) &gt;&gt;=
landRight 4</span> takes place and the result is
<span class="fixed">Just (1,4)</span> as the balance of the birds is
still intact, although just barely. <span class="fixed">Just
(1,4)</span> gets fed to <span class="fixed">landLeft (-1)</span>. This
means that <span class="fixed">landLeft (-1) (1,4)</span> takes place.
Now because of how <span class="fixed">landLeft</span> works, this
results in a <span class="fixed">Nothing</span>, because the resulting
pole is off balance. Now that we have a
<span class="fixed">Nothing</span>, it gets fed to
<span class="fixed">landRight (-2)</span>, but because it's a
<span class="fixed">Nothing</span>, the result is automatically
<span class="fixed">Nothing</span>, as we have nothing to apply
<span class="fixed">landRight (-2)</span> to.

We couldn't have achieved this by just using
<span class="fixed">Maybe</span> as an applicative. If you try it,
you'll get stuck, because applicative functors don't allow for the
applicative values to interact with each other very much. They can, at
best, be used as parameters to a function by using the applicative
style. The applicative operators will fetch their results and feed them
to the function in a manner appropriate for each applicative and then
put the final applicative value together, but there isn't that much
interaction going on between them. Here, however, each step relies on
the previous one's result. On every landing, the possible result from
the previous one is examined and the pole is checked for balance. This
determines whether the landing will succeed or fail.

We may also devise a function that ignores the current number of birds
on the balancing pole and just makes Pierre slip and fall. We can call
it <span class="fixed">banana</span>:

``` haskell:hs
banana :: Pole -> Maybe Pole
banana _ = Nothing
```

Now we can chain it together with our bird landings. It will always
cause our walker to fall, because it ignores whatever's passed to it and
always returns a failure. Check it:

``` haskell:hs
ghci> return (0,0) >>= landLeft 1 >>= banana >>= landRight 1
Nothing
```

The value <span class="fixed">Just (1,0)</span> gets fed to
<span class="fixed">banana</span>, but it produces a
<span class="fixed">Nothing</span>, which causes everything to result in
a <span class="fixed">Nothing</span>. How unfortunate!

Instead of making functions that ignore their input and just return a
predetermined monadic value, we can use the
<span class="fixed">&gt;&gt;</span> function, whose default implementation
is this:

``` haskell:hs
(>>) :: (Monad m) => m a -> m b -> m b
m >> n = m >>= \_ -> n
```

Normally, passing some value to a function that ignores its parameter
and always just returns some predetermined value would always result in
that predetermined value. With monads however, their context and meaning
has to be considered as well. Here's how <span class="fixed">&gt;&gt;</span>
acts with <span class="fixed">Maybe</span>:

``` haskell:hs
ghci> Nothing >> Just 3
Nothing
ghci> Just 3 >> Just 4
Just 4
ghci> Just 3 >> Nothing
Nothing
```

If you replace <span class="fixed">&gt;&gt;</span> with
<span class="fixed">&gt;&gt;= \\ -&gt;</span>, it's easy to see why it acts
like it does.

We can replace our <span class="fixed">banana</span> function in the
chain with a <span class="fixed">&gt;&gt;</span> and then a
<span class="fixed">Nothing</span>:

``` haskell:hs
ghci> return (0,0) >>= landLeft 1 >> Nothing >>= landRight 1
Nothing
```

There we go, guaranteed and obvious failure!

It's also worth taking a look at what this would look like if we hadn't
made the clever choice of treating <span class="fixed">Maybe</span>
values as values with a failure context and feeding them to functions
like we did. Here's how a series of bird landings would look like:

``` haskell:hs
routine :: Maybe Pole
routine = case landLeft 1 (0,0) of
    Nothing -> Nothing
    Just pole1 -> case landRight 4 pole1 of 
        Nothing -> Nothing
        Just pole2 -> case landLeft 2 pole2 of
            Nothing -> Nothing
            Just pole3 -> landLeft 1 pole3
```

<img src="/assets/centaur.png" class="right"
width="297" height="331" alt="john joe glanton" />

We land a bird on the left and then we examine the possibility of
failure and the possibility of success. In the case of failure, we
return a <span class="fixed">Nothing</span>. In the case of success, we
land birds on the right and then do the same thing all over again.
Converting this monstrosity into a neat chain of monadic applications
with <span class="fixed">&gt;&gt;=</span> is a classic example of how the
<span class="fixed">Maybe</span> monad saves us a lot of time when we
have to successively do computations that are based on computations that
might have failed.

Notice how the <span class="fixed">Maybe</span> implementation of
<span class="fixed">&gt;&gt;=</span> features exactly this logic of seeing
if a value is <span class="fixed">Nothing</span> and if it is, returning
a <span class="fixed">Nothing</span> right away and if it isn't, going
forward with what's inside the <span class="fixed">Just</span>.

In this section, we took some functions that we had and saw that they
would work better if the values that they returned supported failure. By
turning those values into <span class="fixed">Maybe</span> values and
replacing normal function application with
<span class="fixed">&gt;&gt;=</span>, we got a mechanism for handling
failure pretty much for free, because <span class="fixed">&gt;&gt;=</span>
is supposed to preserve the context of the value to which it applies
functions. In this case, the context was that our values were values
with failure and so when we applied functions to such values, the
possibility of failure was always taken into account.

<span id="do-notation"></span>

## do notation

Monads in Haskell are so useful that they got their own special syntax
called <span class="fixed">do</span> notation. We've already encountered
<span class="fixed">do</span> notation when we were doing I/O and there
we said that it was for gluing together several I/O actions into one.
Well, as it turns out, <span class="fixed">do</span> notation isn't just
for <span class="fixed">IO</span>, but can be used for any monad. Its
principle is still the same: gluing together monadic values in sequence.
We're going to take a look at how <span class="fixed">do</span> notation
works and why it's useful.

Consider this familiar example of monadic application:

``` haskell:hs
ghci> Just 3 >>= (\x -> Just (show x ++ "!"))
Just "3!"
```

Been there, done that. Feeding a monadic value to a function that
returns one, no big deal. Notice how when we do this,
<span class="fixed">x</span> becomes <span class="fixed">3</span> inside
the lambda. Once we're inside that lambda, it's just a normal value
rather than a monadic value. Now, what if we had another
<span class="fixed">&gt;&gt;=</span> inside that function? Check this out:

``` haskell:hs
ghci> Just 3 >>= (\x -> Just "!" >>= (\y -> Just (show x ++ y)))
Just "3!"
```

Ah, a nested use of <span class="fixed">&gt;&gt;=</span>! In the outermost
lambda, we feed <span class="fixed">Just "!"</span> to the lambda
<span class="fixed">\y -&gt; Just (show x ++ y)</span>. Inside this
lambda, the <span class="fixed">y</span> becomes
<span class="fixed">"!"</span>. <span class="fixed">x</span> is still
<span class="fixed">3</span> because we got it from the outer lambda.
All this sort of reminds me of the following expression:

``` haskell:hs
ghci> let x = 3; y = "!" in show x ++ y
"3!"
```

The main difference between these two is that the values in the former
example are monadic. They're values with a failure context. We can
replace any of them with a failure:

``` haskell:hs
ghci> Nothing >>= (\x -> Just "!" >>= (\y -> Just (show x ++ y)))
Nothing
ghci> Just 3 >>= (\x -> Nothing >>= (\y -> Just (show x ++ y)))
Nothing
ghci> Just 3 >>= (\x -> Just "!" >>= (\y -> Nothing))
Nothing
```

In the first line, feeding a <span class="fixed">Nothing</span> to a
function naturally results in a <span class="fixed">Nothing</span>. In
the second line, we feed <span class="fixed">Just 3</span> to a function
and the <span class="fixed">x</span> becomes
<span class="fixed">3</span>, but then we feed a
<span class="fixed">Nothing</span> to the inner lambda and the result of
that is <span class="fixed">Nothing</span>, which causes the outer
lambda to produce <span class="fixed">Nothing</span> as well. So this is
sort of like assigning values to variables in
<span class="fixed">let</span> expressions, only that the values in
question are monadic values.

To further illustrate this point, let's write this in a script and have
each <span class="fixed">Maybe</span> value take up its own line:

``` haskell:hs
foo :: Maybe String
foo = Just 3   >>= (\x ->
      Just "!" >>= (\y ->
      Just (show x ++ y)))
```

To save us from writing all these annoying lambdas, Haskell gives us
<span class="fixed">do</span> notation. It allows us to write the
previous piece of code like this:

``` haskell:hs
foo :: Maybe String
foo = do
    x <- Just 3
    y <- Just "!"
    Just (show x ++ y)
```

<img src="/assets/owld.png" class="right"
width="269" height="348" alt="90s owl" />

It would seem as though we've gained the ability to temporarily extract
things from <span class="fixed">Maybe</span> values without having to
check if the <span class="fixed">Maybe</span> values are
<span class="fixed">Just</span> values or
<span class="fixed">Nothing</span> values at every step. How cool! If
any of the values that we try to extract from are
<span class="fixed">Nothing</span>, the whole
<span class="fixed">do</span> expression will result in a
<span class="fixed">Nothing</span>. We're yanking out their (possibly
existing) values and letting <span class="fixed">&gt;&gt;=</span> worry
about the context that comes with those values. It's important to
remember that <span class="fixed">do</span> expressions are just
different syntax for chaining monadic values.

In a <span class="fixed">do</span> expression, every line is a monadic
value. To inspect its result, we use <span class="fixed">&lt;-</span>. If
we have a <span class="fixed">Maybe String</span> and we bind it with
<span class="fixed">&lt;-</span> to a variable, that variable will be a
<span class="fixed">String</span>, just like when we used
<span class="fixed">&gt;&gt;=</span> to feed monadic values to lambdas. The
last monadic value in a <span class="fixed">do</span> expression, like
<span class="fixed">Just (show x ++ y)</span> here, can't be used with
<span class="fixed">&lt;-</span> to bind its result, because that wouldn't
make sense if we translated the <span class="fixed">do</span> expression
back to a chain of <span class="fixed">&gt;&gt;=</span> applications.
Rather, its result is the result of the whole glued up monadic value,
taking into account the possible failure of any of the previous ones.

For instance, examine the following line:

``` haskell:hs
ghci> Just 9 >>= (\x -> Just (x > 8))
Just True
```

Because the left parameter of <span class="fixed">&gt;&gt;=</span> is a
<span class="fixed">Just</span> value, the lambda is applied to
<span class="fixed">9</span> and the result is a
<span class="fixed">Just True</span>. If we rewrite this in
<span class="fixed">do</span> notation, we get:

``` haskell:hs
marySue :: Maybe Bool
marySue = do 
    x <- Just 9
    Just (x > 8)
```

If we compare these two, it's easy to see why the result of the whole
monadic value is the result of the last monadic value in the
<span class="fixed">do</span> expression with all the previous ones
chained into it.

Our tightwalker's routine can also be expressed with
<span class="fixed">do</span> notation.
<span class="fixed">landLeft</span> and
<span class="fixed">landRight</span> take a number of birds and a pole
and produce a pole wrapped in a <span class="fixed">Just</span>, unless
the tightwalker slips, in which case a
<span class="fixed">Nothing</span> is produced. We used
<span class="fixed">&gt;&gt;=</span> to chain successive steps because each
one relied on the previous one and each one had an added context of
possible failure. Here's two birds landing on the left side, then two
birds landing on the right and then one bird landing on the left:

``` haskell:hs
routine :: Maybe Pole
routine = do
    start <- return (0,0)
    first <- landLeft 2 start
    second <- landRight 2 first
    landLeft 1 second
```

Let's see if he succeeds:

``` haskell:hs
ghci> routine
Just (3,2)
```

He does! Great. When we were doing these routines by explicitly writing
<span class="fixed">&gt;&gt;=</span>, we usually said something like
<span class="fixed">return (0,0) &gt;&gt;= landLeft 2</span>, because
<span class="fixed">landLeft 2</span> is a function that returns a
<span class="fixed">Maybe</span> value. With
<span class="fixed">do</span> expressions however, each line must
feature a monadic value. So we explicitly pass the previous
<span class="fixed">Pole</span> to the
<span class="fixed">landLeft</span> <span class="fixed">landRight</span>
functions. If we examined the variables to which we bound our
<span class="fixed">Maybe</span> values,
<span class="fixed">start</span> would be
<span class="fixed">(0,0)</span>, <span class="fixed">first</span> would
be <span class="fixed">(2,0)</span> and so on.

Because <span class="fixed">do</span> expressions are written line by
line, they may look like imperative code to some people. But the thing
is, they're just sequential, as each value in each line relies on the
result of the previous ones, along with their contexts (in this case,
whether they succeeded or failed).

Again, let's take a look at what this piece of code would look like if
we hadn't used the monadic aspects of <span class="fixed">Maybe</span>:

``` haskell:hs
routine :: Maybe Pole
routine = 
    case Just (0,0) of 
        Nothing -> Nothing
        Just start -> case landLeft 2 start of
            Nothing -> Nothing
            Just first -> case landRight 2 first of
                Nothing -> Nothing
                Just second -> landLeft 1 second
```

See how in the case of success, the tuple inside
<span class="fixed">Just (0,0)</span> becomes
<span class="fixed">start</span>, the result of
<span class="fixed">landLeft 2 start</span> becomes
<span class="fixed">first</span>, etc.

If we want to throw the Pierre a banana peel in
<span class="fixed">do</span> notation, we can do the following:

``` haskell:hs
routine :: Maybe Pole
routine = do
    start <- return (0,0)
    first <- landLeft 2 start
    Nothing
    second <- landRight 2 first
    landLeft 1 second
```

When we write a line in <span class="fixed">do</span> notation without
binding the monadic value with <span class="fixed">&lt;-</span>, it's just
like putting <span class="fixed">&gt;&gt;</span> after the monadic value
whose result we want to ignore. We sequence the monadic value but we
ignore its result because we don't care what it is and it's prettier
than writing <span class="fixed">\_ &lt;- Nothing</span>, which is
equivalent to the above.

When to use <span class="fixed">do</span> notation and when to
explicitly use <span class="fixed">&gt;&gt;=</span> is up to you. I think
this example lends itself to explicitly writing
<span class="fixed">&gt;&gt;=</span> because each step relies specifically
on the result of the previous one. With <span class="fixed">do</span>
notation, we had to specifically write on which pole the birds are
landing, but every time we used that came directly before. But still, it
gave us some insight into <span class="fixed">do</span> notation.

In <span class="fixed">do</span> notation, when we bind monadic values
to names, we can utilize pattern matching, just like in
<span class="fixed">let</span> expressions and function parameters.
Here's an example of pattern matching in a <span class="fixed">do</span>
expression:

``` haskell:hs
justH :: Maybe Char
justH = do
    (x:xs) <- Just "hello"
    return x
```

We use pattern matching to get the first character of the string
<span class="fixed">"hello"</span> and then we present it as the result.
So <span class="fixed">justH</span> evaluates to
<span class="fixed">Just 'h'</span>.

What if this pattern matching were to fail? When matching on a pattern
in a function fails, the next pattern is matched. If the matching falls
through all the patterns for a given function, an error is thrown and
our program crashes. On the other hand, failed pattern matching in
<span class="fixed">let</span> expressions results in an error being
produced right away, because the mechanism of falling through patterns
isn't present in <span class="fixed">let</span> expressions. When
pattern matching fails in a <span class="fixed">do</span> expression,
the <span class="fixed">fail</span> function is called. It's part of the
<span class="fixed">Monad</span> type class and it enables failed
pattern matching to result in a failure in the context of the current
monad instead of making our program crash. Its default implementation is
this:

``` haskell:hs
fail :: (Monad m) => String -> m a
fail msg = error msg
```

So by default it does make our program crash, but monads that
incorporate a context of possible failure (like
<span class="fixed">Maybe</span>) usually implement it on their own. For
<span class="fixed">Maybe</span>, its implemented like so:

``` haskell:hs
fail _ = Nothing
```

It ignores the error message and makes a
<span class="fixed">Nothing</span>. So when pattern matching fails in a
<span class="fixed">Maybe</span> value that's written in
<span class="fixed">do</span> notation, the whole value results in a
<span class="fixed">Nothing</span>. This is preferable to having our
program crash. Here's a <span class="fixed">do</span> expression with a
pattern that's bound to fail:

``` haskell:hs
wopwop :: Maybe Char
wopwop = do
    (x:xs) <- Just ""
    return x
```

The pattern matching fails, so the effect is the same as if the whole
line with the pattern was replaced with a
<span class="fixed">Nothing</span>. Let's try this out:

``` haskell:hs
ghci> wopwop
Nothing
```

The failed pattern matching has caused a failure within the context of
our monad instead of causing a program-wide failure, which is pretty
neat.

<span id="the-list-monad"></span>

## The list monad

<img src="/assets/deadcat.png" class="left"
width="235" height="230" alt="dead cat" />

So far, we've seen how <span class="fixed">Maybe</span> values can be
viewed as values with a failure context and how we can incorporate
failure handling into our code by using <span class="fixed">&gt;&gt;=</span>
to feed them to functions. In this section, we're going to take a look
at how to use the monadic aspects of lists to bring non-determinism into
our code in a clear and readable manner.

We've already talked about how lists represent non-deterministic values
when they're used as applicatives. A value like
<span class="fixed">5</span> is deterministic. It has only one result
and we know exactly what it is. On the other hand, a value like
<span class="fixed">\[3,8,9\]</span> contains several results, so we can
view it as one value that is actually many values at the same time.
Using lists as applicative functors showcases this non-determinism
nicely:

``` haskell:hs
ghci> (*) <$> [1,2,3] <*> [10,100,1000]
[10,100,1000,20,200,2000,30,300,3000]
```

All the possible combinations of multiplying elements from the left list
with elements from the right list are included in the resulting list.
When dealing with non-determinism, there are many choices that we can
make, so we just try all of them, and so the result is a
non-deterministic value as well, only it has many more results.

This context of non-determinism translates to monads very nicely. Let's
go ahead and see what the <span class="fixed">Monad</span> instance for
lists looks like:

``` haskell:hs
instance Monad [] where
    return x = [x]
    xs >>= f = concat (map f xs)
    fail _ = []
```

<span class="fixed">return</span> does the same thing as
<span class="fixed">pure</span>, so we should already be familiar with
<span class="fixed">return</span> for lists. It takes a value and puts
it in a minimal default context that still yields that value. In other
words, it makes a list that has only that one value as its result. This
is useful for when we want to just wrap a normal value into a list so
that it can interact with non-deterministic values.

To understand how <span class="fixed">&gt;&gt;=</span> works for lists, it's
best if we take a look at it in action to gain some intuition first.
<span class="fixed">&gt;&gt;=</span> is about taking a value with a context
(a monadic value) and feeding it to a function that takes a normal value
and returns one that has context. If that function just produced a
normal value instead of one with a context,
<span class="fixed">&gt;&gt;=</span> wouldn't be so useful because after one
use, the context would be lost. Anyway, let's try feeding a
non-deterministic value to a function:

``` haskell:hs
ghci> [3,4,5] >>= \x -> [x,-x]
[3,-3,4,-4,5,-5]
```

When we used <span class="fixed">&gt;&gt;=</span> with
<span class="fixed">Maybe</span>, the monadic value was fed into the
function while taking care of possible failures. Here, it takes care of
non-determinism for us. <span class="fixed">\[3,4,5\]</span> is a
non-deterministic value and we feed it into a function that returns a
non-deterministic value as well. The result is also non-deterministic,
and it features all the possible results of taking elements from the
list <span class="fixed">\[3,4,5\]</span> and passing them to the
function <span class="fixed">\x -&gt; \[x,-x\]</span>. This function takes
a number and produces two results: one negated and one that's unchanged.
So when we use <span class="fixed">&gt;&gt;=</span> to feed this list to the
function, every number is negated and also kept unchanged. The
<span class="fixed">x</span> from the lambda takes on every value from
the list that's fed to it.

To see how this is achieved, we can just follow the implementation.
First, we start off with the list <span class="fixed">\[3,4,5\]</span>.
Then, we map the lambda over it and the result is the following:

``` haskell:hs
[[3,-3],[4,-4],[5,-5]]
```

The lambda is applied to every element and we get a list of lists.
Finally, we just flatten the list and voila! We've applied a
non-deterministic function to a non-deterministic value!

Non-determinism also includes support for failure. The empty list
<span class="fixed">\[\]</span> is pretty much the equivalent of
<span class="fixed">Nothing</span>, because it signifies the absence of
a result. That's why failing is just defined as the empty list. The
error message gets thrown away. Let's play around with lists that fail:

``` haskell:hs
ghci> [] >>= \x -> ["bad","mad","rad"]
[]
ghci> [1,2,3] >>= \x -> []
[]
```

In the first line, an empty list is fed into the lambda. Because the
list has no elements, none of them can be passed to the function and so
the result is an empty list. This is similar to feeding
<span class="fixed">Nothing</span> to a function. In the second line,
each element gets passed to the function, but the element is ignored and
the function just returns an empty list. Because the function fails for
every element that goes in it, the result is a failure.

Just like with <span class="fixed">Maybe</span> values, we can chain
several lists with <span class="fixed">&gt;&gt;=</span>, propagating the
non-determinism:

``` haskell:hs
ghci> [1,2] >>= \n -> ['a','b'] >>= \ch -> return (n,ch)
[(1,'a'),(1,'b'),(2,'a'),(2,'b')]
```

<img src="/assets/concatmap.png" class="left"
width="399" height="340" alt="concatmap" />

The list <span class="fixed">\[1,2\]</span> gets bound to
<span class="fixed">n</span> and <span class="fixed">\['a','b'\]</span>
gets bound to <span class="fixed">ch</span>. Then, we do
<span class="fixed">return (n,ch)</span> (or
<span class="fixed">\[(n,ch)\]</span>), which means taking a pair of
<span class="fixed">(n,ch)</span> and putting it in a default minimal
context. In this case, it's making the smallest possible list that still
presents <span class="fixed">(n,ch)</span> as the result and features as
little non-determinism as possible. Its effect on the context is
minimal. What we're saying here is this: for every element in
<span class="fixed">\[1,2\]</span>, go over every element in
<span class="fixed">\['a','b'\]</span> and produce a tuple of one
element from each list.

Generally speaking, because <span class="fixed">return</span> takes a
value and wraps it in a minimal context, it doesn't have any extra
effect (like failing in <span class="fixed">Maybe</span> or resulting in
more non-determinism for lists) but it does present something as its
result.



When you have non-deterministic values interacting, you can view their
computation as a tree where every possible result in a list represents a
separate branch.



Here's the previous expression rewritten in
<span class="fixed">do</span> notation:

``` haskell:hs
listOfTuples :: [(Int,Char)]
listOfTuples = do
    n <- [1,2]
    ch <- ['a','b']
    return (n,ch)
```

This makes it a bit more obvious that <span class="fixed">n</span> takes
on every value from <span class="fixed">\[1,2\]</span> and
<span class="fixed">ch</span> takes on every value from
<span class="fixed">\['a','b'\]</span>. Just like with
<span class="fixed">Maybe</span>, we're extracting the elements from the
monadic values and treating them like normal values and
<span class="fixed">&gt;&gt;=</span> takes care of the context for us. The
context in this case is non-determinism.

Using lists with <span class="fixed">do</span> notation really reminds
me of something we've seen before. Check out the following piece of
code:

``` haskell:hs
ghci> [ (n,ch) | n <- [1,2], ch <- ['a','b'] ]
[(1,'a'),(1,'b'),(2,'a'),(2,'b')]
```

Yes! List comprehensions! In our <span class="fixed">do</span> notation
example, <span class="fixed">n</span> became every result from
<span class="fixed">\[1,2\]</span> and for every such result,
<span class="fixed">ch</span> was assigned a result from
<span class="fixed">\['a','b'\]</span> and then the final line put
<span class="fixed">(n,ch)</span> into a default context (a singleton
list) to present it as the result without introducing any additional
non-determinism. In this list comprehension, the same thing happened,
only we didn't have to write <span class="fixed">return</span> at the
end to present <span class="fixed">(n,ch)</span> as the result because
the output part of a list comprehension did that for us.

In fact, list comprehensions are just syntactic sugar for using lists as
monads. In the end, list comprehensions and lists in
<span class="fixed">do</span> notation translate to using
<span class="fixed">&gt;&gt;=</span> to do computations that feature
non-determinism.

List comprehensions allow us to filter our output. For instance, we can
filter a list of numbers to search only for that numbers whose digits
contain a <span class="fixed">7</span>:

``` haskell:hs
ghci> [ x | x <- [1..50], '7' `elem` show x ]
[7,17,27,37,47]
```

We apply <span class="fixed">show</span> to <span class="fixed">x</span>
to turn our number into a string and then we check if the character
<span class="fixed">'7'</span> is part of that string. Pretty clever. To
see how filtering in list comprehensions translates to the list monad,
we have to check out the <span class="fixed">guard</span> function and
the <span class="fixed">MonadPlus</span> type class. The
<span class="fixed">MonadPlus</span> type class is for monads that can
also act as monoids. Here's its definition:

``` haskell:hs
class Monad m => MonadPlus m where
    mzero :: m a
    mplus :: m a -> m a -> m a
```

<span class="fixed">mzero</span> is synonymous to
<span class="fixed">mempty</span> from the
<span class="fixed">Monoid</span> type class and
<span class="fixed">mplus</span> corresponds to
<span class="fixed">mappend</span>. Because lists are monoids as well as
monads, they can be made an instance of this type class:

``` haskell:hs
instance MonadPlus [] where
    mzero = []
    mplus = (++)
```

For lists <span class="fixed">mzero</span> represents a
non-deterministic computation that has no results at all — a failed
computation. <span class="fixed">mplus</span> joins two
non-deterministic values into one. The <span class="fixed">guard</span>
function is defined like this:

``` haskell:hs
guard :: (MonadPlus m) => Bool -> m ()
guard True = return ()
guard False = mzero
```

It takes a boolean value and if it's <span class="fixed">True</span>,
takes a <span class="fixed">()</span> and puts it in a minimal default
context that still succeeds. Otherwise, it makes a failed monadic value.
Here it is in action:

``` haskell:hs
ghci> guard (5 > 2) :: Maybe ()
Just ()
ghci> guard (1 > 2) :: Maybe ()
Nothing
ghci> guard (5 > 2) :: [()]
[()]
ghci> guard (1 > 2) :: [()]
[]
```

Looks interesting, but how is it useful? In the list monad, we use it to
filter out non-deterministic computations. Observe:

``` haskell:hs
ghci> [1..50] >>= (\x -> guard ('7' `elem` show x) >> return x)
[7,17,27,37,47]
```

The result here is the same as the result of our previous list
comprehension. How does <span class="fixed">guard</span> achieve this?
Let's first see how <span class="fixed">guard</span> functions in
conjunction with <span class="fixed">&gt;&gt;</span>:

``` haskell:hs
ghci> guard (5 > 2) >> return "cool" :: [String]
["cool"]
ghci> guard (1 > 2) >> return "cool" :: [String]
[]
```

If <span class="fixed">guard</span> succeeds, the result contained
within it is an empty tuple. So then, we use
<span class="fixed">&gt;&gt;</span> to ignore that empty tuple and present
something else as the result. However, if
<span class="fixed">guard</span> fails, then so will the
<span class="fixed">return</span> later on, because feeding an empty
list to a function with <span class="fixed">&gt;&gt;=</span> always results
in an empty list. A <span class="fixed">guard</span> basically says: if
this boolean is <span class="fixed">False</span> then produce a failure
right here, otherwise make a successful value that has a dummy result of
<span class="fixed">()</span> inside it. All this does is to allow the
computation to continue.

Here's the previous example rewritten in <span class="fixed">do</span>
notation:

``` haskell:hs
sevensOnly :: [Int]
sevensOnly = do
    x <- [1..50]
    guard ('7' `elem` show x)
    return x
```

Had we forgotten to present <span class="fixed">x</span> as the final
result by using <span class="fixed">return</span>, the resulting list
would just be a list of empty tuples. Here's this again in the form of a
list comprehension:

``` haskell:hs
ghci> [ x | x <- [1..50], '7' `elem` show x ]
[7,17,27,37,47]
```

So filtering in list comprehensions is the same as using
<span class="fixed">guard</span>.

### A knight's quest

Here's a problem that really lends itself to being solved with
non-determinism. Say you have a chess board and only one knight piece on
it. We want to find out if the knight can reach a certain position in
three moves. We'll just use a pair of numbers to represent the knight's
position on the chess board. The first number will determine the column
he's in and the second number will determine the row.

<img src="/assets/chess.png" class="center"
width="760" height="447" alt="hee haw im a horse" />

Let's make a type synonym for the knight's current position on the chess
board:

``` haskell:hs
type KnightPos = (Int,Int)
```

So let's say that the knight starts at <span class="fixed">(6,2)</span>.
Can he get to <span class="fixed">(6,1)</span> in exactly three moves?
Let's see. If we start off at <span class="fixed">(6,2)</span> what's
the best move to make next? I know, how about all of them! We have
non-determinism at our disposal, so instead of picking one move, let's
just pick all of them at once. Here's a function that takes the knight's
position and returns all of its next moves:

``` haskell:hs
moveKnight :: KnightPos -> [KnightPos]
moveKnight (c,r) = do
    (c',r') <- [(c+2,r-1),(c+2,r+1),(c-2,r-1),(c-2,r+1)
               ,(c+1,r-2),(c+1,r+2),(c-1,r-2),(c-1,r+2)
               ]
    guard (c' `elem` [1..8] && r' `elem` [1..8])
    return (c',r')
```

The knight can always take one step horizontally or vertically and two
steps horizontally or vertically but its movement has to be both
horizontal and vertical. <span class="fixed">(c',r')</span> takes on
every value from the list of movements and then
<span class="fixed">guard</span> makes sure that the new move,
<span class="fixed">(c',r')</span> is still on the board. If it it's
not, it produces an empty list, which causes a failure and
<span class="fixed">return (c',r')</span> isn't carried out for that
position.

This function can also be written without the use of lists as a monad,
but we did it here just for kicks. Here is the same function done with
<span class="fixed">filter</span>:

``` haskell:hs
moveKnight :: KnightPos -> [KnightPos]
moveKnight (c,r) = filter onBoard
    [(c+2,r-1),(c+2,r+1),(c-2,r-1),(c-2,r+1)
    ,(c+1,r-2),(c+1,r+2),(c-1,r-2),(c-1,r+2)
    ]
    where onBoard (c,r) = c `elem` [1..8] && r `elem` [1..8]
```

Both of these do the same thing, so pick one that you think looks nicer.
Let's give it a whirl:

``` haskell:hs
ghci> moveKnight (6,2)
[(8,1),(8,3),(4,1),(4,3),(7,4),(5,4)]
ghci> moveKnight (8,1)
[(6,2),(7,3)]
```

Works like a charm! We take one position and we just carry out all the
possible moves at once, so to speak. So now that we have a
non-deterministic next position, we just use
<span class="fixed">&gt;&gt;=</span> to feed it to
<span class="fixed">moveKnight</span>. Here's a function that takes a
position and returns all the positions that you can reach from it in
three moves:

``` haskell:hs
in3 :: KnightPos -> [KnightPos]
in3 start = do 
    first <- moveKnight start
    second <- moveKnight first
    moveKnight second
```

If you pass it <span class="fixed">(6,2)</span>, the resulting list is
quite big, because if there are several ways to reach some position in
three moves, it crops up in the list several times. The above without
<span class="fixed">do</span> notation:

``` haskell:hs
in3 start = return start >>= moveKnight >>= moveKnight >>= moveKnight
```

Using <span class="fixed">&gt;&gt;=</span> once gives us all possible moves
from the start and then when we use <span class="fixed">&gt;&gt;=</span> the
second time, for every possible first move, every possible next move is
computed, and the same goes for the last move.

Putting a value in a default context by applying
<span class="fixed">return</span> to it and then feeding it to a
function with <span class="fixed">&gt;&gt;=</span> is the same as just
normally applying the function to that value, but we did it here anyway
for style.

Now, let's make a function that takes two positions and tells us if you
can get from one to the other in exactly three steps:

``` haskell:hs
canReachIn3 :: KnightPos -> KnightPos -> Bool
canReachIn3 start end = end `elem` in3 start
```

We generate all the possible positions in three steps and then we see if
the position we're looking for is among them. So let's see if we can get
from <span class="fixed">(6,2)</span> to
<span class="fixed">(6,1)</span> in three moves:

``` haskell:hs
ghci> (6,2) `canReachIn3` (6,1)
True
```

Yes! How about from <span class="fixed">(6,2)</span> to
<span class="fixed">(7,3)</span>?

``` haskell:hs
ghci> (6,2) `canReachIn3` (7,3)
False
```

No! As an exercise, you can change this function so that when you can
reach one position from the other, it tells you which moves to take.
Later on, we'll see how to modify this function so that we also pass it
the number of moves to take instead of that number being hardcoded like
it is now.

<span id="monad-laws"></span>

## Monad laws

<img src="/assets/judgedog.png" class="right"
width="343" height="170"
alt="the court finds you guilty of peeing all over everything" />

Just like applicative functors, and functors before them, monads come
with a few laws that all monad instances must abide by. Just because
something is made an instance of the <span class="fixed">Monad</span>
type class doesn't mean that it's a monad, it just means that it was
made an instance of a type class. For a type to truly be a monad, the
monad laws must hold for that type. These laws allow us to make
reasonable assumptions about the type and its behavior.

Haskell allows any type to be an instance of any type class as long as
the types check out. It can't check if the monad laws hold for a type
though, so if we're making a new instance of the
<span class="fixed">Monad</span> type class, we have to be reasonably
sure that all is well with the monad laws for that type. We can rely on
the types that come with the standard library to satisfy the laws, but
later when we go about making our own monads, we're going to have to
manually check the if the laws hold. But don't worry, they're not
complicated.

### Left identity

The first monad law states that if we take a value, put it in a default
context with <span class="fixed">return</span> and then feed it to a
function by using <span class="fixed">&gt;&gt;=</span>, it's the same as
just taking the value and applying the function to it. To put it
formally:

- <span class="label law">return x &gt;&gt;= f</span> is the same damn thing
  as <span class="label law">f x</span>

If you look at monadic values as values with a context and
<span class="fixed">return</span> as taking a value and putting it in a
default minimal context that still presents that value as its result, it
makes sense, because if that context is really minimal, feeding this
monadic value to a function shouldn't be much different than just
applying the function to the normal value, and indeed it isn't different
at all.

For the <span class="fixed">Maybe</span> monad
<span class="fixed">return</span> is defined as
<span class="fixed">Just</span>. The <span class="fixed">Maybe</span>
monad is all about possible failure, and if we have a value and want to
put it in such a context, it makes sense that we treat it as a
successful computation because, well, we know what the value is. Here's
some <span class="fixed">return</span> usage with
<span class="fixed">Maybe</span>:

``` haskell:hs
ghci> return 3 >>= (\x -> Just (x+100000))
Just 100003
ghci> (\x -> Just (x+100000)) 3
Just 100003
```

For the list monad <span class="fixed">return</span> puts something in a
singleton list. The <span class="fixed">&gt;&gt;=</span> implementation for
lists goes over all the values in the list and applies the function to
them, but since there's only one value in a singleton list, it's the
same as applying the function to that value:

``` haskell:hs
ghci> return "WoM" >>= (\x -> [x,x,x])
["WoM","WoM","WoM"]
ghci> (\x -> [x,x,x]) "WoM"
["WoM","WoM","WoM"]
```

We said that for <span class="fixed">IO</span>, using
<span class="fixed">return</span> makes an I/O action that has no
side-effects but just presents a value as its result. So it makes sense
that this law holds for <span class="fixed">IO</span> as well.

### Right identity

The second law states that if we have a monadic value and we use
<span class="fixed">&gt;&gt;=</span> to feed it to
<span class="fixed">return</span>, the result is our original monadic
value. Formally:

- <span class="label law">m &gt;&gt;= return</span> is no different than
  just <span class="label law">m</span>

This one might be a bit less obvious than the first one, but let's take
a look at why it should hold. When we feed monadic values to functions
by using <span class="fixed">&gt;&gt;=</span>, those functions take normal
values and return monadic ones. <span class="fixed">return</span> is
also one such function, if you consider its type. Like we said,
<span class="fixed">return</span> puts a value in a minimal context that
still presents that value as its result. This means that, for instance,
for <span class="fixed">Maybe</span>, it doesn't introduce any failure
and for lists, it doesn't introduce any extra non-determinism. Here's a
test run for a few monads:

``` haskell:hs
ghci> Just "move on up" >>= (\x -> return x)
Just "move on up"
ghci> [1,2,3,4] >>= (\x -> return x)
[1,2,3,4]
ghci> putStrLn "Wah!" >>= (\x -> return x)
Wah!
```

If we take a closer look at the list example, the implementation for
<span class="fixed">&gt;&gt;=</span> is:

``` haskell:hs
xs >>= f = concat (map f xs)
```

So when we feed <span class="fixed">\[1,2,3,4\]</span> to
<span class="fixed">return</span>, first
<span class="fixed">return</span> gets mapped over
<span class="fixed">\[1,2,3,4\]</span>, resulting in
<span class="fixed">\[\[1\],\[2\],\[3\],\[4\]\]</span> and then this
gets concatenated and we have our original list.

Left identity and right identity are basically laws that describe how
<span class="fixed">return</span> should behave. It's an important
function for making normal values into monadic ones and it wouldn't be
good if the monadic value that it produced did a lot of other stuff.

### Associativity

The final monad law says that when we have a chain of monadic function
applications with <span class="fixed">&gt;&gt;=</span>, it shouldn't matter
how they're nested. Formally written:

- Doing <span class="label law">(m &gt;&gt;= f) &gt;&gt;= g</span> is just like
  doing <span class="label law">m &gt;&gt;= (\x -&gt; f x &gt;&gt;= g)</span>

Hmmm, now what's going on here? We have one monadic value,
<span class="fixed">m</span> and two monadic functions
<span class="fixed">f</span> and <span class="fixed">g</span>. When
we're doing <span class="fixed">(m &gt;&gt;= f) &gt;&gt;= g</span>, we're
feeding <span class="fixed">m</span> to <span class="fixed">f</span>,
which results in a monadic value. Then, we feed that monadic value to
<span class="fixed">g</span>. In the expression <span class="fixed">m
&gt;&gt;= (\x -&gt; f x &gt;&gt;= g)</span>, we take a monadic value and we feed
it to a function that feeds the result of <span class="fixed">f x</span>
to <span class="fixed">g</span>. It's not easy to see how those two are
equal, so let's take a look at an example that makes this equality a bit
clearer.

Remember when we had our tightrope walker Pierre walk a rope while birds
landed on his balancing pole? To simulate birds landing on his balancing
pole, we made a chain of several functions that might produce failure:

``` haskell:hs
ghci> return (0,0) >>= landRight 2 >>= landLeft 2 >>= landRight 2
Just (2,4)
```

We started with <span class="fixed">Just (0,0)</span> and then bound
that value to the next monadic function, <span class="fixed">landRight
2</span>. The result of that was another monadic value which got bound
into the next monadic function, and so on. If we were to explicitly
parenthesize this, we'd write:

``` haskell:hs
ghci> ((return (0,0) >>= landRight 2) >>= landLeft 2) >>= landRight 2
Just (2,4)
```

But we can also write the routine like this:

``` haskell:hs
return (0,0) >>= (\x ->
landRight 2 x >>= (\y ->
landLeft 2 y >>= (\z ->
landRight 2 z)))
```

<span class="fixed">return (0,0)</span> is the same as
<span class="fixed">Just (0,0)</span> and when we feed it to the lambda,
the <span class="fixed">x</span> becomes
<span class="fixed">(0,0)</span>. <span class="fixed">landRight</span>
takes a number of birds and a pole (a tuple of numbers) and that's what
it gets passed. This results in a <span class="fixed">Just (0,2)</span>
and when we feed this to the next lambda, <span class="fixed">y</span>
is <span class="fixed">(0,2)</span>. This goes on until the final bird
landing produces a <span class="fixed">Just (2,4)</span>, which is
indeed the result of the whole expression.

So it doesn't matter how you nest feeding values to monadic functions,
what matters is their meaning. Here's another way to look at this law:
consider composing two functions, <span class="fixed">f</span> and
<span class="fixed">g</span>. Composing two functions is implemented
like so:

``` haskell:hs
(.) :: (b -> c) -> (a -> b) -> (a -> c)
f . g = (\x -> f (g x))
```

If the type of <span class="fixed">g</span> is <span class="fixed">a -&gt;
b</span> and the type of <span class="fixed">f</span> is
<span class="fixed">b -&gt; c</span>, we arrange them into a new function
which has a type of <span class="fixed">a -&gt; c</span>, so that its
parameter is passed between those functions. Now what if those two
functions were monadic, that is, what if the values they returned were
monadic values? If we had a function of type <span class="fixed">a -&gt; m
b</span>, we couldn't just pass its result to a function of type
<span class="fixed">b -&gt; m c</span>, because that function accepts a
normal <span class="fixed">b</span>, not a monadic one. We could
however, use <span class="fixed">&gt;&gt;=</span> to make that happen. So by
using <span class="fixed">&gt;&gt;=</span>, we can compose two monadic
functions:

``` haskell:hs
(<=<) :: (Monad m) => (b -> m c) -> (a -> m b) -> (a -> m c)
f <=< g = (\x -> g x >>= f)
```

So now we can compose two monadic functions:

``` haskell:hs
ghci> let f x = [x,-x]
ghci> let g x = [x*3,x*2]
ghci> let h = f <=< g
ghci> h 3
[9,-9,6,-6]
```

Cool. So what does that have to do with the associativity law? Well,
when we look at the law as a law of compositions, it states that
<span class="label law">f &lt;=&lt; (g &lt;=&lt; h)</span> should be the same as
<span class="label law">(f &lt;=&lt; g) &lt;=&lt; h</span>. This is just another
way of saying that for monads, the nesting of operations shouldn't
matter.

If we translate the first two laws to use
<span class="fixed">&lt;=&lt;</span>, then the left identity law states that
for every monadic function <span class="fixed">f</span>,
<span class="label law">f &lt;=&lt; return</span> is the same as writing
just <span class="label law">f</span> and the right identity law says
that <span class="label law">return &lt;=&lt; f</span> is also no different
from <span class="label law">f</span>.

This is very similar to how if <span class="fixed">f</span> is a normal
function, <span class="fixed">(f . g) . h</span> is the same as
<span class="fixed">f . (g . h)</span>, <span class="fixed">f .
id</span> is always the same as <span class="fixed">f</span> and
<span class="fixed">id . f</span> is also just
<span class="fixed">f</span>.

In this chapter, we took a look at the basics of monads and learned how
the <span class="fixed">Maybe</span> monad and the list monad work. In
the next chapter, we'll take a look at a whole bunch of other cool
monads and we'll also learn how to make our own.

