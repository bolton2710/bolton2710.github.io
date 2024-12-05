---
layout: post
title: "Test Plotly with python backend"
author: "Bolton Tran"
categories: tutorials
tags: [test]
---
The interactive plot below
was created by
hosting a 
python-Flask (a Web Server Gateway Interface application) script
on https://www.pythonanywhere.com/.
The script contains python functions
that perform mathematical operations.
The slider below send values of
(a) to the PythonAnywhere app,
which performs a sine operation
and outputs json arrays.
The json arrays
are then parsed back to Plotly
written in JavaScript,
which created the plot.
<div id="plot"></div>

<label for="slider">Adjust \( a \):</label>
<input type="range" id="slider" min="0.1" max="10" step="0.1" value="1">
<span id="value">1</span>

<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
<script>
  const plotDiv = document.getElementById('plot');
  const slider = document.getElementById('slider');
  const valueSpan = document.getElementById('value');
  // Initialize plot
  function fetchDataAndUpdatePlot(a) {
    fetch(`https://bolton2710.pythonanywhere.com/compute?a=${a}`)
      .then(response => response.json())
      .then(data => {
        const plotData = [{ x: data.x, y: data.y, type: 'scatter', name: 'y = sin(ax)' }];
        const layout = { title: 'Interactive Sine Plot', xaxis: { title: 'x' }, yaxis: { title: 'y' } };
        Plotly.newPlot(plotDiv, plotData, layout);
      })
      .catch(err => console.error("Error fetching data:", err));
  }
  // Initial plot
  fetchDataAndUpdatePlot(1);
  // Update plot on slider change
  slider.addEventListener('input', () => {
    const a = parseFloat(slider.value);
    valueSpan.textContent = a;
    fetchDataAndUpdatePlot(a);
  });
</script>
