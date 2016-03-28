## 8.exe ##

This is a windows x86 binary.

This can also be solved using the same method as `2.exe`, except that it requires one thing to be nop'd before it can be solved.

`ExitProcess` is called before the decryption therefore it must be nop'd.

![](https://github.com/Jumboperson/camsctf-writeups/blob/master/Reversing-8/not_yet_nopped.png?raw=true)

Breakpoint after the decryption:

![](https://github.com/Jumboperson/camsctf-writeups/blob/master/Reversing-8/flag.png?raw=true)

The flag is `{mem_is_beyond_good}`.
