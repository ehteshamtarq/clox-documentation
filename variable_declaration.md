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



# how compiler access global variable ?

In **clox**, **global variables** are handled differently from local variables. Global variables are stored in a **global table** rather than on the stack. Here's how the compiler accesses global variables:

### 1. **Declaration of Global Variables**

When a global variable is declared in Lox (the language implemented by clox), it is stored in a global hash table (called the **globals table**). This table stores variables by their name as a key and their value as a corresponding entry.

```lox
var globalVar = 42;  // This is a global variable
```

### 2. **How the Compiler Handles Global Variables**

During compilation, global variables are treated differently from local variables:

- The compiler keeps track of **global variables** by their names, rather than using a stack slot like for local variables.
- Whenever the compiler encounters a **global variable**, it generates bytecode that performs a lookup in the global variables table (a hash table).

### 3. **Bytecode Instructions for Globals**

When the compiler encounters global variables, it generates the following bytecode instructions:

- **OP_DEFINE_GLOBAL**: When the global variable is defined, this instruction is emitted to store the variable's value in the global table.
- **OP_GET_GLOBAL**: When a global variable is accessed, this instruction is emitted to fetch the value of the variable from the global table.
- **OP_SET_GLOBAL**: When a global variable is updated, this instruction is emitted to update the variable's value in the global table.

### 4. **Storage in the Global Table**

The **global variables** are stored in a **hash table** (usually referred to as the `globals` table). This is a simple key-value store where:

- The **key** is the name of the variable (a string).
- The **value** is the actual value of the global variable (stored as a `Value`, which can hold any Lox data type).

For example:
```c
// Example of storing a global variable
tableSet(&vm.globals, name, value);
```

### 5. **Accessing Global Variables**

Whenever the bytecode is executed to **access** a global variable, the VM performs a lookup in the `globals` hash table to retrieve or update the variable's value.

- **For reading** a global variable, the bytecode for `OP_GET_GLOBAL` uses the variable's name as the key and retrieves its corresponding value from the global table.
- **For writing** to a global variable, the `OP_SET_GLOBAL` bytecode updates the value in the global table.

### Example of Global Variable Access

Consider the following Lox code:
```lox
var globalVar = 10;

fun example() {
  print globalVar; // Accessing a global variable
}

example();
```

Here’s what happens:

1. **At Compilation Time**:
   - When the compiler sees the declaration `var globalVar = 10`, it emits the `OP_DEFINE_GLOBAL` bytecode.
   - When it encounters `print globalVar`, it emits `OP_GET_GLOBAL` to look up `globalVar` by its name.

2. **At Execution Time**:
   - The `OP_DEFINE_GLOBAL` instruction stores the value `10` in the `globals` hash table under the key `"globalVar"`.
   - The `OP_GET_GLOBAL` instruction looks up `"globalVar"` in the `globals` table and retrieves its value `10` to be printed.

### Example Bytecode:

For the above code, the compiler might generate bytecode that looks like this:

1. **Defining a global**:
   - Bytecode:
     ```
     OP_CONSTANT (10)      // Pushes the constant 10
     OP_DEFINE_GLOBAL (globalVar)  // Defines the global variable "globalVar"
     ```

2. **Accessing a global**:
   - Bytecode:
     ```
     OP_GET_GLOBAL (globalVar)  // Fetches the value of "globalVar"
     OP_PRINT           // Prints the value (10)
     ```

### Summary:

- **Global variables** are stored in a **globals hash table** in clox, unlike local variables, which are stored on the stack.
- The **compiler** emits bytecode (`OP_DEFINE_GLOBAL`, `OP_GET_GLOBAL`, `OP_SET_GLOBAL`) to interact with global variables.
- The **VM** uses the **globals table** to store and retrieve global variables by their names during execution.