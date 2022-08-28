---
layout: post
title: "Signal Processing Operations"
date: 2022-08-27
---

For analog signals, most of the processing operations are performed in the time domain, on the other side, for discrete-time signals the time and frequency domain operations are employed.

# Simple Time-Domain Operations

The signal processing operations are building blocks that will allow to design a more complex set of processing tasks. These operations are detailed as follows:

#### Scaling

The multiplication of the signal by a positive/negative constant. For analog signals, this operation is called _amplification_ if the multiplier is greater than one, otherwise is called _attenuation_. More formally, if \\(x(t)\\) is the analog signal, the scaling operation generates a \\(y(t) = \alpha x(t)\\).

#### Delay

Generates a replica signal delayed in respect to the original. For an analog signal \\( x(t) \\), a delayed signal will be \\( y(t) = x(t - t_0) \\) where \\( t_0 \\) is the amount of time units in which the signal is delayed. If \\( t_0 \\) is positive, then this is a delay operation, otherwise, is an _advance_ operation.

#### Addition

Two or more signals can be added to generate a new one. Lets say \\( y(t) = x_1(t) + x_2(t) - x_3(t) \\) is a signal formed by the addition (and substraction) of three signals.

#### Product

Another elementary operation is the product of two signals, for example \\( y(t) = x_1(t) x_2(t) \\)

#### Integration and differentiation.

The integration of a signal \\( x(t) \\) will generate a signal \\( y(t) = \int_{-\infty}^{t} x(\tau) \mathrm{d}\tau \\), while the differentiation is defined as \\( w(t) = \frac{\mathrm{d}x(t)}{\mathrm{d}t} \\).

It should be noticed that the first three elementary operations, _scaling_, _delay_ and _addition_, can be also applied to discrete-time domain signals. However, _integration_ and _differentiation_ are used mostly for discrete-time domain signals.

# Filtering

The main objective of filtering is to alter the spectrum according to some provided specifications. For example, a filter can be designed to allow some specific frequency components to stay in the signal, while others can be blocked. A _passband_ is the range of frequencies allowed to pass through the filter, whereas a _stopband_ is the range of frequencies blocked by the filter.

More formally, a filter is characterized by an impulse response \\( h(t) \\), then an output signal is given by:

$$ y(t) = \int_{-\infty}^{\infty} h(t - \tau) x(\tau) \mathrm{d}\tau $$

Or using the frequency domain, the same equation can be expressed as:

$$ \mathrm{Y}(j\Omega) = \mathrm{H}(j\Omega) \mathrm{X}(j\Omega)$$

## Common filter classifications

#### Lowpass filter

A _lowpass_ filter allows all low-frequency components below a specified frequency \\( f_p \\), called the _passband edge frequency_, and blocks all high-frequency components above \\( f_s \\), called the _stopband edge frequency_.

#### Highpass filter

A _highpass_ filter passes all high-frequency components above \\( f_p \\) and blocks all low-frequency below \\( f_s \\).

#### Bandpass filter

This filter passes all frequency components between two passband edge frequencies \\( f_{p1} \\) and \\( f_{p2} \\) where \\( f_{p1} < f_{p2} \\), and blocks all frequency components below \\( f_{s1} \\) and above \\( f_{s2} \\).

#### Bandstop filter

A _bandstop_ filter blocks all frequency components between two stopband edge frequencies, \\( f_{s1} \\) and \\( f_{s2} \\), and allow all frequencies below and above of \\( f_{p1} \\) and \\( f_{p2} \\).

A bandstop filter that only blocks a specific frequency is called a _notch filter_.




