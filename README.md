## Buffer Overflow

Code and steps used to help with buffer overflows. Worked great for me personally during the OSCP Exam.

### Immunity Debugger
1. Download and install: https://debugger.immunityinc.com/ID_register.py
   - Always run it as Administrator.
2. Use File -> Attach for already running apps and services or use File -> Open to run executable.
3. Unpause application.
4. Use the Windows menu to jump between mona results, log data, and CPU.

### Mona
1. Download: https://raw.githubusercontent.com/corelan/mona/master/mona.py
2. Copy to the PyCommands folder (default path is C:\Program Files\Immunity Inc\Immunity Debugger\PyCommands).
3. Set working directory for Mona in Immunity Debugger:
   - *!mona config -set workingfolder c:\mona\[path]*

## Fuzzing
1. Update IP and Port accordingly
2. Run *python3 fuzz.py*
3. This will output the number of bytes the app crashed at.

## Finding the EIP Offset
1. In a new console window run 
   - **/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l [length]** with length being the number of bytes from fuzzing + a padding
2. Restart the application
3. Run **python3 exploit.py**
4. Over in the Immunity Debugger, note down the EIP Register value
5. Get the EIP Offset by running the pattern_offset.rb script
   - **/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l [length] -q [eip_register]**
7. Update exploit.py
   - Empty the Payload variable
   - Set the offset variable to the offset found in step 5
   - set the retn variable to BBBB
8. Run exploit.py again and check the EIP Register in the Registers window in Immunity. It should be 42424242 now.

## Finding the Bad Chars
1. In Immunity Debugger, generate a bytearray using mona, and exclude the null byte (\x00):
   - !mona bytearray -b "\x00"
2. Run python3 badchars.py
   - Copy the chars from the output of the badchars.py script into the payload variable of the exploit.py script.
	 - Run python3 exploit.py
	 - In Immunity Debugger, compare the characters using mona
		 - !mona compare -f C:\mona\bytearray.bin -a [esp_address]
3. Update badchars.py with the bad chars in the listRem varible
   - Example: listRem = “\\x00\\xa1\\x2e\\x2f”.split("\\x")
4. Run badchars.py and update the payload variable in exploit.py with the output
5. In Immunity Debugger, compare the characters using mona - Repeat until no bad chars left.
   - !mona compare -f C:\mona\[path]\bytearray.bin -a [esp_address]
