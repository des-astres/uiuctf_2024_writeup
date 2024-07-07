# UIUCTF 2024 Writeup: X marked the spot

On 30 June 2024, I took part in my very first competitive CTF, as part of the APT42 team.  
The event was UIUCTF 2024, a jeopardy CTF with challenges in various categories.  
I picked a cryptography challenge titled "X marked the spot" that seemed beginner-friendly.

## Challenge

For this cryptography challenge, we were given two files.  

One file, called "ct", contained the following cipher text:  

```python
\x1d\n\x1c\x12\x16\x00\x11\x1fX\x106\x1b\x17S\x1e.\x1c\x0cZ.\x11\x12^\x03\x1c;\x0b\x04\x169^\x1d]T6\x05\nU5B\x06\x00HPCGK\x0c
```

The other file was a Python file containing the following code:  

```python
from itertools import cycle

flag = b"uiuctf{????????????????????????????????????????}"
# len(flag) = 48
key  = b"????????"
# len(key) = 8
ct = bytes(x ^ y for x, y in zip(flag, cycle(key)))

with open("ct", "wb") as ct_file:
    ct_file.write(ct)
```
## The code

The first step to solving this challenge is understanding the code.  

First, we have a byte string variable named "flag", whose content clearly resembles the flag format for this CTF.  

Secondly, we have "key", another byte string. Presumably, finding our key's value will help us get the flag.  

Then, we have this line:  
```python
ct = bytes(x ^ y for x, y in zip(flag, cycle(key)))
```
- The bytes() function returns a bytes object. It can create a bytes object (such as a byte string) or convert an object into a bytes object.
- The zip() function, to quote the official [Python documentation](https://docs.python.org/fr/3.10/library/functions.html#zip), will "iterate over several iterables in parallel, producing tuples with an item from each one." This stops when the shortest iterable is exhausted.
- The cycle() function, from the itertools module, returns each element from an iterable, while also saving a copy of that element. Upon reaching the end of the iterable, cycle() repeats this process with the saved copy of the iterable, returning (and copying) each element, indefinitely.
- " ^ " is the bitwise XOR (exclusive or) operator. XOR takes two bits; if they have differing values, the result of the operation is&nbsp;1. Else, the result is 0.
  - "x ^ y" will return the value obtained by XORing each bit in x with the corresponding bit in y.
  - For instance, in the ASCII table, 'a' has a binary value of 01100001, and 'b' has a binary value of 01100010. Thus, "a ^ b" is 00000011, which is the binary code for the end of text (EOT) character.

To summarize, this line creates a byte string (ct). It does so by iterating over *flag* and *key* (cycling through *key* until *flag* is exhausted) and getting the XOR of both strings, character by character.

Finally, the resulting byte string is written into a file.

## Finding the key

The ct file contains 48 characters. The flag's length is 48.  
We already know 8 of the characters that the flag must contain, due to the flag format being uiuctf{...}: the flag will begin with "uiuctf{" and end with "}". Conveniently, the key we need to find contains precisely 8 characters.  
Let's see what happens when we get the XOR of "uiuctf{}" and a second byte string, formed by the first seven characters + last character found in our ct file.

```python
def xor_bstrings(bstr1, bstr2):
    return bytes(x ^ y for x, y in zip(bstr1, bstr2))


key = xor_bstrings(b"\x1d\n\x1c\x12\x16\x00\x11\x0c", b"uiuctf{}")

print(f'Key: {key}')
```
The resulting key is: **"hciqbfjq"**.

## Decoding the cipher text

Now, we look for the XOR of the cipher text and the key we just found.  

```python
from itertools import cycle

ct = b"\x1d\n\x1c\x12\x16\x00\x11\x1fX\x106\x1b\x17S\x1e.\x1c\x0cZ.\x11\x12^\x03\x1c;\x0b\x04\x169^\x1d]T6\x05\nU5B\x06\x00HPCGK\x0c"

key = b"hciqbfjq"

flag = bytes(x ^ y for x, y in zip(ct, cycle(key)))

print(f'Flag: {flag}')
```
The resulting string is: **"uiuctf{n0s_ju5t_to3_st4rtXbut_4l57_th3_3nc!!!!!}"**. Could this be our flag?  

Seeing that the result fit the flag format, I assumed it was... and was somewhat disconcerted when it turned out not to be the case.

## Getting the flag

I did not get the chance to try other solutions before the CTF ended, but here are my thoughts as to what the actual flag may be.

"uiuctf{n0s_ju5t_to3_st4rtXbut_4l57_th3_3nc!!!!!}" reads like a leetspeak sentence, with underscores separating each word.
Except not all words are immediately decipherable, and there's a conspicuous 'X' where we reasonably could have expected an underscore.
What was this challenge's name again?

'X' is the 26th character in our false flag. In the original cipher text, ';' is the 26th character.  
How would our key change if that semi-colon had been meant to be decoded as an underscore?

The XOR of ';' and '_' is 'd'. Since we need to cycle through the key to decode the cipher text, we can determine that 'd' would be the second character in our key.  
Using "hdiqbfjq" as a key, we get the following:
"unuctf{n0t_ju5t_th3_st4rt_but_4l50_th3_3nd!!!!!}"

Taking care to respect the flag format, the correct flag is probably:
**uiuctf{n0t_ju5t_th3_st4rt_but_4l50_th3_3nd!!!!!}**
