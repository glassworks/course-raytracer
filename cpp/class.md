# Class

A class is the central tenet of the Object Oriented programming paradigm, and will be very useful for the architecture of our raytracer.

We will use classes for our mathematical modelling, and make our operations much easier to perform.

For this example, we will model a **color**, which is just a vector (or **tuple**) contining three values: red, green and blue: 

```
(r, g, b)

Red = (1, 0, 0)
Green = (0, 1, 0)
Blue = (0, 0, 1)
Yellow = (1, 1, 0)
....
White = (1, 1, 1)
Black = (0, 0, 0)
```

Have you ever heard of a **pixel**? It is in fact just one tuple containing the three float values for red, green and blue !


In C++ we manage dependencies by first specifying *interfaces* of our classes (using **h** files), and the actual implementation of the interfaces in the **cpp** files.

Let us define our interface for representing a Color (in `src/raymath/Color.hpp`) :


```cpp
#include <iostream>

class  Color
{
private:
  float r = 0;
  float b = 0;
  float g = 0;
public:
  Color();
  Color(float r, float g, float b);
  ~ Color();

  Color operator+(Color const& col);
  friend std::ostream & operator<<(std::ostream & _stream, Color const & col);
};

```

We define three private values, `r`, `g` and `b`. We also define two contructor functions :

- One that will initialise the color to black
- One that takes three parameters and initialises the color accordingly
  
It is also always good practice to include a **destructor* function - the function that will be called when the object instance is destroyed (to clean up memory).

We have defined two **operators** which will make our life easier when using this class :

- `+`  : which will allow us to use the symbol `+`  between two objects of type `Color`, performing the color equivalent of addition
- `<<` : which will allow us to serialize our `Color` to an `iostream` for easier debugging.

Now let us look at the implementation (in `src/raymath/Color.cpp`) :


```cpp
#include <iostream>
#include "Color.hpp"

Color:: Color() : r(0), b(0), g(0)
{  
}

Color:: Color(float iR, float iG, float iB) : r(iR), g(iG), b(iB)
{  
}

Color::~ Color()
{
}

/**
 * Implementation of the + operator :
 * Adding two colors is done by just adding the different components together :
 * (r1, g1, b1) + (r2, g2, b2) = (r1 + r2, g1 + g2, b1 + b2)
 */
Color Color::operator+(Color const& col) {
  Color c;
  c.r = r + col.r;
  c.g = g + col.g;
  c.b = b + col.b;
  return c;
}

/**
 * Here we implement the << operator :
 * We take each component and append it to he stream, giving it a nice form on the console
 */
std::ostream & operator<<(std::ostream & _stream, Color const & col) {  
  return _stream << "(" << col.r << "," << col.g << "," << col.b << ")";
}
```

Note that we provide actual implementations of the functions we defined in the header file !

## Compilation of this library

We want to develop a number of classes like `Color` in our project. It is useful to group them into a **library**.

In CMake, we do this as follows:

1. Add the file `src/raymath/CMakeLists.txt`, with the instruction to build our new class :

```cmake
add_library(raymath 
  ${CMAKE_CURRENT_SOURCE_DIR}/Color.cpp
)
```

2. Modify our root `CMakeLists.txt` file :


```cmake
cmake_minimum_required(VERSION 3.5.0)
project(raytracer VERSION 0.1.0 LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

add_executable(raytracer main.cpp)

# Add these lines
target_include_directories(raytracer PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           "${PROJECT_SOURCE_DIR}/src/raymath"
                           )

add_subdirectory(./src/raymath)

target_link_libraries(raytracer PUBLIC raymath)
```

## Using the library

Now we can use our `Color` class in our project :

```cpp
#include <iostream>

// Include our class definition - we can read it thanks to `target_include_directories`
#include "Color.hpp"

using namespace std;

int main()
{    
    Color red(1, 0, 0);
    Color green(0, 1, 0);
    Color black;

    // We can chain our color instances using << thanks to our operator !
    cout << "Red : " << red << std::endl;
    cout << "Green : " << green << std::endl;
    cout << "Black : " << black << std::endl;

    // We can perform an addition + operation thanks to our operator !
    Color yellow = red + green;

    cout << "Yellow : " << yellow << std::endl;    
}

```
