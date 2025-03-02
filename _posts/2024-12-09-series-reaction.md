---
layout: post
title: "1. Reaction in series"
author: "Bolton Tran"
categories: tutorials
tags: [rxn]
image: "/assets/img/che1.jpg"
---
Consider a gas-phase reaction in series:
$$A \overset{k_1}{\rightarrow} B \overset{k_2}{\rightarrow} C$$
taking place in an isothermal batch reactor,
we want to solve for the concentrations of $$A$$, $$B$$, and $$C$$
versus time.

We write the rate of change of concentration of each species with respect to time as follows:
<div>
\begin{align}
\frac{dC_A}{dt} &= -k_1 C_A\\
\frac{dC_B}{dt} &= k_1 C_A - k_2 C_B\\
\frac{dC_C}{dt} &= k_2 C_B
\end{align}
</div>
We can solve the above ordinary differential equations (ODEs)
once values for rate constants $$k_1$$ and $$k_2$$ are known.
We can use an approximate form of transition state theory (TST)
to calculate $$k_1$$ and $$k_2$$:
<div>
\begin{align}
k_1 &= \frac{RT}{h}\exp{\left(-\frac{E^\dagger_1}{RT}\right)}\\
k_2 &= \frac{RT}{h}\exp{\left(-\frac{E^\dagger_2}{RT}\right)}
\end{align}
</div>
The activation energies $$E^\dagger_1$$ and $$E^\dagger_2$$
and the temperature $$T$$ control reaction rates.
From a simple collision theory perspective,
larger activation energies slow reaction down
since the probability of particles' collision
having enough energy for chemical reaction
is lower;
higher reaction temperature speeds reaction up
since the average kinetic energy of particles 
and the chance of them colliding increase.
Equivalent TST interpretation involves
invoking the shift in a quasi-equilibrium
between the transition state and the initial state.

<span>$$T$$ (K):</span>
<input type="range" id="Tslider" min="400" max="500" step="1" value="400">
<span id="Tspan">400</span>

<span>$$E^\dagger_1$$ (kJ/mol):</span>
<input type="range" id="EA1slider" min="140" max="150" step="0.1" value="140">
<span id="EA1span">140</span>

<span>$$E^\dagger_2$$ (kJ/mol):</span>
<input type="range" id="EA2slider" min="150" max="160" step="0.1" value="150">
<span id="EA2span">150</span>

<div id="plotDiv" style="width: 100%; height: 500px; margin: 0px auto;"></div>

Solving ODEs is a must-have skill
for chemical engineers.
While many ODE solver in different coding languages 
(MATLAB, Wolfram, Scipy) are available,
knowing how to set up an ODE problem
in one code
allows for easy transfer to another code.
Demonstration using Python/Scipy
is shown below:

```python
import numpy as np
from scipy.integrate import odeint #module for ODE solver

#Constants
R=8.3144/1000 #kJ/mol.K
h=4.135667696E-15*96.49/3600 #kJ/mol.h

#Solve kinetic ODE at a given temperature and EAs
def solveODE(EA1, EA2, T):
    #Rate constants
    k1=(R*T/h)*np.exp(-EA1/R/T)
    k2=(R*T/h)*np.exp(-EA2/R/T)  
    #Define ODEs
    def dcdt(c,t):
        #c is array for concentrations
        cA=c[0]; cB=c[1]; cC=c[2]
        dcAdt=-k1*cA
        dcBdt=k1*cA - k2*cB
        dcCdt=k2*cB
        return [dcAdt, dcBdt, dcCdt]
    #Initial parameters
    c0=[2,0,0] #mol/L
    #solve ODE
    cs=odeint(dcdt, c0, time)
    return cs

#Define a time range
t0=0 
t1=50
time=np.linspace(t0,t1,100) #hr

#Invoke ODEsolve function for a given set of parameters
para=[150,160,400] #EA1, EA2, T
result=solveODE(*para) #solve
print(result)
```

For anyone who is curious:
the interactive plot
was created by
hosting a 
python-Flask (a Web Server Gateway Interface application) script
on https://www.pythonanywhere.com/.
The script contains python functions
that perform mathematical operations.
The sliders send values of
($$E^\dagger_1$$, $$E^\dagger_2$$, $$T$$) to the PythonAnywhere app,
which performs the above Scipy-ODE solver
and outputs results as json arrays.
The json arrays
are then parsed back to Plotly
written in JavaScript,
which created the plot interactively.

This interactive exercise
is heavily inspired by the 
[LearnChemE](https://learncheme.com/simulations/kinetics-reactor-design/series-reactions-in-a-batch-reactor/) group
at the University of Colorado Boulder.
Check out their library for other cool interaction simulations
for chemical engineering students.

<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
<script>
  const plotDiv = document.getElementById('plotDiv');
  const Tslider = document.getElementById('Tslider');
  const Tspan = document.getElementById('Tspan');
  const EA1slider = document.getElementById('EA1slider');
  const EA1span = document.getElementById('EA1span');
  const EA2slider = document.getElementById('EA2slider');
  const EA2span = document.getElementById('EA2span');
  // Initialize plot
  function fetchDataAndUpdatePlot(EA1, EA2, T) {
    const url = `https://bolton2710.pythonanywhere.com/che1?EA1=${EA1}&EA2=${EA2}&T=${T}`;
    fetch(url)
      .then(response => response.json())
      .then(data => {
        const ca = data.ca.map(Number);
        const cb = data.cb.map(Number);
        const cc = data.cc.map(Number);
        const t = data.t.map(Number);
        const plotData = [
          { x: t, y: ca, type: 'scatter', line:{color:'blue'}, name:'$C_A$', hoverinfo: 'x+y'},
          { x: t, y: cb, type: 'scatter', line:{color:'red'}, name:'$C_B$', hoverinfo: 'x+y'},
          { x: t, y: cc, type: 'scatter', line:{color:'green'}, name:'$C_C$', hoverinfo: 'x+y'}];
        const layout = {
          xaxis: {title: 'Time (hr)', showgrid: false, tickmode: 'linear', ticks: 'outside', fixedrange: true, range: [0,50], tickangle:0, dtick: 5, tickfont:{size:16}, titlefont:{size:17}},
          yaxis: {title: 'Concentration (mol/L)', showgrid: false, tickmode: 'linear', ticks: 'outside', fixedrange: false, range: [0,2.1], tickfont:{size:16}, titlefont:{size:17}},
          legend: {x: 0.5, y: 1.05, xanchor: 'center', yanchor: 'bottom', orientation: 'h', font: {size:16}},
          }
        Plotly.newPlot(plotDiv, plotData, layout);
      })
      .catch(err => console.error("Error fetching data:", err));
  }
  // Initial plot
  let T = 400;
  let EA1 = 140;
  let EA2 = 150
  fetchDataAndUpdatePlot(EA1,EA2,T);
  // Update plot on slider change
  Tslider.addEventListener('input', () => {
    T = parseFloat(Tslider.value);
    Tspan.textContent = T;
    fetchDataAndUpdatePlot(EA1,EA2,T);
  });
  EA1slider.addEventListener('input', () => {
    EA1 = parseFloat(EA1slider.value);
    EA1span.textContent = EA1;
    fetchDataAndUpdatePlot(EA1,EA2,T);
  });
  EA2slider.addEventListener('input', () => {
    EA2 = parseFloat(EA2slider.value);
    EA2span.textContent = EA2;
    fetchDataAndUpdatePlot(EA1,EA2,T);
  });  
</script>
