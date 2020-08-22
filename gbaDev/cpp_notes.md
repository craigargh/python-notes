# C++ Notes

C++ is a compiled language that is built on top of the C programming language.


Additional topics to research:
- Arrays


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

The standard maths operations are all available in C++. These include:

- addition `+`
- subtraction `-`
- multiplication `*`
- division `/`
- module `%`
- brackets `()`

Maths operators can be used to calculate the value of a variable:

```cpp
int points = 10;

points = points + 1;
points = points * 2;
```

To output the value of a variable

```cpp
std::cout << points << "\n";
```

### Variables and cout

When outputting to standard out, data is displayed on a single line unless the `"\n"` special character is used.

For example
```cpp
std::cout << "Hello "
std::cout << "my "
std::cout << "darling\n"

```

Will output
```
Hello my darling
```

Alternatively chaining can be used to write this on a single line:

```cpp
int score = 10;

std::cout << "Score: " << score << " points";
```

### User Input

Similar to `cout`, the `cin` command can be used for user data input from the command line.

For example:

```cpp
int score;

std::cout << "Enter your score:"
std::cin >> score;
```

The `cin` function can be used to set the value of a variables using the `>>` pipe. The data type of the value will be cast to the type of the variable [CHECK THIS].

### Doubles

A `double` is a data type that can store decimal points. 

For example 

```cpp
double height = 32.6;
```

## Conditionals and Logic


An if statement allows you to control the flow of your programs based on conditions.

The basic structure of an if statement includes:
- The `if` keyword
- A condition in brackets
- A body

In other words:

```cpp
if (condition){
    // run this;
}
```

The code in the body will only execute if the condition results in `true`.

Here's an example:

```cpp
#include <iostream>

int price;
std::cout << "Enter the price: "
std::cin >> price;

if (price > 10.0){
    std::cout << "\nToo expensive\n"
}

if (price <= 10.0){
    std::cout << "\nYou can afford this\n"
}
```

The comparison operators in C++ are:
- equal to `==`
- not equal to `!=`
- greater than `>`
- less than `<`
- greater than or equal to `>=`
- less than or equal to `<=`

The `else` clause when used with an if statement will execute code when the `if` condition is `false`.

Here's the above example rewritten with an `else`:


```cpp
#include <iostream>

int price;
std::cout << "Enter the price: "
std::cin >> price;

if (price > 10.0){
    std::cout << "\nToo expensive\n"
} else {
    std::cout << "\nYou can afford this\n"
}
```

The `else if` statement can be used to add additional conditions to an `if` statement:


```cpp
#include <iostream>

int price;
std::cout << "Enter the price: "
std::cin >> price;

if (price > 10.0){
    std::cout << "\nToo expensive\n"
} else if(price == 10.00){
    std::cout << "\nExactly the right amount\n"
} else {
    std::cout << "\nYou can afford this\n"
}
```

### Switch Statements

The `switch` statement are similar to `if` statements, however they only allow you to check if a value matches.

The benefit of a `switch` statements are that they can check if multiple values match with a simpler syntax than `if` statements.

The structure of a `switch` statement:

```cpp
switch(number) {
    case 1:
      // do this;
      break;
    case 2:
      // do this;
      break;
    default:
      // do this;
      break;
}
```

Here's an example with actual values:

```cpp
#include <iostream>

std::cout << "Enter a number (1-5): "

swtich (number) {
    case 1:
      std::cout << "One\n";
      break;
    case 2:
      std::cout << "Two\n";
      break;
    case 1:
      std::cout << "Three\n";
      break;
    case 1:
      std::cout << "Four\n";
      break;
    case 1:
      std::cout << "Five\n";
      break;
    default:
       std::cout << "Number want not between 1 and 5\n";
       break;
}
```


### Logical operators

C++ doesn't have boolean type, instead it just uses the `int` type with values `0` and `1`.

To make this easier the values `true` and `false` evaluate to `1` and `0` respectively.

Boolean operators allow the combination of Boolean values.

The and operator (`&&`) evaluates to `true` only if both boolean values are `true`. If either or both are `false`, then it will evaluate to `false`.

The or operator (`||`) evaluates to `true` if either boolean values are `true`. Only if both are `false`, then it will evaluate to `false`.

The not operator (`!`) reverses the value of Boolean values. It will change `true` to `false` and `false` to `true`.

## Loops

Loops are used to repeat code. There are two main types of loop in C++: while loops and for loops.

A while loop is similar to an if statement. It will run if a condition is met. The difference is that an if statement will only run code once at most, whereas as while loop can repeat it many times.

The structure of a while loop is similar to an if statment:

```cpp
while (condition){
    // do stuff here;
}
```

Here's an example program:

```cpp
#include <iostream>


int main(){
    int sheep;

    std::cout << "How many sheep? ";
    std::cin >> sheep;
    std::cout << "\n";

    int count = 0;

    while (count < sheep){
        std::cout >> "Baa!\n";
        count ++;
    }
}
```

For loops are different to while loops in that they will repeat a certain number of times.

The basic stucture is:

```cpp
for (int i = 0; i < 20; i++){
    // do something here;
}
```

Here's the earlier program rewritten with a for loop instead:

```cpp
#include <iostream>

int main(){
    int sheep;

    std::cout << "How many sheep? ";
    std::cin >> sheep;
    std::cout << "\n";

    for (int i = 0; i < sheep; i++){
        std::cout >> "Baa!\n";
    }
}
```

The operators in the for loop setup can be changed to decrement instead of increment:

```cpp
#include <iostream>

int main(){
    int sheep;

    std::cout << "How many sheep? "
    std::cin >> sheep;
    std::cout << "\n";

    for (int i = sheep; i > 0; i--){
        std::cout >> "Baa!\n";
    }
}
```

## Errors

There are different categories of errors with your code:
- Compile-time error
- Link-time error
- Run-time error
- Logic error

Compile-time errors happen when an error is detected during compiling. These errors are caused by:
- syntax errors: for example forgetting a `;`
- type errors: for example trying to set a string value for an int variable

Link-time errors are caused when functions or libraries are missing or can't be found.

Run-time errors happen during the execution of a program. The program will compile fine, but the error will occur after the program starts running. For example a division by 0 is a run-time error.

Logic errors are where a mistake in the programmers' code makes the program behave in unintended ways during run-time.


## Vectors

Vectors are used to store a collection of data values within a single variable. Collections use indexes to reference their values. Indexes start at 0. All data in a vector is the same data type.

To use vectors, you must include the vector header file:

```cpp
#include <vector>
```

Vectors are defined with a data type:

```cpp
#include <vector>

std::vector<int> ages; 
```

Like other variables, you can initialise a vector with values when you create it:

```cpp
#include <vector>

std::vector<int> ages = {1, 6, 3};
```

Alternatively you can create an empty vector with a specified size:

```cpp
#include <vector>

std::vector<int> ages(3);
```

The values in vectors are accessed using indexes:

```cpp
#include <iostream>
#include <vector>

std::vector<int> ages = {1, 6, 3};

std::cout << ages[0] << "\n";
std::cout << ages[2] << "\n";
```


Unlike arrays, you can easily resize a vector. To add an item to the end of a vector the `push_back()` method is used:

```cpp
#include <iostream>
#include <vector>

std::vector<int> ages = {1, 6, 3};

ages.push_back(2);
```

To remove an item from the end of a vector, the `pop_back()` method is used:

```cpp
#include <iostream>
#include <vector>

std::vector<int> ages = {1, 6, 3};

int last_age = ages.pop_back();
```

The `.size()` method is used to get the length of a vector:

```cpp
#include <iostream>
#include <vector>

std::vector<int> ages = {1, 6, 3};

int length = ages.size();
```

You use a for loop to iterate over items in a vector:

```cpp
#include <iostream>
#include <vector>

std::vector<int> ages = {1, 6, 3};

for (int i = 0; i < ages.size(); i++){
    std::cout << ages[i];
}
```


## Functions


Data types:
- double
- int
- bool
- char
- std::string
- std::vector