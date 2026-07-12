# How to optimize IO in C++

[Link to the repository containing all examples](https://github.com/MiztonCodes/OptimizeIO)

Back when I was at my first class of Data Structures and Algorithms, I started to solve competitive programming questions in judges like CodeForces, I didn't know why my code was slower if my solution was efficient (at least in theory), my professor explain to the class the reason why our programs were slow, because of I/O is an expensive task for computers.

To start working with the examples in this article just clone the repository and play with it as long the article explains how to run the code.

```bash
git clone https://github.com/MiztonCodes/OptimizeIO.git
```

# Why `std::cout` and `std::cin` are slow?

In C++ by default, both `std::cin` and `std::cout` streams are synchronized with standard C `scanf` and `printf` streams and at the same time every single call to `std::cin`  flushes the `std::cout` buffer, because `std::cin` before reading input, wants to output any pending prompt to the user, which can cause performance issues because reading and writing are expensive tasks due to the need to make calls to the operating system, the solution is to disable all of this synchronizations and manually flushing the output buffer whenever it is needed.

*Note: For convenience in all examples I'll use `#include<bits/stdc++.h>` header to simplify the imports, this header includes everything we'll need and this header only works on GCC compiler.*

## What is input and output?

When you run your program, it is like an isolated box inside your computer, ready to do some work, the **input** process involves moving raw data from an external source like a keyboard,  a file or even the network to your program *(reading)*, the **output** process involves moving data from your program to an external destination like a monitor, the console, a file, the network or even an external device *(writing)*, for both processes your program does not read or write directly to the **input** and **output** sources, because each source has its own way to transfer data, C++ provides a uniform interface to hide that complexity through the usage of **streams**.

## What is a Stream?

In C++ and other programming languages, a **stream** is an abstraction layer that hides the complexity of dealing with external sources to *read* and *write* data from your program, so you can use a unified interface with the following operators for writing `dest << data`  and reading `source >> dest`.  The **stream** is able to translate the raw input data into data types that your program can understand like `int`, `bool`, `char`, so basically the **stream** is only responsible to transport data.

## What is a Buffer?

A **buffer** is a contiguous block of memory located at the RAM, reserved to *batch* data to reduce the amount of calls to the operating system, once the **buffer** has reached its full capacity the operating system performs one single efficient transfer of **buffer's** data to its final destination, so the memory of the **buffer** can be reused and then wait until the **buffer** reached its full capacity again, this makes **buffers** the perfect place to hold data during your program execution.

## Why reading and writing are expensive?

As in real life the travel from shorter distances almost always implies shorter travel times and  plus the planning time, reading and writing data is expensive because the data must travel from one location to its destination and at same time the CPU needs to manage the transfer of data.

Reading and writing data implies much more than just moving data from one location to its destination,  when a reading or writing operation is needed a call to the operating system is made and the process first makes the CPU to switch the context of whatever it was doing before, then verifies permission and perform operations on memory to finally move the data, while all this happens the CPU spends a lot time and operations waiting to move the data to its destination.

## Why C++ streams are synchronized with C streams?

This is due to the C++ history, C++ at first was C with classes, nowadays to keep compatibility and consistency it keeps synchronized standard input and output streams, so you can mix both styles in a single program and expects the output order be preserved when mixing both styles, but it introduces performance issues, especially when you don't need that synchronization like in competitive programming or newer projects where compatibility with C is not needed.

## Optimize reading

As I said earlier, just disable all synchronizations and manually flush the output stream when needed, just add the following lines to the beginning of your `main` function.

**Warning: When you disable synchronization, don't mix both styles because it can produce an undefined behavior**

```cpp
// Disable synchronization with standard C scanf and printf streams
std::ios_base::sync_with_stdio(false); 
// Disable synchronization between std::cin and std::cout streams
std::cin.tie(nullptr); 
```

*Note: You only need to untie `std::cin` like in the code snippet above because `std::cout` is not tied to anything, so just remember to only untie `std::cin`.*

To demonstrate performance gains, you can run the following tests to verify it by yourself.

*Note: To demonstrate a noticeable performance gain, use as input the `dataset.txt` file hosted in the repository of this example, this file has a size of 75 MB and contains 10 million integer numbers from 1 to 10'000'000 this is a really simple example to demonstrate performance gains even when your program only read data and don't perform any other computation.*

### Compilation

To compile and run the next examples just use the following commands.

```bash
# Move the section directory
cd 01_Optimize_Writing

# Give execution permissions to compilation scripts
chmod +x run_without_optimization.sh
chmod +x run_with_optimization.sh

# Run the examples below
./run_without_optimization.sh
./run_with_optimization.sh
```

### Running without optimization

```cpp
#include <bits/stdc++.h>

int main() {
  // Get the current time before executing the heavy part.
  auto start = std::chrono::high_resolution_clock::now();
	
	// Heavy part reading data.
  int val = 0;
  for (long long i = 0; i < 10'000'000; ++i) {
    std::cin >> val;
  }

  // Get the current time after executing the heavy part.
  auto end = std::chrono::high_resolution_clock::now();

  // Calculate the exeuction time lapse.
  std::chrono::duration<double> diff = end - start;

  // Using std::cerr to print diagnostic data immediately.
  std::cerr << "Time taken: " << diff.count() << " seconds" << std::endl;
}
```

You will get an output like this

```
Time taken: 5.39379 seconds
```

### Running with optimization

```cpp
#include <bits/stdc++.h>

int main() {
  // Optimization
  std::ios_base::sync_with_stdio(false);
  std::cin.tie(nullptr);

  // Get the current time before executing the heavy part.
  auto start = std::chrono::high_resolution_clock::now();
	
	// Heavy part reading data.
  int val = 0;
  for (long long i = 0; i < 10'000'000; ++i) {
    std::cin >> val;
  }

  // Get the current time after executing the heavy part.
  auto end = std::chrono::high_resolution_clock::now();

  // Calculate the exeuction time lapse.
  std::chrono::duration<double> diff = end - start;

  // Using std::cerr to print diagnostic data immediately.
  std::cerr << "Time taken: " << diff.count() << " seconds" << std::endl;
}
```

You will get an output like this

```
Time taken: 0.667611 seconds
```

Even in this really simple example we can observe a performance gain of five times faster execution, just disabling the synchronization of streams.

## Optimize writing

Avoid the usage of `std::endl` to add a line break , because `std::endl` add a line break and at the same time it flushes the buffer (which is an expensive operation), so if you only want to add a line break  just use the `\n` character like in the next example:

```cpp
/* 
	Avoid this style to add line breaks,
	it will add a line break at the end of the string
	and flush the output buffer which will cause
	performance issue.
*/
std::cout << "Woody!" << std::endl;

/*
	Prefer this style to add line breaks,
	it will add a line break at the end of the string
	and will not flush the output, so the string is going
	to be batched at the output buffer,
*/
std::cout << "Woody!\n";
```

To demonstrate performance gains, you can run the following tests to verify it by yourself.

### Compilation

To compile and run the next examples just use the following commands.

```bash
# Move the section directory
cd 01_Optimize_Reading

# Give execution permissions to compilation scripts
chmod +x run_without_optimization.sh
chmod +x run_with_optimization.sh

# Run the examples below
./run_without_optimization.sh
./run_with_optimization.sh
```

### Running with `std::endl`

```cpp
#include <bits/stdc++.h>

int main() {
  /*
	  In this case the following lines does not affect in any
	  case because we are not reading data.
  */
  std::ios_base::sync_with_stdio(false);
  std::cin.tie(nullptr);

  // Get the current time before executing the heavy part.
  auto start = std::chrono::high_resolution_clock::now();

  // Heavy part writing data.
  for (int i = 1; i <= 10'000'000; ++i) {
    std::cout << "Iteration " << i << std::endl;
  }

  // Get the current time after executing the heavy part.
  auto end = std::chrono::high_resolution_clock::now();

  // Calculate the exeuction time lapse.
  std::chrono::duration<double> diff = end - start;

  // Using std::cerr to print diagnostic data immediately.
  std::cerr << "Time taken: " << diff.count() << " seconds" << std::endl;
}
```

You will get an output like this

```
Time taken: 5.76418 seconds
```

### Running with `\n`

```cpp
#include <bits/stdc++.h>

int main() {
  /*
	  In this case the following lines does not affect in any
	  case because we are not reading data.
  */
  std::ios_base::sync_with_stdio(false);
  std::cin.tie(nullptr);

  // Get the current time before executing the heavy part.
  auto start = std::chrono::high_resolution_clock::now();

  // Heavy part writing data.
  for (int i = 1; i <= 10'000'000; ++i) {
    std::cout << "Iteration " << i << "\n";
  }

  // Get the current time after executing the heavy part.
  auto end = std::chrono::high_resolution_clock::now();

  // Calculate the exeuction time lapse.
  std::chrono::duration<double> diff = end - start;

  // Using std::cerr to print diagnostic data immediately.
  std::cerr << "Time taken: " << diff.count() << " seconds" << std::endl;
}
```

You will get an output like this

```
Time taken: 1.47316 seconds
```

Even in this really simple example we can observe a performance gain of five times faster execution, just by not flushing the output buffer every single time we need to add a line break.

## Miscellaneous 

### Compilation

To compile and run the next examples just use the following commands.

```bash
# Move the section directory
cd 02_Miscellaneous

# Give execution permissions to compilation scripts
chmod +x run_error_log.sh
chmod +x run_format_output.sh
chmod +x run_redirect_io.sh
chmod +x run_operator_overloading.sh

# Run the examples below
./run_error_log.sh
./run_format_output.sh
./run_redirect_io.sh
./run_operator_overloading.sh
```

### Special `std::cerr` stream for errors and diagnostics

In addition to the standard `std::cin` reading and `std::cout` writing streams, C++ provides another `std::cerr` stream to write errors and diagnostic data, the difference between `std::cout` and `std::cerr` is mainly the way each of them outputs data, while `std::cout` holds data in a buffer, `std::cerr` is unbuffered and outputs data immediately after a call to it, so if you use `std::cout` and the program execution is interrupted, the output buffer is going to be destroyed along the program and data stored in the buffer will be lost.


```cpp
#include <bits/stdc++.h>

int main() {
  std::cout << "First std::cout call\n";
  std::cerr << "Using this to print error messages and diagnostic data.\n";
  std::cout << "Second std::cout call\n";
}
```

You will get this output

```
First std::cout call
Using this to print error messages and diagnostic data.
Second std::cout call
```

As you can read a call to `std::cerr` writes immediately to the console because it is unbuffered.

## Formatting output numbers

If you are using C++20 or higher, you can use the modern `std::format` to print in fixed-point, and scientific notations like in the following example.

```cpp
#include <bits/stdc++.h>

int main() {
  double my_number = 7.2069;

  // Default formatting
  std::cout << std::format("Default notation: {}\n", my_number);

  // Fixed-point and rounded up with 2 decimal places
  std::cout << std::format("Fixed and rounded up (2 decimal): {:.2f}\n", my_number);

  // Fixed-point and floored with 2 decimal places
  std::cout << std::format("Fixed and floored (2 decimal): {:.2f}\n", std::floor(my_number));

  // Fixed-pint and ceiled with 2 decimal places
  std::cout << std::format("Fixed and ceiled (2 decimal): {:.2f}\n", std::ceil(my_number));

  // Scientific notation
  std::cout << std::format("Scientific and rounded up: {:.2e}\n", my_number);

  return 0;
}
```

You will get an output like this

```
Default notation: 7.2069
Fixed and rounded up (2 decimal): 7.21
Fixed and floored (2 decimal): 7.00
Fixed and ceiled (2 decimal): 8.00
Scientific and rounded up: 7.21e+00
```

This is useful when you need to deal with decimal places and rounding decimal numbers based on the question requirements.

### Redirect streams to files

It is very tedious to read and write data directly from the terminal, and have mixed output along with logs, errors, diagnostics and output data, so you can redirect all of this input and output categories to files in your filesystem when you execute the program.

The operating system has multiple channels to read and write data, so when you run your program you can redirect channels to files like in the following example.

```bash
./bin 0< input.txt 1> output.txt 2> errors.txt
```

* The *channel 0* is meant to be used to read input data.
* The *channel 1* is meant to be used to write output data.
* The *channel 2* is meant to be used to write error data.

And you can still use `std::cin`, `std::cout` and `std::cerr` just like any other program and have a more cleaner workspace, this is helpful if you have a really large `input.txt` dataset file and copy pasting to the terminal is not an option.

*Extra: There is also an standard `std::log` stream that is buffered and writes data to the error channel, use it only when you need to write some extra information but you don't need it immediately.*

### Redirect I/O streams to custom ones

In C++ you can keep using `std::cin` and `std::cout` but redirect the streams to read and write from a custom string or a file streams and then rollback the original ones to read and write from the standard input and output.

In the following example a **RAII** *(Resource Acquisition Is Initialization)* pattern is implemented to ensure the swap and rollback of buffers is secure even if an exception happens or the program ends early.

*Note: You can run this example using the files provided at the repository of the article.*

```cpp
#include <bits/stdc++.h>

/*
  This class will hold the overall logic to
  swap buffers given a stream and a new buffer
  using the RAII pattern.
*/
class stream_redir {
private:
  // Holds the stream
  std::ios& _stream;
  // Holds the original buffer of the stream
  std::streambuf* _origin_buff;
public:
  /*
    The constructor will save a reference to the stream
    and will hold a pointer to the original buffer while
    assigning the new buffer to the stream.
  */
  stream_redir(
    std::ios& stream,
    std::streambuf* new_buffer
  ): _stream(stream), 
    _origin_buff(stream.rdbuf(new_buffer)) {}
  // The destructor automatically restores the original buffer.
  ~stream_redir() {
    _stream.rdbuf(_origin_buff);
  }
  // Prevent copying to avoid multiple restorations of the same buffer.
  stream_redir(const stream_redir&) = delete;
  stream_redir operator=(const stream_redir&) = delete;
};

void redirect_output_to_string() {
  std::cout << "=== REDIRECT OUTPUT TO A STRING ===\n";
  std::stringstream custom_out;
  std::cout << "Before redirection\n";
  // In this scope the redirection will happens.
  {
    stream_redir redir(std::cout, custom_out.rdbuf());
    std::cout << "This message is going to be stored at custom_out buffer";
  } // At the end of the scope, buffer restoration happens.
  std::cout << "After redirection\n";
  std::cout << "custom_out contents:\n";
  std::cout << custom_out.rdbuf() << "\n";
}

void redirect_input_to_string() {
  std::cout << "=== REDIRECT INPUT TO A STRING ===\n";
  std::string input_data = "I'm happy learning C++";
  std::stringstream custom_in(input_data);
  std::string read_data;
  read_data.reserve(input_data.size());
  // In this scope the redirection will happens.
  {
    stream_redir redir(std::cin, custom_in.rdbuf());
    while (std::getline(std::cin, read_data)) {}
  } // At the end of the scope, buffer restoration happens.
  std::cout << "read_data contents:\n";
  std::cout << read_data << "\n";
}

void redirect_output_to_file() {
  std::cout << "=== REDIRECT OUTPUT TO A FILE ===\n";
  std::filebuf custom_out;
  custom_out.open("dataset_temp.txt", std::ios::out);
  std::vector<int> nums{1, 2, 3, 4, 5};
  std::cout << "Before redirection\n";
  // In this scope the redirection will happens.
  {
    stream_redir redir(std::cout, &custom_out);
    for (const int& n : nums) {
      std::cout << n << "\n";
    }
  } // At the end of the scope, buffer restoration happens.
  std::cout << "After redirection\n";
}

void redirect_input_to_file() {
  std::cout << "=== REDIRECT INPUT TO A FILE ===\n";
  std::filebuf custom_in;
  // Here read data from the file created by redirect_output_to_file function
  custom_in.open("dataset_temp.txt", std::ios::in);
  constexpr size_t MAX_NUM_SIZE = 5;
  std::vector<int> nums;
  nums.reserve(MAX_NUM_SIZE);
  std::cout << "Before reading nums.size() == " << nums.size() << "\n";
  // In this scope the redirection will happens.
  {
    stream_redir redir(std::cin, &custom_in);
    for (size_t i = 0; i < MAX_NUM_SIZE; ++i) {
      int n = 0;
      std::cin >> n;
      nums.push_back(n);
    }
  } // At the end of the scope, buffer restoration happens.
  std::cout << "After reading nums.size() == " << nums.size() << "\n";
  for (const auto& n : nums) std::cout << n << " ";
  std::cout << "\n";
}

int main() {
  // Optimization for reading and writing
  std::ios_base::sync_with_stdio(false);
  std::cin.tie(nullptr);
  // Examples
  redirect_output_to_string();
  redirect_input_to_string();
  redirect_output_to_file();
  redirect_input_to_file();
}
```

You will get an output like this

```
=== REDIRECT OUTPUT TO A STRING ===
Before redirection
After redirection
custom_out contents:
This message is going to be stored at custom_out buffer
=== REDIRECT INPUT TO A STRING ===
read_data contents:
I'm happy learning C++
=== REDIRECT OUTPUT TO A FILE ===
Before redirection
After redirection
=== REDIRECT INPUT TO A FILE ===
Before reading nums.size() == 0
After reading nums.size() == 5
1 2 3 4 5
```

So as you can see, the streams can be redirected to read and write data to other sources, this is useful for testing when you try to mock certain input or output sources while keep using the standard input and output streams.

### Stream operators overloading

Sometimes you will need to read and write data based on a format or structure, for example to read data and create an instance of a class or struct, maybe you want to print all your vector public members to a certain format so other programs can understand your data.

In C++ you can overload the `<<` output operator and the `>>` input operator to work with complex data types like in the next example.

```cpp
#include <bits/stdc++.h>

namespace mizton {
  struct vector_3d {
    int x, y, z;
  };
};

/*
  Overloading of output operator on output streams.
*/
std::ostream& operator<<(std::ostream& os, const mizton::vector_3d& v) {
  os << "vector{" << v.x << "," << v.y << "," << v.z << "}\n";
  return os;
}

/*
  Overloading of input operator on input streams.
*/
std::istream& operator>>(std::istream& is, mizton::vector_3d& v) {
  int x = 0, y = 0, z = 0;

  /*
    In this case we don't allow negative numbers
    as coordinate values, so we will set the read operation
    as failed, so the calling code can handle the error.
  */
  if (is >> x >> y >> z) {
    if (x < 0 || y < 0 || z < 0) {
      is.setstate(std::ios::failbit); // Indicates input conversion fails
      std::cerr << "Negative coordinate value can't be converted to a mizton::vector_3d structure\n";
    } else {
      v.x = x;
      v.y = y;
      v.z = z;
    }
  } else {
    is.setstate(std::ios::badbit); // Indicates a loss of integrity
    std::cerr << "Lost data while trying to read a mizton::vector_3d structure\n";
  }

  return is;
}

int main() {
  // Mock input stream
  std::stringstream input_stream(R"(
    3
    1 0 1
    1 0 3
    5 6 1
  )");
  // Reading from input_stream
  int vector_size = 0;
  input_stream >> vector_size;
  assert(vector_size >= 0); // Ensure the first number read is positive
  std::vector<mizton::vector_3d> v_list(vector_size);
  for (int i = 0; i < vector_size; ++i) {
    /*
      If input reading fails, just ends the program
      normally performing a clean up of variables and
      calling destructors and return exit code of a failure.
    */
    if (!(input_stream >> v_list[i])) {
      return EXIT_FAILURE;
    }
  }
  // Writing data to standard output
  for (const auto& v : v_list) {
    std::cout << v;
  }
  return EXIT_SUCCESS;
}
```

After running the code above, you will get an output like this

```
vector{1,0,1}
vector{1,0,3}
vector{5,6,1}
```

If you change some of the values for vectors to negative, the program will ends and an error like this will be out

```
Negative coordinate value can't be converted to a mizton::vector_3d structure
```

## Conclusions

In real life, using streams and buffers in C++ can be really complex and allow us to create really performant systems, I have shared what I found useful to deal with, especially when I was working with online judges like **CodeForces** to test my solution.
