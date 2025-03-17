---
layout: post
title: "1. The basics"
author: "Bolton Tran"
categories: tutorials
tags: [fourier]
image: "/assets/img/fourier1.png"
---
Fourier transform is a common technique used in signal processing.
Its applications extend to 
physics and chemistry,
particularly in molecular simulations
where thermal fluctuation can muddle the observables (e.g., energy, atomic position).
We will explore the basics of Fourier transform here,
before jumping into some applications in subsequent posts.

Note that the math of Fourier transform is better documented else where
(e.g., <a href="https://mathworld.wolfram.com/FourierTransform.html" target="_blank">Wolfram</a>).
I want to simply outline the practical use of Fourier transform in this tutorial.

Let's consider a periodic time series,
following a sine function with some frequency and amplitude.
Consider next random noise
that blurs the periodicity of this time series.
How may we recover the frequency and amplitude
of the original signal (i.e., the sine function)? \\
$\rightarrow$ Fourier transform converts the time series into a frequency space,
allowing for quick discerning of the periodicity in our signal.

Using Python, I constructed below an example
of a sine time series with random Gaussian noise.
Playing with the interactive plot below 
should give a better visualization of what Fourier transform entails.
```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.fft import rfft, rfftfreq
#Define time range
dt=0.001 #step size
time=np.arange(0, 1, dt)
#Create signal as a sine function with random Gaussian noise
freq=5
amplitude=2
noise=1
signal=amplitude*np.sin(2*np.pi*freq*time) + noise*np.random.randn(len(time))
#(Discrete) Fourier transform
yf=rfft(signal)
freqs=rfftfreq(len(signal), dt)
#Plot
plt.figure()
plt.plot(t, signal) #signal in real time
plt.figure()
plt.plot(freqs, np.abs(yf)) #signal in frequency
plt.show()
```
<div style="display: flex; justify-content: space-between; width: 100%;">
    <div style="width: 30%; text-align: left;">
        <span>Frequency ($s^{-1}$):</span>
        <input type="range" id="fslider" min="1" max="50" step="1" value="10" style="width: 80%;">
        <span id="fspan">10</span>
    </div>
    <div style="width: 30%; text-align: left;">
        <span>Amplitude:</span>
        <input type="range" id="amslider" min="1" max="5" step="0.1" value="5" style="width: 80%;">
        <span id="amspan">5</span>
    </div>
    <div style="width: 30%; text-align: left;">
        <span>Noise:</span>
        <input type="range" id="noiseslider" min="0" max="5" step="0.1" value="0" style="width: 80%;">
        <span id="noisespan">0</span>
    </div>
</div>
<div id="plotDiv" style="width: 100%; height: 500px; margin: 0px auto;"></div>

<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
<script>
  const plotDiv = document.getElementById('plotDiv');
  const fslider = document.getElementById('fslider');
  const amslider = document.getElementById('amslider');
  const noiseslider = document.getElementById('noiseslider');
  // Initialize plot
  function fetchDataAndUpdatePlot(freq, amplitude, noise) 
  {
    const url = `https://bolton2710.pythonanywhere.com/fourier1?freq=${freq}&amplitude=${amplitude}&noise=${noise}`;
    fetch(url)
      .then(response => response.json())
      .then(data => {
        const signal = data.signal.map(Number);
        const t = data.t.map(Number);
        const fourier = data.yf.map(Number);
        const freqs = data.freqs.map(Number);
        const plotData = [
          {x: t, y: signal, type: 'scatter', line:{color:'blue'}, name:'Signal', hoverinfo: 'x+y', xaxis: 'x1', yaxis: 'y1'},
          {x: freqs, y: fourier, type: 'scatter', line:{color:'red'}, name:'Fourier', hoverinfo: 'x+y', xaxis: 'x2', yaxis: 'y2'}
          ];
        const layout = {
          grid: { rows: 2, columns: 1, pattern: 'independent' },
          xaxis: {title: 'Time (s)', fixedrange: true, range: [0,1], tickfont:{size:16}, titlefont:{size:17}},
          yaxis: {title: 'Amplitude', fixedrange: false, tickfont:{size:16}, titlefont:{size:17}},
          xaxis2: {title: 'Frequency (Hz)', fixedrange: true, range: [0,200], tickfont:{size:16}, titlefont:{size:17}},
          yaxis2: {title: 'Magnitude', fixedrange: false, tickfont:{size:16}, titlefont:{size:17}},
          margin: {l: 85, r: 20, t: 30, b: 40},
          showlegend: false
        };
        Plotly.newPlot(plotDiv, plotData, layout);
      })
      .catch(err => console.error("Error fetching data:", err));
  }
  // Initial plot
  let freq = 10;
  let amplitude = 5;
  let noise = 0;
  fetchDataAndUpdatePlot(freq, amplitude, noise);
  // Update plot on slider change
  fslider.addEventListener('input', () => {
    freq = parseFloat(fslider.value);
    fspan.textContent = freq;
    fetchDataAndUpdatePlot(freq, amplitude, noise);
  });
  amslider.addEventListener('input', () => {
    amplitude = parseFloat(amslider.value);
    amspan.textContent = amplitude;
    fetchDataAndUpdatePlot(freq, amplitude, noise);
  });
  noiseslider.addEventListener('input', () => {
    noise = parseFloat(noiseslider.value);
    noisespan.textContent = noise;
    fetchDataAndUpdatePlot(freq, amplitude, noise);
  });
</script>