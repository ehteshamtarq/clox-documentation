# Late Binding
## Global variable are late binding while local variable are early binding
Late binding refers to the process where the exact method or variable that a program needs to access is determined at runtime, rather than at compile time. This dynamic resolution of identifiers (like variable names or method calls) happens "late" in the execution process, hence the name.

### Key Characteristics of Late Binding:
1. **Runtime Resolution**: The exact variable, function, or method that a name refers to isn't determined until the program is running.
2. **Flexibility**: Late binding allows more dynamic behavior in a program because the exact method or variable that gets executed or accessed can depend on runtime conditions.
3. **Performance**: Since the resolution happens at runtime, it tends to be slower compared to early (or static) binding, which happens at compile time.
4. **Used in Dynamic Typing**: Languages that support dynamic typing or dynamic dispatch often use late binding. For instance, method calls in Python and JavaScript are late bound because the exact method to invoke is determined based on the type of the object at runtime.

### Example of Late Binding:

In dynamically-typed languages like JavaScript, variables are resolved at runtime:

```javascript
function greet() {
    return "Hello!";
}

const sayHello = greet;  // 'greet' resolves at runtime

console.log(sayHello()); // Output: "Hello!"
```

In this case, the function `greet` is assigned to `sayHello`, but the exact function being referenced (i.e., `greet`) is determined at runtime.

### Late Binding in the Context of Global Variables in Lox:
In Lox (the language you're working with), global variables are resolved via late binding. This means that when the interpreter runs, it looks up the global variable’s name during execution, rather than knowing at compile time which variable is being referenced. This can make things slower because the interpreter needs to check the environment (like a dictionary or hash map) to find the variable during runtime.

In contrast, **early binding** (which you'll use for local variables in Clox) resolves the variable at compile time, making it faster during execution.

