# Wii-Nunchuk-Experiment
Initally derived from partsnotincluded article, use a Nunchuk on PC.
Much of the below will be quoted directly from the article, with any points to change addressed immediately after.
https://www.partsnotincluded.com/experiment-wii-nunchuk-controller-for-csgo/
Experiment: Wii Nunchuk Controller for CS:GO
Dave Madison | February 28, 2019

> Earlier this week I was browsing Reddit and came across this interesting post of someone playing a game of Counter-Strike: Global Offensive (CS:GO) using a Wii Nunchuk to aim. They used a cheap Chinese “Classic Controller to USB” adapter to connect the Nunchuk to their PC, then set up JoyToKey to convert the gamepad inputs into mouse movements.
This was pretty interesting, but I thought I could do one better. You see, I’m currently working on my own project that uses two Nunchuks for a custom controller. So when I ran across that Reddit post, I already had a breadboard on my desk with a Teensy LC, two NXC breakout boards, and two Wii Nunchuks wired and ready to go. Destiny was calling…

__While I do not plan to implement support for two simultaneous Nunchuks, it would be interesting to use the full features of the nunchuk, including the accelerometer. The code used in this project already utilizes the analog stick and buttons, although only for digitial button presses on a keyboard.__

# The Plan

> To start, I had to pare down the controls. Normally when playing CS:GO you have a whole host of controls to deal with: you have to handle movement, aiming, firing, reloading, walking, selecting weapons, dropping weapons, buying weapons, jumping, crouching, the list goes on and on…

> Each Nunchuk has the following controls surfaces:

    1 Analog Joystick (XY, 8 bit)
    2 Buttons (C/Z)
    3-Axis Accelerometer (XYZ, 10 bit)

> Since I was building this quickly I decided outright to ignore the accelerometers. It takes too long to code the kinematics and there’s too much interference from the tactile control surfaces.
Plan to incorperate accelerometers into modular functions, utilize the analog axes or create digital "lean" bindings.

__There's no need to build quickly on this project; it's better to explore all possibilities over time than rush something out to be the first to claim "It's possible*, but..." Kinematics do take a while to code, but tilt analog axes and digital lean bindings are still useful input methods for this.__ 

> The joysticks are much more straight-forward. You would obviously map the two joysticks to ‘move’ and ‘aim’ – necessary controls that are typically mapped respectively to the left and right joysticks on most gamepads for first person shooter (FPS) games.

__One joystick, in this case, can be mapped to move, perhaps to legacy bindings to give FPS tank controls while another more reliable input method can be used to aim across the pitch and yaw. Also important to note is that the joystick is mapped strictly to WASD in this case, but it's a limitation of the library.__

> This leaves the four buttons to handle, well, everything else. In order of priority:

>    Right Z: Fire – The big ‘Z’ button on the right Nunchuk handles firing the weapon. It’s a shooting game, you have to be able to shoot. Again mirroring a traditional gamepad where ‘fire’ is the right trigger.
    Left C: Use – The small ‘C’ button on the left Nunchuk is the ‘use’ key. This handles everything from opening the buy menu at the start of the round (for purchasing weapons and equipment), to swapping your gun with one on the ground or defusing the bomb.
    Right C: Switch Weapons – The ‘C’ button on the right Nunchuk switches your weapon. You need to be able to switch weapons if you want to plant the bomb or if you run out of ammo. If you’ve ever played Counter-Strike, you also know that you run at different speeds depending on your equipped weapon, so switching is necessary to get places faster.
    Left Z: Jump – The ‘Z’ button the left Nunchuk jumps. From all of the remaining controls it was a toss-up as to which was most important. Jumping lets you get to places you otherwise wouldn’t be able to, although this is at the expensive of zoom (no AWP-ing!), crouch, or reload. With only four buttons, sacrifices must be made…
    
__Digital lean bindings are possible for the last two actions.__

> I decided right off the bat that I would do this using keyboard and mouse (KBM) commands rather than DirectInput (“Joystick”), just for simplicity. I’ve used a controller with CS:GO in the past and from what I remember it was a little finicky to set up, so I decided to go the easy route and stick to the KBM commands that I knew off the top of my head and I was sure would be plug + play. With keyboard and mouse, here are the final mappings:

> Left Nunchuk
Joystick: Movement (WASD)
Button C: Use (E)
Button Z: Jump (Spacebar)
Right Nunchuk
Joystick: Aiming (Mouse XY)
Button C: Switch Weapons (Mouse Scroll Up)
Button Z: Fire (Mouse LMB)

# The Hardware

> The hardware to get this working is quite simple. I’m using a Teensy LC microcontroller, which runs at the same logic level as the Nunchuks (3.3V) and has two available I²C buses so it can connect to both Nunchuks without a multiplexer. Attached to the Teensy are two Nintendo Extension Control (NXC) breakout boards of my own creation, wired to power (3.3V / GND) and the I²C pins (18/19, 22/23).
![Teensy_DualNunchuk](https://user-images.githubusercontent.com/12158234/140527692-57179d38-a73f-46bf-8970-847915ca6f8f.jpg)

> Last but not least, the code to make it all work. I’m using two of my own libraries here:

>    NintendoExtensionCtrl to handle the communication with the Nunchuk controllers
>    HID Buttons to simplify the handling of the USB commands for the keyboard and mouse

> I’m also making use of the built-in Keyboard and Mouse libraries from the Teensyduino software to make the microcontroller act as a composite USB device.

__Need to see if Teensyduino has a direct input library to map to the analog axes of another controller, while NinExtCtrl can still be used to initalize communication.__

> The program should be fairly straight-forward, although it’s a tad bit messy since I wrote it in a hurry. Within the setup function the program initializes the I²C buses, sets them to a faster rate (100 kHz -> 400 kHz), and then loops until both Nunchuks are connected and initialized. It then saves the starting position of the right Nunchuk’s joystick to use for the aiming comparison in the main program loop.

__A method that records the starting position of joystick and accelerometer can be put into setup()/init(), with a specific combination to recall this method to recenter if needed (or simply reset the microcontroller).__

> Within the program loop (loop function, per Arduino), the program polls the controllers for new data and assigns the USB outputs based on control surface states. The WASD keys use a comparison with a deadzone in the center of the joystick, while the aiming joystick acts directly on the mouse position – linearly and at a 4th of the max range (~25 pixels per update in each direction). The buttons are 1:1 with their HID counterparts with the exception of the ‘switch weapons’ button, which will only scroll once for each press.

> You can see in the title that I’ve marked this as an “experiment”, because this is significantly rougher around the edges than most of the stuff I post on the blog. This is not a fully fledged project with all of the usual trappings, but just a quick and dirty experiment to see what I could do with my resources in half an hour. Using that benchmark I’m quite happy with the result.

> Bearing that in mind, there is plenty of room for improvement if I wanted to dedicate some more time to this and make it a fully fledged controller. Some ideas off the top of my head include:

  - Accelerometer controls (jump! reload!)
  - Nunchuk auto-detect and reconnection
  - Exponential mouses aiming
  - Joystick auto-ranging
