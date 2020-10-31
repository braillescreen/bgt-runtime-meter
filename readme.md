# Runtime Meter
Say you run a bgt game server. And every now and then this server crashes because some kind of runtime error has appeared. Wouldn't it be nice to efficiently grab the runtime error, log it, tell you about it, and reboot the server all in seconds? This program does just that!
This program, along with a virtual x display buffer, really opens up the possibility of bgt game servers on linux using wine, which is cheeper, and even faster as far as pings and such!
This program has been in development for a while, originally starting out where you would have to use a text file to tell it what program to run, now expanded to run a preprogrammed notifier and get the location of any runtime process so you can run multiple servers on the same box with only one meter!
## Setup instructions:
First you will need to have a working and running display before any processes are launched, this especially applies to linux VPSs. This is required in order for the communication between the buggy servers and the meter to work. Windows servers come with this by default but you may have to disconnect remote desktop sessions in a quirky way to keep it going. If you can use NVDA Remote, you will have a display running.
You may also need a file in the same directory as any processes you wish to check for runtimes called runtime_notify.exe. This gets run when a runtime has been logged, and it is then its job to pol the file latest_runtime.txt and notify you about it, usually done via prowl or other such services. This app does not receive any command line arguments from the meter. This file is optional.
It is recommended to have the server start with a show_game_window statement to have a stable window handle to work from.
You can run programs in any order, because the runtime meter just looks for the runtime error dialog.
If you would like to make sure your program gets run and you only have one program you are monitoring with this meter, you can create a file called rmprogram.txt and put the filename, including extension, inside it of the destination program.
However, the meter has win API calls to get the pid and full path of any running process, which has been confirmed to work in latest packages of wine, so this will also work on linux boxes. So rmprogram.txt is not really necessary.
You can instead put some kind of descriptive name in rmprogram.txt and have your notifier read it for the title of the notification, so that you can use one notifier codebase but still tell exactly which application crashed.
During it's execution, if your server encounters a runtime error, the standard BGT Runtime Error dialog appears. The meter is sniffing for this dialog, and when it finds it, it will do the following.
* It will attach to the window and grab the text of the static runtime text control. Since this control ID is universal across the diferent types of runtime errors, we can now remove &yes and such from your output.
* Along with storing the text, it stores the process ID and attempts to find the full process path based on this pid, so it can then rerun the faulty app.
* It then looks for two controls, "&Yes" and "OK". Note it is using control IDs to match them.
    - If "&Yes" is found, it will activate that control, to copy stack trace text to the clipboard. This appears to be broken as of now.
    - Otherwise if it finds the OK button, it will click it but then won't attempt to read the clipboard, since no stack trace would have been delivered.
    - If all else fails, after it waits 200 milliseconds for things to occur, if that PID is still active it just kills it.
* Then it begins writing logs. It writes these logs relative to the folder it found the server was in, or if the folder check failed its current directory.
    - The file runtimes.txt contains a complete history of all runtime errors detected, and the latest ones get stuck on the bottom of it.
    - The file latest-runtime.txt contains only the latest error, the one that triggered this cycle, along with some other info.
* Each of these times it will write the current date/time, the process name, the text of the text control, and the call stack if available. As I said if you're running within wine the call stack isn't available because the clipboard doesn't seem to work, so you just get an extra blank line.
* After that, it executes the program runtime_notify.exe in the same directory as the server (if it exists) to dispatch the latest error to whatever notifiers you have present. It is the job of the notifier to read in the text of latest_runtime.txt and forward it to somehow ding you.
* Finally, it reruns the program that crashed and goes back to sleep. Note that it has a sleep period of 200ms, that is every 200ms it will check for the existence of that window. This was added in the latest meter to keep it off your CPU, thus allowing more cycles for the server.
This has been proven on a linux vps to work with multiple processes involved, each of them did get their own runtime logs and correct relaunches. You will need the package Xvfb.
## Test Suite
A miniature test suite is included  to see if your systems are functioning properly. You will need to compile the bgt scripts and autoit script to executables for this test suite to function. 
The launcher file will run the meter, wait a bit, and run a program that will then also wait and then crash. A fake notifier is also provided which just prints out the text of the latest runtime log. The runtime test also checks to see if the latest runtime log exists, and if it does will simply know this, so you don't have to worry about loops.
When the runtime tester finds that latest runtime file, it then asks you if you would like to initiate a memory overflow. Certain badly optomized servers are proan to these errors. If you hit yes, it will immediately go over what RAM it can use, producing a bad allocation error in this case. So you can see for yourself as the meter handles this. Since it doesn't read the latest runtime log it will ask you again, just hit no and it will go away.
So, enjoy the runtime meter, and in the long run may it stabilize your servers!
## Runtime Notifiers
I have made a few runtime notifiers, and have included these as BGT scripts in the "RuntimeNotifiers" directory. There is one that sends one to Prowl, also finding the name of the application by reading a file called "rmname.txt" in the same directory as the notifier.
