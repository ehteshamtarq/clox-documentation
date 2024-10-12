```
typedef struct
{
    Chunk *chunk;           // The chunk of bytecode to execute.
    uint8_t *ip;            // The instruction pointer (IP).
    Value stack[STACK_MAX]; // Stack Implementation in the VM
    Value *stackTop;        // To keep track of the top of the stack
    Table strings;
    Obj* objects;
} VM;
``` 

## Chunk *chunk; (Bytecode)
- **Purpose**: This field points to a chunk of bytecode, which contains the compiled instructions that the virtual machine executes.
Example:
 
 ```
print 1 + 2;
```
This source code is compiled into bytecode, which might look something like this (simplified):

```
OP_CONSTANT 1
OP_CONSTANT 2
OP_ADD
OP_PRINT
OP_RETURN
```
- These instructions are stored in a chunk. Each instruction corresponds to an operation the VM will perform. The chunk pointer in the VM structure will point to this sequence of bytecode instructions.

## uint8_t *ip; (Instruction Pointer)
- **Purpose**: The instruction pointer keeps track of where the VM is in the bytecode sequence. It moves from one instruction to the next as the VM executes the program.

Example:
When the VM starts executing the bytecode chunk mentioned above, the ip will initially point to the first instruction (OP_CONSTANT 1). As the program executes, the ip will move through each instruction:

- First, it points to OP_CONSTANT 1 and pushes the constant 1 onto the stack.
- Then it moves to OP_CONSTANT 2 and pushes 2 onto the stack.
- Then OP_ADD is executed, which pops the two values, adds them, and pushes the result (3) onto the stack.
Finally, OP_PRINT pops the result from the stack and prints it, followed by OP_RETURN to terminate execution.

## Value stack[STACK_MAX]; (VM Stack)
- **Purpose**: The stack is a fixed-size array that stores intermediate values during execution. The VM uses this stack to store operands for operations, temporary variables, and results of expressions.

Example:

Continuing with the print 1 + 2; example, the stack will be used to store intermediate values:

After OP_CONSTANT 1: The stack will contain 1.
After OP_CONSTANT 2: The stack will contain 1, 2.
After OP_ADD: The stack will contain 3 (since 1 + 2 = 3).
After OP_PRINT: The stack is empty (since 3 has been printed and popped off the stack).

```
Value stack[STACK_MAX];
```

This stack is fixed in size, and STACK_MAX defines the maximum number of values it can hold. In practice, this is a small fixed value because the stack only holds temporary values needed during the execution of bytecode.

## Value *stackTop; (Top of Stack)
- **Purpose**: This pointer points to the next available slot on the stack, effectively keeping track of the current "top" of the stack.

Example:

When OP_CONSTANT 1 is executed, the 1 is pushed onto the stack, and the stackTop is incremented to point to the next empty slot. Similarly:

- Before OP_CONSTANT 1: stackTop points to the first slot.
- After OP_CONSTANT 1: stackTop moves to the second slot.
- After OP_CONSTANT 2: stackTop moves to the third slot.
- After OP_ADD: Two values are popped from the stack, and stackTop moves back to the second slot, where the result (3) is stored.
- The stackTop pointer ensures efficient stack operations (push/pop).

```
Value *stackTop;
```
For example, in a push operation, the VM will do something like:

```
*vm.stackTop = value; // Push value onto the stack.
vm.stackTop++;        // Move stackTop to the next available slot.
```
## Table strings; (String Interning)

**Purpose**: This is a hash table used for string interning, which ensures that identical strings are stored only once in memory, improving efficiency.

Example:
Suppose the program contains the following code:

```
var greeting = "hello";
var anotherGreeting = "hello";
```
Both string literals ("hello") will be interned. The strings hash table is used to store unique strings. When the VM encounters the second "hello", it checks if it already exists in the strings table. If it does, it reuses the same string object rather than creating a new one.

This process saves memory and speeds up string comparison (since comparing interned strings can be done by comparing pointers instead of character-by-character).

The Table is implemented as a hash table, with the string as the key and the pointer to the interned string object as the value.

## Obj* objects; (Garbage Collection)

**Purpose**: This is a pointer to a linked list of all dynamically allocated objects in the VM (e.g., strings, arrays, functions). This list is crucial for garbage collection and memory management.

Example:
When the program dynamically creates a new string, array, or object, the VM allocates memory for that object and adds it to this linked list. For example:

```var name = "Alice";
The string "Alice" will be allocated dynamically, and the VM will add it to the objects list.```

The objects field is used by the garbage collector to track all objects in the VM. When a garbage collection cycle occurs, the VM walks through the objects list and identifies which objects are still in use. Any objects that are no longer referenced by the program are freed.

```
Obj* objects;```
The Obj type is a union-like structure that represents different kinds of objects (e.g., strings, arrays, closures). Every dynamically allocated object (string, function, etc.) has a common header that links it to the next object in the list.

**Putting It All Together: Execution Example**
Let’s walk through the execution of the following Lox code:

```
print "Hello, World!";
```
**Bytecode Generation**: The code is compiled into bytecode:


```
OP_CONSTANT (push "Hello, World!" onto the stack)
OP_PRINT    (pop and print the value)
OP_RETURN   (end execution)

```
The chunk holds this bytecode.

**Instruction Pointer (ip)**: Initially, ip points to OP_CONSTANT, the first instruction in the bytecode sequence.

**Stack Operations**:

**OP_CONSTANT**: The constant "Hello, World!" is pushed onto the stack. The stack now holds one value.
**OP_PRINT**: The string is popped off the stack and printed. The stack is now empty.
String Interning: The string "Hello, World!" is stored in the strings table so that if this string is encountered again, it will reuse the same object.

**Object List**: The dynamically allocated string "Hello, World!" is added to the objects list for garbage collection purposes.

**End of Execution**: When the OP_RETURN instruction is encountered, the VM exits, and garbage collection will eventually clean up the "Hello, World!" string, if no references to it remain.