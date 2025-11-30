# Hero Writeup
by Azumi Yasukohchi

https://cybertalents.com/challenges/malware/hero

## Overview


## Category
Reversing Malware


## Approach


**1. Setting up the HackTheBox environment**

 I am going to use Kali Linux for this CTF



............................................................................................................................
.
.
............................................................................................................................
.
.
.............................................................................................................................
.
.
............................................................................................................................


**2. Unzipping and looking through the Hero.zip**



Unzipped:


<img width="536" height="384" alt="image" src="https://github.com/user-attachments/assets/c6356006-453c-4acb-be66-bdc1def5c210" />


............................................................................................................................
.
.
............................................................................................................................
.
.
.............................................................................................................................
.
.
............................................................................................................................



**3. Understanding general concept of "Hero.unknown"**

Seeing contents inside the file(I added comments so it's easier to understand):

```
# def gen(i):
#     return i ^ 11

  1           0 LOAD_CONST               0 (<code object gen at 0x7f1e8f527710, file "<dis>", line 1>)   # push code object for function gen
              2 LOAD_CONST               1 ('gen')                                                       # push the name "gen"
              4 MAKE_FUNCTION            0                                                               # create a function object gen
              6 STORE_NAME               0 (gen)                                                         # store it in variable name gen

# def gen2(i):
#     return 14 ** i

  4           8 LOAD_CONST               2 (<code object gen2 at 0x7f1e8f5277c0, file "<dis>", line 4>)  # push code object for function gen2
             10 LOAD_CONST               3 ('gen2')                                                      # push the name "gen2"
             12 MAKE_FUNCTION            0                                                               # create a function object gen2
             14 STORE_NAME               1 (gen2)                                                        # store it in variable name gen2

# f = open("flag.txt", "r")

  7          16 LOAD_NAME                2 (open)                                                        # load built-in function open
             18 LOAD_CONST               4 ('flag.txt')                                                  # push string "flag.txt"
             20 LOAD_CONST               5 ('r')                                                         # push string "r" (read mode)
             22 CALL_FUNCTION            2                                                               # call open('flag.txt', 'r')
             24 STORE_NAME               3 (f)                                                           # f = open("flag.txt", "r")

# o = []

  8          26 BUILD_LIST               0                                                               # build empty list []
             28 STORE_NAME               4 (o)                                                           # o = []

# r = f.readlines()[0]

  9          30 LOAD_NAME                3 (f)                                                           # load f
             32 LOAD_METHOD              5 (readlines)                                                   # load method f.readlines
             34 CALL_METHOD              0                                                               # call f.readlines()
             36 LOAD_CONST               6 (0)                                                           # push constant 0
             38 BINARY_SUBSCR                                                                              # index result[0]
             40 STORE_NAME               6 (r)                                                           # r = f.readlines()[0]

# for i in range(len(r)):
#     o.append(ord(r[i]))

 10          42 LOAD_NAME                7 (range)                                                       # load range
             44 LOAD_NAME                8 (len)                                                         # load len
             46 LOAD_NAME                6 (r)                                                           # load r
             48 CALL_FUNCTION            1                                                               # len(r)
             50 CALL_FUNCTION            1                                                               # range(len(r))
             52 GET_ITER                                                                                # get iterator over range(...)
        >>   54 FOR_ITER                22 (to 78)                                                       # for i in range(len(r)):
             56 STORE_NAME               9 (i)                                                           #   i = loop index

 12          58 LOAD_NAME                4 (o)                                                           #   load list o
             60 LOAD_METHOD             10 (append)                                                      #   load o.append
             62 LOAD_NAME               11 (ord)                                                         #   load ord
             64 LOAD_NAME                6 (r)                                                           #   load r (flag line)
             66 LOAD_NAME                9 (i)                                                           #   load i
             68 BINARY_SUBSCR                                                                            #   r[i]
             70 CALL_FUNCTION            1                                                               #   ord(r[i])
             72 CALL_METHOD              1                                                               #   o.append(ord(r[i]))
             74 POP_TOP                                                                                #   discard return value of append
             76 JUMP_ABSOLUTE           54                                                               # loop back to FOR_ITER

# s = []

 14     >>   78 BUILD_LIST               0                                                               # build empty list []
             80 STORE_NAME              12 (s)                                                           # s = []

# for i in range(len(o)):
#     t = gen(i)
#     f = gen2(t)
#     s.append(~(f * o[i]))

 15          82 LOAD_NAME                7 (range)                                                       # load range
             84 LOAD_NAME                8 (len)                                                         # load len
             86 LOAD_NAME                4 (o)                                                           # load o
             88 CALL_FUNCTION            1                                                               # len(o)
             90 CALL_FUNCTION            1                                                               # range(len(o))
             92 GET_ITER                                                                                # get iterator over range(...)
        >>   94 FOR_ITER                40 (to 136)                                                      # for i in range(len(o)):
             96 STORE_NAME               9 (i)                                                           #   i = loop index

 16          98 LOAD_NAME                0 (gen)                                                         #   load function gen
            100 LOAD_NAME                9 (i)                                                           #   load i
            102 CALL_FUNCTION            1                                                               #   gen(i)
            104 STORE_NAME              13 (t)                                                           #   t = gen(i)

 17         106 LOAD_NAME                1 (gen2)                                                        #   load function gen2
            108 LOAD_NAME               13 (t)                                                           #   load t
            110 CALL_FUNCTION            1                                                               #   gen2(t)
            112 STORE_NAME               3 (f)                                                           #   f = gen2(t)

 18         114 LOAD_NAME               12 (s)                                                           #   load list s
            116 LOAD_METHOD             10 (append)                                                      #   load s.append
            118 LOAD_NAME                3 (f)                                                           #   load f
            120 LOAD_NAME                4 (o)                                                           #   load o
            122 LOAD_NAME                9 (i)                                                           #   load i
            124 BINARY_SUBSCR                                                                            #   o[i]
            126 BINARY_MULTIPLY                                                                          #   f * o[i]
            128 UNARY_INVERT                                                                            #   ~(f * o[i])
            130 CALL_METHOD              1                                                               #   s.append(~(f * o[i]))
            132 POP_TOP                                                                                #   discard return value of append
            134 JUMP_ABSOLUTE           94                                                               #   loop back to FOR_ITER

# print(s)

 20     >>  136 LOAD_NAME               14 (print)                                                       # print(...)
            138 LOAD_NAME               12 (s)                                                           # load s
            140 CALL_FUNCTION            1                                                               # print(s)
            142 POP_TOP                                                                                # discard print result

# print(len(s))

 21         144 LOAD_NAME               14 (print)                                                       # print(...)
            146 LOAD_NAME                8 (len)                                                         # load len
            148 LOAD_NAME               12 (s)                                                           # load s
            150 CALL_FUNCTION            1                                                               # len(s)
            152 CALL_FUNCTION            1                                                               # print(len(s))
            154 POP_TOP                                                                                # discard print result
            156 LOAD_CONST               7 (None)                                                        # push None
            158 RETURN_VALUE                                                                             # return None from top-level code

# def gen(i):
#     return i ^ 11

Disassembly of <code object gen at 0x7f1e8f527710, file "<dis>", line 1>:
  2           0 LOAD_FAST                0 (i)                                                           # load argument i
              2 LOAD_CONST               1 (11)                                                          # load constant 11
              4 BINARY_XOR                                                                                # compute i ^ 11
              6 RETURN_VALUE                                                                             # return i ^ 11

# def gen2(i):
#     return 14 ** i

Disassembly of <code object gen2 at 0x7f1e8f5277

```



And this code is basically this


```
def gen(i):
    return i ^ 11

def gen2(i):
    return 14 ** i

f = open("flag.txt", "r")
o = []
r = f.readlines()[0]

for i in range(len(r)):
    o.append(ord(r[i]))

s = []
for i in range(len(o)):
    t = gen(i)              # t = i ^ 11
    f = gen2(t)             # f = 14 ** t
    s.append(~(f * o[i]))   # encrypt each character

print(s)
print(len(s))


```





What this script does:

1. Reads flag.txt
2. Converts each char to its ASCII code with ord()
3. For each index i, computes a big factor and then:
4. multiplies it by ord(char)\
5. applies bitwise NOT ~\
6. Stores that list in s and prints it


and the printed s list should be exactly what’s inside the flag.enc.

Since logic for s is what prints the flag, let's focus on that part.








............................................................................................................................
.
.
............................................................................................................................
.
.
.............................................................................................................................
.
.
............................................................................................................................



**4. Understanding the encryption formula**


```
s = []
for i in range(len(o)):
    t = gen(i)              # t = i ^ 11
    f = gen2(t)             # f = 14 ** t
    s.append(~(f * o[i]))   # encrypt each character
```

1. It creates an empty list s and loops over each position i of o (the ASCII codes of the flag characters).
2. For each i, it computes a per-position key by first scrambling the index t = i ^ 11, then raising 14 to that power: f = 14 ** t.
3. It encrypts the character by MULTIPY f * o[i], flipping all bits with ~, and appending the result to s --> producing the big negative integers.





............................................................................................................................
.
.
............................................................................................................................
.
.
.............................................................................................................................
.
.
............................................................................................................................


**5. Create a new formula by reversing the encryption formula**


```
# a = enc[i]  (the encrypted number at index i)
t = gen(i)                 # t = i ^ 11
f = gen2(t)                # f = 14 ** (i ^ 11)
val = ~a                   # undo NOT:   val = f * ord(char)
o_i = val // f             # undo *:     o_i = ord(char)
chars.append(chr(o_i))     # number -> character
```

Logic:
1. Rebuild the per-index key: t = i ^ 11, f = 14 ** t.
2. Undo the NOT: val = ~a --> val = f * ord(char). #a is encrypted number
3. Undo multiply (divide it instead): o_i = val // f, then chr(o_i) --> the character.


for example, let's say a = encrypted number = - 70001

a = -70001, which came from a = ~(1000 * 70).

Since ~x = -x - 1, the inverse is also ~ (it undoes itself).

So:

val = ~a 
       = ~(-70001) 
       = -(-70001) - 1 
       = 70001 - 1 
       = 70000


**val = 70000 **
    |
    V
    = f * o[i] 
    = 1000 * 70

so:

**f = 1000**
o[i] = 70




o_i = **val** // **f **
    = 70000 // 1000 
    = 70

chr(o_i) = chr(70) = F (ASCII)

-70001 --> 70000 --> 70 --> 'F' 



......................................................................................................................
.
.
............................................................................................................................
.
.
.............................................................................................................................
.
.
............................................................................................................................


**6. Finding the flag**


Now we know that it is located in: /opt/psychosis/diamorphine.ko

We will go to that directory and see if there would be any hints or text file I can read.

<img width="602" height="192" alt="image" src="https://github.com/user-attachments/assets/370784de-805e-489d-94ea-dbcb00f57ce0" />

I found the flag! Thank you for reading!

