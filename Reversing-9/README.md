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

```int main(unsigned short argc, char** argv)
{
	char str[] = "<encrypted crap>";
	char troystr[0x30];
	char* pString = str;
	if (check_debug())
	{
		memcpy(troystr, <troy crap>, 0x30);
	    pString = otherPtr;
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
