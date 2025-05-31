---
layout: post
title: "2. Microkinetic reactions"
author: "Bolton Tran"
categories: tutorials
tags: [rxn]
image: "/assets/img/che2.jpg"
---
Many chemical reactions
take place on a catalyst surface.
Mechanisms of surface catalytic reactions
are often broken down
to elementary steps,
often involved adsorption, reaction, and desorption.
Setting up and solving these microkinetic models
are the basis for 
analyzing catalytic reactions.
As usual,
the theories and model derivation
could be found else where;
the practical problem solving in Python
is the main focus here.

<h2>
Microkinetic model
</h2>
The microkinetic model
draws parallel from
<a href="https://bolton2710.github.io/series-reaction" target="_blank">Part 1. Reactions in series</a>,
i.e.,
we will solve a systems of ODE.
The difference is the way
we construct the chemical equations:

<div>
\begin{align}
A_{(g)} + \ast \text{ } &\overset{k^f_1}{\underset{k^r_1}{\longleftrightarrow{}}} \ast A\\
B_{(g)} + \ast \text{ } &\overset{k^f_2}{\underset{k^r_2}{\longleftrightarrow{}}} \ast B\\
\ast A + \ast B \text{ } &\overset{k^f_3}{\underset{k^r_3}{\longleftrightarrow{}}} \ast C + \ast\\
\ast C \text{ } &\overset{k_4}{\longrightarrow{}} C_{(g)} + \ast 
\end{align}
</div>

This set of chemical equations represents
a bimolecular reaction between $$A$$ and $$B$$
taking place on a catalyst surface.
The catalyst active site is denoted as $$\ast$$,
where gaseous $$A_{(g)}$$ and $$B_{(g)}$$ molecules
can adsorb/desorb to and from (first two equations).
Adsorbed $$\ast A$$ and $$\ast B$$ react
to form $$\ast C$$ adsorbed on one site 
and freeing up one site (third equation).
Finally,
$$C_{(g)}$$ desorbs from the surface (last equation)
as the product of this catalytic cycle.

For simplicity,
we assume that
the partial pressure (i.e., concentration) of $$A_{(g)}$$ and $$B_{(g)}$$
is time invariant,
and that the partial pressure of $$C_{(g)}$$ is negligible
(i.e., no re-adsorption of desorbed $$C$$).
But in principle,
the partial pressures of gaseous species
can be solved with respect to time as well.

We write the ODEs
for the rate of change in concentration of each **surface** species:
<div>
\begin{align}
\frac{d\theta_A}{dt} &= k^f_1 \theta_v p_A - k^r_1\theta_A - k^f_3 \theta_A \theta_B + k^r_3 \theta_v \theta_C\\
\frac{d\theta_B}{dt} &= k^f_2 \theta_v p_B - k^r_2\theta_B - k^f_3 \theta_A \theta_B + k^r_3 \theta_v \theta_C \\
\frac{d\theta_C}{dt} &= k^f_3 \theta_A \theta_B - k^r_3 \theta_v \theta_C - k_4 \theta_C \\
\frac{d\theta_v}{dt} &= -k^f_1 \theta_v p_A + k^r_1\theta_A - k^f_2 \theta_v p_B + k^r_2\theta_B 
                        +k^f_3 \theta_A \theta_B - k^r_3 \theta_v \theta_C + k_4 \theta_C\\
\end{align}
</div>
Here,
$$\theta$$ represents the fractional coverage of
chemical species $$A$$, $$B$$, $$C$$,
and vacant site $$v$$ on the surface.
That is,
$$\theta_v + \theta_A + \theta_B + \theta_C = 1$$ at all time.
The ODEs already ensure the detailed balance of surface species,
we just need to set the initial coverage sum to 1 
when solving for the equations.

The rate constants are temperature dependent,
and differ slightly depending on the elementary steps:
<div>
\begin{align}
k_{\text{ads}} &= \frac{A}{\sqrt{2\pi m k_B T}} \\
k_{\text{des}} &= \frac{k_BT}{h}\exp{\left(-\frac{\Delta E_{\text{des}}}{k_BT}\right)} \\
k_{\text{rxn}} &= \frac{k_BT}{h}\exp{\left(-\frac{\Delta E^\dagger_{\text{rxn}}}{k_BT}\right)}
\end{align}
</div>
For adsorption and desorption,
we use the Hertz-Knudsen formulation.
The rate constant for adsorption
is related to a loss in transtional entropy
of a gaseous molecule with 3D motion
to an adsorbed molecule with 1D motion (and no enthalpic component).
For simplicity,
let's assume a mass of $$m=10$$ amu (a Neon atom)
and an area of $$A=4$$ Å<sup>2</sup> for each active site.
The rate constants for desorption and reaction 
are purely enthalpic loss
and follow the transition state theory.
I have arbitrarily chosen
the desorption energy of $A$, $B$, and $C$
to be $50$, $40$, and $40$ kJ/mol, respectively.
Likewise,
the forward and backward activation energy
of the surface reaction are
$80$ and $60$ kJ/mol.

<h2>
Time series solution
</h2>
In Python,
the microkinetic ODEs
are set up similarly as in 
<a href="https://bolton2710.github.io/series-reaction" target="_blank">Part 1</a>
(see below).
The time variable
is set up in a logarithmic scale,
spanning from picosecond ($10^{-12}$ s)
to second ($1$ s).

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy.integrate import odeint
#Constants
kB=1.380649E-23 #J/K
mass=10*1.66054e-27 #kg
area=(2E-10)**2 #meter^2
h=6.6260707E-34 #J.s
#Choose temperature and pressures
T=300
pA=1
pB=1
#Functions for rate constants
def kads(T):
    return area/np.sqrt(2*np.pi*mass*kB*T)*101325 #s-1
def kdes(T, Edes):
    return kB*T/h*np.exp(-Edes/8.314/T*1000) #s-1
def krxn(T, Eact):
    return kB*T/h*np.exp(-Eact/8.314/T*1000) #s-1
#Calculate rate constants
k1f=kads(T)
k1r=kdes(T, 50)
k2f=kads(T)
k2r=kdes(T, 40)
k3f=krxn(T, 80)
k3r=krxn(T, 60)
k4f=kdes(T, 40)
#Create time array in log scale
time_min=1E-12
time_max=1E2
logtime=np.linspace(np.log10(time_min), np.log10(time_max), int((np.log10(time_max) - np.log10(time_min))*10))
#ODE setup
def dcdt(thetas, t):
    tv=thetas[0]; tA=thetas[1]; tB=thetas[2]; tC=thetas[3] #vacant, A, B, and C
    dtAdt=k1f*tv*pA - k1r*tA - k3f*tA*tB + k3r*tC*tv
    dtBdt=k2f*tv*pB - k2r*tB - k3f*tA*tB + k3r*tC*tv
    dtCdt=k3f*tA*tB - k3r*tC*tv - k4f*tC
    dtvdt=-k1f*tv*pA + k1r*tA - k2f*tv*pB + k2r*tB + k3f*tA*tB - k3r*tC*tv + k4f*tC
    return [dtvdt, dtAdt, dtBdt, dtCdt]
#Solve ODE
theta0=[1,0,0,0] #initial guess
thetas=odeint(dcdt, theta0, 10**logtime) #solve in real time
#Plot in semilog
[plt.semilogx(10**logtime,thetas.T[i]) for i in range(4)]
plt.show()
```
<div style="display: flex; justify-content: space-between; width: 100%;">
    <div style="width: 30%; text-align: left;">
        <span>Temperature (K):</span>
        <input type="range" id="Tslider" min="300" max="800" step="10" value="300" style="width: 80%;">
        <span id="Tspan">300</span>
    </div>
    <div style="width: 30%; text-align: left;">
        <span>Pressure of A (bar):</span>
        <input type="range" id="PAslider" min="0.1" max="2" step="0.1" value="1" style="width: 80%;">
        <span id="PAspan">1</span>
    </div>
    <div style="width: 30%; text-align: left;">
        <span>Pressure of B (bar):</span>
        <input type="range" id="PBslider" min="0.1" max="2" step="0.1" value="1" style="width: 80%;">
        <span id="PBspan">1</span>
    </div>
</div>
<div id="plotDiv1" style="width: 100%; height: 500px; margin: 0px auto;"></div>

In above interactive plot of
surface coverage versus time,
there is a few key observations:\\
1) After a "long" time (above microsecond),
all microscopic steps reach steady-state, i.e.,
the coverages no longer vary with time.\\
2) The rates of appearance of adsorbed $A$ or $B$
depend on the temperature and the respective partial pressures.\\
3) Species $C$ always appear after $A$ and $B$. \\
4) Across all temperatures and pressures,
$A$ species cover the surface more than $B$
because the desorption of $A$ is more costly in energy.\\
5) The coverage $C$ is very small (see right y-axis)
because the rate of forming $C$ is much smaller
the the rate of desorbing $C$.

<h2>
Steady-state solution
</h2>
While time-varying coverages are interesting 
and sometimes important for fundamental catalysis research,
let's say we now care only about
the system at steady-state.
Per the interactive time series,
the system reaches steady state
at $10^2$ second for all temperatures and pressures.
For chemical reaction engineers,
the natural question is
how the steady-state coverages
and production rate
vary with temperatures and pressures.
The rate of production of $C$
is the rate of desorption of $\ast C$,
which is the catalytic turnover frequency 
(TOF, unit s<sup>-1</sup>).

The Python code can be appended to
obtain the steady-state coverages
and TOF
at each temperature
and any given pressures.
There are two ways to go about this:\\
1) Solve the ODEs and extract the final values in the time-series.\\
2) Sovle for a set of nonlinear equations 
that assume all time-derivatives equal to zero.
As seen below,
the nonlinear equations
need to have an explicit site balance for
$$\theta_v + \theta_A + \theta_B + \theta_C = 1$$.
Otherwise,
it is set up with the exact same equations.
```python
#ODE eqns
def dcdt(thetas, t):
    tv=thetas[0]; tA=thetas[1]; tB=thetas[2]; tC=thetas[3] #vacant, A, B, and C
    dtAdt=k1f*tv*pA - k1r*tA - k3f*tA*tB + k3r*tC*tv
    dtBdt=k2f*tv*pB - k2r*tB - k3f*tA*tB + k3r*tC*tv
    dtCdt=k3f*tA*tB - k3r*tC*tv - k4f*tC
    dtvdt=-k1f*tv*pA + k1r*tA - k2f*tv*pB + k2r*tB + k3f*tA*tB - k3r*tC*tv + k4f*tC
    return [dtvdt, dtAdt, dtBdt, dtCdt]
#Steady-state non-linear eqns
def stst(thetas):
    tv=thetas[0]; tA=thetas[1]; tB=thetas[2]; tC=thetas[3] #vacant, A, B, and C
    dtAdt=k1f*tv*pA - k1r*tA - k3f*tA*tB + k3r*tC*tv
    dtBdt=k2f*tv*pB - k2r*tB - k3f*tA*tB + k3r*tC*tv
    dtCdt=k3f*tA*tB - k3r*tC*tv - k4f*tC
    sitebal=tv+tA+tB+tC-1
    return [dtAdt, dtBdt, dtCdt, sitebal]
```
The module 
<a href="https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.fsolve.html" target="_blank">fsolve</a> 
in scipy
solves the nonlinear equations numerically 
from an initial guess (not initial condition as in ODEs).
Clearly,
solving for the steady-state solution
this way is computationall faster.

<div style="display: flex; justify-content: space-between; width: 100%;">
<div style="width: 48%;">
{% highlight python %}
#----Extract last value of ODEs (slower)---
from scipy.integrate import odeint
#Set temperature range
temps=np.arange(300, 801, 10)
#define Pa and Pb
pA=1
pB=1
#Looping through temperatures
rateTs=[]
thetaTs=[]
for T in temps:
    #recompute rate constants at each temp
    k1f=kads(T)
    k1r=kdes(T, 50)
    k2f=kads(T)
    k2r=kdes(T, 40)
    k3f=krxn(T, 80)
    k3r=krxn(T, 60)
    k4f=kdes(T, 40)
    #solve ODE
    thetas=odeint(dcdt, theta0, 10**logtime)
    #Get the final thetas
    thetaTs.append(thetas[-1])
    #Get the rate of C desorption
    rateTs.append(k4f*thetas[-1, 3])
#Plot
plt.plot(temps,thetaTs)
plt.figure()
plt.plot(temps, rateTs)
plt.show()
{% endhighlight %}
</div>
<div style="width: 48%;">
{% highlight python %}
#----Solve steady-state eqns (faster)---
from scipy.optimize import fsolve
#Set temperature range
temps=np.arange(300, 801, 10)
#define Pa and Pb
pA=1
pB=1
#Looping through temperatures
rateTs=[]
thetaTs=[]
for T in temps:
    #recompute rate constants at each temp
    k1f=kads(T)
    k1r=kdes(T, 50)
    k2f=kads(T)
    k2r=kdes(T, 40)
    k3f=krxn(T, 80)
    k3r=krxn(T, 60)
    k4f=kdes(T, 40)
    #solve non-linear steady state eqns
    thetas=fsolve(eqns, theta0)
    #Save thetas
    thetaTs.append(thetas)
    #Save rate of C desorption
    rateTs.append(k4f*thetas[3])
#Plot
plt.plot(temps,thetaTs)
plt.figure()
plt.plot(temps, rateTs)
plt.show()
{% endhighlight %}
</div>
</div>

<div style="display: flex; justify-content: center; width: 100%;">
    <div style="width: 30%; text-align: left;">
        <span>Pressure of A (bar):</span>
        <input type="range" id="PAslider2" min="0.1" max="2" step="0.1" value="1" style="width: 80%;">
        <span id="PAspan2">1</span>
    </div>
    <div style="width: 30%; text-align: left;">
        <span>Pressure of B (bar):</span>
        <input type="range" id="PBslider2" min="0.1" max="2" step="0.1" value="1" style="width: 80%;">
        <span id="PBspan2">1</span>
    </div>
</div>
<div id="plotDiv2" style="width: 100%; height: 800px; margin: 0px auto;"></div>

A few key observations:\\
1) The steady-state coverages of $A$ and $B$
start going down above some temperature
because the rate of adsoprtion goes down
and the rate of desorption goes up with temperature (see Hertz-Knudsen equations).\\
2) The TOF increases initally with temperature
because the rate of reaction forming $C$ goes up.
The TOF decreases above some temperature
when the coverages of $C$ goes down.\\
3) The "optimal" temperature where
the TOF is maximized changes
more sensitively with $A$ than with $B$.

<h2>
Extension to reactor types
</h2>
A key assumption for the current toy model
(aside from using arbitrary $A$, $B$, and $C$ species)
is that the partial pressures 
are assumed to be time-invariant.
In reality,
surface reactions 
will consume $A$ and $B$ from
and produce $C$ to
the gas phase,
which in-turn affects surface kinetics
and the steady-state solutions.

Establishing time-dependent partial pressures
intimately connects
with establishing the type of reactors.
In a batch reactor,
the partial pressures 
and surface coverages
vary with time
until everything reaches equilibrium.
In a plug flow reactor,
the pseudo-steady-state coverages and pressure
vary across the length of the reactor
and also depends on the gas flow rate.

Solving for these reactor types
may become a subject of a future post :).

<!-- Plotly script -->
<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
<script>
  // Initialize plot
  function Plotly1(T, PA, PB) 
  {
    const url = `https://bolton2710.pythonanywhere.com/che2?T=${T}&pA=${PA}&pB=${PB}`;
    fetch(url)
      .then(response => response.json())
      .then(data => {
        const time = data.t.map(Number);
        const thetas = data.thetas;
        const plotData = [{x: time, y: thetas[0], mode: 'lines', hoverinfo: 'x+y', name: "$\\theta_v$", line: { color: 'black' }, xaxis: 'x1', yaxis: 'y1', showlegend: true},
                          {x: time, y: thetas[1], mode: 'lines', hoverinfo: 'x+y', name: "$\\theta_A$", line: { color: 'red' }, xaxis: 'x1', yaxis: 'y1', showlegend: true},
                          {x: time, y: thetas[2], mode: 'lines', hoverinfo: 'x+y', name: "$\\theta_B$", line: { color: 'blue' }, xaxis: 'x1', yaxis: 'y1', showlegend: true},
                          {x: time, y: thetas[3], mode: 'lines', hoverinfo: 'x+y', name: "$\\theta_C$", line: { color: 'green' }, xaxis: 'x1', yaxis: 'y2', showlegend: true}]
        const layout = {
            xaxis: { title: "Log time (s)", type: "log" , showgrid: false, tickmode: 'linear', ticks: 'outside', tickangle:0, dtick: 3, tickfont:{size:16}, titlefont:{size:17}},
            yaxis: { title: "$\\theta_v \\;/\\; \\theta_A \\;/\\; \\theta_B$",  range: [0, Math.max(...thetas[1].map(Number), ...thetas[2].map(Number)) * 1.1], 
                    showline: true, showgrid: false, ticks: 'outside', tickfont:{size:16}, titlefont:{size:17}, automargin: true },
            yaxis2: { title: "$\\theta_C$", range: [0, Math.max(...thetas[3].map(Number)) * 1.1], 
                    showline: true, showgrid: false, ticks: 'outside', tickfont:{size:16}, titlefont:{size:17}, overlaying: 'y', side: 'right', automargin: true },
            legend: {x: 0.5, y: 1.05, xanchor: 'center', yanchor: 'bottom', orientation: 'h', font: {size:16}}};
        Plotly.newPlot(plotDiv1, plotData, layout);
      })
      .catch(err => console.error("Error fetching data:", err));
  }
  function Plotly2(PA, PB) 
  {
    const url = `https://bolton2710.pythonanywhere.com/che2b?pA=${PA}&pB=${PB}`;
    fetch(url)
      .then(response => response.json())
      .then(data => {
        const temp = data.temp.map(Number);
        const thetas = data.thetas;
        const rates = data.rates.map(Number);
        const plotData = [{x: temp, y: thetas[0], mode: 'lines', hoverinfo: 'x+y', name: "$\\theta_v$", line: { color: 'black' }, xaxis: 'x1', yaxis: 'y1', showlegend: true},
                          {x: temp, y: thetas[1], mode: 'lines', hoverinfo: 'x+y', name: "$\\theta_A$", line: { color: 'red' }, xaxis: 'x1', yaxis: 'y1', showlegend: true},
                          {x: temp, y: thetas[2], mode: 'lines', hoverinfo: 'x+y', name: "$\\theta_B$", line: { color: 'blue' }, xaxis: 'x1', yaxis: 'y1', showlegend: true},
                          {x: temp, y: thetas[3], mode: 'lines', hoverinfo: 'x+y', name: "$\\theta_C$", line: { color: 'green' }, xaxis: 'x1', yaxis: 'y2', showlegend: true},
                          {x: temp, y: rates, mode: 'lines', hoverinfo: 'x+y', line: { color: 'black' }, xaxis: 'x2', yaxis: 'y3', showlegend: false}]
        const layout = {
            grid: { rows: 2, columns: 1, pattern: 'independent' },
            xaxis: { title: "Temperature (K)", showgrid: false, tickmode: 'linear', ticks: 'outside', tickangle:0, dtick: 50, tickfont:{size:16}, titlefont:{size:17}},
            yaxis: { title: "$\\text{Steady state}\\;\\;\\theta_v/\\theta_A/\\theta_B$",  range: [0, Math.max(...thetas[1].map(Number), ...thetas[2].map(Number)) * 1.1], 
            showline: true, showgrid: false, ticks: 'outside', tickfont:{size:16}, titlefont:{size:17}, automargin: true },
            yaxis2: { title: "$\\text{Steady state}\\;\\;\\theta_C$", range: [0, Math.max(...thetas[3].map(Number)) * 1.1], 
            showline: true, showgrid: false, ticks: 'outside', tickfont:{size:16}, titlefont:{size:17}, overlaying: 'y', side: 'right', automargin: true },
            xaxis2: { title: "Temperature (K)", domain: [0, 1], anchor: 'y3', showgrid: false, tickmode: 'linear', ticks: 'outside', tickangle: 0, dtick: 50, tickfont: {size:16}, titlefont:{size:17}},
            yaxis3: { title: "TOF (s<sup>-1</sup>)", range: [0, Math.max(...rates) * 1.1], showline: true, showgrid: false, ticks: 'outside', tickfont: {size:16}, titlefont:{size:17}, domain: [0, 0.4], automargin: true },
            legend: {x: 0.5, y: 1.05, xanchor: 'center', yanchor: 'bottom', orientation: 'h', font: {size:16}}};
        Plotly.newPlot(plotDiv2, plotData, layout);
      })
      .catch(err => console.error("Error fetching data:", err));
  }
  // Initial plot
  let T = parseFloat(Tslider.value);
  let PA = parseFloat(PAslider.value);
  let PB = parseFloat(PBslider.value);
  let PA2 = parseFloat(PAslider2.value);
  let PB2 = parseFloat(PBslider2.value);
  Plotly1(T, PA, PB);
  Plotly2(PA2, PB2) 
  // Update plot on slider change
  Tslider.addEventListener('input', () => {
    T = parseFloat(Tslider.value)
    Tspan.textContent = T;
    Plotly1(T, PA, PB);
  });
  PAslider.addEventListener('input', () => {
    PA = parseFloat(PAslider.value)
    PAspan.textContent = PA;
    Plotly1(T, PA, PB);
  });
  PBslider.addEventListener('input', () => {
    PB = parseFloat(PBslider.value)
    PBspan.textContent = PB;
    Plotly1(T, PA, PB);
  });
  PAslider2.addEventListener('input', () => {
    PA2 = parseFloat(PAslider2.value)
    PAspan2.textContent = PA2;
    Plotly2(PA2, PB2);
  });
  PBslider2.addEventListener('input', () => {
    PB2 = parseFloat(PBslider2.value)
    PBspan2.textContent = PB2;
    Plotly2(PA2, PB2);
  });
</script>