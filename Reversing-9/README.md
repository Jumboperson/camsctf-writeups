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

(I will continue to update this, taking a break to eat...)

