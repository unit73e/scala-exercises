# List Answers

1.

Because `Char` is a subtype of `Any`.

Lists are covariant: if `A` is a subtype of `B` then `List[A]` is also a
subtype of `List[B]`. Since `Char` is a subtype of `Any` then `List[Char]` is
also a subtype of `List[Any]`.

2.

Because the nearest common super-type of `Char` and `String` is `Any`.

Lists in Scala are homogeneous: all elements of the list must have the same
type. Lists are also covariant: if `A` is a subtype of `B` then `List[A]` is
also a subtype of `List[B]`.

Since lists are homogeneous, a list is constructed with elements of different
types will have its own type inferred to a common type among its elements.
Since lists are covariant, the type will be inferred to the nearest common
super-type among all its elements. For `Char` and `String` that would be `Any`:

    scala> List('a', "b")
    res0: List[Any] = List(a, b)

Below is a simple example to prove this:

    scala> class A()
    defined class A
    
    scala> class B() extends A()
    defined class B
    
    scala> class C() extends B()
    defined class C
    
    scala> class D() extends B()
    defined class D
    
    scala> List(new C(), new D()): List[B]
    res0: List[B] = List(C@7582ff54, D@67545b57)
    
    scala> List(new C(), new D()): List[A]
    res1: List[A] = List(C@49a64d82, D@344561e0)
    
    scala> List(new C(), new D()): List[Any]
    res2: List[Any] = List(C@34e20e6b, D@15ac59c2)
    
    scala> List(new C(), new D())
    res3: List[B] = List(C@7a360554, D@424de326)


Both `C` and `D` are both subtypes of `B`, `A` and `Any`. Since lists are
covariant, it is possible to construct a list of any of those types. Moreover,
if the type of the list is not explicitly specified, it will be inferred to the
nearest common super-type, in this case, `B`.

3.

Because `Char` is automatically converted to `Int`.

Although `Char` is not a subtype of `Int` it is automatically converted to
`Int` if required. Since the list was being constructed with `Char` and `Int`,
the `Char` is automatically converted to `Int` so that the list is homogeneous:

    scala> List('a', 1)
    res4: List[Int] = List(97, 1)

Note that in Scala automatic conversion and implicit conversion are treated
differently. In particular, the type inference mechanism ignores implicit
conversions:

    scala> import scala.language.implicitConversions
    import scala.language.implicitConversions
    
    scala> class A
    defined class A
    
    scala> class B
    defined class B
    
    scala> implicit def a2b(b: B): A = new A()
    a2b: (b: B)A
    
    scala> List(new A(), new B()): List[A]
    res0: List[A] = List(A@598bd2ba, A@5a755cc1)
    
    scala> List(new A(), new B())
    res1: List[Object] = List(A@47d93e0d, B@475b7792)

A list of type `A` can be constructed from `A` and `B` elements, thanks to the
implicit method `a2b`. However, if the type of the list is not explicitly
specified, it is not inferred to be `A`.

4.

It is of type `List[Nothing]`:

    scala> List()
    res0: List[Nothing] = List()

5.

Because every type is a subtype of `Nothing`.

6.

    scala> 1 :: 2 :: 3 :: Nil
    res0: List[Int] = List(1, 2, 3)

7.

The insertion sort iterates each element of a list to build a new sorted list.

To make things simple we start by showing how to implement the insert sort for
type `Int` in ascending order:

```scala
def isort(l: List[Int]): List[Int] = {
  def insert(x: Int, xs: List[Int]): List[Int] = xs match {
    case Nil => List(x)
    case y :: ys =>
      if (x <= y) x :: xs
      else y :: insert(x, ys)
  }

  l match {
    case Nil => Nil
    case x :: xs => insert(x, isort(xs))
  }
}
```

The `insert` private method inserts an element in a list in the position that
should satisfy the ascending order. An example should make this clear:

    scala> insert(2, List(1, 4, 3))
    res1: List[Int] = List(1, 2, 4, 3)

In the example `x = 2` is being inserted in the `xs = List(1, 2, 4, 3)` list.
To satisfy the ascending order, `x = 2` must be placed after any number that is
less than itself. This means it ends up being between `1` and `4`.

The `isort` method simply calls `insert` for every element of the given list.
The result is a sorted list. Here's an example of execution of this method:

    scala> isort(List(3,2,5,0,1))
    res0: List[Int] = List(0, 1, 2, 3, 5)

The algorithm is complete but only works with lists of `Int`. To be able to
sort a list of any type the method must be generic:

```scala
def isort[T <% Ordered[T]](l: List[T]): List[T] = {
  def insert(x: T, xs: List[T]): List[T] = xs match {
    case Nil => List(x)
    case y :: ys =>
      if (x <= y) x :: xs
      else y :: insert(x, ys)
  }

  l match {
    case Nil => Nil
    case x :: xs => insert(x, isort(xs))
  }
}
```

With these modifications, `isort` can be used with any type as long as that
type can be ordered. This is achieved by adding `[T <% Ordered[T]]` and
replacing every `Int` with `T`. Simply making `isort` polymorphic is not enough
because not every type has the `<=` method. To solve this problem we added the
view bound `Ordered[T]` to `T`, allowing only types that can used as it they
were `Ordered[T]`. In practice this means `T` must be either a `Ordered[T]` or
there is an implicit method that converts `T` to `Ordered[T]`. For example,
`Int` is not a subtype of `Ordered[T]` but has an implicit method that converts
to `Ordered[T]`.

Now you can sort a list of `Char` as well:

    scala> isort(List('b', 'a', 'z', 'e'))
    res0: List[Char] = List(a, b, e, z)

Finally, to have a fully generic sort method, it should possible to specify how
we want to order the elements of the list:

```scala
def isort[T](l: List[T])(less: (T, T) => Boolean): List[T] = {
  def insert(x: T, xs: List[T]): List[T] = xs match {
    case Nil => List(x)
    case y :: ys =>
      if (less(x, y)) x :: xs
      else y :: insert(x, ys)
  }

  l match {
    case Nil => Nil
    case x :: xs => insert(x, isort(xs)(less))
  }
}
```

Now `isort` has a curried parameter `less`, which is a function that takes two
elements of type `T` and returns true if the first element is less than the
second. The `<=` operator is replaced with `less` and this makes any sort order
possible. Also we don't need `T` to be view bound to `Ordered[T]` anymore
because any condition for ordering works, not just `<=`.

Bellow is an example of a list of `Int` sorted in descending order:

    scala> isort(List(2,3,0,5,1,6))(_ > _)
    res0: List[Int] = List(6, 5, 3, 2, 1, 0)

Of course this means any ordering condition is possible, like sorting a list of
words by length is ascending order:

    scala> isort(List("abc", "ab", "abcde", "a"))(_.length < _.length)
    res2: List[String] = List(a, ab, abc, abcde)

8.

Lists can be pattern matched anywhere in the code:

    scala> val List(a, b, c) = List('a', 'b', 'c')
    a: Char = a
    b: Char = b
    c: Char = c

9.

10.

The `:::` method concatenates two lists:

    scala> List(1) ::: List(2) ::: List(3)
    res0: List[Int] = List(1, 2, 3)

11.

12.

The time costs is `O(n)` because the first list must be fully traversed.

13.

The `length` method gets the length of a list:

    scala> List(1, 2, 3).length
    res0: Int = 3

14.

The time cost is `O(n)` because the list must be fully traversed. Lists in
Scala are immutable and for that reason there is no state that stores the
length of a list.

15.

The `head` method gets the first element of a list:

    scala> List(1, 2, 3).head
    res0: Int = 1

The time cost of the operation is `O(1)`.

16.

The `last` method gets the last element of a list:

    scala> List(1, 2, 3).last
    res0: Int = 3

Note that the time cost of this operation is `O(n)` because the whole list must
be traversed to get the last element. Lists in Scala are immutable, so there is
no state that stores the last element of the list.

17.

The `init` method gets all elements of a list, except the last:

    scala> List(1, 2, 3).init
    res0: List[Int] = List(1, 2)

Note that the time cost of this operation is `O(n)`. To copy every element of
the list, the whole list must be traversed, except for the last element.

18.

The `tail` method gets all elements of a list, except the first:

    scala> List(1, 2, 3).tail
    res0: List[Int] = List(2, 3)

The time cost of the operation is `O(1)`. Lists are immutable, so there is no
possibility of modification. For that reason, instead of copying every element,
except the first, a reference should suffice.

Here is a possible implementation of `tail`:

    ```scala
    def myTail[T](l: List[T]) = l match {
      case Nil   => Nil
      case x::xs => xs
    }
    ```

Using the same list in our previous example:

    scala> myTail(List(1,2,3))
    res0: List[Int] = List(2, 3)

19.

The `reverse` method gets a list in reverse:

    scala> List(1,2,3).reverse
    res0: List[Int] = List(3, 2, 1)

20.

The `take` method gets the `n` first elements of a list:

    scala> List(1,2,3,4,5).take(3)
    res0: List[Int] = List(1, 2, 3)

21.

The `drop` method gets all elements of a list except the first `n`:

    scala> List(1,2,3,4,5).drop(3)
    res0: List[Int] = List(4, 5)

22.

The `sliptAt` returns a pair with the first `n` elements in the first container
and the remaining elements in the second:

    scala> List(1,2,3,4,5).splitAt(3)
    res0: (List[Int], List[Int]) = (List(1, 2, 3),List(4, 5))

23.

The `apply` method gets the element of a list at index `i`:

    scala> List(1,2,3,4,5).apply(1)
    res0: Int = 2

If the name of a method is not specified, the `apply` method is called. So the code can be shortened to:

    scala> List(1,2,3,4,5)(1)
    res0: Int = 2

Note that the index of a list starts in zero and the time cost of getting the
`i` element is `i + 1` steps.

24.

The `indices` method gets the indices of a list:

    scala> List(1,2,3,4,5).indices
    res0: scala.collection.immutable.Range = Range(0, 1, 2, 3, 4)

25.

The `flatten` method takes a list of lists and flattens to a single list:

    scala> List(List(1, 2), List(3, 4), List(5, 6)).flatten
    res0: List[Int] = List(1, 2, 3, 4, 5, 6)

26.

The `zip` method takes two lists and forms a list of pairs:

    scala> List(1, 2, 3) zip List('a', 'b', 'c')
    res0: List[(Int, Char)] = List((1,a), (2,b), (3,c))

27.

One possible solution is using a combination of `indices` and `zip`:

    scala> val l = List('a', 'b', 'c')
    l: List[Char] = List(a, b, c)
    
    scala> l zip l.indices
    res0: List[(Char, Int)] = List((a,0), (b,1), (c,2))

However this is not the most efficient solution because the list must be
traversed twice. The most efficient solution is using the `zipWithIndex`
method:

    scala> List('a', 'b', 'c').zipWithIndex
    res0: List[(Char, Int)] = List((a,0), (b,1), (c,2))

28.

That would be the opposite of `zip`, the `unzip` method:

    scala> List((1, 'a'), (2, 'b'), (3, 'c')).unzip
    res0: (List[Int], List[Char]) = (List(1, 2, 3),List(a, b, c))

