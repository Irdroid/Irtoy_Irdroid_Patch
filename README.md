## The built in driver for Irtoy / Irdroid in LIRC has a bug that makes the modules to lock from time to time, which require to replug the Irdroy/Irdroid and restart LIRC
This issue seems to be present in LIRC since long time (probably 10 years) and no one came up to a solution until now!

In the last two weeks we were researching the above issue which seems to exist since a long time (many posts on the Internet with that issue with no resolution). This issue was never
reported from users on MS Windows / WinLIRC and we have performed a code review of WinLirc (https://github.com/leg0/WinLIRC/blob/master/DLL/Irdroid/SendReceiveData.cpp) and in particular
the driver / plugin responsible for communicating with Irtoy / Irdroid.

# The Following observations were made:

1. Device initialization e.g, entering samling mode with "s" is sent once
2. then before reading the result 100ms delay is added after which the result is read (obviously the device need some time to communicate back the result which is "S01")
3. After entering sampling mod the command 0x24 is sent once, followed by 20ms delay
4. Then the command 0x25 is sent , followed by 20ms delay
5  And finally the 0x26 command is sent , followed by 20ms delay

The above commands are sent just once upon device initialization in WinLirc (which is not the case in LIRC where these commands are sent each time a ir transmission is made)which sometimes makes the device to lock and deenumerate.

The solution is a patch for LIRC, that makes the plugin to properly send the above initialization commands with the required timing.

1. Clone LIRC from git -  git clone git://git.code.sf.net/p/lirc/git lirc
2. cd into lirc/plugins
3. Download the patch wget https://gist.githubusercontent.com/Irdroid/df859ae9b7cddd8fccde879804e056da/raw/c6664b43fd493371d8f89d515f40a99c3fc69d3c/Irtoy_Irdroid_Lirc_Stability.patch
4. Apply the patch via : patch -u irtoy.c Irtoy_Irdroid_Lirc_Stability.patch
5. cd into lirc source main directory and run ./autogen.sh , ./configure , make and make install 

* make sure you run lircd together with either irw running in the background or irexec

For those of you who can't or dont want to compile the irtoy driver, you can use the x86_64 binary (MD5 hash - 059a85b7f167d34d85ec93be791b6868 ) attached below. locate and overwrite the irtoy.so file on your system (x86_64) with the attached file.
* for Ubuntu , the file location is  /usr/lib/x86_64-linux-gnu/lirc/plugins/
