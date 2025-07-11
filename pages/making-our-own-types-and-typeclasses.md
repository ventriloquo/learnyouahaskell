---
title: Making Our Own Types and Typeclasses
---




<center>
<a href="modules" class="prevlink">Modules</a>
<a href="/pages/chapters">Table of contents</a>
<a href="input-and-output" class="nxtlink">Input and Output</a>
</center>




In the previous chapters, we covered some existing Haskell types and
typeclasses. In this chapter, we'll learn how to make our own and how to
put them to work!

<span id="algebraic-data-types"></span>

## Algebraic data types intro

So far, we've run into a lot of data types.
<span class="fixed">Bool</span>, <span class="fixed">Int</span>,
<span class="fixed">Char</span>, <span class="fixed">Maybe</span>, etc.
But how do we make our own? Well, one way is to use the *data* keyword
to define a type. Let's see how the <span class="fixed">Bool</span> type
is defined in the standard library.

``` haskell:hs
data Bool = False | True
```

<span class="fixed">data</span> means that we're defining a new data
type. The part before the <span class="fixed">=</span> denotes the type,
which is <span class="fixed">Bool</span>. The parts after the
<span class="fixed">=</span> are *value constructors*. They specify the
different values that this type can have. The
<span class="fixed">|</span> is read as *or*. So we can read this as:
the <span class="fixed">Bool</span> type can have a value of
<span class="fixed">True</span> or <span class="fixed">False</span>.
Both the type name and the value constructors have to be capital cased.

In a similar fashion, we can think of the <span class="fixed">Int</span>
type as being defined like this:

``` haskell:hs
data Int = -2147483648 | -2147483647 | ... | -1 | 0 | 1 | 2 | ... | 2147483647
```

<img src="/assets/caveman.png" class="left"
width="220" height="215" alt="caveman" />

The first and last value constructors are the minimum and maximum
possible values of <span class="fixed">Int</span>. It's not actually
defined like this, the ellipses are here because we omitted a heapload
of numbers, so this is just for illustrative purposes.

Now, let's think about how we would represent a shape in Haskell. One
way would be to use tuples. A circle could be denoted as
<span class="fixed">(43.1, 55.0, 10.4)</span> where the first and second
fields are the coordinates of the circle's center and the third field is
the radius. Sounds OK, but those could also represent a 3D vector or
anything else. A better solution would be to make our own type to
represent a shape. Let's say that a shape can be a circle or a
rectangle. Here it is:

``` haskell:hs
data Shape = Circle Float Float Float | Rectangle Float Float Float Float 
```

Now what's this? Think of it like this. The
<span class="fixed">Circle</span> value constructor has three fields,
which take floats. So when we write a value constructor, we can
optionally add some types after it and those types define the values it
will contain. Here, the first two fields are the coordinates of its
center, the third one its radius. The
<span class="fixed">Rectangle</span> value constructor has four fields
which accept floats. The first two are the coordinates to its upper left
corner and the second two are coordinates to its lower right one.

Now when I say fields, I actually mean parameters. Value constructors
are actually functions that ultimately return a value of a data type.
Let's take a look at the type signatures for these two value
constructors.

``` haskell:hs
ghci> :t Circle
Circle :: Float -> Float -> Float -> Shape
ghci> :t Rectangle
Rectangle :: Float -> Float -> Float -> Float -> Shape
```

Cool, so value constructors are functions like everything else. Who
would have thought? Let's make a function that takes a shape and returns
its surface.

``` haskell:hs
surface :: Shape -> Float
surface (Circle _ _ r) = pi * r ^ 2
surface (Rectangle x1 y1 x2 y2) = (abs $ x2 - x1) * (abs $ y2 - y1)
```

The first notable thing here is the type declaration. It says that the
function takes a shape and returns a float. We couldn't write a type
declaration of <span class="fixed">Circle -&gt; Float</span> because
<span class="fixed">Circle</span> is not a type,
<span class="fixed">Shape</span> is. Just like we can't write a function
with a type declaration of <span class="fixed">True -&gt; Int</span>. The
next thing we notice here is that we can pattern match against
constructors. We pattern matched against constructors before (all the
time actually) when we pattern matched against values like
<span class="fixed">\[\]</span> or <span class="fixed">False</span> or
<span class="fixed">5</span>, only those values didn't have any fields.
We just write a constructor and then bind its fields to names. Because
we're interested in the radius, we don't actually care about the first
two fields, which tell us where the circle is.

``` haskell:hs
ghci> surface $ Circle 10 20 10
314.15927
ghci> surface $ Rectangle 0 0 100 100
10000.0
```

Yay, it works! But if we try to just print out
<span class="fixed">Circle 10 20 5</span> in the prompt, we'll get an
error. That's because Haskell doesn't know how to display our data type
as a string (yet). Remember, when we try to print a value out in the
prompt, Haskell first runs the <span class="fixed">show</span> function
to get the string representation of our value and then it prints that
out to the terminal. To make our <span class="fixed">Shape</span> type
part of the <span class="fixed">Show</span> typeclass, we modify it like
this:

``` haskell:hs
data Shape = Circle Float Float Float | Rectangle Float Float Float Float deriving (Show)
```

We won't concern ourselves with deriving too much for now. Let's just
say that if we add <span class="fixed">deriving (Show)</span> at the end
of a *data* declaration, Haskell automagically makes that type part of
the <span class="fixed">Show</span> typeclass. So now, we can do this:

``` haskell:hs
ghci> Circle 10 20 5
Circle 10.0 20.0 5.0
ghci> Rectangle 50 230 60 90
Rectangle 50.0 230.0 60.0 90.0
```

Value constructors are functions, so we can map them and partially apply
them and everything. If we want a list of concentric circles with
different radii, we can do this.

``` haskell:hs
ghci> map (Circle 10 20) [4,5,6,6]
[Circle 10.0 20.0 4.0,Circle 10.0 20.0 5.0,Circle 10.0 20.0 6.0,Circle 10.0 20.0 6.0]
```

Our data type is good, although it could be better. Let's make an
intermediate data type that defines a point in two-dimensional space.
Then we can use that to make our shapes more understandable.

``` haskell:hs
data Point = Point Float Float deriving (Show)
data Shape = Circle Point Float | Rectangle Point Point deriving (Show)
```

Notice that when defining a point, we used the same name for the data
type and the value constructor. This has no special meaning, although
it's common to use the same name as the type if there's only one value
constructor. So now the <span class="fixed">Circle</span> has two
fields, one is of type <span class="fixed">Point</span> and the other of
type <span class="fixed">Float</span>. This makes it easier to
understand what's what. Same goes for the rectangle. We have to adjust
our <span class="fixed">surface</span> function to reflect these
changes.

``` haskell:hs
surface :: Shape -> Float
surface (Circle _ r) = pi * r ^ 2
surface (Rectangle (Point x1 y1) (Point x2 y2)) = (abs $ x2 - x1) * (abs $ y2 - y1)
```

The only thing we had to change were the patterns. We disregarded the
whole point in the circle pattern. In the rectangle pattern, we just
used a nested pattern matching to get the fields of the points. If we
wanted to reference the points themselves for some reason, we could have
used as-patterns.

``` haskell:hs
ghci> surface (Rectangle (Point 0 0) (Point 100 100))
10000.0
ghci> surface (Circle (Point 0 0) 24)
1809.5574
```

How about a function that nudges a shape? It takes a shape, the amount
to move it on the x axis and the amount to move it on the y axis and
then returns a new shape that has the same dimensions, only it's located
somewhere else.

``` haskell:hs
nudge :: Shape -> Float -> Float -> Shape
nudge (Circle (Point x y) r) a b = Circle (Point (x+a) (y+b)) r
nudge (Rectangle (Point x1 y1) (Point x2 y2)) a b = Rectangle (Point (x1+a) (y1+b)) (Point (x2+a) (y2+b))
```

Pretty straightforward. We add the nudge amounts to the points that
denote the position of the shape.

``` haskell:hs
ghci> nudge (Circle (Point 34 34) 10) 5 10
Circle (Point 39.0 44.0) 10.0
```

If we don't want to deal directly with points, we can make some
auxilliary functions that create shapes of some size at the zero
coordinates and then nudge those.

``` haskell:hs
baseCircle :: Float -> Shape
baseCircle r = Circle (Point 0 0) r

baseRect :: Float -> Float -> Shape
baseRect width height = Rectangle (Point 0 0) (Point width height)
```

``` haskell:hs
ghci> nudge (baseRect 40 100) 60 23
Rectangle (Point 60.0 23.0) (Point 100.0 123.0)
```

You can, of course, export your data types in your modules. To do that,
just write your type along with the functions you are exporting and then
add some parentheses and in them specify the value constructors that you
want to export for it, separated by commas. If you want to export all
the value constructors for a given type, just write
<span class="fixed">..</span>.

If we wanted to export the functions and types that we defined here in a
module, we could start it off like this:

``` haskell:hs
module Shapes 
( Point(..)
, Shape(..)
, surface
, nudge
, baseCircle
, baseRect
) where
```

By doing <span class="fixed">Shape(..)</span>, we exported all the value
constructors for <span class="fixed">Shape</span>, so that means that
whoever imports our module can make shapes by using the
<span class="fixed">Rectangle</span> and
<span class="fixed">Circle</span> value constructors. It's the same as
writing <span class="fixed">Shape (Rectangle, Circle)</span>.

We could also opt not to export any value constructors for
<span class="fixed">Shape</span> by just writing
<span class="fixed">Shape</span> in the export statement. That way,
someone importing our module could only make shapes by using the
auxilliary functions <span class="fixed">baseCircle</span> and
<span class="fixed">baseRect</span>. <span class="fixed">Data.Map</span>
uses that approach. You can't create a map by doing
<span class="fixed">Map.Map \[(1,2),(3,4)\]</span> because it doesn't
export that value constructor. However, you can make a mapping by using
one of the auxilliary functions like
<span class="fixed">Map.fromList</span>. Remember, value constructors
are just functions that take the fields as parameters and return a value
of some type (like <span class="fixed">Shape</span>) as a result. So
when we choose not to export them, we just prevent the person importing
our module from using those functions, but if some other functions that
are exported return a type, we can use them to make values of our custom
data types.

Not exporting the value constructors of a data types makes them more
abstract in such a way that we hide their implementation. Also, whoever
uses our module can't pattern match against the value constructors.

<span id="record-syntax"></span>

## Record syntax

<img src="/assets/record.png" class="right"
width="208" height="97" alt="record" />

OK, we've been tasked with creating a data type that describes a person.
The info that we want to store about that person is: first name, last
name, age, height, phone number, and favorite ice-cream flavor. I don't
know about you, but that's all I ever want to know about a person. Let's
give it a go!

``` haskell:hs
data Person = Person String String Int Float String String deriving (Show)
```

O-kay. The first field is the first name, the second is the last name,
the third is the age and so on. Let's make a person.

``` haskell:hs
ghci> let guy = Person "Buddy" "Finklestein" 43 184.2 "526-2928" "Chocolate"
ghci> guy
Person "Buddy" "Finklestein" 43 184.2 "526-2928" "Chocolate"
```

That's kind of cool, although slightly unreadable. What if we want to
create a function to get seperate info from a person? A function that
gets some person's first name, a function that gets some person's last
name, etc. Well, we'd have to define them kind of like this.

``` haskell:hs
firstName :: Person -> String
firstName (Person firstname _ _ _ _ _) = firstname

lastName :: Person -> String
lastName (Person _ lastname _ _ _ _) = lastname

age :: Person -> Int
age (Person _ _ age _ _ _) = age

height :: Person -> Float
height (Person _ _ _ height _ _) = height

phoneNumber :: Person -> String
phoneNumber (Person _ _ _ _ number _) = number

flavor :: Person -> String
flavor (Person _ _ _ _ _ flavor) = flavor
```

Whew! I certainly did not enjoy writing that! Despite being very
cumbersome and BORING to write, this method works.

``` haskell:hs
ghci> let guy = Person "Buddy" "Finklestein" 43 184.2 "526-2928" "Chocolate"
ghci> firstName guy
"Buddy"
ghci> height guy
184.2
ghci> flavor guy
"Chocolate"
```

There must be a better way, you say! Well no, there isn't, sorry.

Just kidding, there is. Hahaha! The makers of Haskell were very smart
and anticipated this scenario. They included an alternative way to write
data types. Here's how we could achieve the above functionality with
record syntax.

``` haskell:hs
data Person = Person { firstName :: String
                     , lastName :: String
                     , age :: Int
                     , height :: Float
                     , phoneNumber :: String
                     , flavor :: String
                     } deriving (Show) 
```

So instead of just naming the field types one after another and
separating them with spaces, we use curly brackets. First we write the
name of the field, for instance, <span class="fixed">firstName</span>
and then we write a double colon <span class="fixed">::</span> (also
called Paamayim Nekudotayim, haha) and then we specify the type. The
resulting data type is exactly the same. The main benefit of this is
that it creates functions that lookup fields in the data type. By using
record syntax to create this data type, Haskell automatically made these
functions: <span class="fixed">firstName</span>,
<span class="fixed">lastName</span>, <span class="fixed">age</span>,
<span class="fixed">height</span>,
<span class="fixed">phoneNumber</span> and
<span class="fixed">flavor</span>.

``` haskell:hs
ghci> :t flavor
flavor :: Person -> String
ghci> :t firstName
firstName :: Person -> String
```

There's another benefit to using record syntax. When we derive
<span class="fixed">Show</span> for the type, it displays it differently
if we use record syntax to define and instantiate the type. Say we have
a type that represents a car. We want to keep track of the company that
made it, the model name and its year of production. Watch.

``` haskell:hs
data Car = Car String String Int deriving (Show)
```

``` haskell:hs
ghci> Car "Ford" "Mustang" 1967
Car "Ford" "Mustang" 1967
```

If we define it using record syntax, we can make a new car like this.

``` haskell:hs
data Car = Car {company :: String, model :: String, year :: Int} deriving (Show)
```

``` haskell:hs
ghci> Car {company="Ford", model="Mustang", year=1967}
Car {company = "Ford", model = "Mustang", year = 1967}
```

When making a new car, we don't have to necessarily put the fields in
the proper order, as long as we list all of them. But if we don't use
record syntax, we have to specify them in order.

Use record syntax when a constructor has several fields and it's not
obvious which field is which. If we make a 3D vector data type by doing
<span class="fixed">data Vector = Vector Int Int Int</span>, it's pretty
obvious that the fields are the components of a vector. However, in our
<span class="fixed">Person</span> and <span class="fixed">Car</span>
types, it wasn't so obvious and we greatly benefited from using record
syntax.

<span id="type-parameters"></span>

## Type parameters

A value constructor can take some values parameters and then produce a
new value. For instance, the <span class="fixed">Car</span> constructor
takes three values and produces a car value. In a similar manner, *type
constructors* can take types as parameters to produce new types. This
might sound a bit too meta at first, but it's not that complicated. If
you're familiar with templates in C++, you'll see some parallels. To get
a clear picture of what type parameters work like in action, let's take
a look at how a type we've already met is implemented.

``` haskell:hs
data Maybe a = Nothing | Just a
```

<img src="/assets/yeti.png" class="left"
width="209" height="260" alt="yeti" />

The <span class="fixed">a</span> here is the type parameter. And because
there's a type parameter involved, we call
<span class="fixed">Maybe</span> a type constructor. Depending on what
we want this data type to hold when it's not
<span class="fixed">Nothing</span>, this type constructor can end up
producing a type of <span class="fixed">Maybe Int</span>,
<span class="fixed">Maybe Car</span>, <span class="fixed">Maybe
String</span>, etc. No value can have a type of just
<span class="fixed">Maybe</span>, because that's not a type per se, it's
a type constructor. In order for this to be a real type that a value can
be part of, it has to have all its type parameters filled up.

So if we pass <span class="fixed">Char</span> as the type parameter to
<span class="fixed">Maybe</span>, we get a type of
<span class="fixed">Maybe Char</span>. The value
<span class="fixed">Just 'a'</span> has a type of
<span class="fixed">Maybe Char</span>, for example.

You might not know it, but we used a type that has a type parameter
before we used <span class="fixed">Maybe</span>. That type is the list
type. Although there's some syntactic sugar in play, the list type takes
a parameter to produce a concrete type. Values can have an
<span class="fixed">\[Int\]</span> type, a
<span class="fixed">\[Char\]</span> type, a
<span class="fixed">\[\[String\]\]</span> type, but you can't have a
value that just has a type of <span class="fixed">\[\]</span>.

Let's play around with the <span class="fixed">Maybe</span> type.

``` haskell:hs
ghci> Just "Haha"
Just "Haha"
ghci> Just 84
Just 84
ghci> :t Just "Haha"
Just "Haha" :: Maybe [Char]
ghci> :t Just 84
Just 84 :: (Num t) => Maybe t
ghci> :t Nothing
Nothing :: Maybe a
ghci> Just 10 :: Maybe Double
Just 10.0
```

Type parameters are useful because we can make different types with them
depending on what kind of types we want contained in our data type. When
we do <span class="fixed">:t Just "Haha"</span>, the type inference
engine figures it out to be of the type <span class="fixed">Maybe
\[Char\]</span>, because if the <span class="fixed">a</span> in the
<span class="fixed">Just a</span> is a string, then the
<span class="fixed">a</span> in <span class="fixed">Maybe a</span> must
also be a string.

Notice that the type of <span class="fixed">Nothing</span> is
<span class="fixed">Maybe a</span>. Its type is polymorphic. If some
function requires a <span class="fixed">Maybe Int</span> as a parameter,
we can give it a <span class="fixed">Nothing</span>, because a
<span class="fixed">Nothing</span> doesn't contain a value anyway and so
it doesn't matter. The <span class="fixed">Maybe a</span> type can act
like a <span class="fixed">Maybe Int</span> if it has to, just like
<span class="fixed">5</span> can act like an
<span class="fixed">Int</span> or a <span class="fixed">Double</span>.
Similarly, the type of the empty list is
<span class="fixed">\[a\]</span>. An empty list can act like a list of
anything. That's why we can do <span class="fixed">\[1,2,3\] ++
\[\]</span> and <span class="fixed">\["ha","ha","ha"\] ++ \[\]</span>.

Using type parameters is very beneficial, but only when using them makes
sense. Usually we use them when our data type would work regardless of
the type of the value it then holds inside it, like with our
<span class="fixed">Maybe a</span> type. If our type acts as some kind
of box, it's good to use them. We could change our
<span class="fixed">Car</span> data type from this:

``` haskell:hs
data Car = Car { company :: String
               , model :: String
               , year :: Int
               } deriving (Show)
```

To this:

``` haskell:hs
data Car a b c = Car { company :: a
                     , model :: b
                     , year :: c 
                     } deriving (Show)
```

But would we really benefit? The answer is: probably no, because we'd
just end up defining functions that only work on the
<span class="fixed">Car String String Int</span> type. For instance,
given our first definition of <span class="fixed">Car</span>, we could
make a function that displays the car's properties in a nice little
text.

``` haskell:hs
tellCar :: Car -> String
tellCar (Car {company = c, model = m, year = y}) = "This " ++ c ++ " " ++ m ++ " was made in " ++ show y
```

``` haskell:hs
ghci> let stang = Car {company="Ford", model="Mustang", year=1967}
ghci> tellCar stang
"This Ford Mustang was made in 1967"
```

A cute little function! The type declaration is cute and it works
nicely. Now what if <span class="fixed">Car</span> was
<span class="fixed">Car a b c</span>?

``` haskell:hs
tellCar :: (Show a) => Car String String a -> String
tellCar (Car {company = c, model = m, year = y}) = "This " ++ c ++ " " ++ m ++ " was made in " ++ show y
```

We'd have to force this function to take a
<span class="fixed">Car</span> type of <span class="fixed">(Show a) =&gt;
Car String String a</span>. You can see that the type signature is more
complicated and the only benefit we'd actually get would be that we can
use any type that's an instance of the <span class="fixed">Show</span>
typeclass as the type for <span class="fixed">c</span>.

``` haskell:hs
ghci> tellCar (Car "Ford" "Mustang" 1967)
"This Ford Mustang was made in 1967"
ghci> tellCar (Car "Ford" "Mustang" "nineteen sixty seven")
"This Ford Mustang was made in \"nineteen sixty seven\""
ghci> :t Car "Ford" "Mustang" 1967
Car "Ford" "Mustang" 1967 :: (Num t) => Car [Char] [Char] t
ghci> :t Car "Ford" "Mustang" "nineteen sixty seven"
Car "Ford" "Mustang" "nineteen sixty seven" :: Car [Char] [Char] [Char]
```

<img src="/assets/meekrat.png" class="right"
width="150" height="267" alt="meekrat" />

In real life though, we'd end up using <span class="fixed">Car String
String Int</span> most of the time and so it would seem that
parameterizing the <span class="fixed">Car</span> type isn't really
worth it. We usually use type parameters when the type that's contained
inside the data type's various value constructors isn't really that
important for the type to work. A list of stuff is a list of stuff and
it doesn't matter what the type of that stuff is, it can still work. If
we want to sum a list of numbers, we can specify later in the summing
function that we specifically want a list of numbers. Same goes for
<span class="fixed">Maybe</span>. <span class="fixed">Maybe</span>
represents an option of either having nothing or having one of
something. It doesn't matter what the type of that something is.

Another example of a parameterized type that we've already met is
<span class="fixed">Map k v</span> from
<span class="fixed">Data.Map</span>. The <span class="fixed">k</span> is
the type of the keys in a map and the <span class="fixed">v</span> is
the type of the values. This is a good example of where type parameters
are very useful. Having maps parameterized enables us to have mappings
from any type to any other type, as long as the type of the key is part
of the <span class="fixed">Ord</span> typeclass. If we were defining a
mapping type, we could add a typeclass constraint in the *data*
declaration:

``` haskell:hs
data (Ord k) => Map k v = ...
```

However, it's a very strong convention in Haskell to *never add
typeclass constraints in data declarations.* Why? Well, because we don't
benefit a lot, but we end up writing more class constraints, even when
we don't need them. If we put or don't put the <span class="fixed">Ord
k</span> constraint in the *data* declaration for
<span class="fixed">Map k v</span>, we're going to have to put the
constraint into functions that assume the keys in a map can be ordered.
But if we don't put the constraint in the data declaration, we don't
have to put <span class="fixed">(Ord k) =&gt;</span> in the type
declarations of functions that don't care whether the keys can be
ordered or not. An example of such a function is
<span class="fixed">toList</span>, that just takes a mapping and
converts it to an associative list. Its type signature is
<span class="fixed">toList :: Map k a -&gt; \[(k, a)\]</span>. If
<span class="fixed">Map k v</span> had a type constraint in its *data*
declaration, the type for <span class="fixed">toList</span> would have
to be <span class="fixed">toList :: (Ord k) =&gt; Map k a -&gt; \[(k,
a)\]</span>, even though the function doesn't do any comparing of keys
by order.

So don't put type constraints into *data* declarations even if it seems
to make sense, because you'll have to put them into the function type
declarations either way.

Let's implement a 3D vector type and add some operations for it. We'll
be using a parameterized type because even though it will usually
contain numeric types, it will still support several of them.

``` haskell:hs
data Vector a = Vector a a a deriving (Show)

vplus :: (Num t) => Vector t -> Vector t -> Vector t
(Vector i j k) `vplus` (Vector l m n) = Vector (i+l) (j+m) (k+n)

vectMult :: (Num t) => Vector t -> t -> Vector t
(Vector i j k) `vectMult` m = Vector (i*m) (j*m) (k*m)

scalarMult :: (Num t) => Vector t -> Vector t -> t
(Vector i j k) `scalarMult` (Vector l m n) = i*l + j*m + k*n
```

<span class="fixed">vplus</span> is for adding two vectors together. Two
vectors are added just by adding their corresponding components.
<span class="fixed">scalarMult</span> is for the scalar product of two
vectors and <span class="fixed">vectMult</span> is for multiplying a
vector with a scalar. These functions can operate on types of
<span class="fixed">Vector Int</span>, <span class="fixed">Vector
Integer</span>, <span class="fixed">Vector Float</span>, whatever, as
long as the <span class="fixed">a</span> from <span class="fixed">Vector
a</span> is from the <span class="fixed">Num</span> typeclass. Also, if
you examine the type declaration for these functions, you'll see that
they can operate only on vectors of the same type and the numbers
involved must also be of the type that is contained in the vectors.
Notice that we didn't put a <span class="fixed">Num</span> class
constraint in the *data* declaration, because we'd have to repeat it in
the functions anyway.

Once again, it's very important to distinguish between the type
constructor and the value constructor. When declaring a data type, the
part before the <span class="fixed">=</span> is the type constructor and
the constructors after it (possibly separated by
<span class="fixed">|</span>'s) are value constructors. Giving a
function a type of <span class="fixed">Vector t t t -&gt; Vector t t t -&gt;
t</span> would be wrong, because we have to put types in type
declaration and the vector *type* constructor takes only one parameter,
whereas the value constructor takes three. Let's play around with our
vectors.

``` haskell:hs
ghci> Vector 3 5 8 `vplus` Vector 9 2 8
Vector 12 7 16
ghci> Vector 3 5 8 `vplus` Vector 9 2 8 `vplus` Vector 0 2 3
Vector 12 9 19
ghci> Vector 3 9 7 `vectMult` 10
Vector 30 90 70
ghci> Vector 4 9 5 `scalarMult` Vector 9.0 2.0 4.0
74.0
ghci> Vector 2 9 3 `vectMult` (Vector 4 9 5 `scalarMult` Vector 9 2 4)
Vector 148 666 222
```

<span id="derived-instances"></span>

## Derived instances

<img src="/assets/gob.png" class="right"
width="112" height="350" alt="gob" />

In the [Typeclasses 101](types-and-typeclasses#typeclasses-101) section,
we explained the basics of typeclasses. We explained that a typeclass is
a sort of an interface that defines some behavior. A type can be made an
*instance* of a typeclass if it supports that behavior. Example: the
<span class="fixed">Int</span> type is an instance of the
<span class="fixed">Eq</span> typeclass because the
<span class="fixed">Eq</span> typeclass defines behavior for stuff that
can be equated. And because integers can be equated,
<span class="fixed">Int</span> is a part of the
<span class="fixed">Eq</span> typeclass. The real usefulness comes with
the functions that act as the interface for
<span class="fixed">Eq</span>, namely <span class="fixed">==</span> and
<span class="fixed">/=</span>. If a type is a part of the
<span class="fixed">Eq</span> typeclass, we can use the
<span class="fixed">==</span> functions with values of that type. That's
why expressions like <span class="fixed">4 == 4</span> and
<span class="fixed">"foo" /= "bar"</span> typecheck.

We also mentioned that they're often confused with classes in languages
like Java, Python, C++ and the like, which then baffles a lot of people.
In those languages, classes are a blueprint from which we then create
objects that contain state and can do some actions. Typeclasses are more
like interfaces. We don't make data from typeclasses. Instead, we first
make our data type and then we think about what it can act like. If it
can act like something that can be equated, we make it an instance of
the <span class="fixed">Eq</span> typeclass. If it can act like
something that can be ordered, we make it an instance of the
<span class="fixed">Ord</span> typeclass.

In the next section, we'll take a look at how we can manually make our
types instances of typeclasses by implementing the functions defined by
the typeclasses. But right now, let's see how Haskell can automatically
make our type an instance of any of the following typeclasses:
<span class="fixed">Eq</span>, <span class="fixed">Ord</span>,
<span class="fixed">Enum</span>, <span class="fixed">Bounded</span>,
<span class="fixed">Show</span>, <span class="fixed">Read</span>.
Haskell can derive the behavior of our types in these contexts if we use
the *deriving* keyword when making our data type.

Consider this data type:

``` haskell:hs
data Person = Person { firstName :: String
                     , lastName :: String
                     , age :: Int
                     }
```

It describes a person. Let's assume that no two people have the same
combination of first name, last name and age. Now, if we have records
for two people, does it make sense to see if they represent the same
person? Sure it does. We can try to equate them and see if they're equal
or not. That's why it would make sense for this type to be part of the
<span class="fixed">Eq</span> typeclass. We'll derive the instance.

``` haskell:hs
data Person = Person { firstName :: String
                     , lastName :: String
                     , age :: Int
                     } deriving (Eq)
```

When we derive the <span class="fixed">Eq</span> instance for a type and
then try to compare two values of that type with
<span class="fixed">==</span> or <span class="fixed">/=</span>, Haskell
will see if the value constructors match (there's only one value
constructor here though) and then it will check if all the data
contained inside matches by testing each pair of fields with
<span class="fixed">==</span>. There's only one catch though, the types
of all the fields also have to be part of the
<span class="fixed">Eq</span> typeclass. But since both
<span class="fixed">String</span> and <span class="fixed">Int</span>
are, we're OK. Let's test our <span class="fixed">Eq</span> instance.

``` haskell:hs
ghci> let mikeD = Person {firstName = "Michael", lastName = "Diamond", age = 43}
ghci> let adRock = Person {firstName = "Adam", lastName = "Horovitz", age = 41}
ghci> let mca = Person {firstName = "Adam", lastName = "Yauch", age = 44}
ghci> mca == adRock
False
ghci> mikeD == adRock
False
ghci> mikeD == mikeD
True
ghci> mikeD == Person {firstName = "Michael", lastName = "Diamond", age = 43}
True
```

Of course, since <span class="fixed">Person</span> is now in
<span class="fixed">Eq</span>, we can use it as the
<span class="fixed">a</span> for all functions that have a class
constraint of <span class="fixed">Eq a</span> in their type signature,
such as <span class="fixed">elem</span>.

``` haskell:hs
ghci> let beastieBoys = [mca, adRock, mikeD]
ghci> mikeD `elem` beastieBoys
True
```

The <span class="fixed">Show</span> and <span class="fixed">Read</span>
typeclasses are for things that can be converted to or from strings,
respectively. Like with <span class="fixed">Eq</span>, if a type's
constructors have fields, their type has to be a part of
<span class="fixed">Show</span> or <span class="fixed">Read</span> if we
want to make our type an instance of them. Let's make our
<span class="fixed">Person</span> data type a part of
<span class="fixed">Show</span> and <span class="fixed">Read</span> as
well.

``` haskell:hs
data Person = Person { firstName :: String
                     , lastName :: String
                     , age :: Int
                     } deriving (Eq, Show, Read)
```

Now we can print a person out to the terminal.

``` haskell:hs
ghci> let mikeD = Person {firstName = "Michael", lastName = "Diamond", age = 43}
ghci> mikeD
Person {firstName = "Michael", lastName = "Diamond", age = 43}
ghci> "mikeD is: " ++ show mikeD
"mikeD is: Person {firstName = \"Michael\", lastName = \"Diamond\", age = 43}"
```

Had we tried to print a person on the terminal before making the
<span class="fixed">Person</span> data type part of
<span class="fixed">Show</span>, Haskell would have complained at us,
claiming it doesn't know how to represent a person as a string. But now
that we've derived a <span class="fixed">Show</span> instance for it, it
does know.

<span class="fixed">Read</span> is pretty much the inverse typeclass of
<span class="fixed">Show</span>. <span class="fixed">Show</span> is for
converting values of our a type to a string,
<span class="fixed">Read</span> is for converting strings to values of
our type. Remember though, when we use the
<span class="fixed">read</span> function, we have to use an explicit
type annotation to tell Haskell which type we want to get as a result.
If we don't make the type we want as a result explicit, Haskell doesn't
know which type we want.

``` haskell:hs
ghci> read "Person {firstName =\"Michael\", lastName =\"Diamond\", age = 43}" :: Person
Person {firstName = "Michael", lastName = "Diamond", age = 43}
```

If we use the result of our <span class="fixed">read</span> later on in
a way that Haskell can infer that it should read it as a person, we
don't have to use type annotation.

``` haskell:hs
ghci> read "Person {firstName =\"Michael\", lastName =\"Diamond\", age = 43}" == mikeD
True
```

We can also read parameterized types, but we have to fill in the type
parameters. So we can't do <span class="fixed">read "Just 't'" :: Maybe
a</span>, but we can do <span class="fixed">read "Just 't'" :: Maybe
Char</span>.

We can derive instances for the <span class="fixed">Ord</span> type
class, which is for types that have values that can be ordered. If we
compare two values of the same type that were made using different
constructors, the value which was made with a constructor that's defined
first is considered smaller. For instance, consider the
<span class="fixed">Bool</span> type, which can have a value of either
<span class="fixed">False</span> or <span class="fixed">True</span>. For
the purpose of seeing how it behaves when compared, we can think of it
as being implemented like this:

``` haskell:hs
data Bool = False | True deriving (Ord)
```

Because the <span class="fixed">False</span> value constructor is
specified first and the <span class="fixed">True</span> value
constructor is specified after it, we can consider
<span class="fixed">True</span> as greater than
<span class="fixed">False</span>.

``` haskell:hs
ghci> True `compare` False
GT
ghci> True > False
True
ghci> True < False
False
```

In the <span class="fixed">Maybe a</span> data type, the
<span class="fixed">Nothing</span> value constructor is specified before
the <span class="fixed">Just</span> value constructor, so a value of
<span class="fixed">Nothing</span> is always smaller than a value of
<span class="fixed">Just something</span>, even if that something is
minus one billion trillion. But if we compare two
<span class="fixed">Just</span> values, then it goes to compare what's
inside them.

``` haskell:hs
ghci> Nothing < Just 100
True
ghci> Nothing > Just (-49999)
False
ghci> Just 3 `compare` Just 2
GT
ghci> Just 100 > Just 50
True
```

But we can't do something like <span class="fixed">Just (\*3) &gt; Just
(\*2)</span>, because <span class="fixed">(\*3)</span> and
<span class="fixed">(\*2)</span> are functions, which aren't instances
of <span class="fixed">Ord</span>.

We can easily use algebraic data types to make enumerations and the
<span class="fixed">Enum</span> and <span class="fixed">Bounded</span>
typeclasses help us with that. Consider the following data type:

``` haskell:hs
data Day = Monday | Tuesday | Wednesday | Thursday | Friday | Saturday | Sunday
```

Because all the value constructors are nullary (take no parameters, i.e.
fields), we can make it part of the <span class="fixed">Enum</span>
typeclass. The <span class="fixed">Enum</span> typeclass is for things
that have predecessors and successors. We can also make it part of the
<span class="fixed">Bounded</span> typeclass, which is for things that
have a lowest possible value and highest possible value. And while we're
at it, let's also make it an instance of all the other derivable
typeclasses and see what we can do with it.

``` haskell:hs
data Day = Monday | Tuesday | Wednesday | Thursday | Friday | Saturday | Sunday 
           deriving (Eq, Ord, Show, Read, Bounded, Enum)
```

Because it's part of the <span class="fixed">Show</span> and
<span class="fixed">Read</span> typeclasses, we can convert values of
this type to and from strings.

``` haskell:hs
ghci> Wednesday
Wednesday
ghci> show Wednesday
"Wednesday"
ghci> read "Saturday" :: Day
Saturday
```

Because it's part of the <span class="fixed">Eq</span> and
<span class="fixed">Ord</span> typeclasses, we can compare or equate
days.

``` haskell:hs
ghci> Saturday == Sunday
False
ghci> Saturday == Saturday
True
ghci> Saturday > Friday
True
ghci> Monday `compare` Wednesday
LT
```

It's also part of <span class="fixed">Bounded</span>, so we can get the
lowest and highest day.

``` haskell:hs
ghci> minBound :: Day
Monday
ghci> maxBound :: Day
Sunday
```

It's also an instance of <span class="fixed">Enum</span>. We can get
predecessors and successors of days and we can make list ranges from
them!

``` haskell:hs
ghci> succ Monday
Tuesday
ghci> pred Saturday
Friday
ghci> [Thursday .. Sunday]
[Thursday,Friday,Saturday,Sunday]
ghci> [minBound .. maxBound] :: [Day]
[Monday,Tuesday,Wednesday,Thursday,Friday,Saturday,Sunday]
```

That's pretty awesome.

<span id="type-synonyms"></span>

## Type synonyms

Previously, we mentioned that when writing types, the
<span class="fixed">\[Char\]</span> and <span class="fixed">String
</span> types are equivalent and interchangeable. That's implemented
with *type synonyms*. Type synonyms don't really do anything per se,
they're just about giving some types different names so that they make
more sense to someone reading our code and documentation. Here's how the
standard library defines <span class="fixed">String</span> as a synonym
for <span class="fixed">\[Char\]</span>.

``` haskell:hs
 type String = [Char]
 
```

<img src="/assets/chicken.png" class="left"
width="169" height="225" alt="chicken" />

We've introduced the *type* keyword. The keyword might be misleading to
some, because we're not actually making anything new (we did that with
the *data* keyword), but we're just making a synonym for an already
existing type.

If we make a function that converts a string to uppercase and call it
<span class="fixed">toUpperString</span> or something, we can give it a
type declaration of <span class="fixed">toUpperString :: \[Char\] -&gt;
\[Char\]</span> or <span class="fixed">toUpperString :: String -&gt;
String</span>. Both of these are essentially the same, only the latter
is nicer to read.

When we were dealing with the <span class="fixed">Data.Map</span>
module, we first represented a phonebook with an association list before
converting it into a map. As we've already found out, an association
list is a list of key-value pairs. Let's look at a phonebook that we
had.

``` haskell:hs
phoneBook :: [(String,String)]
phoneBook =    
    [("betty","555-2938")   
    ,("bonnie","452-2928")   
    ,("patsy","493-2928")   
    ,("lucille","205-2928")   
    ,("wendy","939-8282")   
    ,("penny","853-2492")   
    ]
```

We see that the type of <span class="fixed">phoneBook</span> is
<span class="fixed">\[(String,String)\]</span>. That tells us that it's
an association list that maps from strings to strings, but not much
else. Let's make a type synonym to convey some more information in the
type declaration.

``` haskell:hs
type PhoneBook = [(String,String)]
```

Now the type declaration for our phonebook can be
<span class="fixed">phoneBook :: PhoneBook</span>. Let's make a type
synonym for <span class="fixed">String</span> as well.

``` haskell:hs
type PhoneNumber = String
type Name = String
type PhoneBook = [(Name,PhoneNumber)]
```

Giving the <span class="fixed">String</span> type synonyms is something
that Haskell programmers do when they want to convey more information
about what strings in their functions should be used as and what they
represent.

So now, when we implement a function that takes a name and a number and
sees if that name and number combination is in our phonebook, we can
give it a very pretty and descriptive type declaration.

``` haskell:hs
inPhoneBook :: Name -> PhoneNumber -> PhoneBook -> Bool
inPhoneBook name pnumber pbook = (name,pnumber) `elem` pbook
```

If we decided not to use type synonyms, our function would have a type
of <span class="fixed">String -&gt; String -&gt; \[(String,String)\] -&gt;
Bool</span>. In this case, the type declaration that took advantage of
type synonyms is easier to understand. However, you shouldn't go
overboard with them. We introduce type synonyms either to describe what
some existing type represents in our functions (and thus our type
declarations become better documentation) or when something has a
long-ish type that's repeated a lot (like
<span class="fixed">\[(String,String)\]</span>) but represents something
more specific in the context of our functions.

Type synonyms can also be parameterized. If we want a type that
represents an association list type but still want it to be general so
it can use any type as the keys and values, we can do this:

``` haskell:hs
type AssocList k v = [(k,v)]
```

Now, a function that gets the value by a key in an association list can
have a type of <span class="fixed">(Eq k) =&gt; k -&gt; AssocList k v -&gt;
Maybe v</span>. <span class="fixed">AssocList</span> is a type
constructor that takes two types and produces a concrete type, like
<span class="fixed">AssocList Int String</span>, for instance.



*Fonzie says:* Aaay! When I talk about *concrete types* I mean like
fully applied types like <span class="fixed">Map Int String</span> or if
we're dealin' with one of them polymorphic functions,
<span class="fixed">\[a\]</span> or <span class="fixed">(Ord a) =&gt;
Maybe a</span> and stuff. And like, sometimes me and the boys say that
<span class="fixed">Maybe</span> is a type, but we don't mean that,
cause every idiot knows <span class="fixed">Maybe</span> is a type
constructor. When I apply an extra type to
<span class="fixed">Maybe</span>, like <span class="fixed">Maybe
String</span>, then I have a concrete type. You know, values can only
have types that are concrete types! So in conclusion, live fast, love
hard and don't let anybody else use your comb!



Just like we can partially apply functions to get new functions, we can
partially apply type parameters and get new type constructors from them.
Just like we call a function with too few parameters to get back a new
function, we can specify a type constructor with too few type parameters
and get back a partially applied type constructor. If we wanted a type
that represents a map (from <span class="fixed">Data.Map</span>) from
integers to something, we could either do this:

``` haskell:hs
type IntMap v = Map Int v
```

Or we could do it like this:

``` haskell:hs
type IntMap = Map Int
```

Either way, the <span class="fixed">IntMap</span> type constructor takes
one parameter and that is the type of what the integers will point to.



*Oh yeah*. If you're going to try and implement this, you'll probably
going to do a qualified import of <span class="fixed">Data.Map</span>.
When you do a qualified import, type constructors also have to be
preceeded with a module name. So you'd write <span class="fixed">type
IntMap = Map.Map Int</span>.



Make sure that you really understand the distinction between type
constructors and value constructors. Just because we made a type synonym
called <span class="fixed">IntMap</span> or
<span class="fixed">AssocList</span> doesn't mean that we can do stuff
like <span class="fixed">AssocList \[(1,2),(4,5),(7,9)\]</span>. All it
means is that we can refer to its type by using different names. We can
do <span class="fixed">\[(1,2),(3,5),(8,9)\] :: AssocList Int
Int</span>, which will make the numbers inside assume a type of
<span class="fixed">Int</span>, but we can still use that list as we
would any normal list that has pairs of integers inside. Type synonyms
(and types generally) can only be used in the type portion of Haskell.
We're in Haskell's type portion whenever we're defining new types (so in
*data* and *type* declarations) or when we're located after a
<span class="fixed">::</span>. The <span class="fixed">::</span> is in
type declarations or in type annotations.

Another cool data type that takes two types as its parameters is the
<span class="fixed">Either a b</span> type. This is roughly how it's
defined:

``` haskell:hs
data Either a b = Left a | Right b deriving (Eq, Ord, Read, Show)
```

It has two value constructors. If the <span class="fixed">Left</span> is
used, then its contents are of type <span class="fixed">a</span> and if
<span class="fixed">Right</span> is used, then its contents are of type
<span class="fixed">b</span>. So we can use this type to encapsulate a
value of one type or another and then when we get a value of type
<span class="fixed">Either a b</span>, we usually pattern match on both
<span class="fixed">Left</span> and <span class="fixed">Right</span> and
we different stuff based on which one of them it was.

``` haskell:hs
ghci> Right 20
Right 20
ghci> Left "w00t"
Left "w00t"
ghci> :t Right 'a'
Right 'a' :: Either a Char
ghci> :t Left True
Left True :: Either Bool b
```

So far, we've seen that <span class="fixed">Maybe a</span> was mostly
used to represent the results of computations that could have either
failed or not. But somtimes, <span class="fixed">Maybe a</span> isn't
good enough because <span class="fixed">Nothing</span> doesn't really
convey much information other than that something has failed. That's
cool for functions that can fail in only one way or if we're just not
interested in how and why they failed. A
<span class="fixed">Data.Map</span> lookup fails only if the key we were
looking for wasn't in the map, so we know exactly what happened.
However, when we're interested in how some function failed or why, we
usually use the result type of <span class="fixed">Either a b</span>,
where <span class="fixed">a</span> is some sort of type that can tell us
something about the possible failure and <span class="fixed">b</span> is
the type of a successful computation. Hence, errors use the
<span class="fixed">Left</span> value constructor while results use
<span class="fixed">Right</span>.

An example: a high-school has lockers so that students have some place
to put their Guns'n'Roses posters. Each locker has a code combination.
When a student wants a new locker, they tell the locker supervisor which
locker number they want and he gives them the code. However, if someone
is already using that locker, he can't tell them the code for the locker
and they have to pick a different one. We'll use a map from
<span class="fixed">Data.Map</span> to represent the lockers. It'll map
from locker numbers to a pair of whether the locker is in use or not and
the locker code.

``` haskell:hs
import qualified Data.Map as Map

data LockerState = Taken | Free deriving (Show, Eq)

type Code = String

type LockerMap = Map.Map Int (LockerState, Code)
```

Simple stuff. We introduce a new data type to represent whether a locker
is taken or free and we make a type synonym for the locker code. We also
make a type synonym for the type that maps from integers to pairs of
locker state and code. And now, we're going to make a function that
searches for the code in a locker map. We're going to use an
<span class="fixed">Either String Code</span> type to represent our
result, because our lookup can fail in two ways — the locker can be
taken, in which case we can't tell the code or the locker number might
not exist at all. If the lookup fails, we're just going to use a
<span class="fixed">String</span> to tell what's happened.

``` haskell:hs
lockerLookup :: Int -> LockerMap -> Either String Code
lockerLookup lockerNumber map = 
    case Map.lookup lockerNumber map of 
        Nothing -> Left $ "Locker number " ++ show lockerNumber ++ " doesn't exist!"
        Just (state, code) -> if state /= Taken 
                                then Right code
                                else Left $ "Locker " ++ show lockerNumber ++ " is already taken!"
```

We do a normal lookup in the map. If we get a
<span class="fixed">Nothing</span>, we return a value of type
<span class="fixed">Left String</span>, saying that the locker doesn't
exist at all. If we do find it, then we do an additional check to see if
the locker is taken. If it is, return a <span class="fixed">Left</span>
saying that it's already taken. If it isn't, then return a value of type
<span class="fixed">Right Code</span>, in which we give the student the
correct code for the locker. It's actually a <span class="fixed">Right
String</span>, but we introduced that type synonym to introduce some
additional documentation into the type declaration. Here's an example
map:

``` haskell:hs
lockers :: LockerMap
lockers = Map.fromList 
    [(100,(Taken,"ZD39I"))
    ,(101,(Free,"JAH3I"))
    ,(103,(Free,"IQSA9"))
    ,(105,(Free,"QOTSA"))
    ,(109,(Taken,"893JJ"))
    ,(110,(Taken,"99292"))
    ]
```

Now let's try looking up some locker codes.

``` haskell:hs
ghci> lockerLookup 101 lockers
Right "JAH3I"
ghci> lockerLookup 100 lockers
Left "Locker 100 is already taken!"
ghci> lockerLookup 102 lockers
Left "Locker number 102 doesn't exist!"
ghci> lockerLookup 110 lockers
Left "Locker 110 is already taken!"
ghci> lockerLookup 105 lockers
Right "QOTSA"
```

We could have used a <span class="fixed">Maybe a</span> to represent the
result but then we wouldn't know why we couldn't get the code. But now,
we have information about the failure in our result type.

<span id="recursive-data-structures"></span>

## Recursive data structures

<img src="/assets/thefonz.png" class="right"
width="168" height="301" alt="the fonz" />

As we've seen, a constructor in an algebraic data type can have several
(or none at all) fields and each field must be of some concrete type.
With that in mind, we can make types whose constructors have fields that
are of the same type! Using that, we can create recursive data types,
where one value of some type contains values of that type, which in turn
contain more values of the same type and so on.

Think about this list: <span class="fixed">\[5\]</span>. That's just
syntactic sugar for <span class="fixed">5:\[\]</span>. On the left side
of the <span class="fixed">:</span>, there's a value and on the right
side, there's a list. And in this case, it's an empty list. Now how
about the list <span class="fixed">\[4,5\]</span>? Well, that desugars
to <span class="fixed">4:(5:\[\])</span>. Looking at the first
<span class="fixed">:</span>, we see that it also has an element on its
left side and a list (<span class="fixed">5:\[\]</span>) on its right
side. Same goes for a list like
<span class="fixed">3:(4:(5:6:\[\]))</span>, which could be written
either like that or like <span class="fixed">3:4:5:6:\[\]</span>
(because <span class="fixed">:</span> is right-associative) or
<span class="fixed">\[3,4,5,6\]</span>.

We could say that a list can be an empty list or it can be an element
joined together with a <span class="fixed">:</span> with another list
(that can be either the empty list or not).

Let's use algebraic data types to implement our own list then!

``` haskell:hs
data List a = Empty | Cons a (List a) deriving (Show, Read, Eq, Ord)
```

This reads just like our definition of lists from one of the previous
paragraphs. It's either an empty list or a combination of a head with
some value and a list. If you're confused about this, you might find it
easier to understand in record syntax.

``` haskell:hs
data List a = Empty | Cons { listHead :: a, listTail :: List a} deriving (Show, Read, Eq, Ord)
```

You might also be confused about the <span class="fixed">Cons</span>
constructor here. *cons* is another word for
<span class="fixed">:</span>. You see, in lists,
<span class="fixed">:</span> is actually a constructor that takes a
value and another list and returns a list. We can already use our new
list type! In other words, it has two fields. One field is of the type
of <span class="fixed">a</span> and the other is of the type
<span class="fixed">\[a\]</span>.

``` haskell:hs
ghci> Empty
Empty
ghci> 5 `Cons` Empty
Cons 5 Empty
ghci> 4 `Cons` (5 `Cons` Empty)
Cons 4 (Cons 5 Empty)
ghci> 3 `Cons` (4 `Cons` (5 `Cons` Empty))
Cons 3 (Cons 4 (Cons 5 Empty))
```

We called our <span class="fixed">Cons</span> constructor in an infix
manner so you can see how it's just like <span class="fixed">:</span>.
<span class="fixed">Empty</span> is like <span class="fixed">\[\]</span>
and <span class="fixed">4 \`Cons\` (5 \`Cons\` Empty)</span> is like
<span class="fixed">4:(5:\[\])</span>.

We can define functions to be automatically infix by making them
comprised of only special characters. We can also do the same with
constructors, since they're just functions that return a data type. So
check this out.

``` haskell:hs
infixr 5 :-:
data List a = Empty | a :-: (List a) deriving (Show, Read, Eq, Ord)
```

First off, we notice a new syntactic construct, the fixity declarations.
When we define functions as operators, we can use that to give them a
fixity (but we don't have to). A fixity states how tightly the operator
binds and whether it's left-associative or right-associative. For
instance, <span class="fixed">\*</span>'s fixity is
<span class="fixed">infixl 7 \*</span> and
<span class="fixed">+</span>'s fixity is <span class="fixed">infixl
6</span>. That means that they're both left-associative
(<span class="fixed">4 \* 3 \* 2</span> is <span class="fixed">(4 \* 3)
\* 2</span>) but <span class="fixed">\*</span> binds tighter than
<span class="fixed">+</span>, because it has a greater fixity, so
<span class="fixed">5 \* 4 + 3</span> is <span class="fixed">(5 \* 4) +
3</span>.

Otherwise, we just wrote <span class="fixed">a :-: (List a)</span>
instead of <span class="fixed">Cons a (List a)</span>. Now, we can write
out lists in our list type like so:

``` haskell:hs
ghci> 3 :-: 4 :-: 5 :-: Empty
(:-:) 3 ((:-:) 4 ((:-:) 5 Empty))
ghci> let a = 3 :-: 4 :-: 5 :-: Empty
ghci> 100 :-: a
(:-:) 100 ((:-:) 3 ((:-:) 4 ((:-:) 5 Empty)))
```

When deriving <span class="fixed">Show</span> for our type, Haskell will
still display it as if the constructor was a prefix function, hence the
parentheses around the operator (remember, <span class="fixed">4 +
3</span> is <span class="fixed">(+) 4 3</span>).

Let's make a function that adds two of our lists together. This is how
<span class="fixed">++</span> is defined for normal lists:

``` haskell:hs
infixr 5  ++
(++) :: [a] -> [a] -> [a]
[]     ++ ys = ys
(x:xs) ++ ys = x : (xs ++ ys)
```

So we'll just steal that for our own list. We'll name the function
<span class="fixed">.++</span>.

``` haskell:hs
infixr 5  .++
(.++) :: List a -> List a -> List a 
Empty .++ ys = ys
(x :-: xs) .++ ys = x :-: (xs .++ ys)
```

And let's see if it works ...

``` haskell:hs
ghci> let a = 3 :-: 4 :-: 5 :-: Empty
ghci> let b = 6 :-: 7 :-: Empty
ghci> a .++ b
(:-:) 3 ((:-:) 4 ((:-:) 5 ((:-:) 6 ((:-:) 7 Empty))))
```

Nice. Is nice. If we wanted, we could implement all of the functions
that operate on lists on our own list type.

Notice how we pattern matched on <span class="fixed">(x :-: xs)</span>.
That works because pattern matching is actually about matching
constructors. We can match on <span class="fixed">:-:</span> because it
is a constructor for our own list type and we can also match on
<span class="fixed">:</span> because it is a constructor for the
built-in list type. Same goes for <span class="fixed">\[\]</span>.
Because pattern matching works (only) on constructors, we can match for
stuff like that, normal prefix constructors or stuff like
<span class="fixed">8</span> or <span class="fixed">'a'</span>, which
are basically constructors for the numeric and character types,
respectively.

<img src="/assets/binarytree.png" class="left"
width="323" height="225" alt="binary search tree" />

Now, we're going to implement a *binary search tree*. If you're not
familiar with binary search trees from languages like C, here's what
they are: an element points to two elements, one on its left and one on
its right. The element to the left is smaller, the element to the right
is bigger. Each of those elements can also point to two elements (or
one, or none). In effect, each element has up to two sub-trees. And a
cool thing about binary search trees is that we know that all the
elements at the left sub-tree of, say, 5 are going to be smaller than 5.
Elements in its right sub-tree are going to be bigger. So if we need to
find if 8 is in our tree, we'd start at 5 and then because 8 is greater
than 5, we'd go right. We're now at 7 and because 8 is greater than 7,
we go right again. And we've found our element in three hops! Now if
this were a normal list (or a tree, but really unbalanced), it would
take us seven hops instead of three to see if 8 is in there.

Sets and maps from <span class="fixed">Data.Set</span> and
<span class="fixed">Data.Map</span> are implemented using trees, only
instead of normal binary search trees, they use balanced binary search
trees, which are always balanced. But right now, we'll just be
implementing normal binary search trees.

Here's what we're going to say: a tree is either an empty tree or it's
an element that contains some value and two trees. Sounds like a perfect
fit for an algebraic data type!

``` haskell:hs
data Tree a = EmptyTree | Node a (Tree a) (Tree a) deriving (Show, Read, Eq)
```

Okay, good, this is good. Instead of manually building a tree, we're
going to make a function that takes a tree and an element and inserts an
element. We do this by comparing the value we want to insert to the root
node and then if it's smaller, we go left, if it's larger, we go right.
We do the same for every subsequent node until we reach an empty tree.
Once we've reached an empty tree, we just insert a node with that value
instead of the empty tree.

In languages like C, we'd do this by modifying the pointers and values
inside the tree. In Haskell, we can't really modify our tree, so we have
to make a new sub-tree each time we decide to go left or right and in
the end the insertion function returns a completely new tree, because
Haskell doesn't really have a concept of pointer, just values. Hence,
the type for our insertion function is going to be something like
<span class="fixed">a -&gt; Tree a - &gt; Tree a</span>. It takes an element
and a tree and returns a new tree that has that element inside. This
might seem like it's inefficient but laziness takes care of that
problem.

So, here are two functions. One is a utility function for making a
singleton tree (a tree with just one node) and a function to insert an
element into a tree.

``` haskell:hs
singleton :: a -> Tree a
singleton x = Node x EmptyTree EmptyTree

treeInsert :: (Ord a) => a -> Tree a -> Tree a
treeInsert x EmptyTree = singleton x
treeInsert x (Node a left right) 
    | x == a = Node x left right
    | x < a  = Node a (treeInsert x left) right
    | x > a  = Node a left (treeInsert x right)
```

The <span class="fixed">singleton</span> function is just a shortcut for
making a node that has something and then two empty sub-trees. In the
insertion function, we first have the edge condition as a pattern. If
we've reached an empty sub-tree, that means we're where we want and
instead of the empty tree, we put a singleton tree with our element. If
we're not inserting into an empty tree, then we have to check some
things. First off, if the element we're inserting is equal to the root
element, just return a tree that's the same. If it's smaller, return a
tree that has the same root value, the same right sub-tree but instead
of its left sub-tree, put a tree that has our value inserted into it.
Same (but the other way around) goes if our value is bigger than the
root element.

Next up, we're going to make a function that checks if some element is
in the tree. First, let's define the edge condition. If we're looking
for an element in an empty tree, then it's certainly not there. Okay.
Notice how this is the same as the edge condition when searching for
elements in lists. If we're looking for an element in an empty list,
it's not there. Anyway, if we're not looking for an element in an empty
tree, then we check some things. If the element in the root node is what
we're looking for, great! If it's not, what then? Well, we can take
advantage of knowing that all the left elements are smaller than the
root node. So if the element we're looking for is smaller than the root
node, check to see if it's in the left sub-tree. If it's bigger, check
to see if it's in the right sub-tree.

``` haskell:hs
treeElem :: (Ord a) => a -> Tree a -> Bool
treeElem x EmptyTree = False
treeElem x (Node a left right)
    | x == a = True
    | x < a  = treeElem x left
    | x > a  = treeElem x right
```

All we had to do was write up the previous paragraph in code. Let's have
some fun with our trees! Instead of manually building one (although we
could), we'll use a fold to build up a tree from a list. Remember,
pretty much everything that traverses a list one by one and then returns
some sort of value can be implemented with a fold! We're going to start
with the empty tree and then approach a list from the right and just
insert element after element into our accumulator tree.

``` haskell:hs
ghci> let nums = [8,6,4,1,7,3,5]
ghci> let numsTree = foldr treeInsert EmptyTree nums
ghci> numsTree
Node 5 (Node 3 (Node 1 EmptyTree EmptyTree) (Node 4 EmptyTree EmptyTree)) (Node 7 (Node 6 EmptyTree EmptyTree) (Node 8 EmptyTree EmptyTree))
```

In that <span class="fixed">foldr</span>,
<span class="fixed">treeInsert</span> was the folding function (it takes
a tree and a list element and produces a new tree) and
<span class="fixed">EmptyTree</span> was the starting accumulator.
<span class="fixed">nums</span>, of course, was the list we were folding
over.

When we print our tree to the console, it's not very readable, but if we
try, we can make out its structure. We see that the root node is 5 and
then it has two sub-trees, one of which has the root node of 3 and the
other a 7, etc.

``` haskell:hs
ghci> 8 `treeElem` numsTree
True
ghci> 100 `treeElem` numsTree
False
ghci> 1 `treeElem` numsTree
True
ghci> 10 `treeElem` numsTree
False
```

Checking for membership also works nicely. Cool.

So as you can see, algebraic data structures are a really cool and
powerful concept in Haskell. We can use them to make anything from
boolean values and weekday enumerations to binary search trees and more!

<span id="typeclasses-102"></span>

## Typeclasses 102

<img src="/assets/trafficlight.png" class="right"
width="175" height="480" alt="tweet" />

So far, we've learned about some of the standard Haskell typeclasses and
we've seen which types are in them. We've also learned how to
automatically make our own types instances of the standard typeclasses
by asking Haskell to derive the instances for us. In this section, we're
going to learn how to make our own typeclasses and how to make types
instances of them by hand.

A quick recap on typeclasses: typeclasses are like interfaces. A
typeclass defines some behavior (like comparing for equality, comparing
for ordering, enumeration) and then types that can behave in that way
are made instances of that typeclass. The behavior of typeclasses is
achieved by defining functions or just type declarations that we then
implement. So when we say that a type is an instance of a typeclass, we
mean that we can use the functions that the typeclass defines with that
type.

Typeclasses have pretty much nothing to do with classes in languages
like Java or Python. This confuses many people, so I want you to forget
everything you know about classes in imperative languages right now.

For example, the <span class="fixed">Eq</span> typeclass is for stuff
that can be equated. It defines the functions
<span class="fixed">==</span> and <span class="fixed">/=</span>. If we
have a type (say, <span class="fixed">Car</span>) and comparing two cars
with the equality function <span class="fixed">==</span> makes sense,
then it makes sense for <span class="fixed">Car</span> to be an instance
of <span class="fixed">Eq</span>.

This is how the <span class="fixed">Eq</span> class is defined in the
standard prelude:

``` haskell:hs
class Eq a where
    (==) :: a -> a -> Bool
    (/=) :: a -> a -> Bool
    x == y = not (x /= y)
    x /= y = not (x == y)
```

Woah, woah, woah! Some new strange syntax and keywords there! Don't
worry, this will all be clear in a second. First off, when we write
<span class="fixed">class Eq a where</span>, this means that we're
defining a new typeclass and that's called
<span class="fixed">Eq</span>. The <span class="fixed">a</span> is the
type variable and it means that <span class="fixed">a</span> will play
the role of the type that we will soon be making an instance of
<span class="fixed">Eq</span>. It doesn't have to be called
<span class="fixed">a</span>, it doesn't even have to be one letter, it
just has to be a lowercase word. Then, we define several functions. It's
not mandatory to implement the function bodies themselves, we just have
to specify the type declarations for the functions.



Some people might understand this better if we wrote
<span class="fixed">class Eq equatable where</span> and then specified
the type declarations like <span class="fixed">(==) :: equatable -&gt;
equatable -&gt; Bool</span>.



Anyway, we *did* implement the function bodies for the functions that
<span class="fixed">Eq</span> defines, only we defined them in terms of
mutual recursion. We said that two instances of
<span class="fixed">Eq</span> are equal if they are not different and
they are different if they are not equal. We didn't have to do this,
really, but we did and we'll see how this helps us soon.



If we have say <span class="fixed">class Eq a where</span> and then
define a type declaration within that class like
<span class="fixed">(==) :: a -&gt; -a -&gt; Bool</span>, then when we
examine the type of that function later on, it will have the type of
<span class="fixed">(Eq a) =&gt; a -&gt; a -&gt; Bool</span>.



So once we have a class, what can we do with it? Well, not much, really.
But once we start making types instances of that class, we start getting
some nice functionality. So check out this type:

``` haskell:hs
data TrafficLight = Red | Yellow | Green
```

It defines the states of a traffic light. Notice how we didn't derive
any class instances for it. That's because we're going to write up some
instances by hand, even though we could derive them for types like
<span class="fixed">Eq</span> and <span class="fixed">Show</span>.
Here's how we make it an instance of <span class="fixed">Eq</span>.

``` haskell:hs
instance Eq TrafficLight where
    Red == Red = True
    Green == Green = True
    Yellow == Yellow = True
    _ == _ = False
```

We did it by using the *instance* keyword. So *class* is for defining
new typeclasses and *instance* is for making our types instances of
typeclasses. When we were defining <span class="fixed">Eq</span>, we
wrote <span class="fixed">class Eq a where</span> and we said that
<span class="fixed">a</span> plays the role of whichever type will be
made an instance later on. We can see that clearly here, because when
we're making an instance, we write <span class="fixed">instance Eq
TrafficLight where</span>. We replace the <span class="fixed">a</span>
with the actual type.

Because <span class="fixed">==</span> was defined in terms of
<span class="fixed">/=</span> and vice versa in the *class* declaration,
we only had to overwrite one of them in the instance declaration. That's
called the minimal complete definition for the typeclass — the minimum
of functions that we have to implement so that our type can behave like
the class advertises. To fulfill the minimal complete definition for
<span class="fixed">Eq</span>, we have to overwrite either one of
<span class="fixed">==</span> or <span class="fixed">/=</span>. If
<span class="fixed">Eq</span> was defined simply like this:

``` haskell:hs
class Eq a where
    (==) :: a -> a -> Bool
    (/=) :: a -> a -> Bool
```

we'd have to implement both of these functions when making a type an
instance of it, because Haskell wouldn't know how these two functions
are related. The minimal complete definition would then be: both
<span class="fixed">==</span> and <span class="fixed">/=</span>.

You can see that we implemented <span class="fixed">==</span> simply by
doing pattern matching. Since there are many more cases where two lights
aren't equal, we specified the ones that are equal and then just did a
catch-all pattern saying that if it's none of the previous combinations,
then two lights aren't equal.

Let's make this an instance of <span class="fixed">Show</span> by hand,
too. To satisfy the minimal complete definition for
<span class="fixed">Show</span>, we just have to implement its
<span class="fixed">show</span> function, which takes a value and turns
it into a string.

``` haskell:hs
instance Show TrafficLight where
    show Red = "Red light"
    show Yellow = "Yellow light"
    show Green = "Green light"
```

Once again, we used pattern matching to achieve our goals. Let's see how
it works in action:

``` haskell:hs
ghci> Red == Red
True
ghci> Red == Yellow
False
ghci> Red `elem` [Red, Yellow, Green]
True
ghci> [Red, Yellow, Green]
[Red light,Yellow light,Green light]
```

Nice. We could have just derived <span class="fixed">Eq</span> and it
would have had the same effect (but we didn't for educational purposes).
However, deriving <span class="fixed">Show</span> would have just
directly translated the value constructors to strings. But if we want
lights to appear like <span class="fixed">"Red light"</span>, then we
have to make the instance declaration by hand.

You can also make typeclasses that are subclasses of other typeclasses.
The *class* declaration for <span class="fixed">Num</span> is a bit
long, but here's the first part:

``` haskell:hs
class (Eq a) => Num a where
   ...  
```

As we mentioned previously, there are a lot of places where we can cram
in class constraints. So this is just like writing
<span class="fixed">class Num a where</span>, only we state that our
type <span class="fixed">a</span> must be an instance of
<span class="fixed">Eq</span>. We're essentially saying that we have to
make a type an instance of <span class="fixed">Eq</span> before we can
make it an instance of <span class="fixed">Num</span>. Before some type
can be considered a number, it makes sense that we can determine whether
values of that type can be equated or not. That's all there is to
subclassing really, it's just a class constraint on a *class*
declaration! When defining function bodies in the *class* declaration or
when defining them in *instance* declarations, we can assume that
<span class="fixed">a</span> is a part of <span class="fixed">Eq</span>
and so we can use <span class="fixed">==</span> on values of that type.

But how are the <span class="fixed">Maybe</span> or list types made as
instances of typeclasses? What makes <span class="fixed">Maybe</span>
different from, say, <span class="fixed">TrafficLight</span> is that
<span class="fixed">Maybe</span> in itself isn't a concrete type, it's a
type constructor that takes one type parameter (like
<span class="fixed">Char</span> or something) to produce a concrete type
(like <span class="fixed">Maybe Char</span>). Let's take a look at the
<span class="fixed">Eq</span> typeclass again:

``` haskell:hs
class Eq a where
    (==) :: a -> a -> Bool
    (/=) :: a -> a -> Bool
    x == y = not (x /= y)
    x /= y = not (x == y) 
```

From the type declarations, we see that the <span class="fixed">a</span>
is used as a concrete type because all the types in functions have to be
concrete (remember, you can't have a function of the type
<span class="fixed">a -&gt; Maybe</span> but you can have a function of
<span class="fixed">a -&gt; Maybe a</span> or <span class="fixed">Maybe
Int -&gt; Maybe String</span>). That's why we can't do something like

``` haskell:hs
instance Eq Maybe where
    ...  
```

Because like we've seen, the <span class="fixed">a</span> has to be a
concrete type but <span class="fixed">Maybe</span> isn't a concrete
type. It's a type constructor that takes one parameter and then produces
a concrete type. It would also be tedious to write
<span class="fixed">instance Eq (Maybe Int) where</span>,
<span class="fixed">instance Eq (Maybe Char) where</span>, etc. for
every type ever. So we could write it out like so:

``` haskell:hs
instance Eq (Maybe m) where
    Just x == Just y = x == y
    Nothing == Nothing = True
    _ == _ = False
      
```

This is like saying that we want to make all types of the form
<span class="fixed">Maybe something</span> an instance of
<span class="fixed">Eq</span>. We actually could have written
<span class="fixed">(Maybe something)</span>, but we usually opt for
single letters to be true to the Haskell style. The
<span class="fixed">(Maybe m)</span> here plays the role of the
<span class="fixed">a</span> from <span class="fixed">class Eq a
where</span>. While <span class="fixed">Maybe</span> isn't a concrete
type, <span class="fixed">Maybe m</span> is. By specifying a type
parameter (<span class="fixed">m</span>, which is in lowercase), we said
that we want all types that are in the form of <span class="fixed">Maybe
m</span>, where <span class="fixed">m</span> is any type, to be an
instance of <span class="fixed">Eq</span>.

There's one problem with this though. Can you spot it? We use
<span class="fixed">==</span> on the contents of the
<span class="fixed">Maybe</span> but we have no assurance that what the
<span class="fixed">Maybe</span> contains can be used with
<span class="fixed">Eq</span>! That's why we have to modify our
*instance* declaration like this:

``` haskell:hs
instance (Eq m) => Eq (Maybe m) where
    Just x == Just y = x == y
    Nothing == Nothing = True
    _ == _ = False
      
```

We had to add a class constraint! With this *instance* declaration, we
say this: we want all types of the form <span class="fixed">Maybe
m</span> to be part of the <span class="fixed">Eq</span> typeclass, but
only those types where the <span class="fixed">m</span> (so what's
contained inside the <span class="fixed">Maybe</span>) is also a part of
<span class="fixed">Eq</span>. This is actually how Haskell would derive
the instance too.

Most of the times, class constraints in *class* declarations are used
for making a typeclass a subclass of another typeclass and class
constraints in *instance* declarations are used to express requirements
about the contents of some type. For instance, here we required the
contents of the <span class="fixed">Maybe</span> to also be part of the
<span class="fixed">Eq</span> typeclass.

When making instances, if you see that a type is used as a concrete type
in the type declarations (like the <span class="fixed">a</span> in
<span class="fixed">a -&gt; a -&gt; Bool</span>), you have to supply type
parameters and add parentheses so that you end up with a concrete type.



Take into account that the type you're trying to make an instance of
will replace the parameter in the *class* declaration. The
<span class="fixed">a</span> from <span class="fixed">class Eq a
where</span> will be replaced with a real type when you make an
instance, so try mentally putting your type into the function type
declarations as well. <span class="fixed">(==) :: Maybe -&gt; Maybe -&gt;
Bool</span> doesn't make much sense but <span class="fixed">(==) :: (Eq
m) =&gt; Maybe m -&gt; Maybe m -&gt; Bool</span> does. But this is just
something to think about, because <span class="fixed">==</span> will
always have a type of <span class="fixed">(==) :: (Eq a) =&gt; a -&gt; a -&gt;
Bool</span>, no matter what instances we make.



Ooh, one more thing, check this out! If you want to see what the
instances of a typeclass are, just do <span class="fixed">:info
YourTypeClass</span> in GHCI. So typing <span class="fixed">:info
Num</span> will show which functions the typeclass defines and it will
give you a list of the types in the typeclass.
<span class="fixed">:info</span> works for types and type constructors
too. If you do <span class="fixed">:info Maybe</span>, it will show you
all the typeclasses that <span class="fixed">Maybe</span> is an instance
of. Also <span class="fixed">:info</span> can show you the type
declaration of a function. I think that's pretty cool.

<span id="a-yes-no-typeclass"></span>

## A yes-no typeclass

<img src="/assets/yesno.png" class="left"
width="201" height="111" alt="yesno" />

In JavaScript and some other weakly typed languages, you can put almost
anything inside an if expression. For example, you can do all of the
following: <span class="fixed">if (0) alert("YEAH!") else
alert("NO!")</span>, <span class="fixed">if ("") alert ("YEAH!") else
alert("NO!")</span>, <span class="fixed">if (false) alert("YEAH") else
alert("NO!)</span>, etc. and all of these will throw an alert of
<span class="fixed">NO!</span>. If you do <span class="fixed">if
("WHAT") alert ("YEAH") else alert("NO!")</span>, it will alert a
<span class="fixed">"YEAH!"</span> because JavaScript considers
non-empty strings to be a sort of true-ish value.

Even though strictly using <span class="fixed">Bool</span> for boolean
semantics works better in Haskell, let's try and implement that
JavaScript-ish behavior anyway. For fun! Let's start out with a *class*
declaration.

``` haskell:hs
class YesNo a where
    yesno :: a -> Bool
```

Pretty simple. The <span class="fixed">YesNo</span> typeclass defines
one function. That function takes one value of a type that can be
considered to hold some concept of true-ness and tells us for sure if
it's true or not. Notice that from the way we use the
<span class="fixed">a</span> in the function,
<span class="fixed">a</span> has to be a concrete type.

Next up, let's define some instances. For numbers, we'll assume that
(like in JavaScript) any number that isn't 0 is true-ish and 0 is
false-ish.

``` haskell:hs
instance YesNo Int where
    yesno 0 = False
    yesno _ = True
```

Empty lists (and by extensions, strings) are a no-ish value, while
non-empty lists are a yes-ish value.

``` haskell:hs
instance YesNo [a] where
    yesno [] = False
    yesno _ = True
```

Notice how we just put in a type parameter <span class="fixed">a</span>
in there to make the list a concrete type, even though we don't make any
assumptions about the type that's contained in the list. What else, hmm
... I know, <span class="fixed">Bool</span> itself also holds true-ness
and false-ness and it's pretty obvious which is which.

``` haskell:hs
instance YesNo Bool where
    yesno = id   
```

Huh? What's <span class="fixed">id</span>? It's just a standard library
function that takes a parameter and returns the same thing, which is
what we would be writing here anyway.

Let's make <span class="fixed">Maybe a</span> an instance too.

``` haskell:hs
instance YesNo (Maybe a) where
    yesno (Just _) = True
    yesno Nothing = False
```

We didn't need a class constraint because we made no assumptions about
the contents of the <span class="fixed">Maybe</span>. We just said that
it's true-ish if it's a <span class="fixed">Just</span> value and
false-ish if it's a <span class="fixed">Nothing</span>. We still had to
write out <span class="fixed">(Maybe a)</span> instead of just
<span class="fixed">Maybe</span> because if you think about it, a
<span class="fixed">Maybe -&gt; Bool</span> function can't exist (because
<span class="fixed">Maybe</span> isn't a concrete type), whereas a
<span class="fixed">Maybe a -&gt; Bool</span> is fine and dandy. Still,
this is really cool because now, any type of the form
<span class="fixed">Maybe something</span> is part of
<span class="fixed">YesNo</span> and it doesn't matter what that
<span class="fixed">something</span> is.

Previously, we defined a <span class="fixed">Tree a</span> type, that
represented a binary search tree. We can say an empty tree is false-ish
and anything that's not an empty tree is true-ish.

``` haskell:hs
instance YesNo (Tree a) where
    yesno EmptyTree = False
    yesno _ = True
```

Can a traffic light be a yes or no value? Sure. If it's red, you stop.
If it's green, you go. If it's yellow? Eh, I usually run the yellows
because I live for adrenaline.

``` haskell:hs
instance YesNo TrafficLight where
    yesno Red = False
    yesno _ = True
```

Cool, now that we have some instances, let's go play!

``` haskell:hs
ghci> yesno $ length []
False
ghci> yesno "haha"
True
ghci> yesno ""
False
ghci> yesno $ Just 0
True
ghci> yesno True
True
ghci> yesno EmptyTree
False
ghci> yesno []
False
ghci> yesno [0,0,0]
True
ghci> :t yesno
yesno :: (YesNo a) => a -> Bool
```

Right, it works! Let's make a function that mimics the if statement, but
it works with <span class="fixed">YesNo</span> values.

``` haskell:hs
yesnoIf :: (YesNo y) => y -> a -> a -> a
yesnoIf yesnoVal yesResult noResult = if yesno yesnoVal then yesResult else noResult
```

Pretty straightforward. It takes a yes-no-ish value and two things. If
the yes-no-ish value is more of a yes, it returns the first of the two
things, otherwise it returns the second of them.

``` haskell:hs
ghci> yesnoIf [] "YEAH!" "NO!"
"NO!"
ghci> yesnoIf [2,3,4] "YEAH!" "NO!"
"YEAH!"
ghci> yesnoIf True "YEAH!" "NO!"
"YEAH!"
ghci> yesnoIf (Just 500) "YEAH!" "NO!"
"YEAH!"
ghci> yesnoIf Nothing "YEAH!" "NO!"
"NO!"
```

<span id="the-functor-typeclass"></span>

## The Functor typeclass

So far, we've encountered a lot of the typeclasses in the standard
library. We've played with <span class="fixed">Ord</span>, which is for
stuff that can be ordered. We've palled around with
<span class="fixed">Eq</span>, which is for things that can be equated.
We've seen <span class="fixed">Show</span>, which presents an interface
for types whose values can be displayed as strings. Our good friend
<span class="fixed">Read</span> is there whenever we need to convert a
string to a value of some type. And now, we're going to take a look at
the <span class="label class">Functor</span> typeclass, which is
basically for things that can be mapped over. You're probably thinking
about lists now, since mapping over lists is such a dominant idiom in
Haskell. And you're right, the list type is part of the
<span class="fixed">Functor</span> typeclass.

What better way to get to know the <span class="fixed">Functor</span>
typeclass than to see how it's implemented? Let's take a peek.

``` haskell:hs
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

<img src="/assets/functor.png" class="right"
width="220" height="441" alt="I AM FUNCTOOOOR!!!" />

Alright. We see that it defines one function,
<span class="fixed">fmap</span>, and doesn't provide any default
implementation for it. The type of <span class="fixed">fmap</span> is
interesting. In the definitions of typeclasses so far, the type variable
that played the role of the type in the typeclass was a concrete type,
like the <span class="fixed">a</span> in <span class="fixed">(==) :: (Eq
a) =&gt; a -&gt; a -&gt; Bool</span>. But now, the
<span class="fixed">f</span> is not a concrete type (a type that a value
can hold, like <span class="fixed">Int</span>,
<span class="fixed">Bool</span> or <span class="fixed">Maybe
String</span>), but a type constructor that takes one type parameter. A
quick refresher example: <span class="fixed">Maybe Int</span> is a
concrete type, but <span class="fixed">Maybe</span> is a type
constructor that takes one type as the parameter. Anyway, we see that
<span class="fixed">fmap</span> takes a function from one type to
another and a functor applied with one type and returns a functor
applied with another type.

If this sounds a bit confusing, don't worry. All will be revealed soon
when we check out a few examples. Hmm, this type declaration for
<span class="fixed">fmap</span> reminds me of something. If you don't
know what the type signature of <span class="fixed">map</span> is, it's:
<span class="fixed">map :: (a -&gt; b) -&gt; \[a\] -&gt; \[b\]</span>.

Ah, interesting! It takes a function from one type to another and a list
of one type and returns a list of another type. My friends, I think we
have ourselves a functor! In fact, <span class="fixed">map</span> is
just a <span class="fixed">fmap</span> that works only on lists. Here's
how the list is an instance of the <span class="fixed">Functor</span>
typeclass.

``` haskell:hs
instance Functor [] where
    fmap = map
```

That's it! Notice how we didn't write <span class="fixed">instance
Functor \[a\] where</span>, because from <span class="fixed">fmap :: (a
-&gt; b) -&gt; f a -&gt; f b</span>, we see that the
<span class="fixed">f</span> has to be a type constructor that takes one
type. <span class="fixed">\[a\]</span> is already a concrete type (of a
list with any type inside it), while <span class="fixed">\[\]</span> is
a type constructor that takes one type and can produce types such as
<span class="fixed">\[Int\]</span>,
<span class="fixed">\[String\]</span> or even
<span class="fixed">\[\[String\]\]</span>.

Since for lists, <span class="fixed">fmap</span> is just
<span class="fixed">map</span>, we get the same results when using them
on lists.

``` haskell:hs
map :: (a -> b) -> [a] -> [b]
ghci> fmap (*2) [1..3]
[2,4,6]
ghci> map (*2) [1..3]
[2,4,6]
```

What happens when we <span class="fixed">map</span> or
<span class="fixed">fmap</span> over an empty list? Well, of course, we
get an empty list. It just turns an empty list of type
<span class="fixed">\[a\]</span> into an empty list of type
<span class="fixed">\[b\]</span>.

Types that can act like a box can be functors. You can think of a list
as a box that has an infinite amount of little compartments and they can
all be empty, one can be full and the others empty or a number of them
can be full. So, what else has the properties of being like a box? For
one, the <span class="fixed">Maybe a</span> type. In a way, it's like a
box that can either hold nothing, in which case it has the value of
<span class="fixed">Nothing</span>, or it can hold one item, like
<span class="fixed">"HAHA"</span>, in which case it has a value of
<span class="fixed">Just "HAHA"</span>. Here's how
<span class="fixed">Maybe</span> is a functor.

``` haskell:hs
instance Functor Maybe where
    fmap f (Just x) = Just (f x)
    fmap f Nothing = Nothing
```

Again, notice how we wrote <span class="fixed">instance Functor Maybe
where</span> instead of <span class="fixed">instance Functor (Maybe m)
where</span>, like we did when we were dealing with
<span class="fixed">Maybe</span> and <span class="fixed">YesNo</span>.
<span class="fixed">Functor</span> wants a type constructor that takes
one type and not a concrete type. If you mentally replace the
<span class="fixed">f</span>s with <span class="fixed">Maybe</span>s,
<span class="fixed">fmap</span> acts like a <span class="fixed">(a -&gt;
b) -&gt; Maybe a -&gt; Maybe b</span> for this particular type, which looks
OK. But if you replace <span class="fixed">f</span> with
<span class="fixed">(Maybe m)</span>, then it would seem to act like a
<span class="fixed">(a -&gt; b) -&gt; Maybe m a -&gt; Maybe m b</span>, which
doesn't make any damn sense because <span class="fixed">Maybe</span>
takes just one type parameter.

Anyway, the <span class="fixed">fmap</span> implementation is pretty
simple. If it's an empty value of <span class="fixed">Nothing</span>,
then just return a <span class="fixed">Nothing</span>. If we map over an
empty box, we get an empty box. It makes sense. Just like if we map over
an empty list, we get back an empty list. If it's not an empty value,
but rather a single value packed up in a
<span class="fixed">Just</span>, then we apply the function on the
contents of the <span class="fixed">Just</span>.

``` haskell:hs
ghci> fmap (++ " HEY GUYS IM INSIDE THE JUST") (Just "Something serious.")
Just "Something serious. HEY GUYS IM INSIDE THE JUST"
ghci> fmap (++ " HEY GUYS IM INSIDE THE JUST") Nothing
Nothing
ghci> fmap (*2) (Just 200)
Just 400
ghci> fmap (*2) Nothing
Nothing
```

Another thing that can be mapped over and made an instance of
<span class="fixed">Functor</span> is our <span class="fixed">Tree
a</span> type. It can be thought of as a box in a way (holds several or
no values) and the <span class="fixed">Tree</span> type constructor
takes exactly one type parameter. If you look at
<span class="fixed">fmap</span> as if it were a function made only for
<span class="fixed">Tree</span>, its type signature would look like
<span class="fixed">(a -&gt; b) -&gt; Tree a -&gt; Tree b</span>. We're going
to use recursion on this one. Mapping over an empty tree will produce an
empty tree. Mapping over a non-empty tree will be a tree consisting of
our function applied to the root value and its left and right sub-trees
will be the previous sub-trees, only our function will be mapped over
them.

``` haskell:hs
instance Functor Tree where
    fmap f EmptyTree = EmptyTree
    fmap f (Node x leftsub rightsub) = Node (f x) (fmap f leftsub) (fmap f rightsub)
```

``` haskell:hs
ghci> fmap (*2) EmptyTree
EmptyTree
ghci> fmap (*4) (foldr treeInsert EmptyTree [5,7,3,2,1,7])
Node 28 (Node 4 EmptyTree (Node 8 EmptyTree (Node 12 EmptyTree (Node 20 EmptyTree EmptyTree)))) EmptyTree
```

Nice! Now how about <span class="fixed">Either a b</span>? Can this be
made a functor? The <span class="fixed">Functor</span> typeclass wants a
type constructor that takes only one type parameter but
<span class="fixed">Either</span> takes two. Hmmm! I know, we'll
partially apply <span class="fixed">Either</span> by feeding it only one
parameter so that it has one free parameter. Here's how
<span class="fixed">Either a</span> is a functor in the standard
libraries:

``` haskell:hs
instance Functor (Either a) where
    fmap f (Right x) = Right (f x)
    fmap f (Left x) = Left x
```

Well well, what did we do here? You can see how we made
<span class="fixed">Either a</span> an instance instead of just
<span class="fixed">Either</span>. That's because
<span class="fixed">Either a</span> is a type constructor that takes one
parameter, whereas <span class="fixed">Either</span> takes two. If
<span class="fixed">fmap</span> was specifically for
<span class="fixed">Either a</span>, the type signature would then be
<span class="fixed">(b -&gt; c) -&gt; Either a b -&gt; Either a c</span>
because that's the same as <span class="fixed">(b -&gt; c) -&gt; (Either a)
b -&gt; (Either a) c</span>. In the implementation, we mapped in the case
of a <span class="fixed">Right</span> value constructor, but we didn't
in the case of a <span class="fixed">Left</span>. Why is that? Well, if
we look back at how the <span class="fixed">Either a b</span> type is
defined, it's kind of like:

``` haskell:hs
data Either a b = Left a | Right b
```

Well, if we wanted to map one function over both of them,
<span class="fixed">a</span> and <span class="fixed">b</span> would have
to be the same type. I mean, if we tried to map a function that takes a
string and returns a string and the <span class="fixed">b</span> was a
string but the <span class="fixed">a</span> was a number, that wouldn't
really work out. Also, from seeing what
<span class="fixed">fmap</span>'s type would be if it operated only on
<span class="fixed">Either</span> values, we see that the first
parameter has to remain the same while the second one can change and the
first parameter is actualized by the <span class="fixed">Left</span>
value constructor.

This also goes nicely with our box analogy if we think of the
<span class="fixed">Left</span> part as sort of an empty box with an
error message written on the side telling us why it's empty.

Maps from <span class="fixed">Data.Map</span> can also be made a functor
because they hold values (or not!). In the case of
<span class="fixed">Map k v</span>, <span class="fixed">fmap</span> will
map a function <span class="fixed">v -&gt; v'</span> over a map of type
<span class="fixed">Map k v</span> and return a map of type
<span class="fixed">Map k v'</span>.



Note, the <span class="fixed">'</span> has no special meaning in types
just like it doesn't have special meaning when naming values. It's used
to denote things that are similar, only slightly changed.



Try figuring out how <span class="fixed">Map k</span> is made an
instance of <span class="fixed">Functor</span> by yourself!

With the <span class="fixed">Functor</span> typeclass, we've seen how
typeclasses can represent pretty cool higher-order concepts. We've also
had some more practice with partially applying types and making
instances. In one of the next chapters, we'll also take a look at some
laws that apply for functors.



*Just one more thing!* Functors should obey some laws so that they may
have some properties that we can depend on and not think about too much.
If we use <span class="fixed">fmap (+1)</span> over the list
<span class="fixed">\[1,2,3,4\]</span>, we expect the result to be
<span class="fixed">\[2,3,4,5\]</span> and not its reverse,
<span class="fixed">\[5,4,3,2\]</span>. If we use
<span class="fixed">fmap (\a -&gt; a)</span> (the identity function, which
just returns its parameter) over some list, we expect to get back the
same list as a result. For example, if we gave the wrong functor
instance to our <span class="fixed">Tree</span> type, using
<span class="fixed">fmap</span> over a tree where the left sub-tree of a
node only has elements that are smaller than the node and the right
sub-tree only has nodes that are larger than the node might produce a
tree where that's not the case. We'll go over the functor laws in more
detail in one of the next chapters.



<span id="kinds-and-some-type-foo"></span>

## Kinds and some type-foo

<img src="/assets/typefoo.png" class="right"
width="287" height="400" alt="TYPE FOO MASTER" />

Type constructors take other types as parameters to eventually produce
concrete types. That kind of reminds me of functions, which take values
as parameters to produce values. We've seen that type constructors can
be partially applied (<span class="fixed">Either String</span> is a type
that takes one type and produces a concrete type, like
<span class="fixed">Either String Int</span>), just like functions can.
This is all very interesting indeed. In this section, we'll take a look
at formally defining how types are applied to type constructors, just
like we took a look at formally defining how values are applied to
functions by using type declarations. *You don't really have to read
this section to continue on your magical Haskell quest* and if you don't
understand it, don't worry about it. However, getting this will give you
a very thorough understanding of the type system.

So, values like <span class="fixed">3</span>,
<span class="fixed">"YEAH"</span> or
<span class="fixed">takeWhile</span> (functions are also values, because
we can pass them around and such) each have their own type. Types are
little labels that values carry so that we can reason about the values.
But types have their own little labels, called *kinds*. A kind is more
or less the type of a type. This may sound a bit weird and confusing,
but it's actually a really cool concept.

What are kinds and what are they good for? Well, let's examine the kind
of a type by using the <span class="fixed">:k</span> command in GHCI.

``` haskell:hs
ghci> :k Int
Int :: *
```

A star? How quaint. What does that mean? A <span class="fixed">\*</span>
means that the type is a concrete type. A concrete type is a type that
doesn't take any type parameters and values can only have types that are
concrete types. If I had to read <span class="fixed">\*</span> out loud
(I haven't had to do that so far), I'd say *star* or just *type*.

Okay, now let's see what the kind of <span class="fixed">Maybe</span>
is.

``` haskell:hs
ghci> :k Maybe
Maybe :: * -> *
```

The <span class="fixed">Maybe</span> type constructor takes one concrete
type (like <span class="fixed">Int</span>) and then returns a concrete
type like <span class="fixed">Maybe Int</span>. And that's what this
kind tells us. Just like <span class="fixed">Int -&gt; Int</span> means
that a function takes an <span class="fixed">Int</span> and returns an
<span class="fixed">Int</span>, <span class="fixed">\* -&gt; \*</span>
means that the type constructor takes one concrete type and returns a
concrete type. Let's apply the type parameter to
<span class="fixed">Maybe</span> and see what the kind of that type is.

``` haskell:hs
ghci> :k Maybe Int
Maybe Int :: *
```

Just like I expected! We applied the type parameter to
<span class="fixed">Maybe</span> and got back a concrete type (that's
what <span class="fixed">\* -&gt; \*</span> means. A parallel (although
not equivalent, types and kinds are two different things) to this is if
we do <span class="fixed">:t isUpper</span> and <span class="fixed">:t
isUpper 'A'</span>. <span class="fixed">isUpper</span> has a type of
<span class="fixed">Char -&gt; Bool</span> and <span class="fixed">isUpper
'A'</span> has a type of <span class="fixed">Bool</span>, because its
value is basically <span class="fixed">True</span>. Both those types,
however, have a kind of <span class="fixed">\*</span>.

We used <span class="fixed">:k</span> on a type to get its kind, just
like we can use <span class="fixed">:t</span> on a value to get its
type. Like we said, types are the labels of values and kinds are the
labels of types and there are parallels between the two.

Let's look at another kind.

``` haskell:hs
ghci> :k Either
Either :: * -> * -> *
```

Aha, this tells us that <span class="fixed">Either</span> takes two
concrete types as type parameters to produce a concrete type. It also
looks kind of like a type declaration of a function that takes two
values and returns something. Type constructors are curried (just like
functions), so we can partially apply them.

``` haskell:hs
ghci> :k Either String
Either String :: * -> *
ghci> :k Either String Int
Either String Int :: *
```

When we wanted to make <span class="fixed">Either</span> a part of the
<span class="fixed">Functor</span> typeclass, we had to partially apply
it because <span class="fixed">Functor</span> wants types that take only
one parameter while <span class="fixed">Either</span> takes two. In
other words, <span class="fixed">Functor</span> wants types of kind
<span class="fixed">\* -&gt; \*</span> and so we had to partially apply
<span class="fixed">Either</span> to get a type of kind
<span class="fixed">\* -&gt; \*</span> instead of its original kind
<span class="fixed">\* -&gt; \* -&gt; \*</span>. If we look at the
definition of <span class="fixed">Functor</span> again

``` haskell:hs
class Functor f where 
    fmap :: (a -> b) -> f a -> f b
```

we see that the <span class="fixed">f</span> type variable is used as a
type that takes one concrete type to produce a concrete type. We know it
has to produce a concrete type because it's used as the type of a value
in a function. And from that, we can deduce that types that want to be
friends with <span class="fixed">Functor</span> have to be of kind
<span class="fixed">\* -&gt; \*</span>.

Now, let's do some type-foo. Take a look at this typeclass that I'm just
going to make up right now:

``` haskell:hs
class Tofu t where
    tofu :: j a -> t a j
```

Man, that looks weird. How would we make a type that could be an
instance of that strange typeclass? Well, let's look at what its kind
would have to be. Because <span class="fixed">j a</span> is used as the
type of a value that the <span class="fixed">tofu</span> function takes
as its parameter, <span class="fixed">j a</span> has to have a kind of
<span class="fixed">\*</span>. We assume <span class="fixed">\*</span>
for <span class="fixed">a</span> and so we can infer that
<span class="fixed">j</span> has to have a kind of
<span class="fixed">\* -&gt; \*</span>. We see that
<span class="fixed">t</span> has to produce a concrete value too and
that it takes two types. And knowing that <span class="fixed">a</span>
has a kind of <span class="fixed">\*</span> and
<span class="fixed">j</span> has a kind of <span class="fixed">\* -&gt;
\*</span>, we infer that <span class="fixed">t</span> has to have a kind
of <span class="fixed">\* -&gt; (\* -&gt; \*) -&gt; \*</span>. So it takes a
concrete type (<span class="fixed">a</span>), a type constructor that
takes one concrete type (<span class="fixed">j</span>) and produces a
concrete type. Wow.

OK, so let's make a type with a kind of <span class="fixed">\* -&gt; (\*
-&gt; \*) -&gt; \*</span>. Here's one way of going about it.

``` haskell:hs
data Frank a b  = Frank {frankField :: b a} deriving (Show)
```

How do we know this type has a kind of <span class="fixed">\* -&gt; (\*
-&gt; \*) - &gt; \*</span>? Well, fields in ADTs are made to hold values, so
they must be of kind <span class="fixed">\*</span>, obviously. We assume
<span class="fixed">\*</span> for <span class="fixed">a</span>, which
means that <span class="fixed">b</span> takes one type parameter and so
its kind is <span class="fixed">\* -&gt; \*</span>. Now we know the kinds
of both <span class="fixed">a</span> and <span class="fixed">b</span>
and because they're parameters for <span class="fixed">Frank</span>, we
see that <span class="fixed">Frank</span> has a kind of
<span class="fixed">\* -&gt; (\* -&gt; \*) -&gt; \*</span> The first
<span class="fixed">\*</span> represents <span class="fixed">a</span>
and the <span class="fixed">(\* -&gt; \*)</span> represents
<span class="fixed">b</span>. Let's make some
<span class="fixed">Frank</span> values and check out their types.

``` haskell:hs
ghci> :t Frank {frankField = Just "HAHA"}
Frank {frankField = Just "HAHA"} :: Frank [Char] Maybe
ghci> :t Frank {frankField = Node 'a' EmptyTree EmptyTree}
Frank {frankField = Node 'a' EmptyTree EmptyTree} :: Frank Char Tree
ghci> :t Frank {frankField = "YES"}
Frank {frankField = "YES"} :: Frank Char []
```

Hmm. Because <span class="fixed">frankField</span> has a type of form
<span class="fixed">a b</span>, its values must have types that are of a
similar form as well. So they can be <span class="fixed">Just
"HAHA"</span>, which has a type of <span class="fixed">Maybe
\[Char\]</span> or it can have a value of
<span class="fixed">\['Y','E','S'\]</span>, which has a type of
<span class="fixed">\[Char\]</span> (if we used our own list type for
this, it would have a type of <span class="fixed">List Char</span>). And
we see that the types of the <span class="fixed">Frank</span> values
correspond with the kind for <span class="fixed">Frank</span>.
<span class="fixed">\[Char\]</span> has a kind of
<span class="fixed">\*</span> and <span class="fixed">Maybe</span> has a
kind of <span class="fixed">\* -&gt; \*</span>. Because in order to have a
value, it has to be a concrete type and thus has to be fully applied,
every value of <span class="fixed">Frank blah blaah</span> has a kind of
<span class="fixed">\*</span>.

Making <span class="fixed">Frank</span> an instance of
<span class="fixed">Tofu</span> is pretty simple. We see that
<span class="fixed">tofu</span> takes a <span class="fixed">j a</span>
(so an example type of that form would be <span class="fixed">Maybe
Int</span>) and returns a <span class="fixed">t a j</span>. So if we
replace <span class="fixed">Frank</span> with
<span class="fixed">j</span>, the result type would be
<span class="fixed">Frank Int Maybe</span>.

``` haskell:hs
instance Tofu Frank where
    tofu x = Frank x
```

``` haskell:hs
ghci> tofu (Just 'a') :: Frank Char Maybe
Frank {frankField = Just 'a'}
ghci> tofu ["HELLO"] :: Frank [Char] []
Frank {frankField = ["HELLO"]}
```

Not very useful, but we did flex our type muscles. Let's do some more
type-foo. We have this data type:

``` haskell:hs
data Barry t k p = Barry { yabba :: p, dabba :: t k }
```

And now we want to make it an instance of
<span class="fixed">Functor</span>. <span class="fixed">Functor</span>
wants types of kind <span class="fixed">\* -&gt; \*</span> but
<span class="fixed">Barry</span> doesn't look like it has that kind.
What is the kind of <span class="fixed">Barry</span>? Well, we see it
takes three type parameters, so it's going to be
<span class="fixed">something -&gt; something -&gt; something -&gt; \*</span>.
It's safe to say that <span class="fixed">p</span> is a concrete type
and thus has a kind of <span class="fixed">\*</span>. For
<span class="fixed">k</span>, we assume <span class="fixed">\*</span>
and so by extension, <span class="fixed">t</span> has a kind of
<span class="fixed">\* -&gt; \*</span>. Now let's just replace those kinds
with the *somethings* that we used as placeholders and we see it has a
kind of <span class="fixed">(\* -&gt; \*) -&gt; \* -&gt; \* -&gt; \*</span>.
Let's check that with GHCI.

``` haskell:hs
ghci> :k Barry
Barry :: (* -> *) -> * -> * -> *
```

Ah, we were right. How satisfying. Now, to make this type a part of
<span class="fixed">Functor</span> we have to partially apply the first
two type parameters so that we're left with <span class="fixed">\* -&gt;
\*</span>. That means that the start of the instance declaration will
be: <span class="fixed">instance Functor (Barry a b) where</span>. If we
look at <span class="fixed">fmap</span> as if it was made specifically
for <span class="fixed">Barry</span>, it would have a type of
<span class="fixed">fmap :: (a -&gt; b) -&gt; Barry c d a -&gt; Barry c d
b</span>, because we just replace the
<span class="fixed">Functor</span>'s <span class="fixed">f</span> with
<span class="fixed">Barry c d</span>. The third type parameter from
<span class="fixed">Barry</span> will have to change and we see that
it's conviniently in its own field.

``` haskell:hs
instance Functor (Barry a b) where
    fmap f (Barry {yabba = x, dabba = y}) = Barry {yabba = f x, dabba = y}
```

There we go! We just mapped the <span class="fixed">f</span> over the
first field.

In this section, we took a good look at how type parameters work and
kind of formalized them with kinds, just like we formalized function
parameters with type declarations. We saw that there are interesting
parallels between functions and type constructors. They are, however,
two completely different things. When working on real Haskell, you
usually won't have to mess with kinds and do kind inference by hand like
we did now. Usually, you just have to partially apply your own type to
<span class="fixed">\* -&gt; \*</span> or <span class="fixed">\*</span>
when making it an instance of one of the standard typeclasses, but it's
good to know how and why that actually works. It's also interesting to
see that types have little types of their own. Again, you don't really
have to understand everything we did here to read on, but if you
understand how kinds work, chances are that you have a very solid grasp
of Haskell's type system.



