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

This tells the user how to run the program correctly. The program then exits with status **code 64**, which is often used to indicate a usage error (invalid command-line arguments).




# readFile()

## Function Signature

```
static char *readFile(const char *path)
```

- **static**: This means that readFile() is only accessible within the file where it's defined. It's not visible to other files (helps with encapsulation).

- **char *readFile(const char *path)**: The function returns a pointer to a dynamically allocated string (a char *) and takes one argument, path, which is the file path as a string (a const char *).

## Opening the File

```
FILE *file = fopen(path, "rb");
if (file == NULL)
{
    fprintf(stderr, "Could not open file \"%s\".\n", path);
    exit(74);
}

```

- fopen(path, "rb"): Opens the file at the specified path in binary mode ("rb"), which is used when dealing with raw data (even if the file contains text).

- - **Binary mode** ensures no newline or EOF translation happens on certain platforms (like Windows).

- **Error checking**: If fopen() fails (e.g., the file doesn't exist or can't be accessed), it returns NULL. The function then prints an error message using fprintf() to stderr and exits the program with an exit **code 74**.

**status code 64** - indicates a usage error(invalid command- line arguments)

**status code 74** - indicates that the file cant be opened

##  Moving to the End of the File and Getting File Size

```
fseek(file, 0L, SEEK_END);
size_t fileSize = ftell(file);
rewind(file);
```

- **fseek(file, 0L, SEEK_END)**: Moves the file pointer to the end of the file. This is done to measure the total size of the file.
- **ftell(file)**: Returns the current position of the file pointer (in bytes), which is effectively the size of the file when the pointer is at the end.
- **rewind(file)**: Moves the file pointer back to the beginning of the file. This ensures the file is ready to be read from the start again.

## Allocating Memory to Hold the File Contents

```
char *buffer = (char *)malloc(fileSize + 1);
if (buffer == NULL)
{
    fprintf(stderr, "Not enough memory to read \"%s\".\n", path);
    exit(74);
}
```

- malloc(fileSize + 1): Allocates memory to hold the file contents. The +1 ensures there is space for the null terminator (\0) at the end of the string.
- - The buffer size is fileSize + 1 because we need one extra byte for the null terminator (\0) that will indicate the end of the string.
- **Error checking**: If malloc() fails to allocate memory (e.g., due to insufficient memory), it returns NULL. If this happens, the function prints an error message and exits with error code 74.
##  Reading the File Contents
```
size_t bytesRead = fread(buffer, sizeof(char), fileSize, file);
if (bytesRead < fileSize)
{
    fprintf(stderr, "Could not read file \"%s\".\n", path);
    exit(74);
}
```

**fread(buffer, sizeof(char), fileSize, file)**: Reads fileSize number of bytes from the file and stores them in the buffer. The buffer can hold the entire file since we allocated enough space.
**Error checking**: If fread() reads fewer bytes than expected (bytesRead < fileSize), this could indicate an error, so the function prints an error and exits.

## Null-Terminating the Buffer
```
buffer[bytesRead] = '\0';
```

- **Purpose**: Ensures that the buffer is a valid C string by adding a null terminator (\0) at the end. This allows the buffer to be treated as a regular string in C.

## Closing the File
```
fclose(file);
```

**Purpose**: Closes the file after reading its contents to free system resources.

## Returning the File Contents

```
return buffer;
```

- The function returns the dynamically allocated buffer, which now contains the entire contents of the file as a null-terminated string.