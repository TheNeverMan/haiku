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
the interpreter loop. If first pixel is black (00 00 00) program execution will stop.
