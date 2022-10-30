# The built in driver for Irtoy / Irdroid in LIRC has a bug that makes the modules to lock from time to time, which require to replug the Irdroy/Irdroid and restart LIRC

# Description of the problem:

When the device is transmitting via irsend in a loop (tested with different delays between transmit from 500ms to 2 seconds), and a signal from another IR source such as another remote control is received by the IR receiver the device “hangs” de-enumerates from the usb bus and a replug is needed to start functioning again. This issue sometimes happen even when not transmitting in a loop, but when the user sends ir command with lirc and at the same time pushes remote control keys on his physical remote control which leads to the same result - device de-enumeration 

This issue seems to be present in LIRC since long time and no one came up to a solution until now!

In the last two weeks we were researching the above issue which seems to exist since a long time (many posts on the Internet with that issue with no resolution). This issue was never
reported from users on MS Windows / WinLIRC and we have performed a code review of WinLirc (https://github.com/leg0/WinLIRC/blob/master/DLL/Irdroid/SendReceiveData.cpp) and in particular
the driver / plugin responsible for communicating with Irtoy / Irdroid.

The tests were carried out on a Ubuntu 20.04 system with stock installation of LIRC 0.10.1

# The Following observations were made:

1. Device initialization e.g, entering samling mode with "s" is sent once
2. then before reading the result 100us delay is added after which the result is read (obviously the device need some time to communicate back the result which is "S01")
3. After entering sampling mod the command 0x24 is sent once, followed by 20us delay
4. Then the command 0x25 is sent , followed by 20us delay
5  And finally the 0x26 command is sent , followed by 20us delay

The above commands are sent just once upon device initialization in WinLirc (which is not the case in LIRC where these commands are sent each time a ir transmission is made)which sometimes makes the device to lock and deenumerate.

# The Solution:

1. Clone LIRC from git -  git clone git://git.code.sf.net/p/lirc/git lirc
2. cd into lirc/plugins
3. Download the patch wget https://raw.githubusercontent.com/Irdroid/Irtoy_Irdroid_Patch/main/Irtoy_Irdroid_Lirc_Stability.patch
4. Apply the patch via : patch -u irtoy.c Irtoy_Irdroid_Lirc_Stability.patch
5. cd into lirc source main directory and run ./autogen.sh , ./configure , make and make install 

* make sure you run lircd together with either irw running in the background or irexec (so that we make sure lirc never command the irtoy/irdroid to exit/enter sampling mode)

For those of you who can't or dont want to compile the irtoy driver, you can use the x86_64 binary (MD5 hash - 059a85b7f167d34d85ec93be791b6868 ) - It can be downloaded from this repository (see files above). locate and overwrite the irtoy.so file on your system (x86_64) with the attached file.
* for Ubuntu , the file location is  /usr/lib/x86_64-linux-gnu/lirc/plugins/
