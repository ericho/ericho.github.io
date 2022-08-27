---
layout: post
title: "Notes on Signals and Signal Processing"
date: 2022-08-22
---

_This is a set of notes in Signal and Signal Processing for my understanding, hopefully this can be useful for my future self._

Signals are everywhere in our sorrounding world. Typical examples are music, images, video and communication. A signal, in a formal attempt, is a function of independent variables such time, pressure, temperature, height, distance and so on. Music signals, for instance, represents air pressure as a function of time in a point in space. A photograph is a representation of colors as a function of two spatial coordinates.

Signals are commonly natural generated, however also signals can be created synthetically. Signals are carriers of information and the Signal Processing goal is to retrieve such information using different methods that depends on characteristics of the signal. So, signal processing utilizes the mathematical representation of the signal and algorithms designed to extract information from the signal. The representation of the signal, mathematically speaking, can be defined in terms of functions in the domain of the original independent variables or in the domain of functions of transformed variables. In other words, signals can be processed in their original form or be transformed for further analysis.

## Classification of signals

The independent variables can be continuous or discrete, therefore the signals can be either a continuous or a discrete functions of the independent variables. Even more, signals can be functions of real or complex values. Signals can have a single source or multiple sources. Single source can be named either, a scalar signal or a single channel signal. Signals with multiple sources can be named vector signals or multichannel signals. Also signals can have one of multiple dimensions, a signal of one dimension is a function of a single independent variable. A two dimensions signal is a function of two independent variables. hence, a m-dimensions signal is a function of more than one variable.

Here are some examples of signals and their dimensions:

  - _Speech_ : This is a 1-D signal where the independent variable is time.
  - _Image_ : This is a 2-D signal where the independent variables are the two spatial positions.
  - _Black and White Video_ : Each frame in a video is a signal of 2-D, like an image. However as these frames changes over time, then a third variable is required. Thus the video is a 3-D signal.

Note: Seems that color video will be an example of a signal higher than 3-D, each frame has three components for each color RGB.

In a 1-D signal, the independent variable is usually seen as time. If this variable is continuous, then is called continuous-time signal. These signals are defined at every instant over time. If the independent variable is discrete, then is a discrete-time signal. Discrete-time signals takes certain values at specific instants over time, between each instant the signal is not defined. Discrete-time signals can be described a sequence of numeric values.

With all these terms, the following signals can be defined.

  - _Analog signals_ : A continuous-time signal with continuous amplitude. These are the signals generally found in nature.
  - _Digital signal_ : A discrete-time signal with discrete-valued amplitudes represented within a range of finite numbers. Any signal digitized in a computer would an example of this type of signals.
  - _Sampled-data signal_: A discrete-time signal with continuous-valued amplitudes. These signals can be found in switched capacitors circuits.
  - _Quantized boxcar signal_: A continuous-time signal with discrete-valued amplitudes. A clock in a circuit or data transmission over a wire can be examples of these signals.

$$ \nabla_\boldsymbol{x} J(\boldsymbol{x}) $$
