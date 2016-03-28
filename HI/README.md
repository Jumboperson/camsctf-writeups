## HI ##

This is a small x64 elf executable. 

Upon opening the binary you will see the main function:

![](https://github.com/Jumboperson/camsctf-writeups/blob/master/HI/first_main.png?raw=true)

Now simply convert that long integer to characters by pressing 'R' on it.

![](https://github.com/Jumboperson/camsctf-writeups/blob/master/HI/char_main.png?raw=true)

Reverse the string and get `{StRinGz` which the 0x7D below that is the final bracket, making `{StRinGz}` the flag.
