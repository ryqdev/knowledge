# Closure
In a **closure scenario**, the local variables captured by the closure are typically stored on the **heap**, rather than the stack, to allow them to persist even after the function that created them has returned. Let's break this down:

### What Happens in a Closure Scenario?

- In a regular function, local variables are stored on the **stack**. They exist only for the duration of the function's execution, and once the function returns, the stack is unwound, and those local variables are destroyed.
  
- In the case of a **closure**, certain local variables of the outer function are **captured** by the inner function (the closure). These variables need to "live" as long as the closure is alive, potentially long after the outer function has returned. To achieve this, the variables are moved from the stack to the **heap**.

### Why the Heap?

- The **heap** is a region of memory where data can persist beyond the scope in which it was created. Unlike stack memory, which is cleaned up when a function returns, heap memory persists until it is explicitly released or garbage collected.
  
- Since closures can be returned from a function or passed around, the variables they capture must have a longer lifespan. The runtime (whether Python, JavaScript, etc.) moves these variables to the heap and manages their memory.

### Closure Storage in Different Languages

Let's consider how this works in a few languages:

#### 1. **Python**:
In Python, when a closure captures a variable, that variable is moved to the heap (more specifically, it is stored in the **function object** itself). Python keeps a reference to the captured variables so that they are not garbage collected as long as the closure exists.

```python
def outer(x):
    def inner():
        return x  # x is captured and stored in the closure
    return inner

closure_func = outer(10)
print(closure_func())  # x still accessible via the closure
```
Here, the variable `x` (which would normally be destroyed when `outer` returns) is stored as part of the closure's environment. Python stores this environment in the closure itself, which keeps `x` on the heap.


#### 2. **C++ (Lambdas)**:
In C++, closures can capture variables by reference or by value. When captured by value, the captured variables are stored on the heap so that the lambda can access them even after the outer function returns.

```cpp
auto outer_function(int x) {
    return [x]() {  // x is captured by value and stored on the heap
        return x;
    };
}

auto closure_func = outer_function(10);
std::cout << closure_func();  // Outputs 10
```

In this example, the variable `x` is captured by the lambda expression, and its value persists even after `outer_function` has returned.

#### 3. Rust
```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {v:?}");
    });

    handle.join().unwrap();
}
```
Rust compiler will check wether the closure get the ownership of environment variables or not. In this case, the closure takes ownership of `v` and moves it to the heap.

#### 4. Golang

The potencial bug:
```go
package main

import (
	"sync"
	"time"
)

func main() {
	var wg sync.WaitGroup
	wg.Add(1)
	s := "gopher"
	go func() {
		defer wg.Done()
		time.Sleep(1)
		println(s)
	}()
	s = "hello"
	wg.Wait()
}
```
`s` will be changed to `hello` after captured by closure.
