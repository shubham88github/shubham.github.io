---
title: "My Experience building DSLR Remotes"
excerpt_separator: "<!--more-->"
categories:
  - Arduino
tags:
  - c
  - arduino
  - electronics
---

I have been a big fan of timelapses for a while. Been making timelapses and i enjoying works of others as well.

## Timelapse by me

<iframe width="560" height="315" src="https://www.youtube.com/embed/BVwyrqCnWNM" frameborder="0" allowfullscreen></iframe>

The timelapse above, was taken during an ultimate frisbee session. I used my gopro to take it. Edited using aftereffects.

Taking these timelapse with a gopro, the quality wasnt that good. I would prefer to use my DSLR. But wierdly these DSLR's dont have a built in intervelometer. When i search for them online it can cost a bomb. I started to research if it was doable using an arduino.

Limited by the resourses that was accessible to me, i was able to find out a way to trigger the camera using an arudino with an IR led. I found online, a timing diagram someone took from an oscilloscope with the probes attached to the IR led of a canon IR remote. I used that to recreate the signal, as the timmings were clearly stated on the diagram.

This was quite a long time ago, and this project was done during a time i did not know the importance of documentation. But i have a video of the first achievement.

<iframe width="560" height="315" src="https://www.youtube.com/embed/HwnBu5ztFtA" frameborder="0" allowfullscreen></iframe>

Since step one was achieved, i can trigger a camera using an arduino. The next step was to create something like an intervelometer.

I was able to build an intervelometer in time to show it for the university open day. Along with my smart home project. Here's a short video me demonstrating my prototype at the Computer Science booth.

<iframe width="560" height="315" src="https://www.youtube.com/embed/_lmCRMYSUCA" frameborder="0" allowfullscreen></iframe>

It was quite successfull, but this remote was not practical at all. It needed to be put into a case, and the circuit needs to be on a PCB not breadboard. Before i could go about making the PCB version, i discovered that there was a remote trigger method, using the 2.5mm headphone jack.

You have to short the shutter wire with the ground of after plugging in the 2.5mm jack to the remote port of the cannon DSLR to take a picture. The other wire was for the focus. Using this new knowledge, i improved my remote and this was what i manage to produce.

<iframe width="560" height="315" src="https://www.youtube.com/embed/nphdYtrw-H4" frameborder="0" allowfullscreen></iframe>

The design is still not protable, i was not able to bring it outside. And i was back home for a holiday, this was a good time for me to try to build a PCB version of this remote. And only control the DSLR using the 2.5mm jack. This meant that the amount of power i need was less, also required less complex code. I built the remote which was portable. It looked something like.

<iframe width="560" height="315" src="https://www.youtube.com/embed/NBsuiGOOUSo" frameborder="0" allowfullscreen></iframe>

I used an OLED screen this time. It was looking super cool. I used 2 opto-isolators to trigger the focus and the shutter. I had switches attached to wires coming out of the PCB , just becuse it was easier for me to reach them, rather then having them squished on the PCB. This remote was very practical for me. But it was vulnerable since the components were exposed. I did have trouble with the display not working once in a while. Wires kept loosing contact with the pins of the display, so ended up having to solder the wires to the display's pin header.

Also i love the remote. It had, bramplapse and Astrolapse. Which can only be found in very expensive remote. while this one costs like $15 - $20. Since i had the components lying around, it wasnt much for me.

Sorry for the lack of proper documentation, I will probably have another go at this project. Also properly document it. With the circuit printed on the PCB by a professional PCB fabrication service. (so far ive worked with seedstudio). And i will make the design avaiable to the public. So you all can have it.

But i will provide some code here. This is the code i used to create the following timelapses. It uses ffmpeg, so make sure you have that installed.


<iframe width="560" height="315" src="https://www.youtube.com/embed/e7ottBkpzBU" frameborder="0" allowfullscreen></iframe>
<br/>

<iframe width="560" height="315" src="https://www.youtube.com/embed/wwj24x0zD9g" frameborder="0" allowfullscreen></iframe>
<br/>

```python
import os
import shutil

imagenumber = 0
pics_dir = 'myfootage'
pics_dir_proper =  pics_dir
new_dir = "timelapsefootage/"
outputname = 'timelapse.mp4'

cmd = "mkdir timelapsefootage"
os.system(cmd)

files = []
for filename in os.listdir(pics_dir_proper):
	files.append(str(filename))
files.sort()
for filename in files:
	newname = new_dir + str(imagenumber) + ".JPG"
	cmd = "cp " + pics_dir_proper + "/" + filename + " " + newname
	os.system(cmd)
	imagenumber = imagenumber+1
cmd = 'ffmpeg/ffmpeg -f image2 -i ' + new_dir + '%d.JPG -r 25 -s 960x640 -vcodec libx264 ' + outputname
os.system(cmd)
shutil.rmtree('./timelapsefootage')
```
please place all your pictures in a directory named `myfootage` same directory as the script.

The code above will make sure, you're pictures dont go out of order. Which is a problem i faced at the initial stages working with this script. It was tested with python2.6.

Hope this helps someone.
