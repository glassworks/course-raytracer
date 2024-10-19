# Class

Une classe est le principe central du paradigme de la programmation orientée objet et sera très utile pour l'architecture de notre raytracer.

Nous utiliserons des classes pour notre modélisation mathématique, ce qui facilitera grandement nos opérations.

Pour cet exemple, nous modéliserons une **couleur**, qui est simplement un vecteur (ou **tuple**) contenant trois valeurs : rouge, vert et bleu : 


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

Avez-vous déjà entendu parler d'un **pixel**? Il s'agit en fait d'un tuple contenant les trois valeurs flottantes pour le rouge, le vert et le bleu !

En C++, nous gérons les dépendances en spécifiant d'abord les *interfaces* de nos classes (en utilisant les fichiers **h** ), et l'implémentation réelle des interfaces dans les fichiers **cpp**.

Définissons notre interface pour représenter une couleur (dans `src/raymath/Color.hpp`) :


```cpp
#pragma once

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

  float R();
  float G();
  float B();

  Color operator+(Color const& col);
  Color& operator=(Color const& col);
  friend std::ostream & operator<<(std::ostream & _stream, Color const & col);
};
```

Nous définissons trois valeurs privées, `r`, `g` et `b`. Nous définissons également deux fonctions de construction :

- L'une qui initialise la couleur en noir
- Une autre qui prend trois paramètres et initialise la couleur en conséquence.
  
Il est toujours bon d'inclure une fonction**destructeur** - la fonction qui sera appelée lorsque l'instance de l'objet sera détruite (pour nettoyer la mémoire).

Nous avons défini deux **opérateurs** qui nous faciliteront la vie lors de l'utilisation de cette classe :

- `+` : qui nous permettra d'utiliser le symbole `+` entre deux objets de type `Color`, réalisant ainsi l'équivalent en couleur d'une addition.
- `<<` : qui nous permettra de sérialiser notre `Color` dans un `iostream` pour faciliter le débogage.

Regardons maintenant l'implémentation (dans `src/raymath/Color.cpp`) :


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

float Color::R()
{
  return r;
}

float Color::G()
{
  return g;
}

float Color::B()
{
  return b;
}

/**
 * Implementation of the + operator :
 * Adding two colors is done by just adding the different components together :
 * (r1, g1, b1) + (r2, g2, b2) = (r1 + r2, g1 + g2, b1 + b2)
 */
Color Color::operator+(Color const& col) {
  Color c;
  c.r = fmax(fmin(r + col.r, 1), 0);
  c.g = fmax(fmin(g + col.g, 1), 0);
  c.b = fmax(fmin(b + col.b, 1), 0);
  return c;
}

/**
 * Overriding the assignment operator
 */
Color& Color::operator=(Color const& col) {
  r = col.r;
  g = col.g;
  b = col.b;
  return *this;
}

/**
 * Here we implement the << operator :
 * We take each component and append it to he stream, giving it a nice form on the console
 */
std::ostream & operator<<(std::ostream & _stream, Color const & col) {  
  return _stream << "(" << col.r << "," << col.g << "," << col.b << ")";
}
```

## Compilation de cette bibliothèque

Nous voulons développer un certain nombre de classes comme `Color` dans notre projet. Il est utile de les regrouper dans une **librairie**.

Dans CMake, nous procédons de la manière suivante :

1. Ajoutez le fichier `src/raymath/CMakeLists.txt`, avec l'instruction de construire notre nouvelle classe :


```cmake
add_library(raymath 
  ${CMAKE_CURRENT_SOURCE_DIR}/Color.cpp
)
```

2. Modifiez notre fichier racine `CMakeLists.txt`.


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

## Utilisation de la bibliothèque

Nous pouvons maintenant utiliser notre classe `Color` dans notre projet :

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
