# C Fundamentals

## Chapters

1. [Data Types](#1-data-types)
2. [Pointers](#2-pointers)
3. [Memory Management](#3-memory-management)
4. [C I/O Operations](#4-io-operations)
5. [C Strings](#5-strings)
6. [C File Operations](#6-file)
7. [C File System](#7-c-file-system-(dirent))

## 1. Data Types

### 1.1. Primitive types

#### 1.1.1. ```int``` type
Simply presents a 16-bit signed integer.
```c
int x = 42;
```

#### 1.1.2. ```char``` type
1-byte char variable.
```c
char c = 'A';
```

Chars can do arithmetic like integer types like:

Advancing through letters.
```c
char c = 'A';
char d = c + 1;                                                 // d = 'B';
```

Extracting integer from text.
```c
char c = '5';
int n = c - '0';                                                // n = 5;
```

Check character.
```c
c >= 'A' && c <= 'Z'                                            // Uppercase condition
c >= '0' && c <= '9'                                            // Digit condition
```

#### 1.1.3. ```float``` & ```double``` types

```double``` type uses 64 bits while ```float``` uses 32 bits, hence ```double``` will have more precision.

```c
float temperature = 36.5f;
double pi = 3.1415926535;
```

#### 1.1.4. Arrays
Arrays store multiple values of the same data type.
```c
int numbers[5];
numbers[0] = 10;
numbers[1] = 20;
...
```

#### 1.1.5. Strings
Strings are represented as an array of characters, always ending with ```'\0'``` (the null terminator)
```c
char str[] = "Hello";
str[0] = 'H';
str[1] = 'e';
...
str[5] = '\0';                                              // Null terminator 
```
The null terminator has integer value ```0```, because of this, C strings are usually 1 character longer than it's actual content.
So strings without a known length limit must end with ```'\0'```.

#### 1.1.6. Structs
Structs can be used to represent custom data types with multiple fields like an object.
```c
struct Student {
    int id;
    char name[50];
    double grade;
};
```

#### 1.1.7. Unsigned types
Unsigned types are used to represent only positive numbers, hence they can store larger values than their signed counterparts. For example, an ```unsigned int``` can store values from 0 to 65535, while a signed ```int``` can store values from -32768 to 32767.
```c
unsigned int u = 65535;
```

### 1.2. Custom types
#### 1.2.1. stdint.h
The ```stdint.h``` header file defines a set of typedefs that specify exact-width integer types, such as ```uint8_t```, ```uint16_t```, ```int32_t```, and ```uint64_t```. These types are useful when you need to ensure that your code behaves consistently across different platforms and compilers, as they provide a guaranteed size for the integer types.
```c
uint64_t l = 18446744073709551615ULL;                   // 64-bit unsigned integer
uint32_t a = 4294967295;                                // 32-bit unsigned integer
uint16_t b = 65535;                                     // 16-bit unsigned integer
uint8_t c = 255;                                        // 8-bit unsigned integer
```
#### 1.2.2. size_t
The ```size_t``` type is an unsigned integer type that is used to represent the size of objects in memory. It is defined in the ```stddef.h``` header file and is typically used in functions that deal with memory allocation, such as ```malloc``` and ```sizeof```. It is often used in loops and array indexing, as it provides a safe way to iterate over arrays and other data structures without risking integer overflow or underflow.
```c
size_t size = sizeof(int);                              // size of int in bytes
int arr[10];
for (size_t i = 0; i < sizeof(arr)/sizeof(arr[0]));      // iterate over array using size_t
``` 
### 1.3. Type casting
Type casting is used to convert a variable from one data type to another. In C, you can use the cast operator ```(type)``` to perform type casting

Common casting scenarios include:
- Converting between integer types of different sizes (e.g., ```int``` to ```long```)
- Converting between signed and unsigned types (e.g., ```int``` to ```unsigned int```)
- Converting between floating-point types (e.g., ```float``` to ```double```)
- Converting between integer and floating-point types (e.g., ```int``` to ```float```)

General syntax for type casting is as follows:
```c
type variable = (type) value;
```

## 2. Pointers
Pointers are special variables that store the memory address of another variable. They are used to manipulate data in memory directly, allowing for more efficient and flexible programming.

### 2.1. Pointer declaration and initialization
To declare a pointer, you use the ```*``` operator. The type of the pointer indicates the type of data it points to. For example, to declare a pointer to an integer, you would write:
```c
int *ptr;
```
To initialize a pointer, you can assign it the address of a variable using the ```&``` operator. For example:
```c
int x = 42;
int *ptr = &x;                                  // ptr now points to the memory address of x
```
Now, the value x can be accessed by two ways:
1. Directly using the variable name:
```c
x = 42;
```
2. Indirectly using the pointer:
```c
*p = 42;                                        // dereferencing the pointer to access the value of x
```

### NOTE: A Quick Way to Verbalize Pointers

We can describe pointer syntax using simple words. This makes pointer code easier to read and reduces confusion.

---

### 1. Normal variable

```c
int x = 42;
```

Read it as:

> **`x` is an `int` variable containing `42`.**

---

### 2. Declaring a pointer

```c
int *pX;
```

Read it as:

> **`pX` is a pointer to an `int`.**

Here:

- `pX` → the **pointer**
- `*pX` → the **value being pointed to**, which is an `int`

> **Note:** `*` is part of the pointer declaration. It means that `pX` is a pointer to the specified type.

---

### 3. Getting an address

```c
int *pX = &x;
```

Read it as:

> **`pX` is a pointer to an `int`, initialized with the address of `x`.**

The `&` operator means:

> **"the address of ..."**

Therefore:

```c
&x
```

means:

> **the address of `x`**

So if:

```c
int x = 42;
int *pX = &x;
```

conceptually:

```text
pX ──────────► x
              42
```

---

### 4. Dereferencing a pointer

Once `*` appears **without a type**, it becomes the **dereference operator**.

```c
int y = *pX;
```

Read it as:

> **`y` takes the value stored at the address that `pX` points to.**

If:

```c
int x = 42;
int *pX = &x;
```

then:

```c
*pX
```

means:

> **the value at the address stored in `pX`**

Therefore:

```c
int y = *pX;
```

is equivalent to:

```c
int y = x;
```

and `y` becomes `42`.

---

### 5. The two meanings of `*`

The `*` symbol has different meanings depending on where it appears.

#### During declaration

```c
int *pX;
```

`*` means:

> **`pX` is a pointer to an `int`.**

#### When used with an existing pointer

```c
*pX
```

`*` means:

> **Access the value at the address stored in `pX`.**

This is called **dereferencing**.

---

### Quick Reference

| Syntax | Read it as |
|---|---|
| `int x` | `x` is an `int` |
| `&x` | the address of `x` |
| `int *pX` | `pX` is a pointer to an `int` |
| `pX` | the address stored in `pX` |
| `*pX` | the value at the address stored in `pX` |
| `int *pX = &x` | `pX` points to `x` |
| `int y = *pX` | `y` gets the value pointed to by `pX` |

### Mental Model

```text
int x = 42;

       x
       │
       │  contains 42
       ▼
   ┌────────┐
   │   42   │
   └────────┘
       ▲
       │
       │ points to
       │
   ┌────────┐
   │   pX   │
   └────────┘
```

Think of it this way:

> **`pX` = the address**  
> **`*pX` = the value at that address**  
> **`&x` = the address of `x`**




### 2.2. Pointer arithmetic
Pointer arithmetic allows you to perform operations on pointers, such as incrementing or decrementing their values. When you increment a pointer, it moves to the next memory location of the type it points to. For example, if you have a pointer to an integer, incrementing it will move it to the next integer in memory:
```c
int arr[5] = {1, 2, 3, 4, 5};
int *ptr = arr;                                 // first element of the array
ptr++;                                          // second element of the array
```

Due to this, arrays can also be initialized using pointers, as the name of the array is a pointer to the first element of the array:
```c
int arr[5] = {1, 2, 3, 4, 5};

int *ptr = arr;                                 // ptr points to the first element of the array

for (int i = 0; i < 5; i++) {
    printf("%d ", *(ptr + i));                  // access each element
}
```

## 2.3. Pointers and functions
Pointers can be used to pass variables to functions by reference, allowing the function to modify the original variable.

Example of passing the pointer to a variable to change that exact variable
```c
void change(int *p) {
    *p = 20;
}

int main() {
    int x = 10;
    change(&x);
    printf("%d\n", x);                              // 20
}
```

## 2.4. Pointers to pointers
Pointers can also point to other pointers, allowing for multiple levels of indirection. This is useful when working with dynamic data structures, such as linked lists and trees.
```c
int x = 42;
int *ptr = &x;                                   // pointer to x
int **ptr_to_ptr = &ptr;                         // pointer to pointer to x
printf("%d\n", **ptr_to_ptr);                    // prints 42
```

This can be used to create dynamic arrays of pointers, where each pointer points to a different data structure or object.


## 3. Memory Management
### 3.1. Dynamic memory allocation
Dynamic memory allocation allows you to allocate memory at runtime using functions like ```malloc```, ```calloc```, and ```realloc``` from the ```stdlib.h``` library. This is useful when you don't know the size of the data you need to store at compile time.

#### 3.1.1. malloc
The ```malloc``` function allocates a specified number of bytes of memory and returns a pointer to the allocated memory. The memory is not initialized, so it may contain garbage values.
```c
int *arr = (int *)malloc(5 * sizeof(int));       // allocate memory for 5 integers
```

#### 3.1.2. calloc
The ```calloc``` function allocates memory for an array of elements, initializes the memory to zero, and returns a pointer to the allocated memory.
```c
int *arr = (int *)calloc(5, sizeof(int));        // allocate memory for 5 integers and initialize to zero
```

#### 3.1.3. realloc
The ```realloc``` function changes the size of previously allocated memory. It can be used to increase or decrease the size of the memory block, while keeping the existing data intact. 

Usage in expanding an array with new input:
```c
int *arr = (int *)malloc(5 * sizeof(int));      // allocate memory for 5 integers
// ... fill the array with data ...
arr = (int *)realloc(arr, 10 * sizeof(int));    // resize the array to hold 10 integers
```

### 3.2. Freeing memory
When you are done using dynamically allocated memory, it is important to free it using the ```free``` function to avoid memory leaks.
```c
free(arr);                                      // free the allocated memory
arr = NULL;                                     // set the pointer to NULL to avoid dangling pointer
```

### 3.3. Memory-related functions
#### 3.3.1. ```memcpy```
The ```memcpy``` function copies a specified number of bytes from a source memory location to a destination memory location. It is defined in the ```string.h``` library.

Syntax:
```c
memcpy(dest, src, n);                           // copy n bytes from src to dest
```

Copying whole array:
```c
char src[10] = "Hello";
char dest[10];
memcpy(dest, src, sizeof(src));                 // copy the contents of src to dest
printf("%s\n", dest);                           // prints "Hello"
```
Copying part of an array:
```c
char src[10] = "Hello";
char dest[20];
memcpy(dest, src, 5);                           // copy the first 5 bytes of src to dest
dest[5] = ' ';
dest[6] = 'W';
dest[7] = 'o';
dest[8] = 'r';
dest[9] = 'l';
dest[10] = 'd';
dest[11] = '\0';                                // null-terminate the string
printf("%s\n", dest);                           // prints "Hello World"
```

The char array visualized in memory:

| index  | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 |
|--------|---|---|---|---|---|---|---|---|---|---|----|----|
| src    | H | e | l | l | o |\0 |   |
| dest   | H | e | l | l | o |   | W | o | r | l | d  | \0 |

#### 3.3.2. ```memset```
The ```memset``` function sets a specified number of bytes in a memory block to a given value.

Syntax:
```c
memset(ptr, value, n);                          // set n bytes of memory at ptr to value
```

Usage:
```c
char arr[10];
memset(arr, 0, sizeof(arr));                    // set all bytes in arr to 0
```

#### 3.3.3. ```memmove```
The ```memmove``` function is similar to ```memcpy```, but it is used when the source and destination memory blocks overlap. It ensures that the data is copied correctly without corruption.

Syntax:
```c
memmove(dest, src, n);                          // move n bytes from src to dest
```

Usage:
```c
char str[] = "Hello, World!";
memmove(str, str + 7, 5);                       // move "World" to the beginning of the string
str[5] = '\0';                                  // null-terminate the string
printf("%s\n", str);                            // prints "World"
```

The char array visualized in memory:
| idx  	| 0 	| 1 	| 2 	| 3 	| 4 	| 5  	| 6 	| 7 	| 8 	| 9 	| 10 	| 11 	| 12 	| 13 	|   	|
|------	|---	|---	|---	|---	|---	|----	|---	|---	|---	|---	|----	|----	|----	|----	|---	|
| src  	| H 	| e 	| l 	| l 	| o 	| ,  	|   	| W 	| o 	| r 	| l  	| d  	| !  	| \0 	|   	|
| dest 	| W 	| o 	| r 	| l 	| d 	| \0 	|   	|   	|   	|   	|    	|    	|    	|    	|   	|

Here, the beginning of the substring "World" is 7 positions ahead of the beginning of the string, so we use ```str + 7``` as the source pointer.
Then we use the source itself as the destination pointer, since we want to move the substring to the beginning of the string.
After that, we null-terminate the string at index 5, which is the new end of the string.

### 3.4. Usage examples
Automatically resizing an array with user input:
```c
int size = 1;
int *arr = (int *)malloc(size * sizeof(int));                   // allocate memory for 1 integer
int input;
while (1) {
    printf("Enter a number (or -1 to stop): ");
    scanf("%d", &input);
    if (input == -1) {
        break;
    }
    if (size == 1) {
        arr[0] = input;                                         // store the first input
    } else {
        arr = (int *)realloc(arr, size * sizeof(int));          // resize the array
        arr[size - 1] = input;                                  // store the new input
    }
    size++;
}
```
## 4. I/O Operations
### 4.1. Standard Input and Output
C provides standard input and output functions through the ```stdio.h``` library. The most commonly used functions are ```printf``` for output and ```scanf``` for input.
#### 4.1.1. ```printf```
The ```printf``` function is used to print formatted output to the console. It takes a format string and a variable number of arguments, which are inserted into the format string at the specified placeholders.
```c
int x = 42;
printf("The value of x is: %d\n", x);                           // prints "The value of x is: 42"
```
#### 4.1.2. ```scanf```
The ```scanf``` function is used to read formatted input from the console. It takes a format
string and a variable number of pointers to variables, which are filled with the input values.
```c
int x;
printf("Enter a number: ");
scanf("%d", &x);                                                // reads an integer from input and stores it in x
```
#### 4.1.3. Format specifiers
Format specifiers are used in ```printf``` and ```scanf``` to indicate the type of data being printed or read. Some common format specifiers include:
- ```%d``` for integers
- ```%f``` for floating-point numbers
- ```%c``` for characters
- ```%s``` for strings
- ```%p``` for pointers
- ```%x``` for hexadecimal integers

For character and string outputs, 

### 4.2. Other I/O functions
#### 4.2.1. ```getchar``` and ```putchar```
The ```getchar``` function reads a single character from standard input, while the ```putchar``` function writes a single character to standard output.

Print first character of input:
```c
char c;
printf("Enter a character: ");
c = getchar(); 
putchar(c); 
```

Print all characters of input until newline or EOF:
```c
char c;
c = getchar(); 
while (c != '\n' && c != EOF) { 
    putchar(c);                                             // write each character to output
}
```

Get all characters of input until newline or EOF and store them in a string:
```c
char str[100];
int i = 0;
char c = getchar(); 
while (c != '\n' && c != EOF && i < sizeof(str) - 1) { 
    str[i++] = c;                                           // store each character in the string
    c = getchar();                                          // read next character
}
str[i] = '\0';                                              // null-terminate the string
```

#### 4.2.2. ```fgets```
The ```fgets``` function reads a line of text from a specified input stream and stores it in a string.

This function reads the line as is, including spaces and tabs, until a newline character or the end of the file is reached. The newline character is included in the string, and the string is null-terminated.

Syntax:
```c
char *fgets(char *str, int n, FILE *stream);
```

Read a single line of input:
```c
char str[100];
fgets(str, sizeof(str), stdin);                         // read a line of text from standard input
```

Loop to read multiple lines of input until EOF:
```c
char str[100];
while (fgets(str, sizeof(str), stdin) != NULL) {
    printf("%s", str);                                  // print each line
}
```

This can be changed to use any string as the breaking condition, for example, to stop reading when the user inputs "exit":
```c
char str[100];
while (1) {
    fgets(str, sizeof(str), stdin); 
    if (strcmp(str, "exit\n") == 0) { 
        break; 
    }
    printf("%s", str); 
}
```

Or when the user wants to turn a line into a formatted string using ```sscanf```:
```c
char str[100];
while (fgets(str, sizeof(str), stdin) != NULL) {
    int x, y;
    if(sscanf(str, "%d %d", &x, &y) == 2) {
        int z = x + y;                          // Parse two integers from the input line
        printf("Sum: %d\n", z);                 // Print the sum of the two integers 
    };                                                                                              
}
```

#### 4.2.3. ```fprintf```
The ```fprintf``` function is used to write formatted output to a specified output stream, such as a string. It works similarly to ```printf```, but instead of printing to the console, it writes to the specified string.

Syntax:
```c
int fprintf(FILE *stream, const char *format, ...);
```

Copy formatting from printf to a string:
```c
char buffer[100];
int x = 42;
fprintf(buffer, "The value of x is: %d\n", x);                  // write formatted output to buffer
printf("%s", buffer);                                           // print the contents of buffer
```

#### 4.2.4. ```sprintf```
The ```sprintf``` function is used to write formatted output to a string. It works similarly to ```printf```, but instead of printing to the console, it writes to the specified string.

Syntax:
```c
size_t sprintf(char *str, const char *format, ...);
```

Usage:
```c
char buffer[100];
int x = 42;
sprintf(buffer, "The value of x is: %d\n", x);                  // write formatted output to buffer
printf("%s", buffer);                                           // print the contents of buffer
```

## 5. Strings
### 5.1. String initialization
Strings can be initialized in two ways:
1. Using a string literal:
```c
char str1[] = "Hello, World!";                                 // automatically adds null terminator
char *str = "Hello, World!";                                   // pointer to string literal (read-only)
```
2. Using an array of characters:
```c
char str2[] = {'H', 'e', 'l', 'l', 'o', ',', ' ', 'W', 'o', 'r', 'l', 'd', '!', '\0'}; // manually adds null terminator
```

### 5.2. String functions
#### 5.2.1. Length and comparison
- ```strlen``` returns the length of a string (excluding the null terminator).
```c
char str[] = "Hello, World!";
size_t length = strlen(str);                                    // length = 13
```

- ```strcmp``` compares two strings and returns an integer indicating their relationship.

The comparison is done lexicographically, meaning that the strings are compared character by character. The function returns `-1` if the first string is less than the second string, `0` if they are equal, and `1` if the first string is greater than the second string.
```c
char str1[] = "Hello";
char str2[] = "World";
int result = strcmp(str1, str2);                                // result < 0 because str1 goes before str2
```

- ```strncmp``` compares the first n characters of two strings and returns an integer indicating their relationship.
```c
char str1[] = "Hello";
char str2[] = "Helium";
int result = strncmp(str1, str2, 3);                            // result = 0 because the first 3 characters are equal
```

#### 5.2.2. Search string
- ```strchr``` finds the **first occurrance of a character** in a string and returns the pointer to it.
```c
char str[] = "Hello, world!";
char *ptr = strchr(str, ',');                                   // Pointer to the ',' character
ptr = &str[5];                                                  // Now points to the 6th character
```

- ```strrchr``` finds the **last occurance of a character** in a string and returns the pointer to it.
```c
char str[] = "Hello, world!";
char *ptr = strrchr(str, 'l');                                  // Pointer to the final 'l' character
ptr = &str[10];                                                 // Now points to the 11th character
```

- ```strstr``` finds the **first occurance of a substring** and returns the pointer to the first character of the substring or null.
```c
char str[] = "Hello, world!";
char *ptr = strstr(str, "world");                               // Pointer to the 'w' in 'world'
printf("%s", ptr);                                              // Prints 'world!'
printf("%c", *ptr);                                             // Prints 'w'
```

- ```strpbrk``` finds the **first occurance of a character** the matches any character in the condition string.
```c
char str[] = "Hello, world!";
char *ptr = strpbrk(str, "aeiou");                               // Pointer to the 'e' in 'Hello'
```

- ```strspn``` looks for the length of the initial segment of a string that **consists only of characters present in a second string**. 
```c
char str1[] = "12345abcd678";
char str2[] = "0123456789";
size_t len = strspn(str1, str2);                                // Returns 5 for the '12345' part
```

- ```strcspn``` looks for the length of the initial segment of a string that **does not consist of characters present in a second string**.
```c
char str[] = "Hello, world!\n";
size_t idx = strcspn(str, "\r\n");                                // Finds the length of the text before '\n'
str[idx] = '\0';                                                  // Can use to null-terminate the string
```

#### 5.2.3. String manipulation
- ```strcpy``` copies a string from a source to a destination, including the null terminator.
```c
// Simple usage
char s1[20];
strcpy(s1, "Hello");                                            // Copy to s1
printf("%s", s1);                                               // Prints to hello

// Replacing part of a String
char str[] = "Hello, world!";
char *p = strstr(str, "world");
char name[] = "Alice!";
strcpy(p, name);                                                // String is now "Hello, Alice!"
```

- ```strncpy``` copies up to n characters from a source to a destination, pads with null terminator if source is shorter than n.
```c
char str[] = "Hello, world!";
char *p = strstr(str, "world");
char name[] = "Alice";
strncpy(p, name, 5);                                            // String is now "Hello, Alice!"
```

- ```strcat``` appends source string to the destination string, add new null terminator at the end.
```c
char msg[20] = "Hello";
strcat(msg, " World");                                          // msg becomes "Hello World"
```

- ```strtok``` breaks source string into tokens, modifies source strings by inserting null terminator at each delimiter.
```c
char data[] = "a,b,c";
char *tokens[10];                                               // Capacity for up to 10 tokens
int count = 0;

char *token = strtok(data, ",");
while (token != NULL && count < 10) {
    tokens[count++] = token;                                    // Store pointer to the current token
    token = strtok(NULL, ",");
}
```

## 6. File
### 6.1. C file basics
A file is a collection of data stored on a storage device. In C, files can be opened, read, written, and closed using functions from the ```stdio.h``` library. Files can be opened in different modes, such as read, write, or append.

#### 6.1.1. File pointers
A file pointer is a pointer to a structure that contains information about a file, such as its name, mode, and current position. File pointers are used to perform operations on files.

```c
FILE *fp;                                                      // Declare a file pointer
fp = fopen("example.txt", "r");                                // Open a file in read mode
```

File paths are relative to the current working directory of the program. If the file is not found, ```fopen``` returns NULL.

#### 6.1.2. File modes
The mode in which a file is opened determines the operations that can be performed on it. Common file modes include:
- ```"r"``` - Read mode: Opens a file for reading.
- ```"w"``` - Write mode: Opens a file for writing. If the file already exists, its contents are truncated. If the file does not exist, a new file is created.
- ```"a"``` - Append mode: Opens a file for writing at the end of the file. If the file does not exist, a new file is created.
- ```"r+"``` - Read and write mode: Opens a file for both reading and writing. The file must exist.
- ```"w+"``` - Read and write mode: Opens a file for both reading and writing. If the file already exists, its contents are truncated. If the file does not exist, a new file is created.
- ```"a+"``` - Read and append mode: Opens a file for both reading and writing. The file is created if it does not exist. The writing will always be done at the end of the file, regardless of the current position of the file pointer.

Example usage of file modes:
```c
FILE *fp;
fp = fopen("example.txt", "r");                                // Open a file in read mode
if (fp == NULL) {
    perror("Error opening file");
    return -1;
}

fp = fopen("example.txt", "w");                                // Open a file in write mode
if (fp == NULL) {
    perror("Error opening file");
    return -1;
}
```

#### 6.1.3. Closing files
When you are done working with a file, it is important to close it using the ```fclose``` function to free up system resources.
```c
fclose(fp);                                                    // Close the file
```

#### 6.1.4. File I/O functions
1. ```fgetc``` - Reads a single character from a file.
```c
FILE *fp;
fp = fopen("example.txt", "r"); 
int ch = fgetc(fp);                                            // Read a character from the file
if (ch != EOF) {
    putchar(ch);                                               // Print the character to the console
}
```

2. ```fputc``` - Writes a single character to a file.
```c
FILE *fp;
fp = fopen("example.txt", "w");
fputc('A', fp);                                                // Write a character to the file
```

3. ```fgets``` - Reads a line of text from a file.
```c
FILE *fp;
fp = fopen("example.txt", "r");
char buffer[100];
if (fgets(buffer, sizeof(buffer), fp) != NULL) {               // Read a line of text from the file
    printf("%s", buffer);                                       // Print the line to the console
}

while (fgets(str, sizeof(str), stdin) != NULL) {                // Read ALL lines of text from the file
    printf("%s", str);                                          // Print each line to the console
}
```

4. ```fputs``` - Writes a line of text to a file.
```c
FILE *fp;
fp = fopen("example.txt", "w");
fputs("Hello, World!\n", fp);                                   // Write a line of text to the file
```

### 6.2. ```popen()```

```popen()``` is a C function that lets your program run another program or command and communicate through a ```FILE *``` stream.

```c
FILE *popen(const char *command, const char *mode);
```

The ```command``` here is the input console command, while the ```mode``` is the interaction mode, same way with files with read, write, etc.

#### 6.2.1. Read mode

Read mode takes in the output of the terminal command into a stream to be read via ```fgets()```.

Example usage for read mode:
```c
FILE *fp = popen("ls", "r");        // Runs ls and initializes a FILE stream to its output

char line[256];                     // Line buffer to read each line
char out[4096];                     // Output string containing the entire output
while(fgets(line, sizeof(line), fp) != NULL) {
    strcat(out, line);              // Read each line and combine it into the output string.
}
pclose(fp);
```

#### 6.2.2. Write mode

Write mode on the other hand, writes the input strings from the C code as input to be written into the ```stdin```.

Example usage:
```c
FILE *fp = popen("./add", "w");        // Run an add program
fprintf(fp, "1 2\n");                  // Write input to stdin via C code
pclose(fp);                            // Close the stream when done
```

## 7. C file system (dirent)

In C, the user can interact with the file system using the ```dirent.h``` library.
For proper usage, a `dirent` struct is called as the starting point.

```c
struct dirent {
    ino_t d_ino;                    // inode number
    off_t d_off;                    // Offset
    unsigned short d_reclen;        // Record length
    unsigned char d_type;           // Entry type
    char d_name[];                  // Entry name
}
```

The file name and file type can be accessed for business logic.
- `d_name` can be accessed as a simple string.
```c
char filename[] = entry->d_name;
```
- `d_type` can usually take on one of these two values: `DT_REG` or `DT_DIR`
```c
if (entry->d_type == DT_DIR) {          // If this is a directory
    printf("This is a directory");
}

if (entry->d_type == DT_REG) {          // If this is a file
    printf("This is a file");
}
```

### 7.1. `opendir()`

Similar to `fopen()`, this function returns a `DIR *` object which can be used as the base for other file system interactions. Returns `NULL` on failure.

```c
DIR *dir = opendir(".");                // Open the current directory.
```

### 7.2. `readdir()`

`readdir()` uses the `DIR *` object retrieved via `opendir()`, returning a `struct dirend` singly linked list that we talked about earlier.
```c
DIR *dir = opendir("."); 
struct dirent *entry = readdir(dir);            // Get the first element of the linked list
printf("%d", entry->d_name);                    // Print the filename
```

For each time `readdir()` is called, the next element if the file list is returned, so we use a `while` loop to list all files like so.
```c
DIR *dir = opendir("."); 
struct dirent *entry = readdir(dir);            // Get the first element of the linked list
while ((entry = readdir(dir)) != NULL) {
    printf("%s\n", entry->d_name);
}
```

## 8. Bit manipulation
### 8.1. Accessing the bytes of a variable

To be able to access the bytes of a variable with a datatype, we cast the variable to `unsigned char *`, returning an array of bytes representing that variable. 

Since using `unsigned char *` returns a pointer to the first element of the byte array while `unsigned char` only does a direct value conversion, the pointer version is used.

```c
float f = 3.14f;                            // Initialize the variable value
unsigned char *p = (unsigned char *)&x;     // Cast to unsigned char
```

After this, each byte can be accessed via `p[0]`,`p[1]`, `p[2]`, etc. A for loop can be used to do such a task.
```c
for (size_t i = 0; i < sizeof(x); i++) {
    printf("%02X ", p[i]);                  // Print out as 2 characters
}
```

The individual bit representation of each byte can also be done.
```c
for (int i = 7; i >= 0; i--) {
    printf("%d", (p[0] >> i) & 1);          // Print out all bits on a row
}
```


