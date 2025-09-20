# ASAP Usage Guide

This guide provides comprehensive instructions on how to use the ASAP (Amiga Software Authoring Platform) C++ wrapper library for Amiga development.

## Table of Contents

1. [Project Structure](#project-structure)
2. [Installation](#installation)
3. [Basic Usage Patterns](#basic-usage-patterns)
4. [Complete Example Walkthrough](#complete-example-walkthrough)
5. [Common Patterns](#common-patterns)
6. [Compilation Instructions](#compilation-instructions)
7. [Troubleshooting](#troubleshooting)

## Project Structure

```
ASAP/
├── src/                    # Main header files (90+ wrapper classes)
│   ├── Window.h           # AWindow wrapper
│   ├── RastPort.h         # ARastPort wrapper
│   ├── FileHandle.h       # AFileHandle wrapper
│   ├── Library.h          # ALibrary wrapper
│   └── ...                # 85+ other wrappers
├── src/unittest/          # Example code
│   ├── LINES.C            # Complete working example
│   └── readme.txt         # Compilation instructions
├── archive/               # Original archives
└── README.md              # Project overview
```

## Installation

### For SAS/C
1. Ensure INCLUDE: has SDK:Include_h/ in its path (as per ToolKit SDK) and the ASAP includes are copied there
2. Ensure your compiler can find the headers

### For Storm C
1. Place the `src/` directory anywhere accessible
2. Add the directory to your include path

### For GCC
1. Place the `src/` directory in your include path
2. Use `-I` flag to specify the path: `-I/path/to/ASAP/src`

## Basic Usage Patterns

### 1. Include Headers
Include only the ASAP headers you need:

```cpp
#include <ASAP/Window.h>      // For AWindow class
#include <ASAP/RastPort.h>    // For ARastPort class
#include <ASAP/MsgPort.h>     // For AMsgPort class
#include <ASAP/IntuiMessage.h> // For AIntuiMessage class
```

### 2. Object Creation with Placement New
ASAP uses placement new to create objects from existing Amiga structures:

```cpp
// Traditional Amiga API
NewWindow newWin = { /* ... */ };
struct Window *win = OpenWindow(&newWin);
CloseWindow(win);

// With ASAP
NewWindow newWin = { /* ... */ };
AWindow *win = new(&newWin) AWindow;
delete win;  // Automatically calls CloseWindow()
```

### 3. Method Calls
Call methods directly on ASAP objects:

```cpp
AWindow *win = new(&newWin) AWindow;
win->ActivateWindow();
win->MoveWindow(10, 20);
win->SizeWindow(100, 50);
```

### 4. Accessing Nested Structures
Access nested Amiga structures through ASAP objects:

```cpp
AWindow *win = new(&newWin) AWindow;
ARastPort& rp = *(ARastPort*)win->RPort;
rp.SetAPen(1);
rp.Move(10, 10);
rp.Draw(100, 100);
```

## Complete Example Walkthrough

Let's analyze the `LINES.C` example step by step:

### Step 1: Includes
```cpp
#include <stdlib.h>           // Standard C library
#include <ASAP/Window.h>      // Window management
#include <ASAP/RastPort.h>    // Graphics operations
#include <ASAP/MsgPort.h>     // Message handling
#include <ASAP/IntuiMessage.h> // Intuition messages
```

### Step 2: Window Setup
```cpp
NewWindow The_NewWindow = {
    100,100, 300,200,         // Left, Top, Width, Height
    2,1,                      // Detail pen, Block pen
    IDCMP_CLOSEWINDOW | IDCMP_NEWSIZE | IDCMP_INTUITICKS,  // IDCMP flags
    WINDOWDRAG | WINDOWCLOSE | GIMMEZEROZERO,              // Window flags
    NULL, NULL,               // Gadgets, Checkmark
    (unsigned char *) "ASAP Lines",  // Title
    NULL, NULL,               // Screen, Bitmap
    50,50,640,400,            // Min/Max dimensions
    WBENCHSCREEN              // Screen type
};
```

### Step 3: Create Window Object
```cpp
AWindow *pThe_Window = new (&The_NewWindow) AWindow;
if (pThe_Window) {
    // Window created successfully
}
```

### Step 4: Access RastPort
```cpp
ARastPort& The_RastPort = *(ARastPort *) pThe_Window->RPort;
The_RastPort.SetDrMd(JAM1);  // Set draw mode
```

### Step 5: Message Loop
```cpp
do {
    AMsgPort& The_MsgPort = *(AMsgPort *) pThe_Window->UserPort;
    The_MsgPort.WaitPort();  // Wait for messages
    
    AIntuiMessage *pThis_IntuiMessage;
    do {
        pThis_IntuiMessage = (AIntuiMessage *) The_MsgPort.GetMsg();
        if (pThis_IntuiMessage) {
            ULONG messageClass = pThis_IntuiMessage->Class;
            pThis_IntuiMessage->ReplyMsg();
            
            switch (messageClass) {
                case IDCMP_NEWSIZE: 
                    // Handle window resize
                    break;
                case IDCMP_INTUITICKS: 
                    DrawLines(The_RastPort, xlim, ylim);
                    break;
            }
        }
    } while (pThis_IntuiMessage);
} while (messageClass != IDCMP_CLOSEWINDOW);
```

### Step 6: Cleanup
```cpp
delete pThe_Window;  // Automatically calls CloseWindow()
```

## Common Patterns

### File Operations
```cpp
#include <ASAP/FileHandle.h>
#include <ASAP/FileLock.h>

// Open a file
AFileHandle *file = new("test.txt", MODE_READWRITE) AFileHandle;
if (file) {
    file->Write("Hello World", 11);
    delete file;  // Automatically closes file
}

// Directory operations
AFileLock *lock = new("SYS:", SHARED_LOCK) AFileLock;
if (lock) {
    // Use lock for directory operations
    delete lock;
}
```

### Library Management
```cpp
#include <ASAP/Library.h>

// Open a library
ALibrary *lib = new("graphics.library", 37) ALibrary;
if (lib) {
    // Use library functions
    delete lib;  // Automatically closes library
}
```

### Graphics Operations
```cpp
#include <ASAP/RastPort.h>
#include <ASAP/ColorMap.h>

// Set up graphics
ARastPort *rp = new(gadgetInfo) ARastPort;
rp->SetAPen(1);
rp->SetBPen(0);
rp->SetDrMd(JAM1);

// Draw operations
rp->Move(10, 10);
rp->Draw(100, 100);
rp->RectFill(50, 50, 150, 150);
```

### BOOPSI Objects
```cpp
#include <ASAP/Object.h>
#include <ASAP/IClass.h>

// Create BOOPSI object
AObject *obj = new("window.class", WA_Width, 320, WA_Height, 200, TAG_DONE) AObject;
if (obj) {
    obj->SetAttrs(WA_Title, (ULONG)"My Window", TAG_DONE);
    delete obj;
}
```

## Compilation Instructions

### SAS/C
```bash
sc cxxsrc myprogram.c link
```

### Storm C
```bash
stormc myprogram.c -o myprogram
```

### GCC
```bash
gcc -noixemul -o myprogram myprogram.c
# Or with optimizations:
gcc -O2 -m68030 -noixemul -msmall-code -o myprogram myprogram.c
```

### Makefile Example
```makefile
CC = gcc
CFLAGS = -O2 -m68030 -noixemul -msmall-code
INCLUDES = -I./ASAP/src

myprogram: myprogram.c
	$(CC) $(CFLAGS) $(INCLUDES) -o myprogram myprogram.c
```

## Troubleshooting

### Common Issues

1. **Missing New.h**: Some classes require `#include <New.h>` - this appears to be a missing dependency that should be provided by your compiler.

2. **Include Path**: Ensure the ASAP `src/` directory is in your include path.

3. **Compiler Compatibility**: ASAP was designed for SAS/C, Storm C, and GCC. Other compilers may have issues.

4. **Memory Management**: Always use `delete` on ASAP objects to ensure proper cleanup.

5. **Type Casting**: Some operations require explicit casting, especially when accessing nested structures.

### Debug Tips

1. **Check Return Values**: Always check if object creation succeeded:
   ```cpp
   AWindow *win = new(&newWin) AWindow;
   if (!win) {
       // Handle error
   }
   ```

2. **Use References**: For frequently accessed objects, use references:
   ```cpp
   ARastPort& rp = *(ARastPort*)win->RPort;
   ```

3. **Memory Leaks**: Ensure every `new` has a corresponding `delete`.

## Advanced Usage

### Custom Wrapper Classes
You can extend ASAP by creating your own wrapper classes:

```cpp
class AMyCustom : public MyCustomStruct
{
public:
    inline void MyMethod() { ::MyFunction(this); }
    inline static AMyCustom* Create() { 
        return (AMyCustom*)::CreateMyCustom(); 
    }
};
```

### Template Usage
Some ASAP classes support template-like usage:

```cpp
// For classes with different types
AAslRequest *req = new(FILEREQUEST, tags) AAslRequest;
```

This guide should get you started with ASAP development. The library provides a clean, object-oriented interface to the Amiga API while maintaining zero runtime overhead.
