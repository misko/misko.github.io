---
title: Introduction to estimating direction of arrival
date: 2023-06-11 12:00:00 -800
categories: [signal_processing]
tags: [signal_process,doa,tdoa]
math: true
---

## Motivation

You lose your keys, but not to worry, they have a bluetooth enabled tag attached to them. Using your phone you send a radio request, it emits outwards through space around you, piercing through objects like an unstoppable tsunami! Luckily, the tag hears your plea! You hear the tag emit a sound!, but the sound is faint, you slowly move through space, poorly estimating the gradient of volume, after an initial wrong turn, you backtrack and try again, this time the sound gets louder and louder, until you're face to face with your cat, purring loudly while sitting on top of your keys!

![Finding your keys](/assets/2023-6-11-tag-finding.gif)
*An illustration of what your ears might hear when looking for the source location of a sound*

It feels like there is something natural about associating direction with sound, but how does it work?  

Your brain has two pieces of information, the sound heard from both ears. From this information there are two basic things you or your brain could do, 
1. Use the volume of the sounds. Maybe it sounds louder in your right ear vs your left ear, then it's a pretty good chance the sound is coming from your right. You could turn your head until the sound is equal loudness in both ears, once you stop turning your head you are confident the sound is either directly in front or behind you. Maybe you try moving forward and the sound gets quieter, then it must be coming from behind you. 

2. Maybe your brain is doing something more clever under the hood, a bit of a spoiler, it is! Your brain compares the difference in time between hearing something in your left vs right ear, and uses this to estimate the direction! [interaural time difference (ITD)](https://en.wikipedia.org/wiki/Interaural_time_difference). 

Approach (1) works well in low noise environments when you can clearly hear the signal and can sense a large change in volume with small change in space. But imagine you are guiding a boat on a foggy night, and you hear a loud fog horn, oh no! there must be land nearby?! in the moment you can't tell which ear it sounds louder in, the fog horn is just too loud! What do you do?

Approach (1) uses the volume from each ear but discards the information about what was heard, approach (2) on the other hand discards the volume and uses mostly the time information! Let's think about this for a second, sound travels at roughly 340m/s (1100ft/s), the average width of an adult human head is ~0.145m (6inches), the maximum difference in time (sound-time) between your two ears is 0.0004s! Your brain can somehow compare the left and right ear signals, determine a difference smaller than 0.0004s and use it to give you a sense of direction! 

## Waves

Let's say we want to send our friend an electronically generated hello signal using sound (or maybe radio), how can we do this? 

First we might encode our message into some kind of digital form. There are many ways to do this, let's say we choose a binary representation of [1,-1,1]. Once encoded our next step would be to emit this signal from some kind of source. We might think the whole situation kind of works like this,

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

## Encoding information into waves

![Single wave emission](/assets/2023-6-11-single-wave-emission.gif)
*Transmission of a single frequency wave*

The problem in the previous section was broadcasting discrete values (a binary code) instead of continuous values. We are able to emit a constantly varying function from our source, such as a wave with a single frequency and amplitude (shown above).

How about instead of trying to transmit a constant value, we encode our information into the waves amplitude and frequency. In a very simple case we could use the single wave above and change its amplitude over time. Then when our friend receives the wave, they can decode the message from the amplitudes! This is how [AM radio](https://en.wikipedia.org/wiki/Amplitude_modulation) works. Doing this would result in the illustration below where we successfully send our "hello" message to our friend! 

![AM digital transmission](/assets/2023-6-11-am-digital-transmission.gif)
*The binary message [1,-1,1] is encoded into amplitudes [1.0,0.5,1.0] of a fixed frequency wave that is transmitted to our friend!*

## Direction of arrival estimation

Getting to the real meat of this post, how to solve the original problem: for a set of physical detectors (human ears in the initial example above), given some received signal (sound or radio) how can we estimate the location of the source?

It's important to separate out the direction of a source [Direction Of Arrival - DOA](https://en.wikipedia.org/wiki/Direction_of_arrival) versus the exact location (direction and distance) of the source. If we do not accurately know how the signal degrades over time and direction, the signal strength may not be a good indicator of distance to source. For example if you are standing in a field and hear a faint sound, that sound could be something quiet nearby or it could equally be something very loud far away. You would not know the difference.

## Audio direction of arrival

![Audio Time-Difference-Of_Arrival TDOA](/assets/2023-6-11-tdoa-estimate.gif)
*We fix the location of two microphones and record the sound coming from a source. The signal from each microphone is recorded and shown above. It is clear that the signal from microphone 1 (the microphone farther from the source) is identical to the signal received by microphone 0 , but it is delayed by some time because it is farther away from the source. We can estimate this time delay using cross correlation (right most plot above).* 

To estimate the direction of a sound source we could do something similar to what your brain does [ITD](https://en.wikipedia.org/wiki/Interaural_time_difference). The idea here is that a delay in hearing a sound in one ear versus the other tells us which direction the sound came from. In this section let's replace ears by microphones, record the signal at each, estimate the time delay between the two recordings, and finally use that delay to determine the direction of the source.

### Estimating time delay using cross correlation

To estimate the time delay ([Time Difference Of Arrival TDOA](https://en.wikipedia.org/wiki/Time_of_arrival)) we use a technique called [cross correlation](https://en.wikipedia.org/wiki/Cross-correlation). The idea here is we have two signals, we take a small piece of the first signal and then compare it to every position on the other signal. For each comparison we compute a similarity value (dot product) between the two signals.

![Cross correlation](/assets/2023-6-11-cross-correlation.gif)

In the example above, the signal received by microphone 1 is clearly the same signal as from microphone 0, only it has been delayed by 0.1 seconds. Taking a small slice of the signal from microphone 0 (0.4s to 0.7s), we can compare it to every slice of signal from microphone 1. At each position we compute the dot product and generate a sequence of values called the cross correlation between the signals. Looking at the cross correlation plot , we see a large peak at 0.5s. This is exactly the peak corresponding to a 0.1s delay ([0.1s=0.5s(peak)-0.4s(slice start)])!

### Intuition behind time delay

So let's say we have successfully recorded signals at both microphones and determined the time delay using cross correlation, great, what does that mean?! 

If we detect that there was no delay between the microphones then the distance from the source to both microphones must be identical! There are only two directions this could be, directly ahead or behind!

Directly in front             |  Directly behind
:-------------------------:|:-------------------------:
![](/assets/2023-6-11-front.gif)  |  ![](/assets/2023-6-11-back.gif) 

If the source was directly to the left or right of our microphones, then the time delay must be at its maximal magnitude. If it's to the left , then the left microphone detects it first, and the right microphone is maximally delayed.

Directly to left             |  Directly to right
:-------------------------:|:-------------------------:
![](/assets/2023-6-11-left.gif)  |  ![](/assets/2023-6-11-right.gif) 

So the larger in magnitude the time delay is, the more we know it's to the left or right of our microphones. If the time delay is positive (left microphone received the signal first) then we know the source is to our left. Using both the magnitude and sign of the time delay we should be able to narrow down the direction!


### From time delay to source position (General case)

Given the time delay $\Delta_{time}$ between two received signals, we also know the space delay $\Delta_{space}$, or how much farther in space one signal had to travel relative to the other. We do this by simply multiplying by the speed the wave (speed of sound),

$\Delta_{space} = \Delta_{time} \cdot s_{sound}$ 

To make the math easier (for now) let's assume the source is at some fixed distance $r_0$ from microphone $0$ and $r_1$ from microphone $1$. If we assume $r_0$ and $r_1$ then the location of the source lies at the intersection of two circles centered around each microphone with their respective radius ($r_0$ , $r_1$).

Using the coordinate axis parallel to the microphones we can derive a solution for the source's $x$ coordinate (given $r_0$),

![Detector axis](/assets/2023-6-11-detector-axis.png)

$x^2+y^2=r_0^2$ , $(x-d)^2+y^2=r_1^2$, d = distance between microphones 

$(x-d)^2-x^2=(r_0+\Delta_{space})^2-r_0^2, \Delta_{space}=r_1-r_0$

$x = \frac{d^2-\Delta_{space}^2}{2 \cdot d} - \frac{\Delta_{space}}{d} \cdot r_0$

$y = \sqrt{r_0^2+x^2}$
 
This is interesting! Given $\Delta_{space}$ the $x$ coordinate of the source is linear in terms of $r_0$ (distance between microphone $0$ and the source). To get a better idea of what this looks like in reality, let's fix $\Delta_{space}$ (the difference in distance the signal traveled to the microphones), and solve the above equation for all possible $r_0$.


![Emitter path solution](/assets/2023-6-11-emmiter-path-sol-0.6.gif)
*Fixing $\Delta_{space}=0.6$ and enumerating all possible values for $r_0$ (distance from source to microphone $0$) we can see a path along which the source could be located. As the value of $r_0$ grows the difference in angle formed between the microphone normal vector and the source position converges.*

In the above illustration we fix $\Delta_{space}=0.6$ and solve for all possible positions of the source. From the above it's clear that a single $\Delta_{space}$ gives rise to exactly one path on which the source lies. We can enumerate a contour map of $\Delta_{space}$ across the X/Y plane and see that this is indeed the case.

![Delta space contours](/assets/2023-6-11-contours.png)

In this section we covered how to convert an estimate of $\Delta_{time}$ to $\Delta_{space}$ and then use that to determine a curve on which the source must lay on. One downside of this curve is that near the origin it bends quite a bit, so we cannot be sure of the direction of the sound when the source is close. However when the source is far away from our microphones the curve behaves as a line, and we can be certain of the direction.


### From time delay to source direction (farfield)

In the above section we derived a solution to source direction without additional assumptions. In this section we assume that the distance between our microphones is small relative to the distance between microphone and source. We can see in the figures above that if we are sufficiently far away from the microphones the contours of $\Delta_{space}$ are pretty straight! If we look at the region in space near the microphone array the contours curve quite a bit. This means for a fixed $\Delta_{space}$ we are uncertain about the direction of the source. These observations should motivate the coming assumption and conclusion that given a fixed $\Delta_{space}$ we can estimate direction accurately if the source is far away from our microphones (farfield).

![Farfield approximation](/assets/2023-6-11-farfield_approx.png)

Assuming the distance to our source is large relative to the distance between microphones ($d$) the angle of incidence towards either of our microphones will be very similar $\theta$ and we can solve for $\theta$ given $\Delta_{space}$

$\theta = sin^{-1}( \frac{\Delta_{space}}{d} )$


## Next time

Using the above methods we can estimate sound direction using two (or more)  microphones placed in a linear array. Next we will look how far apart we might want to place our microphones, how sampling speed might affect this and how we can proceed towards estimating direction of radio signals.


