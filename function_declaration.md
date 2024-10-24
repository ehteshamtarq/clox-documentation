
## Function Declarations
1. **Parsing Function Declarations**: 
   When the compiler encounters a `fun` keyword, it knows it’s dealing with a function declaration. The `funDeclaration()` method handles this. It starts by parsing the function name and storing it, similarly to how variable names are handled in the compiler.

   ```c
   uint8_t global = parseVariable("Expect function name.");
   ```

   The function's name is bound to a global variable at the top level. If the function is inside a block or another function, it becomes a local variable.

2. **Initialization in Two Stages**: 
   Normally, variables are initialized in two stages to avoid accessing an uninitialized variable in its own initializer. However, functions don't have this problem since they can't be called until they are fully compiled. This allows recursive functions to work correctly.

   ```c
   markInitialized();
   ```

   The function’s name is marked as initialized before the function body is compiled, so recursive calls inside the body are allowed.

3. **Compiling the Function**: 
   Once the function name is parsed, the actual function definition is compiled. This involves parsing the parameter list and the function body.

   ```c
   consume(TOKEN_LEFT_PAREN, "Expect '(' after function name.");
   consume(TOKEN_RIGHT_PAREN, "Expect ')' after parameters.");
   consume(TOKEN_LEFT_BRACE, "Expect '{' before function body.");
   block();
   ```

   For now, the code handles an empty parameter list. The body of the function starts with `{`, and the block of code is compiled as part of the function body using the `block()` function.

4. **Storing the Function**: 
   Once the function is compiled, the resulting bytecode needs to be stored in the appropriate variable (global or local). This is done by calling `defineVariable(global)` after the function is compiled.

### Compiler Structure and Nested Functions
1. **Nested Functions**: 
   The interesting part here is the handling of nested functions. Each function being compiled gets its own `Compiler` struct. This struct holds information about the current function’s local variables, blocks, etc. Functions can be nested, so there can be multiple `Compiler` structs active at once. 

   ```c
   Compiler compiler;
   initCompiler(&compiler, type);
   beginScope();
   ```

   When a new function is being compiled, a new `Compiler` is created and initialized. This new `Compiler` takes over as the current one, and when the function is done, the previous `Compiler` is restored.

   ```c
   current = current->enclosing;
   ```

   This effectively creates a stack of compilers, with the most recent function being compiled at the top of the stack. Each `Compiler` has an `enclosing` pointer that points to the `Compiler` of the enclosing function (the function that contains the current one). This chain of `Compiler` structs mirrors the structure of nested functions in the source code.

2. **Using the C Stack**: 
   Instead of dynamically allocating memory for these `Compiler` structs, the compiler stores them on the C stack (the call stack). This is efficient but limits the depth of function nesting, as too many nested functions could cause a stack overflow. This wouldn’t be a problem unless there’s deeply nested or malicious code, but some compilers put limits on nesting to prevent stack overflows.

### Function Parameters
1. **Parsing Parameters**: 
   Functions often take parameters, which are treated as local variables. When parsing the parameter list, the compiler adds each parameter as a local variable in the function’s outermost scope (the function's scope).

   ```c
   if (!check(TOKEN_RIGHT_PAREN)) {
       do {
           current->function->arity++;
           if (current->function->arity > 255) {
               errorAtCurrent("Can't have more than 255 parameters.");
           }
           uint8_t constant = parseVariable("Expect parameter name.");
           defineVariable(constant);
       } while (match(TOKEN_COMMA));
   }
   ```

   Each parameter is parsed and added as a local variable, and the function’s arity (the number of parameters it takes) is incremented. There is a limit of 255 parameters, which is enforced by the check in the code.

2. **Function Name Handling**: 
   The name of the function is stored with the function object when it’s compiled. This is done in the `initCompiler()` function.

   ```c
   current->function->name = copyString(parser.previous.start,
                                        parser.previous.length);
   ```

   The name is copied from the source code string since the source string might not exist once compilation is complete. The function object will need to retain its name for use at runtime.

### Conclusion
This process compiles function declarations by:
- Parsing the function name and storing it in a variable.
- Compiling the parameter list and body.
- Storing the function in the variable (global or local).
- Using a stack of `Compiler` structs to handle nested functions, allowing the compiler to recursively compile functions inside other functions.
  
This method ensures that functions are compiled and stored properly, allowing for features like recursion, local functions, and nested functions. The compiler efficiently manages this using the C stack, making the system lean but limited by potential stack overflows from deep nesting.


