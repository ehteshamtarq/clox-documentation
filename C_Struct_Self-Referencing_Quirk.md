Yes, that's a common "quirk" in C due to how it handles struct definitions. In C, when you define a `struct`, the entire declaration of the struct (including its fields) has to be complete before you can reference it by its typedef alias. So, when you're in the middle of defining the `struct`, you can’t use the `typedef` alias because the compiler hasn’t finished processing the full declaration yet.

To work around this, you can give the `struct` itself a name (a *tag*), and use that tag as the type within the `struct` for any self-references or forward declarations.

Here’s an example:

```c
typedef struct Compiler {
    struct Compiler *parent;  // Use the struct tag (Compiler) instead of the typedef alias
    int field1;
    // other fields...
} Compiler;
```

In this example:
- The `typedef` alias (`Compiler`) can't be used until the entire struct definition is complete.
- Inside the `struct`, you use `struct Compiler` to reference the `Compiler` type.

Yes, C can be a little weird with this, but it's just one of those language quirks you get used to. Other languages (like C++) handle this more gracefully with their type systems!