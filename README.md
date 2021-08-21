# haiku
Haiku is small and weird esoteric programming language.
It is visual language which means that it is using images as code to interpret. However not only color of pixels holds information about instructions but also placement
is important for execution. To make language even more weird it is using queue as its primary and only data storage. I have called it Haiku because programs made in it
look simplistic.

FILES
-----
Haiku source code is saved in ppm (image/x-portable-pixmap) binary bitmap format. It is supported by most of image editors. Images are interpreted as they are in interpreter.
Image resoltuion can vary from 10x10 to 999x999. Color depth is 24bit (3 bytes per pixel).

DATA STRUCTURE
--------------
Haiku source code has two sections: initial queue data section and code section. Initial queue data section is first row of image, data specified in this section (if
it is used) is preloaded to queue before start of program. First byte of this section should be FF, second FF (if you wnat to preload data) or 00 (if you want to start with
blank queue). Then rest of bytes is data preloaded to queue, however first 2 bytes are ignored if resolution of your image is less than 100x100, also preloading stops on first FF byte found.
Code section is just source code of your program.
Data Structure of Haiku source code:
1. PPM Header (13-15 bytes)
2. FF byte
3. 00 or FF (preload check)
4. preload data(first row, depends on resolution)(should be filled with 00 if unused)
5. code

CODE
----
Each instruction in Haiku is composed out of two pixel combinations on 3x3 grid. Area where interpreter expects to get second pixel is specified by color of first pixel.
For example if our program looks like this:  
 p 0 0  
 0 0 p  
 0 0 0  
where first p (top left) is colored pixel pointing to second pixel, this instruction is recognized as Print (pops first element from queue and prints it to screen).
Every pixel not used by program should be FF FF FF, white coloured. Color of pixel is composed of two colors: area color and pixel (command) color.
Area color determines 3x3 block where interpreter seeks for next pixel, command color specifies exact command to search for which means correct pixel.
After executing command interpreter jumps to second pixel of last command and treats it like new first pixel and then again jumps to next second pixel, that is
the interpreter loop. If first pixel is black (00 00 00) program execution will stop. Here are areas that could be specified by area color:  
a a e b b  
a a e b b  
f f p g g  
c c h d d  
c c h d d  
a - topleft area  
b - topright area  
c - downleft area  
d - downright area  
e - topleft or topright area (same pixels)  
f - topleft or downleft area (same pixels)  
g - downright or topright area (same pixels)  
h - downleft or downright area (same pixels)  
p - first pixel  
Area color is saved in red and green color byte of pixel, here is structure of bytes representing color of singular pixel:  
aa aa cc  
aa - area color  
cc - command color  
Valid area colors are:
00 00 - topright area  
00 FF - topleft area  
FF 00 - downright area
FF FF - downleft area  

COMMANDS
--------
There are 7 commands in Haiku, their description will have such table:  
a a a b b b  
a a a b b b  
a a p p b b  
c c p p d d  
c c c d d d  
c c c d d d  
Topleft 3x3 block of this table show correct pixel positions for topleft area (rest does accordingly) where p is first pixel and a,b,c and d can be either 0, r or x. 0 means no pixel (white), r means second pixel for this command and x is data pixel (for Put and If command).  
  
  
Print  
Color: 11 (xx xx 11)    
Errors: H  
0 0 0 0 0 0  
r 0 0 0 0 r  
0 0 p p 0 0  
0 0 p p 0 0  
r 0 0 0 0 r 
0 0 0 0 0 0  
Print pops first value from data queue and prints it to screen as ASCII character.  
  
Ask  
Color: 22 (xx xx 22)  
Errors: none  
0 0 r r 0 0  
0 0 0 0 0 0  
0 0 p p 0 0  
0 0 p p 0 0  
0 0 0 0 0 0  
0 0 r r 0 0  
Ask ask user for number from 0 to 255 and puts it in data queue.
  
Increment  
Color: 33 (xx xx 33)  
Errors: H  
0 0 0 0 0 0  
0 r 0 0 r 0  
0 0 p p 0 0  
0 0 p p 0 0  
0 r 0 0 r 0  
0 0 0 0 0 0  
Increment increments first value in data queue by one.
  
Increment  
Color: 44 (xx xx 44)  
Errors: H  
0 0 0 0 0 0  
0 0 0 0 0 0  
0 r p p r 0  
0 r p p r 0  
0 0 0 0 0 0  
0 0 0 0 0 0  
Decrement decrements first value in data queue by one.
  
Remove  
Color: 55 (xx xx 55)  
Errors: H  
0 0 0 0 0 0  
0 0 0 0 0 0  
r 0 p p 0 r  
r 0 p p 0 r  
0 0 0 0 0 0  
0 0 0 0 0 0  
Remove removes first element from data queue.
  
Put/Put Back  
Color: 66 (xx xx 66)  
Error: H  
0 r 0 0 r 0  
0 0 x x 0 0  
0 0 p p 0 0  
0 0 x x 0 0  
0 r 0 0 r 0  
Put if x pixel is white (00 00 00) copies first element from data queue and puts at the end of queue (Put Back). If x pixel is not white it puts value of first byte of x pixel at the end of data queue (Put).  
  
If  
Color: 77 (xx xx 77)  
Error: H A  
r 0 0 0 0 r  
0 0 x x 0 0  
0 0 p p 0 0  
0 0 p p 0 0  
0 0 x x 0 0  
r 0 0 0 0 r  
If checks if first and last element in queue are equal if they are code is executed normally (from r pixel) if they are not equal code jumps to x pixel and treats is as new first pixel (executes code from x pixel).  

ERRORS
------
There are five erros in Haiku. 3 are bitmap related and two are command related. If command can fail with command error this possible error is in error row in command description. When error occurs code execution is stopped and error code char is printed on screen.
Error list:  
1. Queue is empty. (Error code H) This error occurs when command tries to remove something from queue when queue is empty.
2. Queue is too small. (Error code A) This error occurs when queue is too small for If comparision.
3. Invalid Pixel. (Error code I) This error occurs when pixel is pointing to a pixel which is beyond bounds of bitmap.
4. White Pixel. (Error code K) This error occurs when next executed pixel is white.
5. Invalid Color. (Error code U) This error occurs when pixel has invalid color.

DATA
----
Data in Haiku is stored in single byte FIFO queue. Amount of data cells is not limited by program. Data printed to screen is printed as ASCII not numbers, which makes printing out numbers complicated (you can see that in cat program). However user inputs data in decimal format (to allow inputs bigger than 127 and smaller than 34).

SAMPLE PROGRAMS
---------------
hello.ppm  
This is a Hello World program. It is using queue preloading (you can see that first row of image is not empty). The entire code here are just two Print instruction pointing to each other, this is an infinite loop, however when whole queue is emptied the program will end with error code H. So in fact this is simpliest Hello World program, but not errorless  
  
cat.ppm  
This is a cat program, at least its simpliest form. It has queue preloading disabled and its code is composed out of two ask-print instruction. Instructions are pointing to each other so program is infinite. This program shows limitations of Haiku data management, user inputs data to program in decimal format, but output is in ASCII, so in fact this is a binary cat program. Also user can input only one byte (char) at a time.
  
if.ppm  
This program shows conditional abilities of Haiku. If user inputs 97 ('a' code in ASCII) program will finish. However if user inputs anything else program will output a and then user inputted data incremented by one. Remember about decimal input.
