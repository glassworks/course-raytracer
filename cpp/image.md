# Écrire une image

L'objectif ultime de ce projet est d'écrire une série de pixels (Couleurs) dans un fichier, par exemple un fichier PNG.

Nous pouvons modéliser un **fichier** comme une grille 2D de pixels ou de couleurs. En réalité, cette grille n'est qu'un long tableau de pixels `largeur * hauteur`.

Voici donc une façon de modéliser notre fichier image (dans `src/rayimage/Image.hpp`) :

```cpp
#pragma once

#include <iostream>
#include "../raymath/Color.hpp"

class Image
{
private:
  unsigned int width = 0;
  unsigned int height = 0;
  std::vector<Color> buffer;
public:
  Image(unsigned int w, unsigned int h);
  Image(unsigned int w, unsigned int h, Color c);
  ~ Image();

  void SetPixel(unsigned int x, unsigned int y, Color color);
  Color GetPixel(unsigned int x, unsigned int y);

  void WriteFile(const char* filename);
};
```

Et voici l'implémentation (`src/rayimage/Image.cpp`) :

```cpp
#include <iostream>
#include "Image.hpp"
#include "../lodepng/lodepng.h"


Image:: Image(unsigned int w, unsigned int h) : width(w), height(h)
{  
  buffer.resize(width * height);
  for (int i = 0; i < buffer.size(); ++i) {
    buffer[i] = Color();
  }
}

Image:: Image(unsigned int w, unsigned int h, Color c) : width(w), height(h)
{  
  buffer.resize(width * height);
  for (int i = 0; i < buffer.size(); ++i) {
    buffer[i] = c;
  }
}

Image::~ Image()
{
}

void Image::SetPixel(unsigned int x, unsigned int y, Color color) {
  unsigned int index = (y * width) + x;

  if (index >= buffer.size()) { throw std::invalid_argument("Image: Invalid index"); }
  buffer[index] = color;
}

Color Image::GetPixel(unsigned int x, unsigned int y) {
  unsigned int index = (y * width) + x;

  if (index >= buffer.size()) { throw std::invalid_argument("Image: Invalid index"); }
  return buffer[index];
}


void Image::WriteFile(const char * filename) {
  std::vector<unsigned char> image;
  image.resize(width * height * 4);
  for(unsigned index = 0; index < buffer.size(); index++) {
    Color pixel = buffer[index];
    int offset = index * 4;

    image[offset] = (unsigned int)floor(pixel.R() * 255); 
    image[offset + 1] = (unsigned int)floor(pixel.G() * 255); 
    image[offset + 2] = (unsigned int)floor(pixel.B() * 255); 
    image[offset + 3] = 255;      // Alpha
  }

  //Encode the image
  unsigned error = lodepng::encode(filename, image, width, height);

  //if there's an error, display it
  if(error) std::cout << "encoder error " << error << ": "<< lodepng_error_text(error) << std::endl;
}
```

Comment fonctionne ce code ?

1. Nous utilisons un constructeur qui réserve une liste d'objets `Color`. Le nombre d'éléments dans la liste correspond au nombre total de pixels dans l'image.
2. Nous pouvons définir des pixels spécifiques à partir des coordonnées `x` et `y`.
3. Nous pouvons sauvegarder cette grille dans une image. J'utilise l'outil [lodepng](https://lodev.org/lodepng/) que j'ai téléchargé et placé dans le répertoire `src/lodepng`.

Rappelez-vous qu'un fichier image n'est qu'une liste de pixels, et que chaque pixel n'est qu'une liste de 3 nombres (un octet chacun). Donc notre fonction write va juste boucler sur chaque `Color`, et pour chaque composante de la couleur, convertir la valeur float en une valeur byte et l'insérer dans le tableau de l'image finale. Un fichier PNG a une quatrième composante, alpha, que nous n'utilisons pas dans cet exemple, donc nous la fixons simplement à sa valeur maximale.

{% hint style="success" %}
Je vous laisse le soin de configurer les fichiers CMakeLists.txt ! C'est le même processus que nous avons utilisé pour la classe Color.
{% endhint %}

Maintenant que nous avons notre outil d'image, testons-le en étendant notre `main.cpp` :

```cpp
#include <iostream>
#include "Color.hpp"
#include "Image.hpp"

using namespace std;


int main()
{    
    Color red(1, 0, 0);
    Color green(0, 1, 0);
    Color black;

    cout << "Red : " << red << std::endl;
    cout << "Green : " << green << std::endl;
    cout << "Black : " << black << std::endl;

    Color yellow = red + green;

    cout << "Yellow : " << yellow << std::endl;

    // Create an image in memory, and fill it with yellow
    Image image(512, 512, yellow);

    // Make a red square on the top left of the image
    for (int y = 0; y < 100; y++) {
      for (int x = 0; x < 100; x++) {
        image.SetPixel(x, y, Color(1, 0, 0));
      }
    }

    image.WriteFile("test.png");
}
```
