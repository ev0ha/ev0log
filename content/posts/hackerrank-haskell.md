---
title: "HackerRank: List Replication in Haskell"
date: 2022-10-28T19:20:11+02:00
draft: false
---

# Problem description

```
Given a list, repeat each element in the list n amount of times. The input and output portions will be 
handled automatically by the grader. You need to write a function with the recommended method signature.

Input Format
The first line contains the integer S where S is the number of times you need to repeat the elements.
The next X lines each contain an integer. These are the X elements in the array.
```
We want to iterate over the elements of the list and repeat each element n amount of times:

```haskell
f n arr =
    map (replicate n) arr 
```
Easy, right!? This doesn't give us the result we want though, because it creates lists of the repeated elements. This time with example inputs:

```haskell
ghci> map (replicate 3) [1, 2, 3]
[[1,1,1],[2,2,2],[3,3,3]]
```
So we need to flatten the result. Haskell has a built-in function to concat the resulting lists which we will use instead of map.
```haskell
ghci> concatMap (replicate 3) [1, 2, 3]
[1,1,1,2,2,2,3,3,3]
```
This works! It's a straightforward solution that builds on the naive (and not correct!) first attempt but maybe we can solve it in a nicer way.
Let's look at the concatMap function.

```haskell
ghci> :info concatMap
concatMap :: Foldable t => (a -> [b]) -> t a -> [b]
  	-- Defined in ‘Data.Foldable’
```
So concatMap takes a function a -> [b] (in our case replicate) and a container with the Foldable constraint (in our case a list) 
and returns the concatenated lists, each produced by the function, as a result. Great! Foldable is a type class, do other type classes also apply to list?
```haskell
ghci> :info []
type [] :: * -> *
data [] a = [] | a : [a]
  	-- Defined in ‘GHC.Types’
instance Applicative [] -- Defined in ‘GHC.Base’
instance Eq a => Eq [a] -- Defined in ‘GHC.Classes’
instance Functor [] -- Defined in ‘GHC.Base’
instance Monad [] -- Defined in ‘GHC.Base’
instance Monoid [a] -- Defined in ‘GHC.Base’
instance Ord a => Ord [a] -- Defined in ‘GHC.Classes’
instance Semigroup [a] -- Defined in ‘GHC.Base’
instance Show a => Show [a] -- Defined in ‘GHC.Show’
instance Foldable [] -- Defined in ‘Data.Foldable’
instance MonadFail [] -- Defined in ‘Control.Monad.Fail’
instance Read a => Read [a] -- Defined in ‘GHC.Read’
instance Traversable [] -- Defined in ‘Data.Traversable’
```
As we can see: Yes. Quite a lot, actually. List is also an instance of Monad, now let's take a look at that.
```haskell
ghci> :info Monad
type Monad :: (* -> *) -> Constraint
class Applicative m => Monad m where
  (>>=) :: m a -> (a -> m b) -> m b
  (>>) :: m a -> m b -> m b
  return :: a -> m a
  {-# MINIMAL (>>=) #-}
  	-- Defined in ‘GHC.Base’
instance Monad (Either e) -- Defined in ‘Data.Either’
instance Monad [] -- Defined in ‘GHC.Base’
instance Monad Solo -- Defined in ‘GHC.Base’
instance Monad Maybe -- Defined in ‘GHC.Base’
instance Monad IO -- Defined in ‘GHC.Base’
instance Monad ((->) r) -- Defined in ‘GHC.Base’
instance (Monoid a, Monoid b, Monoid c) => Monad ((,,,) a b c)
  -- Defined in ‘GHC.Base’
instance (Monoid a, Monoid b) => Monad ((,,) a b)
  -- Defined in ‘GHC.Base’
instance Monoid a => Monad ((,) a) -- Defined in ‘GHC.Base’
```
See the so called bind operator (>>=)? It takes a monad instance (m a - in our case a list) and a function that produces another monad instance 
(a -> m b - in our case replicate) and returns a monad instance of b (m b). Remember the concatMap function and the similarities should be obvious,
so let's try the bind operator to solve our problem.
```haskell
ghci> [1, 2, 3] >>= replicate 4
[1,1,1,1,2,2,2,2,3,3,3,3]
```
Now that looks nice if you ask me! Complete solution as submitted below.
```haskell
f :: Int -> [Int] -> [Int]
f n arr = -- Complete this function
     arr >>= replicate n
-- This part handles the Input and Output and can be used as it is. Do not modify this part.
main :: IO ()
main = getContents >>=
       mapM_ print. (\(n:arr) -> f n arr). map read. words
```