---
layout: post
title: Remapping the Office key
subtitle: or how to make **Microsoft Ergonomic keyboard for business** actually useful with AutoHotkey
categories: cli
tags: [office-key, AutoHotkey, hints, windows, emoji-key]
---

## finding a new keyboard

Recently I have been pondering to upgrade my keyboard from my trusty friend, the microsoft sculpt keyboard.
Since I have owned multiple of them throughout the years, I have the theory that with time, the input lag they have, gets worse.
What happens is that I will be typing something and there will simply be letter missing in what I have type to my frustration.

Anyway this has lead me to do some research in keyboards and I finally settled on the Microsoft Ergonomic keyboard for business.
It looks and feels super solid and like the sculpt line, fits nicely into my setup. The biggest upside, besides its ergonomics, is wired.
Don't get me wong, I absolutely would have went for a wired sculpt, but as tradition with microsoft, such a thing does not exists, and even though I still struggle a little bit with the ergonomic form factor sometimes I can notice that wired keyboards just react so much better to inputs or at least they do compared to my trusty sculpt.

## discovering the office and the emoji key

So, after I unwrapped and tested the keyboard I noticed a couple of new keys but the two that I saw first where the one with an office flag and one with a emoji.
Great I thought, more hotkeys for me and pondered on. Pressing the emoji key would open the windows 11 emoji menu, which lets you select, copy and paste emojis.
While I appreciate the effort here microsoft, its just not something that I normally do in my workflow.
The office key opens up office.com or the installed office on your computer, which didn't really work for my office version and redirected me to the website anyways.
Also not something I am going to do very often.

Great, two keys I wont use. But hey, let's download the software from microsoft which allows me to remap **every other key** but the ones I actually want to remap.
So to make it clear, if you buy this keyboard there is no way to remap those keys, or even to deactivate them with the default windows tools or the specific keyboard software.

## PowerToys won't work either

<img src="https://user-images.githubusercontent.com/9097313/140762686-b8af7168-51aa-4ba8-b69a-17310e22bffa.png"/>

Normally my first stop for remapping my keys would be the PowerToys of the microsoft community <https://github.com/microsoft/PowerToys>.
But following this issue: <https://github.com/microsoft/PowerToys/issues/3194> it becomes clear, that it simply can not help us.
Turns out this is based on how the hotkeys of the keyboard are implemented.

## The Office Key Sends Shift+Control+Alt+Windows

Normally, the Office key opens up the Office application and has several hotkeys to open up specific Microsoft apps. There are basic hotkeys like Office+W and Office+X to open Word and Excel, or less used ones like  Office+L for LinkedIn, Office+T for Teams, and finally Office+Y Yammer. Needless to say, while coding I don't need them quite often.
The interesting thing here is that the office hotkey is implemented using **Shift+Control+Alt+Windows** or something known as a super key<https://en.wikipedia.org/wiki/Super_key_(keyboard_button)>.
You can read more about it on wikipedia, but essentially, but a super key is nothing more than a separate modifier and in this case microsoft is making sure that it:-

- will not override the hotkey of another program
- is still usable for anyone without the key (try it Shift+Control+Alt+Windows+W opens word ;)))

but finally it creates a problem for me, because I really don't want to use them.

## Disabling the emoji key and the office key

In order to deactivate these keys, you have to execute this powershell script:

```
$ REG ADD HKCU\Software\Classes\ms-officeapp\Shell\Open\Command /t REG_SZ /d rundll32
```

When you press the Office key on its own, it opens up the Office app. What this does is that it modifies the location that gets opened, preventing the app from starting whenever the key is pressed, therefore stopping its execution and the action from execution.

Great, we have deactivated the key, but now what? We have two useless keys on our keyboard.

## AutoHotkey to the rescue

> AutoHotkey is a free, open-source scripting language for Windows that allows users to easily create small to complex scripts for all kinds of tasks such as: form fillers, auto-clicking, macros, etc.
<https://www.autohotkey.com/>

and the feature we are happy about the most is:

> Define hotkeys for the mouse and keyboard, remap keys or buttons and autocorrect-like replacements. Creating simple hotkeys has never been easier; you can do it in just a few lines or less!

I am not going to explain how to install AutoHotkey but I am sure you will find the documentation at <https://www.autohotkey.com/docs/Tutorial.htm#s11> just as helpful as I have.
But what AutoHotkey can do for us, is the bring our two dead keys back to live, or only disable the ones we don't like.

In a nutshell AutoHotkey comes down to running a script with your desired functionality, so lets do just that and create a file called "hotkey.ahk" in our Startup folder for the current user.

- Press the Win + X key for the extended start menu Menu.
- Select Run to open the Run box.
- Type shell:startup and hit Enter to open the Current Users Startup folder.
- create your hotkey.ahk here

We put the file in that location in order for it to be executed on every startup, because thats what we are going to have to do from now on.
Lets deactivate the office key:

```
#NoEnv ; Recommended for performance and compatibility with future AutoHotkey releases.
SetWorkingDir %A_ScriptDir% ; Ensures a consistent starting directory.
#UseHook
#InstallKeybdHook
#SingleInstance force
SendMode Input

#^!+W::
Send ^!+W
return
```

And bam, you have just disabled your office key. Let's add another line for the emoji key:

```
#NoEnv ; Recommended for performance and compatibility with future AutoHotkey releases.
SetWorkingDir %A_ScriptDir% ; Ensures a consistent starting directory.
#UseHook
#InstallKeybdHook
#SingleInstance force
SendMode Input

#^!+W::
Send ^!+W
return
#^!+Space::
Send ^!+W
return
```

And that is taken care of too. But now again, we have two dead keys we can not do anything else with.
For me I decided that I would rather start different applications with those hotkeys so I went online and found this gist:
<https://gist.github.com/teglsbo/1257230>

especially this function:

```
; Function to run a program or activate an already running instance
RunOrActivateProgram(Program, WorkingDir="", WindowSize=""){
	SplitPath Program, ExeFile
	Process, Exist, %ExeFile%
	PID = %ErrorLevel%
	if (PID = 0) {
		Run, %Program%, %WorkingDir%, %WindowSize%
	}else{
		WinActivate, ahk_pid %PID%
	}
}
```

It allows you to open a program if it is not already running or to switch to it if it is. This makes it perfect for our hotkey scenario.

## Open or activate Outlook 2021 with the emoji key

So I would like to assign the emoji button to open Microsoft Outlook. In order to do that I simply need to associate the above function to a hotkey:

```
#^!+Space::RunOrActivateProgram("C:\Program Files\Microsoft Office\root\Office16\OUTLOOK.EXE")
```

And thats it. No more chat menu, simply blissful Outlook.
But now for the sad part.

## The office key itself can not be reassigned

Yes you read that right. We got rid of the emoji but the fact that the office key is a super key, makes it impossible to be simply assigned.
You might want to try something like this:

```
#^!::RunOrActivateProgram("C:\Program Files\Microsoft Office\root\Office16\OUTLOOK.EXE")
```

**but it will not work.** So here is a list with the original office mapping but disabled:

```
#^!+W::
Send ^!+W
return

#^!+T::
Send ^!+T
return

#^!+Y::
Send ^!+Y
return

#^!+O::
Send ^!+O
return

#^!+P::
Send ^!+P
return

#^!+D::
Send ^!+D
return

#^!+L::
Send ^!+L
return

#^!+X::
Send ^!+X
return

#^!+N::
Send ^!+N
return
```

With these and the AutoHotkey documentation you should be all set to set up your specialized environment for yourself.
In the end I have to clearly point out that I would really have expected simple and clear software from Microsoft for this, but maybe we just don't know how to use keyboards yet and need to iterate more at the Microsoft lab ;)

