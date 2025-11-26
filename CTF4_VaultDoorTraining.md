# vault-door-training
by Azumi Yasukohchi

https://play.picoctf.org/practice/challenge/7?category=3&difficulty=1&page=1 

## Overview
This is an easy CTF challenge that helps challengers build their code-reading skills by analyzing a simple program to uncover the solution.



## Category
Reverse Engineering 



## Approach

**1. Looking inside the source code**

<img width="1024" height="695" alt="image" src="https://github.com/user-attachments/assets/2c315b6f-49fe-4627-b0a9-13b4f00829ca" />



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

**2. Answering to a question inside the code**

<img width="1003" height="114" alt="image" src="https://github.com/user-attachments/assets/03d752d0-53f0-4a1f-9ad8-d28b1375b5fc" />

Answer: It is not safe to hardcode passwords in the source code because anyone who gets the code or reverse engineers the program can extract them. In real systems, passwords should be stored securely on the server side, using hashed values or proper authentication mechanisms, not inside client-side code. 


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



**3. Finding the flag inside the source code**


<img width="852" height="48" alt="image" src="https://github.com/user-attachments/assets/8f4d9474-4ac7-4952-8f6b-4dfc7ab0da72" />


There was the flag right inside the source code, no complicted steps. Only thing I need to do was to read the code.

<img width="300" height="139" alt="image" src="https://github.com/user-attachments/assets/8d6f9556-cab1-4a9f-81b8-b4712731edde" />


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


Thanks for reading!
