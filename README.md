<h2>Overwriting entries in the Global Offset Table using a vulnerable printf function</h2>
This write-up builds on what was learned about exploiting printf vulnerabilities in my write-up on the picoCTF <a href="https://github.com/tlkroll/format-string-exploitation/blob/main/README.md">format string 2</a> challenge. In the format string 3 challenge there is a similar vulnerability, but instead of overwriting a variable within the program it will be used to overwrite a pointer to an entry in the Global Offset Table so that it will point to a different entry.
<br><br>
<h2>format string 3</h2>
When the program is run, it prints a line which leaks the address of the GOT entry for the setvbuf function and then waits for input:<br>
<img src="/fs3-1.png">
<br><br>
After supplying some input, it then prints two lines; one which repeats the input and another which prints /bin/sh:<br>
<img src="/fs3-2.png">
<br><br>
Here is the source code for this function:<br>
<img src="/fs3-3.png">
<br><br>
Looking at the source, we can identify two goals. The main objective will be to overwrite the pointer to the puts function with a pointer to the system function so that instead of printing /br/sh, the program will make a system call to /bin/sh instead, granting us a shell. To accomplish this, we will need to exploit the vulnerable printf function which repeats user input.
<br><br>
<h2>The Global Offset Table</h2>
