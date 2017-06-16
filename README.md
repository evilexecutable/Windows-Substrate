# hackManager

Windows process memory hacking framework created in Python and winappdbg.
> So i noticed a couple of people were creating projects using this library I made a couple years ago so I thought I'd continue supporting it from now onwards.

# Installation
`pip install hackManager`

# Quick start guide
To create a simple Python file to renames Notepad.exe into something else, you could achieve that via:
```
from hackManager.hack import Hack

Hack.change_window_title("Notepad.exe", "Changed Notepad.exe")
```
Or to create a hack for a game, you should be able to do something similar to:
```
from hackManager.hack import Hack

FAKE_HEALTH_ADDRESS = 0x01123123 # memory address found using Cheat Engine or your preferred Debugger (Olly, Immunity).
h = Hack("Game.exe")
h.find_process()
h.write_int(FAKE_HEALTH_ADDRESS, 100)
```