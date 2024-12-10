---
layout: post
title: "Series Reaction in Batch Reactor"
author: "Bolton Tran"
categories: tutorials
tags: [che]
---
Consider a gas-phase reaction in series:
$$A \overset{k_1}{\rightarrow} B \overset{k_2}{\rightarrow} C$$
taking place in an isothermal batch reactor.
We want to solve for the concentrations of $$A$$, $$B$$, and $$C$$
versus time.

<input type="range" id="Tslider" min="400" max="500" step="1" value="400">
<label for="Tslider">$$T$$ (K):</label>
<span id="Tspan">400</span>

<input type="range" id="EA1slider" min="140" max="150" step="0.1" value="140">
<label for="EA1slider">$$E^\dagger_1$$ (kJ/mol):</label>
<span id="EA1span">140</span>

<input type="range" id="EA2slider" min="150" max="160" step="0.1" value="150">
<label for="EA2slider">$$E^\dagger_2$$ (kJ/mol):</label>
<span id="EA2span">150</span>

<div id="plot"></div>

The interactive plot below
was created by
hosting a 
python-Flask (a Web Server Gateway Interface application) script
on https://www.pythonanywhere.com/.
The script contains python functions
that perform mathematical operations.
The slider below send values of
($$E^\dagger_1$$, $$E^\dagger_2$$, $$T$$) to the PythonAnywhere app,
which performs a sine operation
and outputs json arrays.
The json arrays
are then parsed back to Plotly
written in JavaScript,
which created the plot.


<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
<script>
  const plotDiv = document.getElementById('plot');
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
        const plotData = [
          { x: data.t, y: data.ca, type: 'scatter', name:'$C_A$', hoverinfo: 'x+y'},
          { x: data.t, y: data.cb, type: 'scatter', name:'$C_B$', hoverinfo: 'x+y'},
          { x: data.t, y: data.cc, type: 'scatter', name:'$C_C$', hoverinfo: 'x+y'}];
        const layout = {
          xaxis: {title: 'Time (hr)', showgrid: false, tickmode: 'linear', ticks: 'outside', fixedrange: true, range: [0,50], tickangle:0, dtick: 5, tickfont:{size:16}, titlefont:{size:17}}, 
          yaxis: {title: 'Concentration (mol/L)', showgrid: false, tickmode: 'linear', ticks: 'outside', fixedrange: true, range: [0,2.1],tickfont:{size:16}, titlefont:{size:17}},
          legend: {x: 0.5, y: 1.05, xanchor: 'center', yanchor: 'bottom', orientation: 'h', font: {size:16}},
          margin: {l: 50, r: 50, b: 50, t: 0}};
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
