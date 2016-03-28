## 1.exe ##

This is a windows x86 binary.

Upon opening the binary in IDA I didn't see anything, so I searched for 'flag' and found this:

![](https://github.com/Jumboperson/camsctf-writeups/blob/master/Reversing-1/string_search.png?raw=true)

put brackets around that and it is the flag, `{not_encrypted_string_flag_whoooo}`
