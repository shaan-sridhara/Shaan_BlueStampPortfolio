# Smart Glasses
[comment]: <> (description)

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Shaan S | Bret Harte | Biomedical Engineering | Incoming 8th Grader

![Headstone Image](image_2025-06-17_161621492.png)
  
[comment]: <> (# Final Milestone)

[comment]: <> (work in progress)

[comment]: <> (# Second Milestone)

[comment]: <> (work in progress)

# First Milestone

My project is called Smart Glasses. The goal is to help people, especially those who can’t see well, by using AI to detect objects around them. I’m building it using a Raspberry Pi 5 and a camera that attaches to it. These parts will go on a glasses frame. The camera takes video of what’s in front of the person, and the AI will figure out what the objects are. Then, the glasses will say what it sees out loud using a speaker so the person knows what’s around them.

So far, I was able to take a picture using the Raspberry Pi and the camera. At first, I tried to set it up without using a monitor or keyboard, but it was really hard and didn’t work for me. Instead, I decided to just plug it into a monitor and use a keyboard and mouse, which made it easier to work on.

My plan now is to find a database of pictures to help train the AI to recognize different objects. Once I get that working, I’ll connect it to a text-to-speech program so the glasses can tell the person what it sees. That way, the Smart Glasses will be able to help people understand what’s around them without needing to see it.

This is my first milestone video:

<iframe width="560" height="315" src="https://www.youtube.com/embed/hwgU-7iSydI?si=aCjqJ5J-OpynQL5M" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

![Milestone_Setup](Screenshot 2025-06-20 161814.png)

## Challenges

One of the main challenges I faced during Milestone 1 was setting up the Raspberry Pi for a headless configuration, which means accessing and controlling the Pi remotely without a monitor or keyboard attached. This setup is very convenient because it allows you to program the Raspberry Pi from another computer over a network. However, due to slow and unstable Wi-Fi, I had trouble maintaining a consistent connection, which made the setup process difficult and sometimes impossible to complete. This experience taught me how important a reliable network is for remote device management and pushed me to troubleshoot connectivity issues while learning more about networking and remote access tools like SSH.

## Next Steps

The next steps for this project are to get the camera module working so it can detect and recognize different objects by using a database of known items. This means setting up software that can compare what the camera sees with stored information to identify objects accurately. At the same time, I plan to set up a continuous video stream so that the camera feed can be viewed and analyzed in real time. Having this live video combined with object detection will let the system recognize and classify objects as they come into view, making it more interactive and useful. To do this, I’ll need to explore different machine learning models, figure out how to run them efficiently on the Raspberry Pi, and make sure the camera, detection program, and database all work smoothly together. These steps will really push the project forward and help build a smart vision system that can respond to what it sees.

## Code



[comment]: <> (# Schematics)

[comment]: <> (work in progress)

[comment]: <> (# Code)

[comment]: <> (work in progress)

# Bill of Materials

| Part                    | Note                                  | Price | Link                                                                                                     |
|-------------------------|-------------------------------------|-------|----------------------------------------------------------------------------------------------------------|
| Raspberry Pi 5          | Latest Raspberry Pi model             | $75   | [Buy here](https://www.raspberrypi.org/products/raspberry-pi-5/)                                         |
| Pi Camera Module        | Official camera for Raspberry Pi     | $30   | [Buy here](https://www.raspberrypi.org/products/camera-module-v3/)                                       |
| Non-prescription Glasses| Black rectangular fashion glasses    | $12   | [Buy here](https://www.amazon.com/GQUEEN-201512-Fashion-Rectangular-Glasses/dp/B00ZRD1MEI/)               |


[comment]: <> (# Other Resources/Examples)

[comment]: <> (work in progress)

# Starter Project
This is the retro arcade console I built for this project, using the VOGURTIME Electronic Soldering Practice Kit. The kit came with pre-manufactured components, including a mini circuit board, joystick, LED display, and tactile buttons. The goal was to assemble a fully functional mini arcade game system by carefully soldering all the components to the PCB. Once completed, the arcade console lights up, plays sound, and allows simple interactive gameplay, making it a fun and practical introduction to basic electronics.

This is a [link](https://www.amazon.com/Electronic-Soldering-Practice-Comfortable-VOGURTIME/dp/B094QRRHC2/ref=sr_1_3?asc_source=01H2RCFWNNZMQFGXGXS3RMXVE5&crid=12C0SOV36FG6M&dib=eyJ2IjoiMSJ9.Prj06eg0mzBHrfW8zuFr43Ott4t2wUOVBo8A8bYw0PqFZRlOEmgR5YwhMy7jXrdI2HlBjVttnEyYLz5CP684SzJyHmVMBp25vNna9o8wjV-df55ilTgj0xMy1CiRwkcnu6xqacZ3JUPlq8C3mQJwmEtoeokndNqpwpdkZBQMplM9vg3M-cfB0xM_nXdjeqHQ3bB707ehrzX6Llp-Euu3CTFzF8wgEqhPwo6RCvzbo5M.yyrFg8EXJr9BL5cOgZF551-8cIl91p0MSy8nGiilcpU&dib_tag=se&keywords=arcade%2Bsolder%2Bproject&qid=1717994267&sprefix=arcade%2Bsolder%2Bprojec%2Caps%2C147&sr=8-3&tag=snxs3-20&th=1) for the Retro Arcade Console:

This is a video of me demoing the Retro Arcade Console:

<iframe width="560" height="315" src="https://www.youtube.com/embed/FWgXDVm8kqY?si=U474cKpsJ89gVsbU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Challenges
One of the biggest challenges I faced was soldering the components precisely. Many of the soldering points were small and tightly packed, which required a steady hand and a lot of focus. It took some trial and error to get the hang of it, but I eventually learned that good soldering comes down to patience, attention to detail, and proper technique. Overall, this was a great project for learning hands-on skills in electronics through a fun and engaging experience.

## Next Steps
My next steps for this project involve beginning the initial phase of my intensive project. This first milestone will focus on assembling and configuring the core hardware components: a Raspberry Pi 5 and the official Pi Camera Module. I will begin by carefully connecting the camera to the Raspberry Pi using the appropriate ribbon cable and ensuring the hardware is properly secured and recognized by the system.

Once the hardware is set up, I will move on to the software configuration. This includes enabling the camera interface through the Raspberry Pi OS settings and installing any necessary libraries or dependencies required to operate the camera. I will be using Visual Studio Code as my development environment to write and test Python scripts that allow the camera to capture still images.

The goal of this milestone is to successfully capture a still photo using the Pi Camera, laying the groundwork for more advanced features later in the project. This step is critical as it ensures that both the hardware and software components are functioning correctly before progressing to more complex parts of my project.

