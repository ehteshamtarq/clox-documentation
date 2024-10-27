 ## **native functions**

### Overview
A **native function** in this context is a function implemented in C but callable from Lox. Native functions in Lox allow the VM (virtual machine) to perform complex operations and interact with the system without requiring bytecode instructions or user-defined functions. To achieve this, a **new object type** is created specifically for native functions.

### Steps to Implement Native Functions

1. **Define a Type for Native Functions**:
   Native functions need to be treated differently from regular Lox functions, so we define a new `ObjNative` structure to represent them:
   ```c
   typedef Value (*NativeFn)(int argCount, Value* args);

   typedef struct {
       Obj obj;           // Inherits the general object header
       NativeFn function; // Pointer to the C function implementing the native behavior
   } ObjNative;
   ```
   - **`NativeFn`** is a type for native functions that return a `Value` (the VM’s universal data type) and take two parameters: `argCount` (number of arguments) and a pointer to the argument list.
   - **`ObjNative`** includes an object header (for generic object handling) and a pointer to the C function implementing the behavior.

2. **Create a Constructor for ObjNative**:
   We define a function, `newNative`, to construct an `ObjNative` object by allocating memory for it and assigning the C function pointer:
   ```c
   ObjNative* newNative(NativeFn function) {
       ObjNative* native = ALLOCATE_OBJ(ObjNative, OBJ_NATIVE);
       native->function = function;
       return native;
   }
   ```
   - `ALLOCATE_OBJ` handles memory allocation for the `ObjNative` object.
   - The `OBJ_NATIVE` type ensures this object is recognized as a native function.

3. **Add the Native Object Type to the VM’s Type System**:
   We expand the `ObjType` enum to recognize `OBJ_NATIVE` as a distinct type:
   ```c
   typedef enum {
       OBJ_FUNCTION,
       OBJ_NATIVE,
       OBJ_STRING,
       ...
   } ObjType;
   ```

4. **Implement Memory Management**:
   Since the `ObjNative` doesn’t own any additional memory, its deallocation is straightforward:
   ```c
   case OBJ_NATIVE:
       FREE(ObjNative, object);
       break;
   ```

5. **Define Macros for Dynamic Typing**:
   Macros make it easy to check and cast values to native functions, allowing the VM to recognize them and manage them dynamically:
   ```c
   #define IS_NATIVE(value) isObjType(value, OBJ_NATIVE)
   #define AS_NATIVE(value) (((ObjNative*)AS_OBJ(value))->function)
   ```

6. **Modify callValue() for Native Function Invocation**:
   When a native function is called, we don’t push a new `CallFrame` because no bytecode needs to be executed. Instead, we directly invoke the C function:
   ```c
   case OBJ_NATIVE: {
       NativeFn native = AS_NATIVE(callee);
       Value result = native(argCount, vm.stackTop - argCount);
       vm.stackTop -= argCount + 1;
       push(result);
       return true;
   }
   ```
   - The C function is called with the argument count and a pointer to the arguments.
   - After the function returns, we pop the arguments and push the result onto the stack.

7. **Define a Helper Function to Register Native Functions**:
   To expose native functions to Lox, we create a `defineNative` function that registers a native function under a specific name in the Lox environment:
   ```c
   static void defineNative(const char* name, NativeFn function) {
       push(OBJ_VAL(copyString(name, (int)strlen(name))));
       push(OBJ_VAL(newNative(function)));
       tableSet(&vm.globals, AS_STRING(vm.stack[0]), vm.stack[1]);
       pop();
       pop();
   }
   ```
   - `defineNative` takes a C function pointer (`function`) and a name (`name`) and makes the native function available in the Lox environment.
   - **Why push and pop?** Memory allocation functions `copyString()` and `newNative()` might trigger garbage collection. By pushing them to the stack, we ensure the GC doesn’t free these items prematurely.

8. **Implement the `clock` Native Function**:
   As an example, the `clockNative` function uses the standard C library’s `clock()` to return the time since the program began:
   ```c
   static Value clockNative(int argCount, Value* args) {
       return NUMBER_VAL((double)clock() / CLOCKS_PER_SEC);
   }
   ```
   - **Purpose**: `clockNative` enables users to measure the runtime of Lox programs in seconds.
   - To make `clockNative` available in Lox, we add `defineNative("clock", clockNative);` to `initVM()`.

9. **Benchmarking with `clock`**:
   With the new `clock` function, you can measure execution time. A recursive Fibonacci function, though inefficient, can be used to stress-test the language’s function-calling capability. Running this code should display the time it takes to compute the 35th Fibonacci number:
   ```lox
   fun fib(n) {
       if (n < 2) return n;
       return fib(n - 2) + fib(n - 1);
   }

   var start = clock();
   print fib(35);
   print clock() - start;
   ```

### Summary
This approach:
- Adds native functions as a separate type that integrates seamlessly with the VM’s object model.
- Demonstrates how to register native functions, handle memory management concerns, and call native C functions from within the Lox VM.
- Expands Lox’s capabilities by exposing a single system-level operation (`clock`) as a proof of concept. 

This architecture allows more native functions to be added easily, helping make Lox a more versatile and useful language. Let me know if you'd like to dive into any of these details further!