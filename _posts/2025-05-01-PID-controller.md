---
layout: post
title: "1. PID controller basics"
author: "Bolton Tran"
categories: tutorials
tags: [control]
image: "/assets/img/pid1.png"
---
<h2>
PID controller
</h2>

<div class="image-container">
  <img src="/assets/img/pid_diagram.png" alt="PID-diagram" class="centered-image" style="width: 100%;">
</div>

Consider a second-order plant process $G_p$. 
While majority of unit processes are first-order (e.g, heater, mixer, liquid tank, etc.),
they are often placed in series with each other,
or with measurement units $G_m$ that themselves follow first-order dynamics (though often with very small time constant).
Therefore, here I consider $G_p$ as a lump of two first-order units,
thereby exhibit second-order dynamics:

<div>
\begin{align} 
\tau^2\frac{d^2C}{dt^2} + 2\tau\zeta\frac{dC}{dt} + C(t) = M(t) + U(t)
\label{eqn:ode}
\end{align}
</div>

Here, $C(t)$ is the process variable, i.e., the variable we wish to control.
$M(t)$ is the manipulated variable, output of the PID controller.
$U(t)$ is the system load.
The parameters $\tau$ and $\zeta$ control the dynamics of process $G_p$,
of which real values are informed by the physical conditions of the system (e.g., heat capacitity, tank volume, flow rates, etc.).

Now onto the controller $G_c$, it takes $E(t)$ as input and yields $M(t)$ as output.
$E(t)$ is the error of the process variable $C(t)$ to the setpoint $R(t)$:
<div>
\begin{align}
E(t) = R(t) - C(t)
\label{eqn:E}
\end{align}
</div>
The controller transform $E(t)$ to $M(t)$, which then alter $C(t)$
to eventually minimize the error $E(t)$ as quickly and safely as possible.
Different controller logics and parameters change how well this can be done.

The Proportional-Integral-Derivative (PID) controller is possibly the most widely used.
PID controller logic works as follows:
<div>
\begin{align}
M(t) = K_p E(t) + K_i \int{E(t)dt} + K_d \frac{dE}{dt}
\label{eqn:M}
\end{align}
</div>
The first term scales the error (P)roportionally with a factor $K_p$.
The second term scales the (I)ntegral or the cumulative error with a factor $K_i$.
The last term scales the (D)erivative or rate of change of the error with a factor $K_d$.

We can solve the above second-order dynamics (Eqn. \ref{eqn:ode}) by discretizing and propagating through some small timestep.
While in theory regular ODE solver can deal with the second-order,
it is particularly challenging because the PID logic requires knowledge of
the instantaneous derivative of the $E(t)$,
which makes things incredibly messy to setup (I tried).

We break the second-order ODE into two first-order ODEs.
We need first a variable change,
<div>
\begin{align}
c_1 &= C\\
c_2 &= \frac{dC}{dt}
\end{align}
</div>
then rearangement of Eqn \ref{eqn:ode}.
<div>
\begin{align}
\frac{dc_1}{dt} &= c_2 (t) \label{eqn:c1}\\
\frac{dc_2}{dt} &= \frac{1}{\tau^2}\left( -c_1(t) -2\tau\zeta c_2(t) + M(t) + U(t)\right) \label{eqn:c2}
\end{align}
</div>
The practical steps---commonly known as the Euler's numerical method---are:
1. Create a time array.
2. Create and customize arrays for time series of setpoint $R$ and load $U$ (e.g., step, pulse, wave etc.).
3. Initiate at $t=0$ for:\
    a. $c_1$ and $c_2$.\
    b. Error (Eqn \ref{eqn:E}), its derivative and integral, which gives $M$ (Eqn \ref{eqn:M}).\
    c. Initiate derivatives of $c_1$ and $c_2$ using Eqns \ref{eqn:c1} and \ref{eqn:c2}.
4. Iterating through each step of the predefined time array. At each step $t$:\
    a. Get setpoint $R$ and load $U$ at this time step from the predefined arrays.\
    b. Update the new $c_1$ and $c_2$ by finite difference to the previous step.\
    c. Update the new error components to give $M$.\
    d. Update the new derivatives of $c_1$ and $c_2$.

Below, we examine the PID controller in action in two scenario: 1) setpoint step-change; 2) load fluctuation.

<h2>
Setpoint step-change
</h2>
Here we solve the closed-loop system (controller and process)
for when there is a step change in the setpoint $R$.
We assume the load $U(t)$ is constant for now,
and does not include in the solver.
This is sometimes refered to as the "servo" problem.
The Python code for solving this is shown below:

```python
import numpy as np
##-----Process parameter-----
tau=20 #process time constant
zeta=1 #damping coefficient
##-----Controller parameter-----
Kp=2 #proportional gain
Ki=0.1 #integral gain
Kd=1 #derivative gain
##-----Time setup-----
tmax=200
dt=0.1
time=np.arange(0, tmax,dt)
##-----Initial condition-----
C0=0 #intial process variable
M0=C0 #same manipulated variable as offset
##Set point and load
tstart=50
R=np.heaviside(time-tstart, 1) + C0 #Step 1 unit up from C0
U=np.zeros(len(time)) #No load considered
##-----Set up numerical solver-----
#initialize arrays
c1=np.zeros(len(time)) #C
c2=np.zeros(len(time)) #dCdt
E=np.zeros(len(time)) #E
M=np.zeros(len(time)) #M
#Initial C
c1[0]=C0 #first order
c2[0]=0 #second order
#Intial E
E[0]=R[0]-C0 #error
Ei=E[0]*dt #integral error
dE=0 #derivative error
#Initial M
M[0]=Kp*E[0] + Ki*Ei + Kd*dE + M0 #PID logic with offset
#Initial derivatives
dc1=c2[0]
dc2=1/tau**2*(-2*tau*zeta*c2[0] -c1[0] + M[0] + U[0])
##-----Solve=propagrate through time-----
for t in range(1,len(time)):
    #Update for this time step
    c1[t]=c1[t-1]+dc1*dt #first order
    c2[t]=c2[t-1]+dc2*dt #second order
    E[t]=R[t]-c1[t] #error
    Ei+=E[t]*dt #cumulate integral error
    dE=(E[t]-E[t-1])/dt #derivative error by finite difference with previous step
    #Propagate derivatives for next step
    M[t]=Kp*E[t] + Ki*Ei + Kd*dE + M0 #PID logic with offset
    dc1=c2[t]
    dc2=1/tau**2*(-2*tau*zeta*c2[t] -c1[t] + M[t] + U[t]) 
##-----Plot-----
import matplotlib.pyplot as plt
plt.plot(time, R) #setpoint
plt.plot(time, c1) #process varibable
plt.plot(time, M) #manipulated variable
plt.plot(time, E) #error
plt.show()
```

<div style="display: flex; justify-content: space-between; width: 100%;">
    <div style="width: 33%; text-align: center; display: flex; flex-direction: row; align-items: center;">
        <span>\[K_p\]</span>
        <input type="range" id="kpslider" min="0" max="10" step="0.1" value="0" style="width: 60%;">
        <span id="kpspan">0</span>
    </div>
    <div style="width: 33%; text-align: center; display: flex; flex-direction: row; align-items: center;">
        <span>\[K_i\]</span>
        <input type="range" id="kislider" min="0" max="2" step="0.01" value="0" style="width: 60%;">
        <span id="kispan">0</span>
    </div>
    <div style="width: 33%; text-align: center; display: flex; flex-direction: row; align-items: center;">
        <span>\[K_d\]</span>
        <input type="range" id="kdslider" min="0" max="20" step="0.2" value="0" style="width: 60%;">
        <span id="kdspan">0</span>
    </div>
</div>
<div id="plotDiv" style="width: 100%; height: 500px; margin: 0px auto;"></div>

As you play with the above interactive plot, consider these aspects:
* *<u>Manually tune the PID parameters</u>*

There are numerical methods to optimize these parameters to meet various criteria (e.g., quarter-wave decay). Here let's just try a simple manual tuning exercise:
1. From an initial system with all $K$'s equal 0, move only $K_p$ up until the process variable ($C$)
oscillate with a small overshoot (i.e., going above the setpoint $R$).
2. Here we see the steady-state (i.e., the long-time plateau) of $C$ does not quite match $R$, which is known problem with using just the (P)roportional controller. Increase $K_i$ very slightly until we see steady-state $C$ matches well with $R$.
3. Now we perhaps want to reduce the time it takes to reach steady-state or reduce the overshoot. Increase $K_d$ to do just that.

* *<u>Observe controller instability</u>*

Controller's stability is an important consideration.
To visualize how instability manifests,
with a small $K_p$, keep increasing $K_i$
until the oscillation starts to progressively increase with time
and never reaches the $R$ target.\
Intuitively, this is because the (I)ntegral error component penalizes the cumulative deviation of $C$ to $R$.
Therefore, at some critical $K_i$ weight, more cumulative error leads to more corrective action that overshoots the target, leading to even more culmulative error, and so on.

* *<u>Observe spike in manipulated variable</u>*

While a controller's objective is to match process variable $C$ to setpoint $R$,
how the manipulated variable $M$ manifests is also important.
To demonstrate, at some small $K_p$ and $K_i$, increase $K_d$ to the max level.
We see while increasing the (D)erivative error helps to "smooth" out the process variable dynamics,
there is a conspicuous spike in the manipulated variable $M$.
This is because the (D) component penalizes sudden changes in the error,
thereby aggressively responses to a step-change in setpoint.
In physical systems,
manipulated variables $M$ usually transfer to gas pressure for pneumatic valves,
of which spikes could be a safety hazard or downright un-actuatable (that's a word?) by the equipment.

<h2>
Load fluctuation
</h2>
Here we consider a constant setpoint $R$ but with changing and fluctuating load $U$.
This is refered to as a "regulator" problem,
which is perhaps more common for chemical engineers.

To set this problem up,
we simply re-customize our setpoint $R$ and $U$ functions.
To simulate noise or fluctuation of the load change,
we can sample from a normal distribution with 
preset average $\Delta U$ ($\Delta$ because it is a change from previous value),
and standard deviation $\delta U$.
```python
##Set point and load
#Setpoint held constant at 1
R=np.zeros(len(time))+1
#Load fluctuation
tstart=500 #time when load changes
delU=10 #how much load change
stdU=2 #how much new load fluctuate
U=np.zeros(len(time))
for i in range(len(U)):
    #Randomize load with a normal distribution
    if time[i] <= tstart: U[i]=np.random.normal(loc=0, scale=1)
    else: U[i]=np.random.normal(loc=delU, scale=stdU)
```

<div style="display: flex; justify-content: space-between; width: 100%;">
    <div style="width: 45%; display: flex; flex-direction: row; align-items: center;">
        <span>\[\Delta U\]</span>
        <input type="range" id="riseslider" min="0" max="10" step="0.5" value="0" style="width: 60%;">
        <span id="risespan">0</span>
    </div>
    <div style="width: 45%; display: flex; flex-direction: row; align-items: center;">
        <span>\[\delta U\]</span>
        <input type="range" id="flucslider" min="0" max="5" step="0.5" value="1" style="width: 60%;">
        <span id="flucspan">1</span>
    </div>
</div>
<div style="display: flex; justify-content: space-between; width: 100%;">
    <div style="width: 33%; text-align: center; display: flex; flex-direction: row; align-items: center;">
        <span>\[K_p\]</span>
        <input type="range" id="kpslider2" min="0" max="10" step="0.1" value="0" style="width: 60%;">
        <span id="kpspan2">0</span>
    </div>
    <div style="width: 33%; text-align: center; display: flex; flex-direction: row; align-items: center;">
        <span>\[K_i\]</span>
        <input type="range" id="kislider2" min="0" max="2" step="0.01" value="0" style="width: 60%;">
        <span id="kispan2">0</span>
    </div>
    <div style="width: 33%; text-align: center; display: flex; flex-direction: row; align-items: center;">
        <span>\[K_d\]</span>
        <input type="range" id="kdslider2" min="0" max="20" step="0.2" value="0" style="width: 60%;">
        <span id="kdspan2">0</span>
    </div>
</div>
<div id="plotDiv2" style="width: 100%; height: 500px; margin: 0px auto;"></div>

From above interactive plot, we can see how the fluctuation in load $U$ manifest into the process variable $C$. The controller effectively dampens the fluctuation such that the fluctuation of $C$ is stretched out over a longer time period in comparison to that of $U$.
As of now, I am not sure whether the PID $K$'s parameters affect the damping of $C$'s fluctuations.

<script src="https://cdn.plot.ly/plotly-2.32.0.min.js"></script>
<script>
  const plotDiv = document.getElementById('plotDiv');
  const kpslider = document.getElementById('kpslider');
  const kislider = document.getElementById('kislider');
  const kdslider = document.getElementById('kdslider');
  const plotDiv2 = document.getElementById('plotDiv2');
  const kpslider2 = document.getElementById('kpslider2');
  const kislider2 = document.getElementById('kislider2');
  const kdslider2 = document.getElementById('kdslider2');
  //Plot R step
  function plot1(Kp, Ki, Kd) 
  {
    const url = `https://bolton2710.pythonanywhere.com/pid1?Kp=${Kp}&Ki=${Ki}&Kd=${Kd}`;
    fetch(url)
      .then(response => response.json())
      .then(data => {
        const t = data.t.map(Number);
        const r = data.r.map(Number);
        const pv = data.pv.map(Number);
        const mv = data.mv.map(Number);
        const e = data.e.map(Number);
        const plotData = [
          {x: t, y: pv, type: 'scatter', line:{color:'blue'}, name:'$C$', hoverinfo: 'x+y', xaxis: 'x1', yaxis: 'y1'},
          {x: t, y: r , type: 'scatter', line:{color:'black', dash:'dash'}, name:'$R$', hoverinfo: 'x+y', xaxis: 'x1', yaxis: 'y1'},
          {x: t, y: e, type: 'scatter', line:{color:'red', dash:'dot'}, name:'$E$', hoverinfo: 'x+y', xaxis: 'x1', yaxis: 'y1'},
          {x: t, y: mv, type: 'scatter', line:{color:'green'}, name:'$M$', hoverinfo: 'x+y', xaxis: 'x2', yaxis: 'y2'}
          ];
        const layout = {
          grid: { rows: 2, columns: 1, pattern: 'independent' },
          xaxis: {fixedrange: true, showgrid: false, range: [0,200], tickfont:{size:16}, titlefont:{size:17}, ticks: 'outside', showline: true},
          xaxis2:{title: 'Time', fixedrange: true, showgrid: false, range: [0,200], tickfont:{size:16}, titlefont:{size:17}, ticks: 'outside', showline: true},
          yaxis: {title: 'Process variable', fixedrange: false, zeroline: false, showgrid: false, tickfont:{size:16}, titlefont:{size:17}, ticks: 'outside'},
          yaxis2: {title: 'Manipulated variable', fixedrange: false, zeroline: false, showgrid: false, tickfont:{size:16}, titlefont:{size:17}, ticks: 'outside'},
          legend: {x: 0.5, y: 1.05, xanchor: 'center', yanchor: 'bottom', orientation: 'h', font: {size:16}}
        };
        Plotly.newPlot(plotDiv, plotData, layout);
      })
      .catch(err => console.error("Error fetching data:", err));
  }
  //Plot U fluc
  function plot2(Kp, Ki, Kd, rise, fluc) 
  {
    const url = `https://bolton2710.pythonanywhere.com/pid2?Kp=${Kp}&Ki=${Ki}&Kd=${Kd}&rise=${rise}&fluc=${fluc}`;
    fetch(url)
      .then(response => response.json())
      .then(data => {
        const t = data.t.map(Number);
        const r = data.r.map(Number);
        const u = data.u.map(Number);
        const pv = data.pv.map(Number);
        const mv = data.mv.map(Number);
        const e = data.e.map(Number);
        const plotData = [
          {x: t, y: u , type: 'scatter', line:{color:'purple'}, name:'$U$', hoverinfo: 'x+y', xaxis: 'x1', yaxis: 'y1'},
          {x: t, y: pv, type: 'scatter', line:{color:'blue'}, name:'$C$', hoverinfo: 'x+y', xaxis: 'x2', yaxis: 'y2'},
          {x: t, y: r , type: 'scatter', line:{color:'black', dash:'dash'}, name:'$R$', hoverinfo: 'x+y', xaxis: 'x2', yaxis: 'y2'},
          {x: t, y: e, type: 'scatter', line:{color:'red', dash:'dot'}, name:'$E$', hoverinfo: 'x+y', xaxis: 'x2', yaxis: 'y2'},
          {x: t, y: mv , type: 'scatter', line:{color:'green'}, name:'$M$', hoverinfo: 'x+y', xaxis: 'x2', yaxis: 'y2'},
          ];
        const layout = {
          grid: { rows: 2, columns: 1, pattern: 'independent' },
          xaxis2: {title: 'Time', fixedrange: true, showgrid: false, range: [0,1000], tickfont:{size:16}, titlefont:{size:17}, ticks: 'outside', showline: true},
          xaxis:{fixedrange: true, showgrid: false, range: [0,1000], tickfont:{size:16}, titlefont:{size:17}, ticks: 'outside', showline: true},
          yaxis2: {title: 'Process variable', fixedrange: false, zeroline: false, showgrid: false, tickfont:{size:16}, titlefont:{size:17}, ticks: 'outside'},
          yaxis: {title: 'Load', fixedrange: false, zeroline: false, showgrid: false, tickfont:{size:16}, titlefont:{size:17}, ticks: 'outside'},
          legend: {x: 0.5, y: 1.05, xanchor: 'center', yanchor: 'bottom', orientation: 'h', font: {size:16}},
          annotations: [{xref: 'x1', yref: 'y1', x: 200, y: 0., ax:-20, ay: +50, text: 'Load change', font:{size: 16, color: 'black'},
          showarrow: true, arrowcolor: 'black', arrowhead:3}]
        };
        Plotly.newPlot(plotDiv2, plotData, layout);
      })
      .catch(err => console.error("Error fetching data:", err));
  }
  // Initial plot
  let Kp = 0;
  let Ki = 0;
  let Kd = 0;
  plot1(Kp, Ki, Kd);
  // Update plot1
  kpslider.addEventListener('input', () => {
    Kp = parseFloat(kpslider.value);
    kpspan.textContent = Kp;
    plot1(Kp, Ki, Kd);
  });
  kislider.addEventListener('input', () => {
    Ki = parseFloat(kislider.value);
    kispan.textContent = Ki;
    plot1(Kp, Ki, Kd);
  });
  kdslider.addEventListener('input', () => {
    Kd = parseFloat(kdslider.value);
    kdspan.textContent = Kd;
    plot1(Kp, Ki, Kd);
  });
  // Update plot2
  let Kp2 = 0;
  let Ki2 = 0;
  let Kd2 = 0;
  let rise = 0;
  let fluc = 1;
  plot2(Kp, Ki, Kd, rise, fluc);
  kpslider2.addEventListener('input', () => {
    Kp2 = parseFloat(kpslider2.value);
    kpspan2.textContent = Kp2;
    plot2(Kp2, Ki2, Kd2, rise, fluc);
  });
  kislider2.addEventListener('input', () => {
    Ki2 = parseFloat(kislider2.value);
    kispan2.textContent = Ki2;
    plot2(Kp2, Ki2, Kd2, rise, fluc);
  });
  kdslider2.addEventListener('input', () => {
    Kd2 = parseFloat(kdslider2.value);
    kdspan2.textContent = Kd2;
    plot2(Kp2, Ki2, Kd2, rise, fluc);
  });
  riseslider.addEventListener('input', () => {
    rise = parseFloat(riseslider.value);
    risespan.textContent = rise;
    plot2(Kp2, Ki2, Kd2, rise, fluc);
  });
  flucslider.addEventListener('input', () => {
    fluc = parseFloat(flucslider.value);
    flucspan.textContent = fluc;
    plot2(Kp2, Ki2, Kd2, rise, fluc);
  });
</script>