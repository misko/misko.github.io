# Wave propagation in the context of direction of arrival (DOA)

## Motivation

You lose your keys, but not to worry, they have a bluetooth enabled tag attached to them. Using your phone you send a radio request, it emits outwards through space around you, piercing through objects like an unstoppable tsunami! Luckily, the tag hears your plea! You hear the tag emit a sound!, but the sound is faint, you slowly move through space, poorly estimating the gradient of volume, after an inital wrong turn, you back track and try again, this time the sound gets louder and louder, until your are face to face with your cat, purring loudly while sitting ontop of them!

![Finding your keys](/assets/2023-6-11-tag-finding.gif)
*An illustration of what your ears might hear when looking the source location of a sound*

It feels like there is something natural about associating direction with sound, but how does it work?  

Your brain has two pieces of information, the sound heard from both ears. From this information there is two basic things you or your brain could do, 
1. Use the volume of the sounds. Maybe it sounds louder in your right ear vs your left ear, then its a pretty good chance the sound is coming from your right. You could turn your head until the sound is equal loudness in both ears, once you stop turning your head you are confident the sound is either directly infront or behind you. Maybe you try moving forward and the sound gets quiter, then it must be coming from behind you. 

2. Maybe your brain is doing something more clever under the hood, a bit of a spoiler, it is! Your brain compares the difference in time between hearing something in your left vs right ear, and uses this to estimate the direction! [interaural time difference (ITD)](https://en.wikipedia.org/wiki/Interaural_time_difference). 

Approach (1) works well in low noise enviornments when you can clearly hear the signal and can sense a large change in volume with small change in space. But imagine you are guiding a boat on a foggy night, and you hear a loud fog horn, oh no! there must be land nearby?! in the moment you cant tell which ear it sounds louder in, the fog horn is just too loud! What do you do?

Approach (1) uses the volume from each ear but discards the information about what was heard, approach (2) on the otherhand discards the volume and uses mostly the information! Let's think about this for a second, sound travels at roughly 340m/s (1100ft/s), the average width of an adult human head is ~0.145m (6inches), the maximum difference in time (sound-time) between your two ears is 0.0004s! Your brain can somehow compare the left and right ear signals, determine a difference smaller than 0.0004s and use it to give you a sense of direction! 

## Waves

Let's say we want to send our friend a electronically generated hello signal using sound (or maybe radio), how can we do this? 

First we might encode our message into some kind of digital form. There are many ways to do this, lets say we choose a binary representation of [1,-1,1]. Once encoded our next step would be to emit this signal from some kind of speaker. We might think the whole situation kind of works like this,

![Digital emission](/assets/2023-6-11-digital-emission.gif) 
*Our first attempt to transmit our message [1,-1,1] to our friend fails. Because our output is not continously varying and therefore it is not actually transmitting*

But this is not exactly how waves work. Waves need to wave!, they can't just be discrete value, well not exactly. Lets try to nail down some properties of waves that are important for our context.

1. Waves originate from a source and propagate uniformly outwards in all directions of space

2. Waves propagate at some speed. For sound that speed is ~340m/s for radio that speed is ~3x10^8 m/s

3. Waves oscillate at some frequency. They go back, and forth, and back, and forth, the rate of back and forths per second is their frequency 

4. Waves have an amplitude. One wave could be twice the disturbance of another, making it have twice the amplitude. One sound could be twice as loud as another 

5. Waves disturb something. Sound waves disturb and propagate through the air and radio waves disturb and propagate through the electromagnetic field

Our problem in the above illustration is that we are just sending 1 or -1 out into the world. That is not really waving! For both sound and radio waves we have to continously vary the output in order to transmit. If the output is constant then nothing is waving if nothing is waving then nothing is being trasmitted. So we can't just transmit 1 or -1! The change in output is actually what is causing the propagation of the wave!

All is not lost though, we just need a layer of abstraction. 

## Encoding information into waves

![Single wave emission](/assets/2023-6-11-single-wave-emission.gif)
*Transmission of a single frequency wave*

The problem in the previous section was broadcasting discrete values (a binary code) instead of continous values. We are able to emit a constantly varying function from our speaker, such as a wave with a single frequency and amplitude (shown above).

How about instead of trying to transmit a constant value, we encode our information into the waves amplitude and frequency. In a very simple case we could use the single wave above and change its amplitude over time. Then when our friend receives the wave, they can decode the message from the amplitudes! This is how [AM radio](https://en.wikipedia.org/wiki/Amplitude_modulation) works. Doing this would result in the below illustration where we successfully send our "hello" message to our friend! 

![AM digital transmission](/assets/2023-6-11-am-digital-transmission.gif)
*The binary message [1,-1,1] is encoded into amplitudes [1.0,0.5,1.0] of a fixed frequency wave that is transmitted to our friend!*

## Audio TDOA

## Audio beamforming

## Radio 

## Radio beamforming


