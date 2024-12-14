<h2>Overwriting entries in the Global Offset Table using a vulnerable printf function</h2>
This write-up builds on what was learned about exploiting printf vulnerabilities in my write-up on the picoCTF <a href="https://github.com/tlkroll/format-string-exploitation/blob/main/README.md">format string 2</a> challenge. In the format string 3 challenge there is a similar vulnerability, but instead of overwriting a variable within the program it will be used to overwrite a pointer to an entry in the Global Offset Table so that it will point to a different entry.
<br><br>
<h2>format string 3</h2>
