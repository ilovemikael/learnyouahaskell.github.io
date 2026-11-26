---
chapter: 5
title: "Recursion"
---

## Hello recursion! {#hello-recursion}

![SOVIET RUSSIA](assets/images/recursion/recursion.png){.left width=250 height=179}


We mentioned recursion briefly in the previous chapter.
In this chapter, we'll take a closer look at recursion, why it's important to Haskell, and how we can work out very concise and elegant solutions to problems by thinking recursively.

If you still don't know what recursion is, read this sentence.
Haha!~!
Just kidding!
Recursion is actually a way of defining functions in which the function applies itself within its definition.
Definitions in mathematics are often given recursively.
For instance, the fibonacci sequence is defined recursively:
First, we define the first two fibonacci numbers non-recursively;
we say that *F(0) = 0* and *F(1) = 1*, meaning that the 0th and 1st fibonacci numbers are 0 and 1, respectively.
Then we say that for any other natural number, that fibonacci number is the sum of the previous two fibonacci numbers.
So *F(n) = F(n-1) + F(n-2)*.
That way, *F(3)* is *F(2) + F(1)*, which is *(F(1) + F(0)) + F(1)*.
Because we've fell through to only non-recursively defined fibonacci numbers, we can safely say that *F(3)* is 2.
Having an element or two in a recursive definition defined non-recursively (like *F(0)* and *F(1)* here) is also called the **edge condition** (or base case) and is important if you want your recursive function to terminate.
If we hadn't defined *F(0)* and *F(1)* non recursively, you'd never get a solution to any number because you'd reach 0 and descend into negative numbers.
All of a sudden, you'd be saying that *F(-2000)* is *F(-2001) + F(-2002)* and there'd still be no end in sight!

Recursion is important to Haskell because unlike in imperative languages, computations are performed by declaring what something *is* rather than declaring *how* you get it.
That's why there are no while or for loops in Haskell; instead, we often have to declare things recursively.

## Maximum awesome {#maximum-awesome}

The `maximum` function takes a list of things that can be ordered (i.e. instances of the `Ord` typeclass) and returns the largest of them.
Think about how you'd implement that in an imperative fashion:
You'd probably set up a variable to hold the maximum value so far and loop through the elements of a list; if an element is bigger than then the current max, you'd replace it with that element.
In the end, the result is the max value remaining.
Whew!
Quite a few words to describe such a simple algorithm!

Now let's see how we could define it recursively:
First, we'd set up an edge condition which says a singleton list's only element is its max;
if the tail's max is bigger, well, then it's the new max.
That's it!
Now let's implement that in Haskell.

```{.haskell:hs}
maximum' :: (Ord a) => [a] -> a
maximum' [] = error "maximum of empty list"
maximum' [x] = x
maximum' (x:xs)
    | x > maxTail = x
    | otherwise = maxTail
    where maxTail = maximum' xs
```

As you can see, pattern matching goes great with recursion!
As most imperative languages don't have pattern matching, you have to make lots of if/else statements to test for edge conditions.
Here, we simply put them out as patterns.
So the first edge condition says that if the list is empty, crash!
Makes sense because, what's the maximum of an empty list?
I don't know.
The second pattern lays another base case:
it says that if it's a singleton list, just give back the only element.

The third pattern is where the action happens;
we use pattern matching to split a list into a head and tail.
This is a very common idiom when recursing lists, so get used to it.
We define `maxTail` as the maximum of the list's remainder in a `where` binding.
Then we check if the head is greater than the maximum of the rest of the list.
If it is, we return the head.
Otherwise, we return the maximum of the rest of the list.

Let's take an example list of numbers and check out how this would work on them: `[2,5,1]`.
If we call `maximum'` on that, the first two patterns won't match.
The third one will and the list is split into `2` and `[5,1]`.
The `where` block wants to know the maximum of `[5,1]`, so we follow that route.
It matches the third pattern again and `[5,1]` is split into `5` and `[1]`.
Again, the `where` block wants to know the maximum of `[1]`.
Because that's the edge condition, it returns `1`.
Finally!
So going up one step, comparing `5` to the maximum of `[1]` (which is `1`), we obviously get back `5`.
So now we know that the maximum of `[5,1]` is `5`.
We go up one step again where we had `2` and `[5,1]`.
Comparing `2` with the maximum of `[5,1]`, which is `5`, we choose `5`.

An even clearer way to write this function is to use `max`.
If you remember, `max` is a function that takes two numbers and returns the bigger of them.
Here's how we could rewrite `maximum'` by using `max`:

```{.haskell:hs}
maximum' :: (Ord a) => [a] -> a
maximum' [] = error "maximum of empty list"
maximum' [x] = x
maximum' (x:xs) = max x (maximum' xs)
```

How's that for elegant?
In essence, the maximum of a list is the max of the first element and the maximum of the tail.

![max](assets/images/recursion/maxs.png){.center width=651 height=267}

## A few more recursive functions {#a-few-more-recursive-functions}

Now that we know how to generally think recursively, let's implement a few functions using it.
First off, we'll implement `replicate`:
it takes an `Int`, some element, and returns a list with several repetitions of that element.
For instance, `replicate 3 5` returns `[5,5,5]`.
Let's think about the edge condition:
My guess is that it's 0 or less.
If we try to replicate something zero times, it should return an empty list;
same for negative numbers, since it doesn't make much sense.

```{.haskell:hs}
replicate' :: (Num i, Ord i) => i -> a -> [a]
replicate' n x
    | n <= 0    = []
    | otherwise = x:replicate' (n-1) x
```

We used guards here instead of patterns because we're testing for a boolean condition.
If `n` is less than or equal to 0, return an empty list;
otherwise, return a list that has `x` as the first element and then `x` replicated n-1 times as the tail.
Eventually, the `(n-1)` part will push our function to the edge.

::: {.hintbox}
**Note:** `Num` is not a subclass of `Ord`.
This is because not every number type has an ordering, e.g. complex numbers aren't ordered;
that's why we have to specify both the `Num` and `Ord` class constraints when adding, subtracting, or comparing.
:::

Next up, we'll implement `take`.
It `take`s a certain number of elements from a list.
For instance, `take 3 [5,4,3,2,1]` returns `[5,4,3]`.
If we try to take 0 or fewer elements from a list or try to take anything from an empty list, we get an empty list;
notice that those are two edge conditions right there.
Let's write that out:

```{.haskell:hs}
take' :: (Num i, Ord i) => i -> [a] -> [a]
take' n _
    | n <= 0   = []
take' _ []     = []
take' n (x:xs) = x : take' (n-1) xs
```

![painter](assets/images/recursion/painter.png){.right width=350 height=276}

The first pattern specifies that if we try to take a 0 or negative number of elements, we get an empty list.
Notice that we're using `_` to match the list because we don't really care what it is in this case.
Also notice that we use a guard, but without an `otherwise` part.
That means that if `n` turns out to be more than 0, the matching will fall through to the next pattern.
The second pattern indicates that if we try to take anything from an empty list, we get an empty list.
The third pattern breaks the list into a head and a tail.
And then we state that taking `n` elements from a list equals a list that has `x` as the head and then a list that takes `n-1` elements from the tail as a tail.
Try using a piece of paper to write down how the evaluation would look like if we try to take, say, 3 from `[4,3,2,1]`.

`reverse` simply reverses a list.
Think about the edge condition.
What is it?
Come on ... it's the empty list!
An empty list reversed equals the empty list itself.
O-kay. Okie dokie.
What about the rest of it?
Well, you could say that if we split a list to a head and a tail, the reversed list is equal to the reversed tail and then the head at the end.

```{.haskell:hs}
reverse' :: [a] -> [a]
reverse' [] = []
reverse' (x:xs) = reverse' xs ++ [x]
```

There we go!

Because Haskell supports infinite lists, our recursion doesn't really have to have an edge condition.
But if it doesn't have it, it will either keep churning at something infinitely or produce an infinite data structure, like an infinite list.
The good thing about infinite lists though is that we can cut them where we want.
`repeat` takes an element and returns an infinite list that just has that element.
A recursive implementation of that is really easy, watch.

```{.haskell:hs}
repeat' :: a -> [a]
repeat' x = x:repeat' x
```

Calling `repeat 3` will give us a list starting with `3` which has infinite 3's as a tail;
calling it would evaluate like `3:repeat 3`, which is `3:(3:repeat 3)`, which is `3:(3:(3:repeat 3))`, etc.
`repeat 3` will never finish evaluating, whereas `take 5 (repeat 3)` will give us a list of five 3's;
essentially, it's `replicate 5 3`.

`zip` takes two lists and zips them together.
`zip [1,2,3] [2,3]` returns `[(1,2),(2,3)]`, because it truncates the longer list to match the length of the shorter one.
How about if we zip something with an empty list?
Well, we get an empty list back then.
So there's our edge condition.
However, `zip` takes two lists as parameters, so there are actually two edge conditions.

```{.haskell:hs}
zip' :: [a] -> [b] -> [(a,b)]
zip' _ [] = []
zip' [] _ = []
zip' (x:xs) (y:ys) = (x,y):zip' xs ys
```

The first two patterns say if the first or second list is empty, an empty list is returned;
the third one says that two zipped lists return their heads paired up with the zipped tails appended.
Zipping `[1,2,3]` and `['a','b']` eventually tries to zip `[3]` with `[]`:
The edge conditions kick in, resulting in `(1,'a'):(2,'b'):[]`, which is identical to `[(1,'a'),(2,'b')]`.

Let's implement one more standard library function --- `elem`.
It takes an element, list, and sees if said element exists within the list;
the empty list is the edge condition, as is most common in lists.
We know an empty list contains no elements, so it's certainly missing the droids we're looking for.

```{.haskell:hs}
elem' :: (Eq a) => a -> [a] -> Bool
elem' a [] = False
elem' a (x:xs)
    | a == x    = True
    | otherwise = a `elem'` xs
```

Pretty simple and expected:
if the element isn't the head, we check the tail;
if we hit an empty list, the result is `False`.

## Quick, sort! {#quick-sort}

We have a list of items that can be sorted;
their type is an instance of the `Ord` typeclass.
And now, we want to sort them!
There's a very cool sorting algorithm called quicksort:
It's a very clever way of sorting items.
While it takes upwards of 10 lines to implement quicksort in imperative languages, the Haskell version is much more short and elegant.
Quicksort has become a sort of poster child for Haskell;
therefore, let's implement it here (despite the fact that implementing quicksort in Haskell is considered really cheesy due to everyone doing it to showcase its elegance).

![quickman](assets/images/recursion/quickman.png){.left width=180 height=235}

So, the type signature is going to be `quicksort :: (Ord a) => [a] -> [a]`:
No surprises there.
The edge condition?
An empty list, as expected;
an empty list is already sorted.
Now here comes the main algorithm: **a sorted list is one with all values less than (or equal to) the head of the list in front (and those values are sorted), then the head of the middle list, and finally all the values larger than the head (also sorted).**
Notice we said *sorted* twice in this definition, so we'll probably have to make the recursive call twice!
Also notice we defined it using the verb *is* to define the algorithm instead of saying *do this, do that, then do that ...*.
That's the beauty of functional programming!
How are we going to filter the list so that we only get elements less or more than the head of our list?
List comprehensions.
So, let's dive in and define this function:

```{.haskell:hs}
quicksort :: (Ord a) => [a] -> [a]
quicksort [] = []
quicksort (x:xs) =
    let smallerSorted = quicksort [a | a <- xs, a <= x]
        biggerSorted = quicksort [a | a <- xs, a > x]
    in  smallerSorted ++ [x] ++ biggerSorted
```

Let's give it a small test run to see if it appears to behave correctly.

```{.haskell:ghci}
ghci> quicksort [10,2,5,3,1,6,7,4,2,3,4,8,9]
[1,2,2,3,3,4,4,5,6,7,8,9,10]
ghci> quicksort "the quick brown fox jumps over the lazy dog"
"        abcdeeefghhijklmnoooopqrrsttuuvwxyz"
```

Booyah!
That's what I'm talking about!
So, if we have, say `[5,1,9,4,6,7,3]`, and wish to sort it, this algorithm will first take the head, which is `5` and jam it between two smaller and greater lists;
at some point, you'll have `[1,4,3] ++ [5] ++ [9,6,7]`.
We know that once the list is fully sorted, the number `5` will stay in fourth place since there are 3 below it and 3 above it.
Now, if we sort `[1,4,3]` and `[9,6,7]`, we have a sorted list!
We sort those two lists using the same function;
eventually, we'll break it up so much that the list empties, and an empty list is already sorted by virtue of its emptiness.
Here's an illustration:

![quicksort](assets/images/recursion/quicksort.png){.center width=799 height=332}

An element that's in place and won't move anymore is represented in [orange]{style=color:#FF6600;font-weight:bold;}.
If you read from left to right, you'll see the sorted list;
although we chose to compare them all to the heads, we could have compared against any element.
In quicksort, the element you compare against is called a pivot:
They're in [green]{style=color:#009900;font-weight:bold} here.
We chose the head as it's easy to retrieve via pattern matching;
elements smaller than the pivot are [light green]{style=color:#0f0;font-weight:bold} and elements larger than the it are [dark green]{style=color:#030;font-weight:bold}.
The yellowish gradient thing represents an application of quicksort.

## Thinking recursively {#thinking-recursively}

We did quite a bit of recursion so far and as you've probably noticed, there's a pattern here:
Usually you define an edge case, a function that does something between some element, and the function applied to the rest;
it doesn't matter whether it's a list, a tree or any other data structure.
A sum is the head of a list plus the sum of the tail;
a product is the head of the list times the product of the tail;
the length of a list is one plus the length of the tail.
Et cetera, et cetera...

![brain](assets/images/recursion/brain.png){.left width=250 height=219}

Of course, there are edge cases:
Usually, it's some scenario where a recursive application is illogical;
when dealing with lists, the edge case is typically the empty one;
if dealing with trees, it's usually a node without any children.

Dealing with numbers recursively is similar:
It frequently revolves around some number and the function applied to said number modified;
we did the factorial function earlier, which is the product of a number and its factorial minus one.
Such recursive applications don't make sense with zero, since factorials are defined only for positive integers. (factorial 0 is 1?)
Often the edge case value turns out to be an identity.
The identity for multiplication is 1 because if you multiply something by it, you get that something back;
also when doing sums of lists, we define the sum of an empty list as 0, which is the identity for addition.
In quicksort, the edge case and identity are the empty list because if you add a list to an empty list, the original is returned.

So when trying to think of a recursive method for solving a problem, if a recursive solution doesn't apply, see if you can use it as an edge case; think about identities and whether you'll break apart the function's parameters (for instance, lists are usually broken into a head and tail via pattern matching), and which part you'll use the recursive call on.

