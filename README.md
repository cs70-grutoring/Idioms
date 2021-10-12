# CS70 Idioms
One goal of CS70 is to teach professional coding practices.  Nearly all languages have *idioms* (agreed upon best practices), and good code will usually adhere to the idioms of the language in which it was written.  By nature, idioms are subjective and often change over time.

Because C++ is such an old and widly-used language, its idioms are not standardized and often a topic of debate. For clarity, we have compiled the following list of common C++ idioms that we teach and enforce in CS70. You may disagree with some of them, but we ask that you follow them for the code that you write in this class.  (This will be good practice, since in industry, you will likely find yourself required to follow coding practices that you may not agree with).

**Hint: You can view the table of contents for this file on GitHub by clicking the menu icon to the left of `README.md`.**

# 1. General
## 1.1 Prefer pre-increment over post-increment

In C++, we can either *pre-increment* a variable by placing `++` in front (such as `++x`) or *post-increment* a variable by placing `++` after (such as `x++`).  Both increase the variable by 1, but pre-increment returns the incremented variable and post-increment returns the value before the variable was incremented.  

Consider the following example:
```C++
int x = 5;
++x; // pre-increment x
x++; // post-increment x

x = 42;
std::cout << ++x << std::endl; // this will set x to 43 and print "43"
x = 42;
std::cout << x++ << std::endl; // this will set x to 43 but print "42"
```

Most of the time (such as in for loops), we only care about incrementing the variable and do not care about the value returned.  When both pre-increment and post-increment have the desired effect, **we should use pre-increment because it is more efficient** (it takes 1 hardware instruction rather than 2).  We also prefer pre-increment to `+= 1`.  


## 1.2 Appropriately use for and while loops
For and while loops perform similar functions, and any for loop can be rewritten as a while loop.  However, the two types of loops carry different implications, so choosing the correct type increases the readability of our code and can help clarify our intent.


### For loops
We should use a **for loop instead of a while loop when the loop requires a variable with the following two properties**:
1. The variable is only used inside of the loop.
2. Some operation or function is performed on the variable after each iteration of the loop.

A common example is when we have an index variable which either increases or decreases after each iteration.  The following example prints the numbers 0 through 4.

```C++
// Approach 1: while loop (not preferred)
size_t i = 0; 
while (i < 5) {
 std::cout << i << std::endl;
 ++i;
} // Assume i is never used again outside of the loop

// Approach 2: for loop (preferred)
for (size_t i = 0; i < 5; ++i) {
  std::cout << i << std::endl;
}
```

We can also use a for loop with an iterator to walk through the elements of a data structure.  For example, the following function prints every element in a `list<int>` (a list of `ints`).

```C++
void printList(list<int> l) {
  for (list<int>::iterator i = l.begin(); i != l.end(); ++i) {
    std::cout << *i << std::endl;
  }
}
```

We can even declare and use multiple iterators in a single for loop.

```C++
void printTwoVectors(list<int> l1, list<int> l2) {
  for (vector::iterator i = l1.begin(), j = l2.begin(); 
    i != l1.end() && j != l2.end(); 
    ++i, ++j) {
      std::cout << *i << ", " << *j << std::endl;
  }
}
```

In these cases, using a for loop increases the readability of our code because it clearly indicates that the for loop variable is *only* associated with this particular loop.  If we instead use a while loop with the variable declared above, the reader is left expecting the variable to be used again later on.  


### While loops
We should use a **while loop instead of a for loop when there is not a particular variable associated with the loop**.  For example, the following code prints and removes all of the elements of an `std::stack`.

```C++
// Approach 1: for loop (not preferred)
void printStack(std::stack& s) {
  for (size_t i = s.size(); s > 0; --i) {
    std::cout << s.top() << std::endl;
    s.pop();
  }
}

// Approach 2: while loop (preferred)
void printStack(std::stack& s) {
  while (!s.empty()) {
    std::cout << s.top() << std::endl;
    s.pop();
  }
}
```

Notice that by using a for loop, Approach 1 introduces a completely unnecessary variable `i`, which decreases efficiency and reduces readability. 

We also need to **use a while loop instead a of a for loop if we need to access the indexing variable outside of the loop**.  For example, the following function prints all powers of 3 that are less than `n` and returns the largest one.

```C++
int threes(int n) {
  int num = 1;
  while (num < n) {
    std::cout << num << std::endl;
    num *= 3;
  }

  // Because we use num outside of the loop, we must use a while loop instead of a for loop
  return num;
}
```



## 1.3 Avoid magic numbers
If your code requires a specific literal value (such as a particular number or string), you should usually **store the value as a constant rather than using the literal directly in your code**.  Not only does this make it easier to change the value in the future, it allows you to give the constant a meaningful name which helps provide the reader with context.

In general, we declare a constant in the `.hpp` file of a class, but if it is only need in a particular function, it may make sense to declare the constant at the beginning of the function body.  


## 1.4 Prefer `size_t` to `int` for non-negative integers
In C++, the `int` data type can store integers (both positive and negative), while the `size_t` datatype is *unsigned* and can only store non-negative integers (0 and up).  **When a variable should never be negative, and should never be compared to a variable that can be negative, we make it of type `size_t`** to indicate to the reader that the value will never be negative.

Because `size_t` is unsigned, if you ever try to set a `size_t` variable to a negative number, it will underflow and give you a very large positive number.

```C++
size_t x = 0;
--x; // x is now the largest possible positive number
```

## 1.5 Use header guards
On its own, `#include` has the danger of including the same code multiple times.  To prevent this, **we should place a header guard at the top of every `.hpp` file** (except for `-private.hpp` files when templating).  A header guard makes sure that a file is only included once by defining a unique variable associated with each file.  For example, if we have a file `sheep.hpp`, we should structure the file as follows. 

```C++
#ifndef SHEEP_HPP_
#define SHEEP_HPP_

// (Contents of sheep.hpp)

#endif  // SHEEP_HPP_
```

This will define the preprocessor macro `SHEEP_HPP_` the first time that we include `sheep.hpp`, and then use the existence of that variable to avoid including `sheep.hpp` a second time if it is included again.  For this to work, must use a unique preprocessor macro name for each `.hpp` file, so it is idiomatic to use the path and name of the file.

You can see a more detailed explanation of header guards in chapter 1.3 of the [CS70 Textbook](https://github.com/MatthewCalligaro/CS70Textbook).



# 2. Conciseness 
## 2.1 Avoid excessive unnecessary variables
In C++, we strive for clear and concise code and avoid unnecessary "bloat" which can distract the reader.  Consider the following two approaches for calculating Euclidean distance:

```C++
float x = 7.2;
float y = 4.4;

// Approach 1: excessive unnecessary variables (not preferred)
float xSquared = x * x;
float ySquared = y * y;
float distance = math.sqrt(xSquared + ySquared);

// Approach 2: concise implementation (preferred)
float distance = math.sqrt(x * x + y * y);
```
The `xSquared` and `ySquared` variables in the first approach are completely unnecessary and distracting; the reader must mentally substitute them into the distance calculation to understand what is going on.  In the second approach, the reader sees the distance calculation all on one line.  Further, unnecessary variables can sometimes decrease performance.

In some cases, however, an unnecessary temporary variable can add clarity by distilling out an intermediate result.  This idiom is more subjective, so you will need to use your best judgement when reviewing your code.  We will only take off points in truly egregious cases such as the provided example. 


## 2.2 Avoid unnecessary conditionals
Unnecessary conditional statements hinder code readability and can significantly reduce performance due to [branch prediction](https://en.wikipedia.org/wiki/Branch_predictor), which you will learn more about in CS105.  Thus, you should usually **use conditional statements only when they are required for the correct execution of your code**.

Below, we discuss a number of common pitfalls involving unnecessary conditionals. 

### Returning booleans
When a function returns whether a condition is true, it should return the condition directly rather than using an `if-else` statement to return `true` or `false`.  Consider the following two implementations of a the `isPositive` function, which checks whether an integer is positive.  

```C++
// Approach 1: if-else (not preferred) 
isPositive (int x) {
  if (x > 0) {
    return true;
  } else {
    return false;
  }
}

// Approach 2: return condition (preferred)
isPositive (int x) {
  return x > 0;
}
```

### Returning inside of an `if`
If we always `return` inside of an `if` block, we should not include `else` for cases not handled by the `if`.  When the `if` condition is met, we will `return` before any of the other code is run, so the `else` is unnecessary. 

For example, suppose that the `Sheep` class has a `getAge()` method which returns `0` if `ageUnknown_` is `true`, and otherwise returns `age_`.  

```C++
// Approach 1: Including the else (not preferred)
size_t Sheep::getAge() {
  if (ageUnknown_) {
    return 0;
  } else {
    return age_;
  }
}

// Approach 2: Not including the else (preferred)
size_t Sheep::getAge() {
  if (ageUnknown_) {
    return 0;
  }
  return age_;
}
```

### Factor conditionals out of loops
When you have a conditional statement inside the body of a loop, consider whether it can be moved outside of the loop.  If so, this will cause it to only run once rather than every iteration of the loop. 

For example, suppose that we want to print the numbers 0 through 4 if and only if the `verbose` flag is set to `true`.  Rather than checking each iteration of the loop, we should only enter the loop if `verbose == true`. 

```C++
// Approach 1: check condition in loop (not preferred)
for (size_t i = 0; i < 4; ++i) {
  if (verbose) {
     std::cout << i << std::endl;
  }
}

// Approach 2: factor condition out of loop (preferred)
if (verbose) {
  for (size_t i = 0; i < 4; ++i) {
    std::cout << i << std::endl;
  }
}
```

### Conditionals already handled by loop conditions
Frequently, we want a loop to only run if a certain condition is true.  Remember that loops already contain a looping condition, so you only need to add an additional `if` statement if the condition cannot be incorporated into the looping condition.

In the following example, the `if` statement can be removed because it is captured by the looping condition.  If `input <= 0`, the loop will not be run because `i < input` will never be true.    
```C++
if (input > 0) {
  for (size_t i = 0; i < input; ++i) {
    std::cout << i << std::endl;
  }
}
```

In the following example, the `if` condition is separate from the looping condition, so it does not need to be removed. Assume that `verbose` is a `bool` which is set somewhere above.  
```C++
if (verbose) {
  for (size_t i = 0; i < input; ++i) {
    std::cout << i << std::endl;
  }
}
```


## 2.3 Avoid duplicate code between cases
Often times, a function will consist of multiple cases.  Some work will be unique to each case, while other work must be done for all cases.  We should always **factor out shared work so that the code doing this work is only written once**.

For example, suppose that the `Sheep` class has a `birthday` method which celebrates the sheep's birthday.  The method will always increment the sheep's age and takes the parameter `likesCake` indicating whether they should eat cake or ice cream.

```C++
// Approach 1: Duplicate code between cases (not preferred)
void Sheep::birthday(bool likesCake) {
  if (likesCake) {
    eatCake();
    ++age_;
  } else {
    eatIceCream();
    ++age_;
  }
}

// Approach 2: Shared work factored out (preferred)
void Sheep::birthday(bool likesCake) {
  if (likesCake) {
    eatCake();
  } else {
    eatIceCream();
  }
  ++age_;
}
```

Shared work does not have to be an exact copy across the cases; sometimes, the shared work looks different in each case but can be rewritten in a more general way that can be executed outside of the cases.


## 2.4 Avoid unreachable code
Unreachable code is code which can never be run.  **We should always remove unreachable code since it can confuse the reader and does not affect the behavior of our program**.  

Here are two examples of unreachable code. 

```C++
// Example 1: Code after a return
int foo(int* x, int* y) {
  int temp = *x;
  *x = *y;
  return *x + *y;
  *y = temp; // This line is unreachable and will never be run
}

// Example 2: Code in a conditional that will never be executed
int x;
std::cin >> x;
if (x > 0 && x < 0) { // This condition will never evaluate to true
  x = 42; // This line is unreachable and will never be run
}
```


## 2.5 Avoid unused variables
An unused variable is a variable which is never read from and therefore does not affect the behavior of our program.  We should **always remove unused variables** because they can confuse the reader and distract from the purpose of our program.  Even if you write to a variable, it is still unused if you never read from it.  Depending on the settings we use, the compiler will often identify unused variables with a warning.

In the following example, `x` and `y` are unused variables and should be removed

```C++
int main() {
  int x = 5; // x is an unused variable

  int y = 0; // y is also unused even though we write to it
  y += 5;

  int z = 8; // because we read from z, it is not an unused variable
  std::cout << z << std::endl;

  return 0;
}
```


## 2.6 Avoid comparing bools to `true` or `false`
Comparing a `bool` to `true` or `false` adds unnecessary obfuscation, making our code more complicated and harder to understand.  For example, the expression `a == true` can always be replaced with `a`, since `a` must already be a bool to be compared to `true`.  Similarly, `a == false` can always be replaced with `!a`.  **Our code should never have the phrase `== true`, `!= true`, `== false`, or `!= false`**.

The following example demonstrates how we can rewrite the conditions of a function to remove unnecessary comparisons. 

```C++
// Approach 1: Compares bools to true and false (not preferred)
void foo(bool a, bool* b) {
  if (a == true) {
    std::cout << "case 1" << std::endl;
  } else {
    std::cout << "case 2" << std::endl;
    if (b[0] == false) {
      std::cout << "subcase 1" << std::endl;
    } else {
      std::cout << "subcase 2" << std::endl;
    }
  }
}

// Approach 2: Removes unnecessary comparisons (preferred)
void foo(bool a, bool* b) {
  if (a) {
    std::cout << "case 1" << std::endl;
  } else {
    std::cout << "case 2" << std::endl;
    if (!b[0]) {
      std::cout << "subcase 1" << std::endl;
    } else {
      std::cout << "subcase 2" << std::endl;
    }
  }
}

```



# 3. Pointers
## 3.1 Prefer `[]` notation
If `p` is a pointer and `n` is an integral datatype (such as `int` or `size_t`), then `p[n]` is syntactic sugar for `*(p + n)`.  In C++, all arrays are pointers, so this is especially helpful for accessing the `n`th element of an array.  We should **use [] notation instead of pointer math whenever applicable**.

```C++
string sheep[] = {"Shawn", "Elliot", "Timmy", "Shirley"};

// Approach 1: Pointer math (not preferred)
std::cout << *(sheep + 2) << std::endl;

// Approach 2: [] notation (preferred)
std::cout << sheep[2] << std::endl;
```


## 3.2 Prefer '->' notation
Suppose that `p` is a pointer to an object with public member function `foo()` and public member variable `x`.  Then, `p->foo()` is syntactic sugar for `(*p).foo()` and `p->x` is syntactic sugar for `(*p).x`.  We should **use arrow notation whenever applicable**.

```C++
Sheep shawn = new Sheep();

// Approach 1: Explicit dereference (not preferred)
(*shawn).pet();

// Approach 2: Arrow notation (preferred)
shawn->pet();
```



# 4. Classes
## 4.1 Prefer member initialization list
When we initialize an object in C++, all of its data member need to be initialized as well.  We specify the value to which each variable is initialized in the member initialization list of our object's constructor.  Any data member not in the member initialization list is initialized with its default constructor.  

Any time we refer to a member variable in the body of the constructor, the variable is in its *use* phase.  Therefore, if we set the variable in the body of the constructor, we really are setting it twice: first, we default initialize it to the *wrong* value, then update it to the correct value.  When we use the member initialization list, we set the variable to the correct value the first time.  For this reason, we should **always use the member initialization list to initialize member variables rather than setting them in the body of the constructor**.  

The member initialization list has the form `: dataMember_{<parameters>}, ...`, where `...` can be a list of more data members with the same format.  The `dataMember_{<parameters>}` term calls the constructor of `dataMember_` and passes it `<parameters>`.  Let's consider a simple parameterized constructor for a `Point` class.
 
```C++
// Approach 1: Set variables in constructor body (not preferred)
Point::Point(double x, double y) { // x_ and y_ are default initialized
  // x_ and y_ are updated to x and y in the use phase
  x_ = x;
  y_ = y;
}

// Approach 2: Initialize variables in member initialization list (preferred)
Point::Point(double x, double y) :
  // x_ and y_ are initialized to x and y
  x_{x}, y_{y} {
  // Nothing to do in constructor body
}
```

Next, let's consider a more complex example. Suppose that `Sheep` has a parameterized constructor taking a `string name` and `size_t age`.  Suppose that the `Barn` class has a `Sheep sheep_` data member and a `Sheep* sheepPointer_` data member (a pointer to a sheep).

```C++
// Approach 1: Set variables in constructor body (not preferred)
Barn::Barn(string name, size_t age) { // sheep_ and sheepPointer_ are default initialized
  // sheep_ and sheepPointer_ are updated to the intended value in the use phase
  sheep_ = Sheep(name, age);
  sheepPointer_ = new Sheep(name, age);
  sheep_.feed();
  sheepPointer_->feed();
}

// Approach 2: Initialize variables in member initialization list (preferred)
Barn::Barn(string name, size_t age) :
   // sheep_ and sheepPointer_ are initialized with the intended value
  sheep_{name, age}, sheepPointer_{new Sheep(name, age)} { 
  // Anything in the use phase still goes in the body of the constructor
  sheep_.feed();
  sheepPointer_->feed();
}
```

Notice that in Approach 1, we default initialized `sheep_`, meaning that we called the `Sheep` default constructor.  If the `Sheep` class did not have a default constructor, it would have caused an error!  This is another advantage of the member initialization list — it allows us to initialize data members that do not have a default constructor.  


## 4.2 Avoid unnecessary `this` keywords
When you are inside of an object in C++, the `this` keyword is a *pointer* to the object.  In other languages like Java and C#, it is idiomatic to use always use the `this` keyword when referring to member methods and variables.  However, in C++, the idiom is to **only use the `this` keyword when necessary, and not when referring to member methods and variables**.

In the following example, both of the `this` keywords are superfluous; we should simply write `insert(length_)`.  
```C++
// Example of unnecessary this keywords (should be removed)
this->insert(this->length_)
```

However, when you need a pointer to the current object, you should use the `this` keyword.  The following if statement checks if the object we are inside of is equal to `rhs`.  Notice that we have to deference `this` because it is a pointer. 
```C++
// Example of necessary this keywords (should not be removed)
if (*this == rhs)
```


## 4.3 Label methods `const` when applicable
When we label a member method as `const`, it promises that the method will not change any of the data members of the object.  It allows us to call that method on `const` instances of the object (such as a `const` reference to the object).  We should **always label a member method as `const` if it does not change any data members**.  

We need to label both the *method declaration* in the `.hpp` and the *method definition* in the `.cpp`.  For example, suppose that our class has a `size()` member method which returns the private data member `size_` without modifying it.

```C++
// In the .hpp:
size_t size() const;

// In the .cpp:
size_t size() const {
  return size_;
}
```


## 4.4 Implement `!=` with `==`
If a class supports `operator==` and `operator!=`, they should always return the opposite value (if `a == b` evaluates to `true`, then `a != b` should evaluate to `false`, and vice versa).  To avoid code duplication, we should **always implement `!=` by calling `==` and negating the result**.  Here is an example for the `Sheep` class. 

```C++
bool operator!=(const Sheep& other) const {
  return !(*this == other);
}
```

We use `*this` to get a reference to ourself since `this` is a pointer.


## 4.5 Prefer synthesized methods when applicable
The compiler can create synthesized versions of the following methods:

1. **Default Constructor** (`Class() = default`): The synthesized default constructor default constructs all private data members.  
2. **Copy Constructor** (`Class(const Class& other) = default`): The synthesized copy constructor copy constructs each private data member parameterized with the corresponding data member in `other`. 
3. **Assignment operator** (`Class& operator=(const Class& other) = default`): The synthesized assignment operator calls the assignment operator on each private data member parameterized with the corresponding data member in `other`.
4. **Destructor** (`~Class() = default`): The synthesized destructor calls the destructor on each private data member.  

If any of these synthesized methods have the desired behavior, **we should use the synthesized version by writing `= default` in the `.hpp` file rather than explicitly implementing the equivalent method in the `.cpp` file**.  

For example, consider the following class. 

```C++
// sheep.hpp
class Sheep {
 public:
  Sheep();
  Sheep(const Sheep& other);
 private:
  list<Sheep*> friends;
```

Suppose that we implemented the default and copy constructors as follows.

```C++
// sheep.cpp
Sheep::Sheep() : friends_{} {
  // Nothing left to do
}

Sheep::Sheep(const Sheep& other) : friends_{other.friends_} {
  // Nothing left to do
}
```

For both constructors, we implemented the exact behavior as the synthesized version!  Thus, we should remove these implementations from `sheep.cpp` and simply use the synthesized version in `sheep.hpp`.  

```C++
// sheep.hpp (using synthesized constructors)
class Sheep {
 public:
  Sheep() = default;
  Sheep(const Sheep& other) = default;
 private:
  list<Sheep*> friends;
}

// sheep.cpp no longer implements Sheep() or Sheep(const Sheep& other)
```


## 4.6 Avoid explicitly calling operators and destructors
In C++, we can define how certain symbols (such as `*` and `==`) act on objects of our class by implementing the corresponding operator method (such as `operator*` and `operator==`).  The operator method is then called whenever we use the corresponding symbol on the object.  For example, `a == b` will call `operator==` of `a` with `b` as the parameter.  We should **always call operators by using the corresponding symbol rather than explicitly calling the operator**.  For example, we should *not* write `a.operator==(b)`, even though it has the exact same behavior as `a == b`.  

Here are some examples which explicitly and implicitly use different operators.

```C++
// Example 1: operator==
Sheep shawn, elliot;
std::cout << shawn.operator==(elliot) << std::endl;  // explicit use (not preferred)
std::cout << shawn == elliot << std::endl;           // implicit use (preferred)

// Example 2: operator=
shawn.operator=(elliot);  // explicit use (not preferred)
shawn = elliot;           // implicit use (preferred)

// Example 3: operator*
vector<size_t> vec = {1, 2, 3}
std::cout << vec.begin().operator*() << std::endl;  // explicit use (not preferred)
std::cout << *vec.begin() << std::endl;             // implicit use (preferred)
```

Similarly, we should **never explicitly call an object's destructor**.  An object's destructor is automatically called in the *destruction* phase of the object's lifetime.  

```C++
Sheep* timmy = new Sheep{};
timmy->~Sheep();  // explict destructor call (not preferred)
delete timmy;     // the Sheep's destructor is implicitly called when we delete timmy
```



# 5. Iterators
## 5.1 Use standard template aliases
When creating an iterator, we should include certain aliases so that it can interface with the Standard Template Library (STL).  The following example shows these aliases for a forward iterator over a data structure containing `strings` (such as the iterator for `TreeStringSet`).

```C++
using value_type = std::string;
using reference = value_type&;
using pointer = value_type*;
using iterator_category = std::forward_iterator_tag;
```

**Our iterator implementation and interface should uses these aliases rather than the explicit types whenever applicable**.  For example, consider the declaration of `operator*` and `operator->` for this iterator. 

```C++
// Approach 1: Explicit types (not preferred)
string& operator*() const;
string* operator->() const;

// Approach 2: STL aliases (preferred)
reference operator*() const;
pointer operator->() const;
```


## 5.2 Implement `operator->` with `operator*`
For an iterator, `operator*` returns the value to which the iterator currently points and `operator->` returns the address of that value.  Because the result of `operator->` is directly dependent on the result of `operator*`, **we should always implement `operator->` for iterators by leveraging `operator*`**.

Suppose that we are writing the iterator for the `Foo` class.  We can implement `operator->` as follows.

```C++
Foo::Iterator::pointer operator->() const {
  return &(**this);
}
```

To understand what `&(**this)` means, let's work inside out.

- `this`: A pointer to the iterator on which we called `operator->`. 
- `*this`: Dereferences `this` to give the iterator itself. 
- `**this`: Calls `operator*` on the iterator, which returns the value to which the iterator currently points.
- `&(**this)`: Gives the address of the value returned by `**this`.   
