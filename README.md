# input-evdev-debounce.patch
Matt Whitlock's debounce patch for input-evdev, updated to work on input-evdev-2.92

This eliminates a minor annoyance I have had to live with for years, namely always having a mouse with a "bouncing (multiple hit) key.  Seems I always do.

Matt Whitlocks original posting is here: http://lists.x.org/archives/xorg-devel/2012-August/033225.html

I actually followed the excellent instructions posted here: http://blog.guntram.de/?p=16
However the patch doesn't work with the current source code, so I have updated it, working fine here.
For the sake of completeness I will summarize the instructions here, but refer users to the original instructions at 
http://blog.guntram.de/?p=16 (these instructions are heavily adopted from that source)

Do note however the setting command line I suggest below for consistent operation.

Acquire the source code and patch with this patch using

  patch -p 1 < ../evdev-debounce-2.92.patch
  
(Note: I use Funtoo and simply put it in /etc/portage/patches/x11-drivers/xf86-input-evdev directory and emerged it)

compile and install input-evdev.  You will also have to install xinput if you don't have it already.
Once installed it does nothing until you set the parameters with xinput.

At the command prompt run ```xinput --list```
This will give a list of all of your input devices.  Find the listing referring to the mouse you wish to debounce
and make a note of the id number.
Here is an example from my computer:

    $ xinput --list
    ⎡ Virtual core pointer                    	id=2	[master pointer  (3)]
    ⎜   ↳ Virtual core XTEST pointer              	id=4	[slave  pointer  (2)]
    ⎜   ↳ Logitech USB Receiver                   	id=6	[slave  pointer  (2)]
    ⎜   ↳ MCE IR Keyboard/Mouse (meson-ir)        	id=9	[slave  pointer  (2)]
    ⎜   ↳ lircmd                                  	id=10	[slave  pointer  (2)]
    ⎜   ↳ Logitech MX5000 Keyboard                	id=11	[slave  pointer  (2)]
    ⎣ Virtual core keyboard                   	id=3	[master keyboard (2)]
    ⎜↳ Virtual core XTEST keyboard             	id=5	[slave  keyboard (3)]
    ⎜↳ meson-ir                                	id=7	[slave  keyboard (3)]
    ⎜↳ cec_input                               	id=8	[slave  keyboard (3)]
        
In my case the third item, Logitech USB Receiver is the (wireless) mouse, id=6

Now run ```xinput --list-props <id number from last step>``` at the command line.  Another example from my computer:

    $ xinput --list-props 6
    Device 'Logitech USB Receiver':
    .....
    Evdev Debounce Delay (267):	20
    .....

Giving environment property item 267 for the debounce variable.  As shown, I have assigned 20 (milliseconds) to this
value, as suggested by Guntram and works great for me.
Finally, Guntram suggests adding this line to your startup (.xinitrc, .xprofile, whatever) file:

    xinput --set-prop --type=int --format=32 <id> <debounce property variable number> <desired delay in milliseconds>

But I found that on occasion my id number and even the debounce variable number would be different, so I recommend
this line, which will always pick the right incantation:

    xinput --set-prop --type=int --format=32 $(xinput --list|grep "USB Receiver"|cut -d "=" -f2|cut -f1) $(xinput --list-props 6|grep Debounce | cut -d "(" -f2|cut -d ")" -f1) 20

Note that you will have to change "USB Receiver" to some value that grep will uniquely find for your mouse and replace the "20" at the end with your desired delay value.

The fix will be in place immediately upon running ```xinput --set-prop...``` and on subsequent startups.
