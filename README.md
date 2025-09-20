# ASAP - Amiga Software Authoring Platform

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**Zero overhead, inlined C++ wrapper classes for the Amiga API**

ASAP provides a collection of 90+ header files, each containing one class that derives from the corresponding Amiga system structure. These classes add no data members, only inlined methods that call the corresponding global prototype functions, substituting the `this` pointer for the structure pointer.

## What is ASAP?

ASAP is designed for C++ programmers who think in terms of objects. The Amiga API is a "flat" API, and ASAP helps categorize these functions into logical "classes" or "categories". 

**Key Benefits:**
- **Zero runtime overhead** - when your compiler allows inline optimization, these are essentially `#define` macros
- **Type safety** - fewer bugs like `CloseWindow(pNotAWindow)`
- **Simplified includes** - just include what you need:
  ```cpp
  #include <ASAP/Window.h>
  #include <ASAP/RastPort.h>
  ```
- **Memory efficiency** - these classes *are* the Amiga structures, no additional storage

## How It Works

The classes are designed to be 100% compatible with Amiga structures. Consider this example:

```cpp
// Traditional Amiga API
struct Window *pThis_Window = OpenWindow(&The_NewWindow); 
CloseWindow(pThis_Window);

// With ASAP
AWindow *pThis_Window = new(&The_NewWindow) AWindow;
delete pThis_Window;
```

Each wrapper class inherits from its corresponding Amiga structure:

```cpp
class AStructure : public Structure
{
 public:
  void Open() { ::OpenStructure(this); }
  void Close() { ::CloseStructure(this); }
};
```

## Installation

### SAS/C
Move the ASAP classes directory into your `cxxinclude` directory:
```
sc:cxxinclude/ASAP/
```

### Storm C
Place the ASAP class directory wherever you want, ensuring it's in your include path.

### Usage
Include the headers as needed:
```cpp
#include <ASAP/HeaderName.h>
```

## Example

Here's a simple example from the included `Lines.c` demo:

```cpp
#include <ASAP/Window.h>
#include <ASAP/RastPort.h>
#include <ASAP/MsgPort.h>
#include <ASAP/IntuiMessage.h>

void main(void)
{
    NewWindow The_NewWindow = {
        100,100, 300,200,
        2,1,                    // detail, block pens
        IDCMP_CLOSEWINDOW | IDCMP_NEWSIZE | IDCMP_INTUITICKS,
        WINDOWDRAG | WINDOWCLOSE | GIMMEZEROZERO,
        NULL, NULL,
        (unsigned char *) "ASAP Lines",
        NULL, NULL,
        50,50,640,400,
        WBENCHSCREEN
    };

    AWindow *pThe_Window = new (&The_NewWindow) AWindow;
    
    if (pThe_Window) {
        ARastPort& The_RastPort = *(ARastPort *) pThe_Window->RPort;
        The_RastPort.SetDrMd(JAM1);
        
        // ... rest of your code ...
        
        delete pThe_Window;
    }
}
```

## Supported Compilers

- SAS/C
- Storm C
- GNU C++ (gcc)

## Library Coverage

ASAP includes wrappers for:
- Core Amiga libraries (exec, intuition, graphics, dos, etc.)
- Additional libraries (wizard.library, msql.library)
- 90+ wrapper classes covering most Amiga API functions

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details.

## Original Author

**Laurie Perrin** - 1998

## Acknowledgements

- Thanks to Matt Sergeant for testing ASAP with gcc and providing compilation instructions
- Thanks to Christophe Sollet for the msql.library wrapper
- Thanks to everyone who provided feedback and encouragement

## Historical Note

ASAP was originally released in 1998 as a beta version. It has been re-released under an MIT license as it is entirely implemented as inline header files and will always be tightly linked into any binary.