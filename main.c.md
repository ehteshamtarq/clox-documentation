## 1. int main(int argc, const char *argv[]):
- The main function takes two parameters:
- - argc: The number of arguments passed to the program, including the program's name.
- - argv[]: An array of strings (character pointers) representing the arguments. argv[0] is the name of the program, and any other arguments follow.
- The program operates based on how many arguments are passed (argc).

## 2. Argument Handling

### if (argc == 1):
- This means no additional arguments were passed (only the program name), so the program enters **REPL mode (Read-Eval-Print Loop)** by calling repl().
REPL is a loop where the user can enter code or commands interactively, and the VM evaluates and responds to each input.
### else if (argc == 2):
- This means one argument was passed in addition to the program name (argv[1]). The program interprets this as a path to a file and calls runFile(argv[1]) to run the file.
### else:
- If more than one argument (in addition to the program name) is passed, the program displays an error message:

``` 
Usage: clox [path]
```

This tells the user how to run the program correctly. The program then exits with status code 64, which is often used to indicate a usage error (invalid command-line arguments).

**status code 6** - indicates a usage error(invalid command- line arguments)



# readFile()

## Function Signature

```
static char *readFile(const char *path)
```

- **static**: This means that readFile() is only accessible within the file where it's defined. It's not visible to other files (helps with encapsulation).

- **char *readFile(const char *path)**: The function returns a pointer to a dynamically allocated string (a char *) and takes one argument, path, which is the file path as a string (a const char *).

