## 9.exe ##

This is a windows x86 binary.

I will be updating this problem as I solve it. Here is my preliminary analysis of the function:

![](https://github.com/Jumboperson/camsctf-writeups/blob/master/Reversing-9/first_view.png?raw=true)

Clearly it has some obfuscation that we will get rid of momentarily with some patches...

Also more encrypted strings. And a function call.

![](https://github.com/Jumboperson/camsctf-writeups/blob/master/Reversing-9/anti_debug.png?raw=true)

The function called checks the isDebugged member in the PEB and returns true if its being debugged.

I since named this function `debug_check`.

![](https://github.com/Jumboperson/camsctf-writeups/blob/master/Reversing-9/more_setup.png?raw=true)

Moving on the debug check looks like it procs the copy of some memory onto the stack where strings were setup and changing of a ptr to the strings on the stack. 

Also a continued setup of strings and another copy of memory to places on the stack where strings are.

![](https://github.com/Jumboperson/camsctf-writeups/blob/master/Reversing-9/wow_more_stuff.png?raw=true)

So we're gonna have to nop that `ExitProcess` but in the meantime there are some MessageBoxes that think they're cool. Hints n stuff.

Also some `call eax` stuff right after setting eax to 0, again more nops incoming.

![](https://github.com/Jumboperson/camsctf-writeups/blob/master/Reversing-9/more_and_more.png?raw=true)

Continuing we have more copies and some aborts (incoming nops) and then finally some hints of the normal algo, GlobalAlloc, then decryption methods.

Immediately after that the function ends.

So lets try with all the nops we want to see what happens...

![](https://github.com/Jumboperson/camsctf-writeups/blob/master/Reversing-9/2_method_first.png?raw=true)

So after it hits that breakpoint the text is something telling me that "Troy" has messed up the flag.

I figure this is what overwrites the normal flag with all those copies so lets try removing them...

![](https://github.com/Jumboperson/camsctf-writeups/blob/master/Reversing-9/nope.png?raw=true)

Well shoot, after nopping those I still don't get anything.

Let us try the same thing but also nopping the pointer change in the debug check which changes what gets decrypted at the end...

Not even gonna add a picture, still nothing. Perhaps I'm doing it wrong, time to overkill...

So I'm going to copy all the pertinent parts of the code into a project of my own to attempt to solve this mess.

![](https://github.com/Jumboperson/camsctf-writeups/blob/master/Reversing-9/decrypt_routine_call.png?raw=true)

This is a call to a decryption routine, its __cdecl and takes 3 params. In order the block to decrypt, the length of the block, and the GlobalAlloc ptr.

Both routines follow this format.

This means the code for my project is going to look kinda like this:

```
int main(unsigned short argc, char** argv)
{
    char str[] = "<encrypted crap>";
    char troystr[0x30];
    char* pString = str;
    if (check_debug())
    {
        memcpy(troystr, <troy crap>, 0x30);
        pString = troystr;
    }
    char otherStr[] = "<more encrypted stuff>";
    char otherTroyStr[0x30]; // ?
    if (check_debug())
    {
        memcpy(otherTroyStr, <other troy crap>, 0x30);
    }
    // MessageBox cancer //
    // Call that prints the string (I think its cout based on the way the call looks, not sure) //
    char buff00[0x30];
    char buff01[0x30];
    memcpy(buff00, <something>, 0x2f);
    memcpy(buff01, <something else>, 0x2f);
    // Another MessageBox //
    char buff02[0x30];
    char buff03[0x30];
    memcpy(buff02, <something else else>, 0x2f);
    memcpy(buff03, <something else else else>, 0x2f);
    HGLOBAL hGlob = GlobalAlloc(0x40, 0x102);
    size_t stLength00 = strlen(buff03);
    first_decrypt(buff03, stLength00, hGlob);
    size_t stLength01 = strlen(buff02);
    second_decrypt(buff02, stLength01, hGlob);
    // Another MessageBox //
    return <MessageBoxRetValue>;
}
```

Now what I noticed when converting this to C is that the decrypts aren't on the stuff originally created on the stack like they are for the rest of reversing problems...

Also no part of the original stuff is overwritten, just it decrypts the wrong one. Meaning we probably need to change the first parameter of the decrypts to the buffers that are used.

I'm gonna be testing real quick and then I'll update again...

### Multi-hour break ###

So I wrote a bunch of code to try to replicate what this program does, I've tried swapping out dec1 and dec2 with stuff but to no avail... Not sure what to do.

```
#include "stdafx.h"
#include <Windows.h>
#include <stdio.h>

void byte_swap(char* loc01, char* loc02)
{
    char temp = *loc01;
    *loc01 = *loc02;
    *loc02 = temp;
}

void first_decrypt(char* pBuff, size_t stLen, HGLOBAL hGlobal)
{
    for (int i = 0; i < 256; ++i)
    {
        ((char*)hGlobal)[i] = i;
    }
    ((char*)hGlobal)[256] = 0;
    ((char*)hGlobal)[257] = 0;

    int index = 0;
    int sum01 = 0;
    for (int i = 0; i < 256; ++i)
    {
        sum01 += ((char*)hGlobal)[i] + pBuff[index];
        byte_swap(&((char*)hGlobal)[i], &((char*)hGlobal)[sum01]);
        index = (index + 1) % stLen;
    }
}

void second_decrypt(char* pBuff, size_t stLen, HGLOBAL hGlobal)
{
    int sec_to_last_byte = ((char*)hGlobal)[256];
    int last_byte = ((char*)hGlobal)[257];

    for (int i = 0; i < stLen; ++i)
    {
        char temp = (sec_to_last_byte + 1) & 0x800000ff;
        sec_to_last_byte = temp;
        last_byte += ((char*)hGlobal)[temp];
        byte_swap(&((char*)hGlobal)[temp], &((char*)hGlobal)[last_byte]);
        pBuff[i] ^= ((char*)hGlobal)[((char*)hGlobal)[last_byte] + ((char*)hGlobal)[sec_to_last_byte]];
    }
    ((char*)hGlobal)[256] = sec_to_last_byte;
    ((char*)hGlobal)[257] = last_byte;
}

bool check_debug()
{
    // Pretend we're in a debugger.
    return true;
}

int main(unsigned short argc, char** argv)
{
    char str[] = "\xb3\xff\xf1\x22\x12\xef\xfe\xf8\x05\xae\x36\x88\xfxb1\x92\x38\xed\xf8\x00\x00\x00";
    char troystr[0x30];
    char* pString = str;

    if (check_debug())
    {
        memcpy(troystr, "!\x11\xB7K\x13C\x91)\x85;\xB85aA\x04J\xDA\xCA\x84\xBB\xDA>\xA6\xB0y\xAC\x8F\xF8\xE5\x90\xC9\x9D\x96]hW\xB9\xA1\x818\x1C\x01!|\xAB\x8F\xED\x00", 0x30);
        pString = troystr;
    }
    
    char otherStr[] = "\x1b\x08\x7c\x7f\x11\x7e\x10\x59\x32\x6e\x48\x30\x5a\x64\x3c\x40\x17\x28\x00\x00\x00";
    char otherTroyStr[0x30]; // ?
    if (check_debug())
    {
        memcpy(otherTroyStr, "pICN{?&\v\x10E.8\t\x11*F\n~\f&e\x18PUVn\x15W\x19C\tffec\vWYnD8.h.|b<\x00", 0x30);
    }
    
    // MessageBox cancer //
    
    // Call that prints the string (I think its cout based on the way the call looks, not sure) //
    
    char buff00[0x30];
    char buff01[0x30];
    
    memcpy(buff00, "\xB0q\xD4\xDDh)\x1D\xCC\xDC\vn\xC3\x94\xF3[\xAA'?!\x1A\xBAw\x13\x97L\xAD\xE4x\xB7<\xB0\xB9\xB6!s;2F}\x846\xA9\xAEu\xE3\xD3\x00", 0x2f);
    memcpy(buff01, "\xB0q\xD4\xDDh)\x1D\xCC\xDC\vn\xC3\x94\xF3[\xAA'?!\x1A\xBAw\x13\x97L\xAD\xE4x\xB7<\xB0\xB9\xB6!s;2F}\x846\xA9\xAEu\xE3\xD3\x00", 0x2f);
    
    // Another MessageBox //
    
    char buff02[0x30];
    char buff03[0x30];
    
    memcpy(buff02, "\xB0q\xD4\xDDh)\x1D\xCC\xDC\vn\xC3\x94\xF3[\xAA'?!\x1A\xBAw\x13\x97L\xAD\xE4x\xB7<\xB0\xB9\xB6!s;2F}\x846\xA9\xAEu\xE3\xD3\x00", 0x2f);
    memcpy(buff03, "m\x03`\x01!\x0F\x10Vb\x0E\x1A\x1A6rkbA?qw\x15\x04EIV[\x06c\x04G9|R\x06%X7Hj*\x05]\f \x18\x04\x00", 0x2f);
    
#define dec1 pString
#define dec2 otherTroyStr
    HGLOBAL hGlob = GlobalAlloc(0x40, 0x102);
    
    size_t stLength00 = strlen(dec1);
    first_decrypt(dec1, stLength00, hGlob);
    
    size_t stLength01 = strlen(dec2);
    second_decrypt(dec2, stLength01, hGlob);
    
    // Another MessageBox //
    printf_s("%s\n%s\n%s\n", pString, otherStr, (char*)hGlob);
    return 0;
}
```

