
# hackManager

Windows process memory hacking framework created in Python and winappdbg.
> So i noticed a couple of people were creating projects using this library I made a couple years ago so I thought I'd continue supporting it from now onwards.

# Installation
`pip install hackManager`

# Examples
**To create a simple Python file to renames Notepad.exe into something else, you could achieve that via:**
```
from hackManager.hack import Hack

Hack.change_window_title("Notepad.exe", "Changed Notepad.exe")
```

**To create a hack for a game, you should be able to do something similar to:**
```
from hackManager.hack import Hack

FAKE_HEALTH_ADDRESS = 0x01123123 # memory address found using Cheat Engine or your preferred Debugger (Olly, Immunity).
h = Hack("Game.exe")
h.write_int(FAKE_HEALTH_ADDRESS, 100)
```

**Internal (main process) function hooking**
> You can hook onto the processes internal functions following this method.
add_internal_hook takes three parameters, the third being optional: function_memory_address, function_handle, signatures.
The signatures parameter allows you to parse values related to this function directly from the stack.
```
from hackManager.hack import Hack
 
 
def function_handle(event, ra, **kwargs):
    print "Function hit!"
 
    
h = Hack("mb_warband.exe")
h.add_internal_hook(0x004961FC, function_handle)
h.hook()
h.safe_exit()
```

**External library (DLL) hooking**
> You can hook onto Kernel and DLL functions within' a process. In this example, we hook onto RUST's(game) Dedicated Server and hook onto its WinSock SendTo DLL function calls. This allows us to sniff(analyze) process-specific traffic. This requires a little research on your part though. For example, you need to know what parameters the DLL functions require. You can easily pull this up on Google Search by typing something like, i.e: "WinSock sendto msdn". The Microsoft Developer Network is fulled with tons of documentation for its DLLs like Winsock. You can add as many hooks as you want, for the same DLL or different ones. Simply call the class-method add_hook(DLL_Name, DLL_Function, function_handler).
```
from hackManager.hack import Hack
 
def sendto(event, ra, s, buf, length, flags, to, tolength):     
    data = event.get_process().peek(buf, length)
    print "Send: " + data + "\n" 
 
h = Hack("rust_server.exe") 
h.add_hook("ws2_32.dll", "sendto", sendto) 
h.hook() 
h.safe_exit()
```

**Call of Duty: Black Ops 2 - No recoil hack**
> Singleplayer & Multiplayer hack that removes every weapons recoil effect.
```
from hackManager.hack import Hack  # Absolute memory address 
 
 
BLACKOPS_RECOIL_ADDRESS = 0x004AF328 # You can also supply the base address and a offset like, i.e.:
# BLACKOPS_RECOIL_ADDRESS = instance.base_address + 0xAF328  
 
# No recoil value 
BLACKOPS_NO_RECOIL_VALUE = 117 
target = "t6mp.exe" 
 
instance = Hack(target) 
print instance.base_address # main modules base address (0x400000) 
 
print instance.read_char(BLACKOPS_RECOIL_ADDRESS) # returns the following:  ( value, label ) 
# label is: t6mp.exe(base address) + offset print 
# instance.read_char(BLACKOPS_RECOIL_ADDRESS) 
 
# update value with 117(NO RECOIL VALUE) 
instance.write_char(BLACKOPS_RECOIL_ADDRESS, BLACKOPS_NO_RECOIL_VALUE)
``` 

**Accessing and modifying data structures within' an application (or game)**
> In this example we are hooking on the game's Winsock sendto DLL function and accessing its Structure directly via ctypes. We are also accessing the data directly via peek. Both methods work great, however, if you want to access Structure fields in a clean manner, the ctypes approach is preferred.
```
from hackManager.hack import Hack 
import ctypes 
 
# Winsock sockaddr structure.
class sockaddr(ctypes.Structure): 
    _fields_ = [         
        ("sa_family", ctypes.c_ushort),
        ("sa_data", ctypes.c_char * 14), 
    ]  
 
 
def sendto(event, ra, s, buf, length, flags, to, tolength):     
    p = event.get_process()     
    data = p.peek(buf, length)
    to_struct = p.read_structure(to, sockaddr) 
    print "BUFFER DATA: " + repr(data) + "\n"     
    print "ACCESSING SPECIFIC STRUCTURE sa_data field:", repr(to_struct.sa_data) 
    print "PEEKING WHOLE STRUCTURE DATA:", repr(p.peek(to, tolength))
 
 game = Hack("game.exe") 
 h.add_hook("ws2_32.dll", "sendto", sendto) 
 h.hook() 
 h.safe_exit()
```

**Retrieving and interacting with running threads**
> You can retrieve the list of the processes running threads with the Hack.get_threads() class-method. You can use this to your advantage to supervise the amount of threads your processes currently has running and to analyze them individually. Remember that hackManager is built on top of winappdbg, thus you are able to execute thread-related class-methods like, i.e: thread_instance.set_name(), thread_instance.is_hidden(), thread_instance.set_process(), thread_instance.kill(), thread_instance.name(), thread_instance.resume(), thread_instance.suspend(), to name a few. When you call Hack.get_threads(), the list of threads are stored on your Hack() instances thread global variable. Thus you can access it with Hack_instance.threads. It's stored as a dictionary. The thread id's being the keys for the dictionary. Check the winappdbg documentation for more information regarding iteraction with threads. Remember: hackManager returns winappdbg.Thread instances.
```
from hackManager.hack import Hack 
h = Hack("game.exe") 
h.get_threads() 
print h.threads # returns dictionary, with the keys being the individual threads id's.
```

**Retrieving the list of imported DLLs(libraries).**
> You can retrieve the list of loaded(imported) DLLs(libraries) within' the process by accessing the module_base_dict global variable. The module_base_dict is a dictionary with the keys being the module names and the values being their base addresses.
```
from hackManager.hack import Hack  
h = Hack("game.exe") 
print h.module_base_dict
```

**Retrieving list of running processes**
> You can retrieve the list of currently running processes by not supplying a target process name within' your Hack() instance. Then you access the list by calling Hack().running.
```
from hackManager.hack import Hack 
h = Hack() 
print h.running
```

# Hack() instance variables overview
**Hack.module_base_dict**
> The module_base_dict is a dictionary with the keys being the module names and the values being their base addresses.

**Hack.threads**
> Python dictionary containing lists of running threads on the target process defined in Hack("PROCESSNAME"). Keys are thread id's, the value being the thread object itself. Refer to the python winappdbg documentation for more information on what type of functionality you can perform on threads.

**Hack.base_address**
> Base address of target process defined in Hack("PROCESSNAME").
