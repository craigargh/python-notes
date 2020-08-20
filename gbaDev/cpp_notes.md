# C++ Notes

C++ is a compiled language that is built on top of the C programming language.


## 1. Basics

### Hello World and main()

A basic hello wolrd program in C++ looks like this:

```cpp
#include <iostream>

int main() {
	std::cout << "Hello world\n";
}
```

The `#include` keyword is used to import libraries into your program. In this case the `<iostream>` is a standard library that is used for input and output.

Functions in C++ start have four parts:
1. The return type
2. The function name
3. Arguments
4. A body

The `main()` function is a special function that is called when you program starts. It doesn't return a value, but has its return type set to `int` by convention. The `main()` function also does not take any arugments.

Strings in C++ are a sequence of characters surrounded by a pair of speech marks e.g. `"This is a string"`.

To output a string to the terminal's stdout, the string is piped using the `<<` operator to the `std::cout` command.


Each statement in C++ is terminated using a `;` (semi-colon).

By convention the C++ file extension is `.cpp`


### Compiling

C++ is a compiled language. To compile a program:

```
g++ file_name.cpp
```

This will generate compiled code in a file called `a.out`, which can be ran using the following command:

```
./a.out
```

The `-o` flag is used to set the name out the output file.

```
g++ file_name.cpp -o my_program
./my_program
```

### Comments

Comments are notes written in code that do not affect the code's execution. They are mainly used for writing explanations of the code for yourself and other developers.

C++ has two types of comments: single-line comments and multi-line comments.


Single line comments start with `//`

```cpp
// This is a comment
```

Multi-line comments start with `/*` and end with `*/`

```cpp
/*
This is also a comment
on multipe lines
*/
```

## Variables and Maths Operations

### Variables

Variables are labels that reference pieces of data. In C++ variables must have a type when they are declared.

Variable have three things:
1. A type
1. A name
1. A value


```cpp
int points = 0;
```

Here the variable type is `int`. the name is `points` and the value is `0`.


It is possible to define a variable without a value:

```cpp
int points;
```

The value of the variable can then be set/initialised later in the program:

```cpp
points = 0;
```

### Maths Operations




