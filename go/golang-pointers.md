# Explanation of Pointers in Golang on a simple metaphor

_Originally published on Jun 10, 2023 [here](https://medium.com/@jannden/explanation-of-pointers-in-golang-on-a-simple-metaphor-d9b3a04de9ad)._

---

Pointers are an essential concept to understand. Yet it's easy to get confused when you are new to Go --- especially so if you are coming from JavaScript.

Let's break it down.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:875/1*DUSIS9ZJgabPS7CesSZwaA.png)

### To explain pointers in Go, let's use a metaphor

Imagine a global object that we'll call the "Memory Map". The keys of this object correspond to memory addresses. Then some value can be assigned to any of these keys.

Now, let's say that we have a variable `x` that has a value of 20. When we declare this variable in Go, a piece of your computer's memory will be allocated to store the value of `x`.

In order for your computer to be able to find the value of `x` stored in its memory, it has to keep a record of the memory address. Let's say that the address of the memory that stores the value of `x` is `0x1234`. The Memory Map would look something like this:

0x1234 -> 20

The keys of the Memory Map (such as 0x1234) are called pointers.

In languages such as JavaScript, it is not possible to find out the keys of the Memory Map. In other words, we don't have access to pointers. But that's not the case with Go.

To find out the memory address of a variable in Go, we use the `&` operator as a prefix to the nickname of the variable. We gave our variable a nickname `x`, which means:

- `x` is a nickname for the value 20
- `&x` is a nickname for the pointer 0x1234

Now, let's say that we want to save the pointer to a separate variable `p`. This is how it would be done:

```react
p := &x
```

The variable `p` now holds a pointer to `x` as its value. If we were to print the value of `p`, we would see `0x1234`.

If `p` itself had a memory address 0x5678, then our Memory Map would now have two rows that would look like this:

0x1234 -> 20
0x5678 -> 0x1234

### Using pointers

Now, let's say that we want to change the value of `x`.

We can do so the standard way:

```react
x = 10
```

Or we can achieve exactly the same result in an alternative way by using the `*` operator on the `p` variable. Remember, the `p` variable stores the memory address to `x`. The `*` operator tells Golang, that you want to tamper with whatever is stored at the memory address saved as a value of `p`. The expression would look like this:

```react
*p = 10
```

In both of the examples above, we set the value of `x` to 10. The Memory Map now looks like this:

0x1234 -> 10
0x5678 -> 0x1234

Note that we didn't change the value of `p` itself. We only changed the value of the memory location that `p` points to.

Pointers can be especially useful when working with large data structures like arrays and structs. Instead of copying the entire data structure every time we want to pass it to a function, we can just pass a pointer to the data structure. This can save a lot of memory and improve performance.

### Using pointers in functions

Here is [a great example](https://gobyexample.com/pointers) of using pointers in functions. Read through the code below. Now it shouldn't be a problem to understand how and why the value of `i` changes:

```react
package main

import "fmt"

func zeroval(ival int) {
    ival = 0
}

func zeroptr(iptr *int) {
    *iptr = 0
}

func main() {
    i := 1
    fmt.Println("initial", i) // prints: initial 1

    zeroval(i)
    fmt.Println("zeroval", i) // prints: zeroval 1

    zeroptr(&i)
    fmt.Println("zeroptr", i) // prints: zeroptr 0

    fmt.Println("pointer", &i) // prints: pointer 0x42131100
}
```

### Conclusion

Hope you got a good grasp on pointers in Go 😊

If you found it helpful, please click the clap 👏 button.
And feel free to comment! I'd be happy to help :)
