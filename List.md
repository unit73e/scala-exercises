# Scala List Exercises

1. How come a list of `Char` can be a list of `Any`?

    ```scala
    List('a', 'b', 'c'): List[Any]
    ```

2. Why is the following list infered to be of type `Any`?

    ```scala
    List('a', "b")
    ``` 
 
3. Why is the following list infered to be of type `Int`?

    ```scala
    List('a', 1)
    ```

4. What is the type of an empty list?

5. How come an empty list can be a list of `String`?

    ```scala
    List(): List[String]
    ```

6. Construct a list using `::` and `Nil`

7. Define the insertion sort method

8. Extract the elements of the following list into variables

    ```scala
    List('a', 'b', 'c')
    ```

9. Why is it possible to pattern match with List(...) and `::`?

10. Concatenate three lists

11. Implement the concatenation method of two lists

12. What is the time cost of the concatenation of two lists?

13. Get the length of a list

14. What is the time cost of getting the length of a list?

15. Get the first element of a list

16. Get the last element of a list

17. Get all elements of a list except the last

18. Get all elements except the first

19. Get a list in reverse

20. Get the first three elements of a list

21. Get all elements of a list except the first three

22. Get a pair with the first three elements of a list and the remaining elements

23. Get the second element of a list

24. Get the indices of a list

25. Convert a list of lists to a single list with all elements of the lists

26. Get a list of pairs from two lists like the example below

    ```scala
    val list1 = List(1, 2, 3)
    val list2 = List('a', 'b', 'c')
    val result = List((1, 'a'), (2, 'b'), (3, 'c'))
    ```

27. Get a list of pairs from a list and the corresponding indices

28. Get two lists from a list of pairs like the example below

    ```scala
    val listOfpairs = List((1, 'a'), (2, 'b'), (3, 'c'))
    val result = (List(1, 2, 3), List('a', 'b', 'c'))
    ```
