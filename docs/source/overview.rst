Overview
=============

**Hack.module_base_dict**
> The module_base_dict is a dictionary with the keys being the module names and the values being their base addresses.

**Hack.threads**
> Python dictionary containing lists of running threads on the target process defined in Hack("PROCESSNAME"). Keys are thread id's, the value being the thread object itself. Refer to the python winappdbg documentation for more information on what type of functionality you can perform on threads.

**Hack.base_address**
> Base address of target process defined in Hack("PROCESSNAME").
