---
layout: post
title: ! 'Just enough kernel to get by (part 2) : Syscall & SSDT'
date: 2013-12-01 16:20:58.000000000 +01:00
---
Last week we detailed the interrupt dispatch internals, today we will focus on the syscall mechanism.

### System overview

The diagram below shows a very high level and simplified view of the Windows architecture (I have omitted subsystem and GDI stuff to keep it simple)


![]({{ site.baseurl }}/images/bigpicture.png)

So basically there are 3 families of processes running :

* System processes like smss.exe, lsass.exe, winlogon.exe etc...
* Services ranging from system services (DHCP, RPC etc ...) to third party services (AVs, SGBD etc ...)
* Applications (Word, Notepad, Firefox,...)

During the life of processes lots of API calls will end up in the kernel to be serviced, and except for the GDI/Windowing stuff, this will be **ntdll.dll** role to perform the syscall and make the transition from user land to kernel.

So basically, when you call **CreateFile** from your application it will go through *kernel32.dll* to end up in *ntdll.dll* in the **ZwCreateFile** function.

Depending on the CPU, **ZwCreateFile** will issue the system call using either :

* **INT 2E** software interrupt (old CPUs)
* **SYSCALL** (AMD) or **SYSENTER** (Intel) instructions

In the first scenario, the **2E** interrupt will be serviced by the **KiSystemService** *ISR* :

![]({{ site.baseurl }}/images/dumpidtkisystemservice.png)

Then **KiSystemService** will call **KiFastCallEntry** which will call the kernel implementation of **ZwCreateFile** : 

![]({{ site.baseurl }}/images/kisystemservice.png)

But how does **KiFastCallEntry** know which function to call in the kernel ?

Let's take a closer look at the second scenario from the beginning and detail it step by step (as it is the most common scenario you will face on today's machines)


Nowdays rather than using the **INT 2E** mechanism the OSes use the **SYSCALL** / **SYSENTER** instructions to perform the system call.

These instructions were introduced to speed up the user to kernel mode transition as using the software interrupt mechanism was to slow (There is **LOTS** of user to kernel switches going on every seconds).

So let's go back to the implementation of **ZwCreateFile** in **ntdll.dl** :

![]({{ site.baseurl }}/images/zwcreatefile.png)

First thing it does is to put **0x25** in the **EAX** register (**remenber this**) and then call a function pointer, so let's follow this pointer :

![]({{ site.baseurl }}/images/systemcallstub.png)


and again, we end up in **KiFastSystemCall** :

![]({{ site.baseurl }}/images/calltokifastsystemcall.png)


And here is the real stuff, the **SYSENTER** call.

Once this instruction is executed, the CPU will switch from user land to kernel mode.
But how does the CPU know which kernel function to call when entering kernel mode ?

The answer lies in the **MSR** registers (*Model-Specific Register*).

These registers are kind of special since you do not access them directly but through 2 dedicated instructions **RDMSR** (read) and **WRMSR** (write).

There is plenty of MSR registers (will get back to some of them in due time), but the one we are interested in right now is **SYSENTER__EIP__MSR** (Offset **0x176**).

This register tells the CPU which OS kernel function to call while handling a **SYSENTER** .

So let's dump it in Windbg :

![]({{ site.baseurl }}/images/rdmsr.png)

Aaahhh what a surprised, we find the **KiFastCallEntry** function, yep the same one called in the **KiSystemService** function we saw previously while describing the good old **INT 2E** mechanism.

So, now we know that regardless the scenario used to perform the system call we will end up in **KiFastCallEntry**.

But how does this function know which kernel function to call ?
Meaning, what is the kernel land equivalent of the API we were calling in our application (in our case **CreateFile**)?

The answer lies in the **KiServiceTable** also known as the **SSDT** (*System Service Descriptor Table*).

This is basically an array of functions pointers where each function referenced is the kernel implementation of the Nt* functions stubs in **ntdll.dll**.

So let's dump it (just a part of it) in Windbg  :

![]({{ site.baseurl }}/images/kiservicetable.png)

This is it, most of the user mode APIs you will ever call from your application (except GDI stuff) will end up here... 
    
At runtime **KiServiceTable** is not exported, but you can use the kernel export **KeServiceDescriptorTable** to access it, we will get back to this later...

So remember the **0x25** i mentioned while talking about **ZwCreateFile** ?

![]({{ site.baseurl }}/images/kiservicetableoff25.png)

Circle is complete :)

Here is a summary of the User to Kernel mode transition mechanism :

![]({{ site.baseurl }}/images/kernel_transition_flow-3.png)


We will get back to the **IDT** / **SSDT** / **MSR** stuff later while talking about rootkits, because as you can imagine, those are perfect spots to perform some **API hooking** at kernel level :)

In the next post i will start describing some kernel object structures and will talk about the Object Manager.
