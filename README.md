<h2>Overwriting entries in the Global Offset Table using a vulnerable printf function</h2>
This write-up builds on what was learned about exploiting printf vulnerabilities in my write-up on the picoCTF <a href="https://github.com/tlkroll/format-string-exploitation/blob/main/README.md">format string 2</a> challenge. In the format string 3 challenge there is a similar vulnerability, but this time instead of overwriting a variable within the program it will be used to overwrite a pointer to an entry in the Global Offset Table so that it will call a different libc function.
<br><br>
<h2>format string 3</h2>
When the program is run, it prints a line which leaks the address of the GOT entry for the <b>setvbuf</b> function and then waits for input:<br><br>
<img src="/fs3-1.png">
<br>
After supplying some input, it then prints two lines; one which repeats the input and another which prints <b>/bin/sh</b>:<br><br>
<img src="/fs3-2.png">
<br>
Here is the source code for this function:<br><br>
<img src="/fs3-3.png">
<br>
Looking at the source, we can identify two goals. The main objective will be to overwrite the pointer to the <b>puts</b> function with a pointer to the <b>system</b> function so that instead of printing <b>/bin/sh</b>, the program will make a <b>system</b> call to <b>/bin/sh</b> instead, granting us a shell. To accomplish this, we will need to exploit the vulnerable printf function which repeats user input.
<br><br>
<h2>The Global Offset Table</h2>
My understanding of the GOT is that it is a copy of libc used by a program which is stored in a random memory location at runtime in order to prevent the type of attack we are performing here. But because we have access to the copy of libc being used and the location of <b>setvbuf</b> has been leaked, we are able to calculate offsets and find the memory locations of other functions within libc.<br><br>
Finding the locations of functions and calculating their offsets can all be accomplished using pwntools, but I wanted to know exactly what was going on under the hood, so I took a look at the libc file which was supplied with this challenge. First, I used the <b>readelf</b> function to find the location of <b>setvbuf</b>:<br><br>
<img src="/fs3-4.png">
<br>
Then, I found the location of <b>system</b>:<br><br>
<img src="/fs3-5.png">
<br>
Finally, I found the difference of the two, which is <b>2ac90</b>:<br><br>
<img src="/fs3-6.jpg">
<br>
To test this theory, I ran the program in gdb and set a breakpoint after the <b>setvbuf</b> leak:<br><br>
<img src="/fs3-7.png">
<br>
Then I subtracted the <b>2ac90</b> offset from the leaked address and looked at the memory in that location:<br><br>
<img src="/fs3-8.png">
<br>
Cool! We've found the location of the <b>system</b> call in memory! Now what points to the <b>puts</b> entry we want to replace?<br><br>
<h2>The GOT, PLT, and GOT.PLT</h2>
<h3>GOT</h3>
As previously mentioned, the Global Offset Table (GOT) is a copy of libc stored in a random location in memory. The location of these addresses must be found at runtime, which is why the setvbuf leak in this program is so valuable.<br>
<h3>PLT</h3>
The Procedure Linkage Table (PLT) is where libc functions are referenced locally in this program. In this example, <b>puts</b> is located at <b>0x401080</b>:<br><br>
<img src="/puts2.png">
<h3>GOT.PLT</h3>
This is the middle-man where the libc functions which are called locally are linked to the GOT. Here we can see the <b>got.plt</b> entry by examining the instructions following the local call. Looking at the memory in this location you can see the GOT address for <b>puts</b>:<br><br>
<img src="/fs3-12.png"><br>
Now all we need to do is overwrite this entry with the offset we calculated earlier (the address for <b>system</b> is different because this is a different instance from earlier):<br><br>
<img src="/fs3-13.png"><br>
And after some bugginess, we have a shell!<br><br>
<img src="/fs3-14.png"><br>
Entering an <b>ls</b> command shows the contents of the local directory where I was working:<br><br>
<img src="/fs3-15.png"><br>
I believe the errors in the above examples are from the breakpoints I set to examine memory. Unfortunately, because of the dynamic locations which are generated every time the program is run and the fact that the payload can not be crafted manually and sent to the program beforehand, a script must be used to complete this challenge. But at least we have a solid grasp of what is going on behind the scenes.<br><br>
<h2>Final Exploit</h2>
I won't go over the breakdown of the printf vulnerability because I already covered that in my <a href="https://github.com/tlkroll/format-string-exploitation/blob/main/README.md">format string 2</a> write-up, but I used those methods to determine that our printf string is in location 38. Using all of the information we have so far, here is the script I used, adapted from Wiebe Willems's script <a href="https://blog.nviso.eu/2024/05/23/format-string-exploitation-a-hands-on-exploration-for-linux/">here</a>:<br><br>
<img src="/fs3-9.png">
<br>
After running this script we have a shell on the remote server and can just cat the flag.txt file!<br>
(I wasn't able to do this in the picoCTF webshell and had to use a Kali VM)<br><br>
I hope this was helpful. This challenge was a great way to learn about the Global Offset Table and also get some practice with gdb and pwntools. Reverse engineering and binary exploitation are the most fun things I have learned about so far on my cybersecurity journey!
