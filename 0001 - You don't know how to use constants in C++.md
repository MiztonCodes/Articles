# You don't know how to use constants in C++

When I took my first university programming class, my professor taught us to define constants using `#define`. It's a practice that should not be used in production code, to write robust, modern C++, you need to understand the distinct roles of `#define`, `const`, and `constexpr`.

While they may seem interchangeable, these three tools operate at different stages of the program's lifecycle, understanding the difference between the **Preprocessor**, **Compile-time** and **Runtime** is the key to choosing the right one.

## 1.- The Macro `#define`

This directive is a C heritage, it is a **preprocessor directive**, not a C++ language feature. When it is used, you are not defining a constant, you are telling the preprocessor to perform a blind text replacement before the compiler even sees your code.

Why you must stop using it?
* **No Type Safety:** it bypasses the C++ type system entirely.
* **No Scoping:** Macros completely ignore `namespace` and scope rules.
* **Debugging Nightmare:** Since the compiler never sees the macro name, error messages will refer to the expanded value, not the name you used.

Only use `#define` for conditional compilation with `#ifdef` or `#ifndef`, it is useful when you want to define some code only for a target plaform.

```cpp
#ifdef __WINDOWS_PLATFORM__
void windows_only_function();
#else
void generic_function();
#endif
```

## 2.- The Immutable Qualifier `const`

This is a C++ keyword that tells the compiler *"This variable's value should not change after initialization."*, unlike macros `const` variables are part of the type system and respect scope rules, it can be initialized with a value known at **compile-time** or values only known at **runtime**.

```cpp
// Value known at compile-time
const int MAX_AGE = 56;

// Value known at runtime
// when the program is launched and you define an immutable variable after a
// reading operation, const will respect immutability, scope rules 
// and type system.
int input_min_age = 0;
std::cin >> input_min_age;
const int MIN_AGE = input_min_age;
```

## 3.- The Compile-Time Constant `constexpr`

Introduced in C++11 and expanded significantly since then, `constexpr` is the most powerful tool for constants, it forces the compiler to evaluate an expression at **compile-time**, it moves the computing cost from the end user's machine at **runtime** to your build process at **compile-time**.

### `constexpr` Functions

You can mark functions as `constexpr`, if you provide constant aguments, the compiler will compute the result immediately.

```cpp
// This function will be removed if it is not used after constant evaluations.
// If the function is called with a not known value at compile-time, 
// the function will remain in the final code and will work properly.
constexpr int calc_cube(int n) { return n * n * n; }

// Will be evaluated at compile-time.
constexpr int my_cube = calc_cube(5);

// This is the result in machine code after compilation.
constexpr int my_cube = 125;
```

### Compile-Time Logic `if constexpr`

A powerful features is `if constexpr`, this allows you to perform **compile-time branching**, the compiler generates code only for the branch that is true, ignoring the other entirely.

```cpp
template <typename T>
void do_something(T value) {
	if constexpr (std::is_pointer_v<T>) {
		std::cout << "Handling a pointer.\n";
	} else {
		std::cout << "Handling a value.\n";
	}
}

int* n = nullptr;

// The compiler will generate a corresponding version for each of these,
// using function overloading.
do_something(n); // Prints "Is is a pointer.\n"
do_something(11); // Prints "It is not a pointer.\n"
```

The compiler is smart enough to know which version of the function to call, optimizing and reducing the time CPU waste on conditional branching.

### Look Up Tables

A powerful optimization technique is a look up table generation, for any data that is known at compile time and doesn't change at **runtime**,  leveraging the power of templates.

```cpp
constexpr int calc_cube(int n) { return n * n * n; }

template<std::size_t N_CUBES>
constexpr auto generate_cubes() {
  std::array<int, N_CUBES> table{};
  for (std::size_t n = 0; n < N_CUBES; ++n) {
    table[n] = calc_cube(n);
  }
  return table;
}

// The code written by us will look like this
constexpr auto cube_table = generate_cubes<3>();

// The compile-time code will look like this
constexpr auto cube_table = {0, 1, 8};
```

### Static assertions

This feature allow us to restrict how a template is used in other parts of our code at **compile-time** using static asserts, a `static_assert` verifies certain codition at **compile-time** enforcing strictly practices.

```cpp
// Declare a constant at compilation time.
constexpr size_t MAX_CONTAINER_SIZE = 10;
// Verifies certain condition at compile-time to prevent undefined behaviour
// in other parts of the code.
static_assert(MAX_CONTAINER_SIZE >= 10);
// Use whatever value was defined at MAX_CONTAINER_SIZE and
// initialize a vector with -11 value at all positions.
std::vector<int> my_vector(MAX_CONTAINER_SIZE, -11);
// Access the last element in a safer way, because we already have 
// a static assert before.
std::cout << my_vector.at(9);
```

## Conclusion

* Avoid `#define` for constants, keep it for **platform-specific** conditional compilation only.
* Use `constexpr` by default, if you can calculate a value at **compile-time**, let the compiler do it for you, it results in predictable code, faster and more optimized binaries.
* Use `const` when a value must be immutable but is only known at **runtime**, like a dynamic value depending from input.

In C++ there are many ways to do something that appear to be the same but in reality they have a complete different behaviour and purpose.
