## 2.exe ##

This is a windows x86 binary.

This write up will go into detail in the process used to solve 2-8, with minor to no modifications for all of them. 

A quick bit of analysis in IDA leads to showing that there are 2 strings stored on the stack in the main function, a GlobalAlloc, and 2 decryption looking rountines which have a little bit of obfuscation.

![](https://github.com/Jumboperson/camsctf-writeups/blob/master/Reversing-2/enc_str_n_globalalloc.png?raw=true)

![](https://github.com/Jumboperson/camsctf-writeups/blob/master/Reversing-2/decryption_routines.png?raw=true)

Now the main issue with actually figuring out what these routines do is that I'm lazy and don't need to figure it out. I skimmed the functions are there are no syscalls and no references to library functions so its just decryption and no malicious code. Therefore I'll just breakpoint after both of them run and grab the flag from the stack.

![](https://github.com/Jumboperson/camsctf-writeups/blob/master/Reversing-2/flag.png?raw=true)

Thus when eip hits after the final decryption call, the flag is on the stack at `ebp-30` and we can submit `{cheating_out_101}` for the points.
