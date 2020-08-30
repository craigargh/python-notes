# C++ Notes

C++ is a compiled language that is built on top of the C programming language.


Additional topics to research:
- Create a section about data types
    - type casting
    - std::string type
    - bool type
    - structs
    - hex
    - binary
- Integer size and memory allocation (e.g. how are 16 bit integers stored in memory?)
- Class inheritance
- Memory allocation
- Arrays
- Char data type
- extern
- Interrupts 


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

Here are some of the data types can be used in C++:
- double
- int
- bool
- char
- std::string

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

Functions are reusable blocks of code. The main elements of a function in C++ are:
- return type of the function
- function name
- parameters/arguments
- function body

Here's the basic structure of a funciton:

```cpp
return_type name(type param1, type param2){
    // do some stuff
    return new_value
}
```

For example a function that calculates a percentage from two doubles and returns the result as a double:

```cpp
double percentage(double num1, double num2){
    return num1/num2;
}
```

Functions do not need to return a value, in which case their return type should be set to `void` and the `return` keyword can be omitted. 

```cpp
#include <iostream>

void say_hello(std::string name){
    std::cout << "Hello " << name;
}
```

Void functions are also sometimes called subroutines.

### Built-in Functions

C++ includes several built-in functions in its standard library.

Built-in functions can be added to programs by including their header file, for example the `cmath` header is used to add the `sqrt()` (square root) function:

```cpp
#include <cmath>

std::cout << sqrt(16);
```

### Declaring and Defining Functions

Functions can be declared before the body of their code is defined. This is useful for organising code so that the main function is near the top. For example

```cpp
#import <iostream>


int age(int birth_year, int current_year);

int main(){
    std::cout << age(1972, 2020);
}

int age(int birth_year, int current_year){
    return current_year - birth_year;
}
```

### Variable Scope

Variables that are defined outside of function can be accessed within functions. These variables are said to have global scope:

```cpp
#include <iostream>


int age = 23;


int main(){
    std::cout << age;
}
```

Variables that are defined within functions can only be accessed within that function. They cannot be accessed by other functions, unless their values is returned to that function.

```cpp
#include <iostream>

int get_age(){
    return 23;
]

int main(){
    std::cout << get_age();
}

```

### Separating Functions into Files

As a program grows larger it is important to split it into multiple files so that it is easier to manage.

In C++ you can split the functions into several files. However, any functions that are used by main need to be declared before they are used. 

In this example the `age()` function is declared in `main.cpp`, but defined in `age.cpp`:

`main.cpp`

```cpp
#include <iostream>


int age(int birth_year, int current_year);

int main(){
    std::cout << age(1972, 2020);
}
```

`age.cpp`

```cpp
int age(int birth_year, int current_year){
    return current_year - birth_year;
}
```

When compiling both file names need to be passed to the compiler:

```bash
g++ main.cpp age.cpp
```


A better way to split functions into several files is to use header files. The header file will declare the function, while the definition is still done in a `.cpp` file. This makes organising functions easier as the `main.cpp` doesn't need to declare all of the functions defined in other files.

The header file should be imported into the `main.cpp` so that it can read the declarations.

Here's the same example split to use a headers file:

```cpp
#include <iostream>
#include "age.hpp"

int main(){
    std::cout << age(1972, 2020);
}
```

`age.hpp`

```cpp
int age(int birth_year, int current_year);
```


`age.cpp`

```cpp
int age(int birth_year, int current_year){
    return current_year - birth_year;
}
```

The header file can use a `.hpp` or `.h` file extension, though `.h` is usually used by standard C programs.

### Inline Functions

Inline functions are functions that are defined in the header file. Their code is substitued for where they are called when the program is compiled. This can slow down compilation times, but increase execution times when used correctly.

Inline functions use the `inline` keyword. They should only be defined in the header file and not defined in `.cpp` files.


`age.hpp`

```cpp
inline 
int age(int birth_year, int current_year){
    return current_year - birth_year;
}
```

### Default Arguments

Default arguments can be declared in functions. They will set the default argument values passed to functions when the function call does not set them.

For example:

```cpp
int age(int birth_year, int current_year=2020){
    return current_year - birth_year;
}

int main(){
    std::cout << age(1952);
}
```

When splitting functions into header files, the default arguments should be included in the declaration in the header file, but not in the definition in the `.cpp` file.


### Function Overloading and Templates

Function overloading is useful when you want a functions with the same name to work for different types. 


```cpp
int age(int birth_year, int current_year){
    return current_year - birth_year;
}

double age(double birth_year, double current_year){
    return current_year - birth_year;
}
```


Alternatively, you can use function templates, which allows you to define a function once that can be used with multiple types. Function templates should be included in header files.

```cpp
template <typename T>
T age(T birth_year, T current_year){
    return current_year - birth_year;
}
```

## Classes and Objects

Classes are user defined data types. You can define attributes and methods that can be used by your classes.

The main components of a class are:

- The `class` keyword
- A name
- attributes
- public methods
- private methods


Defining a basic class:

```cpp
class Dog{
    int age;

public:
    void bark(){
        std::cout << "Bark!";
    }

private:
    void run(){
        std::cout << "Running";
    }
};
```

When splitting class declaration and defintion into header and `cpp` files the syntax for classes is slightly different.

The header would look like this:

```cpp
class Dog{
    int age;

public:
    void bark();

private:
    void run();
};
```

While the `cpp` defintion would only define the behaviour of the functions:

```cpp
void Dog::bark(){
    std::cout << "Bark!";
} 

void Dog::run(){
    std::cout << "Running";
}
```

To use a class, you create objects. Each object has access to the methods and attriutes of the class, however some of them are private so can't be called.

To create an object and then call one of the methods:

```cpp
Dog new_dog;
new_dog.bark();
```

One pattern for classes is to use setters and getters to manage private attribute values:


```cpp
class Dog{
    int age;

public:
    void set_age(int new_age){
        age = age;
    }

    int get_age(){
        return age;
    }
};
```

Broken down into headers and cpp files:

Header:

```cpp
class Dog{
    int age;

public:
    void set_age(int new_age):

    int get_age();
};
```

cpp:

```cpp
void Dog::set_age(int new_age){
    age = age;
}

int Dog::get_age(){
    return age;
}
```

Constructors are a way to set the attributes of a class when it is created.

```cpp
class Dog{
    int age;
    std::string;

public:
    Dog(int new_age, std::string new_name)
    : age(new_age), name(new_name){}
};
```

Split into headers:

```cpp
class Dog{
    int age;
    std::string;

public:
    Dog(int new_age, std::string new_name);
};
```

And cpp:

```cpp
Dog:Dog(int new_age, std::string new_name)
    : age(new_age), name(new_name){}
```

Alternatively you can use standard function syntax in the constructor:

```cpp
Dog:Dog(int new_age, std::string new_name){
    age = new_age;
    name = new_name;
}
```

To create an instance object of a class that has a contructor:

```cpp
Dog my_dog(3, "Barker");
```

Destructors are used to define special behaviour when a class is destroyed:

```cpp
Dog::~Dog(){
    //some custom code here
}
```

The reasons that are classes are destroyed are:
- The object is no longer in scope
- The object is explicitly deleted by the programmer's code
- The program finishes execution

## References and Pointers

Each byte of memory has an address. When a variable is initialised with a value, the value is allocated a number of bytes of memory. 

### Reference variables

Reference variables are variables that point to the same location in memory as another variable. When one variable's value is changed, the other variable's value also changes.

To create a reference variable, the `&` operator is added before a variables name.

For example:
```cpp
int qty_of_apples = 20;
int &number_of_apples = qty_of_apples; 
```

When the value of `qty_of_apples` changes then so will the value of `number_of_apples` and vice-versa.

### Pass by Reference

Normally when a value is passed as an argument to a function, then the function will create a copy of the value in its internatl scope. Changes to the value in the function will not affect the value of the original value passed as an argument.

This can be changed by using the `&` reference operator on function arguments. Adding the `&` to a parameter declaration in a function will mean that any changes to the value in the function, will also affect the original value/variable that was passed to the function.

For example:

```cpp
#include <iostream>

void double_qty(int &number){
  number *= 2;
}

int main(){
  int apples = 10;
  double_qty(apples);

  std::cout << apples;
}
```

The value of `apples` will be output as 20.

### Constants

A `const` (constant) variable is a variable whose value won't change. By doing this the compiler knows to throw an error if our program tries to change the value of a constant.

```cpp
int const conversion_rate = 2;
```

When used with a function argument the `const` keyword, the function knows that the value of the argument won't change throughout the execution of the function:

```cpp
int calculate_conversioon(int const conversion_rate, int const value){
  return conversion_rate * value;
}
```

Constant arguments can be refined further using the reference operator. By referencing the original value, an argument can reduce the need to allocate new memory. 

```cpp
int calculate_conversioon(int const &conversion_rate, int const &value){
  return conversion_rate * value;
}
```


### Memory Address Operator 

The `&` operator has another use: to get the memory address/location of an object. 

For example here it is used to get the address of a variable:

```cpp
int book_number = 12;
std::cout << &book_number;
```

Memory address are returned in hexadecimal. For example `0x0A0170330`.

To remember the use of the `&`:
- When it is used to declare a variable or argument, it is a reference operator
- When it is not used in declaration, it is an address operator



### Pointers

Pointers are a type of variable that stores a memory address. 

```cpp
int book_number = 12;
int* book_pointer = &book_number;
```

The `*` operator ignores spaces so writing the second line any of these ways is valid:

```cpp
int* book_pointer = & book_number;
int *book_pointer = & book_number;
int * book_pointer = & book_number;
```


### Dereference

After initialising a pointer, you can get the value that the pointer references using a deference operator, which is also a `*` operator.

For example:

```cpp
int book_number = 12;
int* book_pointer = &book_number;

std::cout << *book_pointer; // should be 12
```

To tell the different uses of a `*` apart:
- When it is used in the declaration of a variable: it is creating a pointer
- When it is not used in the declaraion: it is a deference

### Null Pointers

When declaring a pointer variable without a value it will reference nothing:

```cpp
int* pointer;
```

This is dangerous as it will hold a garbage value. If we don't yet know what the address should be when declaring the pointer, we can use C++'s `nullptr` value:

```cpp
int* pointer = nullptr;
```

In older code it is more common to see a `NULL` value instead:

```cpp
int* pointer = NULL;
```

