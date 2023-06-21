---
title: Introduction to beamforming
date: 2023-06-21 12:00:00 -800
categories: [signal_processing]
tags: [signal_process,doa,tdoa]
math: true
image: /assets/2023-6-11-post-image.png
description: Let's do some basic beamforming
---

## Motivation

Your friend quiet Bob asked for your help to record his band's live show! Previously he used a single microphone in the bar to record their performances, but there was a lot of noise coming through on the recording and it was hard to hear the actual band. 

### Minor details
 
Some details that are not directly relevant but might help guide this post:
* Speed of sound is ~ $341 m/s$
* Wave frequency is $300hz$ (A relatively bass sound)
* Wavelength here is $341/300=1.13 m$

![Single microphone recording](/assets/2023-6-21-single-microphone-recording.gif)
*Using a single microphone you record the band's rehearsal and confirm it is noisy! The source here is the band and the receiver is the single microphone that is recording*

Being the good friend you are, you venture down to the bands new venue and start recording while they rehearse. Immediately you confirm the hypothesis, using a single microphone the noise is too great, and affects the recording quality.

What can you do?

# More microphones

You get to thinking, if one microphone is too noisy, maybe the average of two microphones might cancel out the (uncorrelated) noise! You get to work, set up an additional microphone and start recording. However you quickly realize the average signal of the two microphones is more noisy than the original recording! How is that possible?

![Average of two microphones half wavelength apart](/assets/2023-6-21-two-microphones-noise-halfwave.gif)
*Using the average of two microphones placed some distance apart results in a worse signal than any single recording from either microphone alone!*

Baffled by the results, you stare at the above analysis and try to figure out what is going on. It looks like the time delay between the two microphones lines up the peaks from one microphones signal with the troughs in the other microphones signal. This is probably causing [destructive interference](https://en.wikipedia.org/wiki/Wave_interference) and erasing the signal. Thinking on your feet you figure that if the current spacing is causing destructive interference (lining up peaks with troughts), then doubling the spacing must line up the peaks with peaks and increase signal! 

![Average of two microphones full wavelength apart](/assets/2023-6-21-two-microphones-noise-fullwave.gif)
*Spacing the microphones a full wavelength apart increases the quality of the averaged signal by quite a bit! At least compared to the original spacing of half a wavelength (above)*

By golly! It worked! The noise is reduced (even though only a little) and the signal prevails. You can't help but wonder, if the angle between the microphones and the direction of the source sound matters, was it just luck that half wavelength was a bad spacing and full wavelength was a good spacing? Is there a better spacing you could choose for this setup? What if instead of changing the spacing, you just artificially reduce the delay from the second microphone, would that fix the averaging issue? By how much time should you shift the signal from the second microphone? 

Based on the previous [post](/posts/introduction_to_doa), you might think to use cross correlation to determine this time delay, and that would work. However let's consider another approach here, one that exploits the wave nature of our signal. If we assume that our signal is wave-like can we just speed up or slow down the [phase](https://en.wikipedia.org/wiki/Phase_(waves)) of the wave to determine the delay? 

If we can find this delay, we could shift one of the signals in such a way to line up peaks with peaks (and troughs with troughs) and sum them together to get a better average signal!

## Beamforming assumptions

Let's assume the signal at microphone 0 (right microphone) is $sin(2 \pi t)$ and the signal at microphone 1 (left microphone) is $sin(2 \pi t+\Delta_{phase})$. If we can figure out $\Delta_{phase}$ then we could shift the signal of microphone 1 by this amount and perfectly line up peaks with peaks! 

It's clear from the above signal assumptions that we must limit $\Delta_{phase}$ to the interval $[-\pi,\pi]$ since any value outside this would just repeat an observation in the original interval $[-\pi,\pi]$. This means that as long as we can confidently say the $\Delta_{phase}$ is bound in the range of $[-\pi,\pi]$ we can use the above assumption. The way we can bound $\Delta_{phase}$ to the desired region is by enforcing that the distance between receivers is less than $\frac{\lambda}/{2}$ ($\lambda$ = wavelength). If this is true, then the distance cannot be more than $\frac{\lambda}{2}$, which is exactly half a wave cycle, which is exactly $\lvert \Delta_{phase} \rvert \lt \pi$. Cool! So if we manage to keep the distance between receivers relatively small, and we can keep the $\Delta_{phase}$ in a nice region to work with.

## Complex numbers

![sin(x+d)](/assets/2023-6-21-sin.png)

Let's say we have some signal $ sin(x) $ and we want to compute $ sin(x+\Delta) $ ($\Delta$ small), how can we do this? It's really not obvious. 

If we bound $x$ to an interval $[-\pi,\pi]$, we can narrow down $x$ to at most two possible values. However one of these values has a positive derivative and the other negative, which means that computing $ sin(x+\Delta) $ will be impossible, because we don't know if we should go a little up or a little down. 

![complex sin(x+d)](/assets/2023-6-21-complex-sin.png)

Luckily enough for us, someone else already developed the theory for complex numbers, and that is exactly what we need here! In the above image we can see that $sin(x)$ is really just the $real$ part of $ e^{i (x - \pi/2)} = cos(x - \pi/2) + i sin(x - \pi/2) = sin(x) + i sin( x - \pi/2)$ .  

If we were to store the original signal somehow as a complex number ($e^{ix}$), we could use [complex multiplication](https://en.wikipedia.org/wiki/Complex_number#Multiplication_and_division_in_polar_form) to easily shift the real part of the signal by $\Delta$, 

$ sin(x+\Delta) = Real( e^{i (x - \pi/2 + \Delta) } = Real(  e^{i (x - \pi/2)} \cdot  e^{i \Delta}) $

## Beamforming

That's a bunch of theory, but does this work? Let's give it a try! Eyeballing the above analysis it looks like the delay between microphones is close to the distance between microphones. Since we chose the distance between microphones to be $\lambda/2$ (half a wavelength), then an offset of $\Delta_{phase}=\pi$ should really help clean up the signal in the average, and that is exactly what we see!

![Shift signal by pi before averaging](/assets/2023-6-21-two-microphones-noise-pi-offset.gif)

If we shift the signal from microphone 1 by $\lambda/2$ or $\Delta_{phase}=\pi$ before averaging signals between microphones we see a boost in resulting signal quality (red vs not shifting in green).

Maybe instead of guessing a fixed $\Delta_{phase}=\pi$ what we could do is enumerate all possible $\Delta_{phase}$ and ask the question, at which $\Delta_{phase}$ do we see the best signal quality? This is beamforming! Using the direction of an emitter or receiver we can offset (steer) our array (of emitters or receivers) to better receive (or emit) in that direction.

Let's try this out! To measure "signal quality" lets take the mean of the absolute value of the signal, that way the larger the spikes (peaks or troughs) the better our "signal quality" measure will be.

![Beamforming](/assets/2023-6-21-beamform-rotate.gif)

From the above we see that our initial guess of the signals being delayed by $\Delta_{phase}=\pi/2=180deg$ is not that far from the best possible signal quality at $\Delta_{phase}=149deg$. 

![Shift signal by 149deg before averaging](/assets/2023-6-21-two-microphones-noise-best149.gif)


## DOA (direction of arrival) using beamforming

Beamforming tells us that the optimal phase delay is $\Delta_{phase}=149deg$ , we can use this in combination with wavelength to compute the difference in distance that the signal had to travel to microphone 1 (vs microphone 0 [right]), 

$\Delta_{space}=\Delta_{phase} \cdot \lambda$

And we can solve for the direction of the source from our receiver axis (using far field assumption from last [time](/posts/introduction_to_doa)),

$\theta=sin^{-1}(\frac{\Delta_{space}}{d})=sin^{-1}( \frac{(149deg/360deg) \cdot \lambda}{\lambda \cdot 0.5})=56deg$


## Conclusion

We can definitely help Bob record a better quality sound by adding an additional microphone to his recording setup. Even more than just increasing the signal quality by an additional microphone we have developed a beamformer to help us evaluate direction of arrival, given some guess of direction ($\Delta_{phase}$) we can compute a signal strength measure. Under the assumption that the signal strength is maximal when we guess the right direction, we can use beamforming to estimate the direction of the source!

We could use the above to add even more microphones to our array and get an even better signal! If we assume the noise is not correlated and 0 centered, we can reduce noise much more by adding microphones to our array. In the approach above we did something a bit different than last time, the critical part from this time is that we exploited the wave nature of our signal. We did this by both bounding the distance between receivers by a factor of the wavelength and also by recording our signal in the complex plane (instead of just the real). 

Given current hardware and physical constraints, using TDOA for estimating DOA from radio in a small space becomes challenging, if not impossible. However, we can (as above) exploit the wave part of the radio signal to still perform beamforming and estimate the direction of the source. 


