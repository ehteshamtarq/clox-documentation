### 1. **Matching the `var` Keyword**
When the parser encounters a `var` keyword, it recognizes that a variable declaration is starting. The `declaration()` function handles this:

```c
static void declaration() {
    if (match(TOKEN_VAR)) {
        varDeclaration();
    } else {
        statement();
    }
    if (parser.panicMode) synchronize();
}
```

If `match(TOKEN_VAR)` is true, it calls `varDeclaration()` to handle the variable declaration.

### 2. **Parsing the Variable**
Inside `varDeclaration()`, the next step is to parse the variable name. The function `parseVariable()` is called to expect and consume the variable identifier (name) token.

```c
static void varDeclaration() {
    uint8_t global = parseVariable("Expect variable name.");
    ...
}
```

#### Parsing the Identifier:
`parseVariable()` ensures that the next token is a valid identifier (the variable name). If it's not, an error is triggered.

```c
static uint8_t parseVariable(const char *errorMessage) {
    consume(TOKEN_IDENTIFIER, errorMessage);
    return identifierConstant(&parser.previous);
}
```

- `consume(TOKEN_IDENTIFIER, ...)`: Ensures the current token is a valid identifier.
- `identifierConstant()`: Converts the identifier into a constant in the bytecode, returning its index for later use.

### 3. **Handling Initializer (Optional Assignment)**
After the variable name, the compiler checks if the variable is being initialized (i.e., assigned a value). If the `=` token is present, it parses an expression that will become the variable's value. Otherwise, the variable is initialized with `nil`.

```c
if (match(TOKEN_EQUAL)) {
    expression();  // Parse the expression after the '='
} else {
    emitByte(OP_NIL);  // Initialize the variable with nil
}
```

- `expression()`: Parses the right-hand side of the assignment (if present) and emits the corresponding bytecode.
- `emitByte(OP_NIL)`: If no assignment is made, the variable gets assigned a `nil` value.

### 4. **Finalizing the Declaration**
After parsing the variable's name and possible initializer, the compiler expects a semicolon (`;`) to finish the declaration. It then emits bytecode to define the variable globally.

```c
consume(TOKEN_SEMICOLON, "Expect ';' after variable declaration.");
defineVariable(global);
```

- `defineVariable(global)`: Emits bytecode to define the variable as a global variable, using the index `global` (returned by `identifierConstant()`).

### 5. **Defining the Variable**
In `defineVariable()`, the compiler emits the bytecode instruction to define a global variable:

```c
static void defineVariable(uint8_t global) {
    emitBytes(OP_DEFINE_GLOBAL, global);
}
```

- `emitBytes(OP_DEFINE_GLOBAL, global)`: The `OP_DEFINE_GLOBAL` operation stores the variable into a global slot, using the index from the `global` argument.

### 6. **Result**
After the `varDeclaration()` function finishes, the variable is fully declared in the global scope, either initialized with a value or set to `nil`, and can be used later in the program.

### Summary of Steps:
1. **Keyword Matching**: The compiler matches the `var` keyword.
2. **Variable Parsing**: The variable name is parsed and stored as a constant.
3. **Optional Assignment**: The variable may be initialized with a value or `nil`.
4. **Finalization**: A semicolon is expected, and the variable is defined globally.

This process ensures that the variable is properly added to the program’s symbol table, and its value (if any) is stored in the bytecode for future use.