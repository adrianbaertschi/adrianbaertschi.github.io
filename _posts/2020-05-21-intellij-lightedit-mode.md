# Setup IntelliJ IDEA LightEdit mode in Windows 10
In the IntelliJ IDEA update 2020.1 [LightEdit mode was inroduced](https://blog.jetbrains.com/idea/2020/04/lightedit-mode/).
To set it up on Windows 10 with Jetbrains Toolbox app 

* Open Jetbrains Toolbox settings > Enable "Generate shell scripts"
* Select a path for scripts to be generated (example C:\Users\myuser\bin)

### For Windows Command prompt:
* Add the directory of the scripts to your "Path" Environment variable
* Make sure to open a new cmd window

### For Git bash
* Make sure the directry of the script in in PATH `echo $PATH`
  * Add with `export PATH=$PATH:"/c/Users/myuser/bin"`
* Copy idea.cmd to file idea and adapt for bash:
* `start /c/Users/myuser/AppData/Local/JetBrains/Toolbox/apps/IDEA-U/ch-0/201.7223.91/bin/idea64.exe "$@"`

Now you can use it as `idea myfile.txt` from Windows cmd and Git bash and a IDEA in LighEdit Mode will open.

More information:
[https://www.jetbrains.com/help/idea/lightedit-mode.html](https://www.jetbrains.com/help/idea/lightedit-mode.html)