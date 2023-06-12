---
title: Introduction to estimating direction of arrival
date: 2023-06-11 12:00:00 -800
categories: [signal_processing]
tags: [signal_process,doa,tdoa]
---

# Motivation

You lose your keys, but not to worry, they have a bluetooth enabled tag attached to them. Using your phone you send a radio request, it emits outwards through space around you, piercing through objects like an unstoppable tsunami! Luckily, the tag hears your plea! You hear the tag emit a sound!, but the sound is faint, you slowly move through space, poorly estimating the gradient of volume, after an initial wrong turn, you backtrack and try again, this time the sound gets louder and louder, until you're face to face with your cat, purring loudly while sitting on top of your keys!

![Finding your keys](/assets/2023-6-11-tag-finding.gif)
*An illustration of what your ears might hear when looking the source location of a sound*

It feels like there is something natural about associating direction with sound, but how does it work?  :q
Ls 

Your brain has two pieces of information, the sound heard from both ears. From this information there is two basic things you or your brain could do, 
1. Use the volume of the sounds. Maybe it sounds louder in your right ear vs your left ear, then it's a pretty good chance the sound is coming from your right. You could turn your head until the sound is equal loudness in both ears, once you stop turning your head you are confident the sound is either directly in front or behind you. Maybe you try moving forward and the sound gets quieter, then it must be coming from behind you. 

2. Maybe your brain is doing something more clever under the hood, a bit of a spoiler, it is! Your brain compares the difference in time between hearing something in your left vs right ear, and uses this to estimate the direction! [interaural time difference (ITD)](https://en.wikipedia.org/wiki/Interaural_time_difference). 

Approach (1) works well in low noise environments when you can clearly hear the signal and can sense a large change in volume with small change in space. But imagine you are guiding a boat on a foggy night, and you hear a loud fog horn, oh no! there must be land nearby?! in the moment you can't tell which ear it sounds louder in, the fog horn is just too loud! What do you do?

Approach (1) uses the volume from each ear but discards the information about what was heard, approach (2) on the other hand discards the volume and uses mostly the information! Let's think about this for a second, sound travels at roughly 340m/s (1100ft/s), the average width of an adult human head is ~0.145m (6inches), the maximum difference in time (sound-time) between your two ears is 0.0004s! Your brain can somehow compare the left and right ear signals, determine a difference smaller than 0.0004s and use it to give you a sense of direction! 

# Waves

Let's say we want to send our friend an electronically generated hello signal using sound (or maybe radio), how can we do this? 

First we might encode our message into some kind of digital form. There are many ways to do this, let's say we choose a binary representation of [1,-1,1]. Once encoded our next step would be to emit this signal from some kind of speaker. We might think the whole situation kind of works like this,

![Digital emission](/assets/2023-6-11-digital-emission.gif) 
*Our first attempt to transmit our message [1,-1,1] to our friend fails. Because our output is not continuously varying and therefore it is not actually transmitting*

But this is not exactly how waves work. Waves need to wave!, they can't just be discrete values, well not exactly. Let's try to nail down some properties of waves that are important for our context.

1. Waves originate from a source and propagate uniformly outwards in all directions of space

2. Waves propagate at some speed. For sound that speed is ~340m/s for radio that speed is ~3x10^8 m/s

3. Waves oscillate at some frequency. They go back, and forth, and back, and forth, the rate of back and forths per second is their frequency 

4. Waves have an amplitude. One wave could be twice the disturbance of another, making it have twice the amplitude. One sound could be twice as loud as another 

5. Waves disturb something. Sound waves disturb and propagate through the air and radio waves disturb and propagate through the electromagnetic field

Our problem in the above illustration is that we are just sending 1 or -1 out into the world. That is not really waving! For both sound and radio waves we have to continuously vary the output in order to transmit. If the output is constant then nothing is waving if nothing is waving then nothing is being transmitted. So we can't just transmit 1 or -1! The change in output is actually what is causing the propagation of the wave!

All is not lost though, we just need a layer of abstraction. 

# Encoding information into waves

![Single wave emission](/assets/2023-6-11-single-wave-emission.gif)
*Transmission of a single frequency wave*

The problem in the previous section was broadcasting discrete values (a binary code) instead of continuous values. We are able to emit a constantly varying function from our speaker, such as a wave with a single frequency and amplitude (shown above).

How about instead of trying to transmit a constant value, we encode our information into the waves amplitude and frequency. In a very simple case we could use the single wave above and change its amplitude over time. Then when our friend receives the wave, they can decode the message from the amplitudes! This is how [AM radio](https://en.wikipedia.org/wiki/Amplitude_modulation) works. Doing this would result in the illustration below where we successfully send our "hello" message to our friend! 

![AM digital transmission](/assets/2023-6-11-am-digital-transmission.gif)
*The binary message [1,-1,1] is encoded into amplitudes [1.0,0.5,1.0] of a fixed frequency wave that is transmitted to our friend!*

# Direction of arrival estimation

Getting to the real meat of this post, how to solve the original problem: for a set of physical detectors (human ears in the initial example above), given some received signal (sound or radio) how can we estimate the location of the source?

It's important to separate out the direction of a source [Direction Of Arrival - DOA](https://en.wikipedia.org/wiki/Direction_of_arrival) versus the exact location (direction and distance) of the source. If we do not accurately know how the signal degrades over time and direction, the signal strength may not be a good indicator of distance to source. For example if you are standing in a field and hear a faint sound, that sound could be something quiet nearby or it could equally be something very loud far away. You would not know the difference.

## Audio TDOA

![Audio Time-Difference-Of_Arrival TDOA](/assets/2023-6-11-tdoa-estimate.gif)

To estimate the direction of a sound source we could do something similar to what your brain does ITD. We can do this by placing microphones where your ears would be (in the example at the top of this page), recording the signal at each, and then estimating the time delay between the two recorded signals. We can estimate the time offset between two signals by sliding the signals over one another and at every time offset computing their similarity (using dot product). Then we find the offset with the highest similarity and that represents the most likely estimated time delay between the two signals. Estimating this delay is called [Time Difference Of Arrival TDOA](https://en.wikipedia.org/wiki/Time_of_arrival).

Once we have the time offset between signals you might be wondering how we then compute the direction of the source. To first get some intuition we could consider what happens when the source is directly in front of the two microphones, so that the distance to each microphone is identical. In this case the time delay would be 0, since by definition the distance is the same. If we consider the other extreme case, when the source is directly in line with our microphones, let's say it's to the right, then the time delay will be at a maximum, since it's the maximum distance separating the two microphones. 

Given the distance between the two microphones is d, our maximum_offset is d/speed_of_wave (341m/s for sound)

* Time offset of -maximum_offset means the source is directly to our left
* Time offset of maximum_offset means the source is directly to our right
* Time offset of 0 means the source is directly ahead or directly behind us

By making the farfield assumption (distance to source is far relative inter-microphone distances) we can draw out some triangles and conclude that given the time delay between microphone signals the direction of the source is arcsin(delay/d)


