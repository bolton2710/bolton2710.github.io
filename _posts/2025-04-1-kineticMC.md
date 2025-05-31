---
layout: post
title: "3. Kinetic Monte Carlo"
author: "Bolton Tran"
categories: tutorials
tags: [rxn]
image: "/assets/img/kmc.png"
---

More to come soon. Play animations below if you're curious :)

<!-- Viewing pre-made kMC animations -->
<select id="kMCList">
    <option value="">Loading stored kMC...</option>
</select>
<button id="viewkMC">View kMC</button>

<iframe id="kMCframe" width="110%" height="850" 
style="border: none; overflow: hidden; display: block"></iframe>

<!-- Let user tests kMC animations -->
<!-- <input type="text" id="nameInput" placeholder="Enter job name">
<button id="generateButton">Generate Animation</button> -->

<!-- JS script -->
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script>

    let myURL = `https://bolton2710.pythonanywhere.com`;

    function loadStoredAnimations() 
    {
        $.ajax({
            url: `${myURL}/list_kMC`,
            type: "GET",
            success: function(data) 
            {
                let select = $("#kMCList");
                select.empty(); // Clear existing options
                if (data.length === 0) {
                    select.append(`<option value="">No stored animations found</option>`);
                } else {
                    select.append(`<option value="">Select an animation</option>`);
                    data.sort();
                    data.forEach(animationId => {
                        select.append(`<option value="${animationId}">${animationId}</option>`);
                    });
                }
            },
            error: function()
            {
                alert("Failed to load stored kMCs.");
            }
        });
    } 

    //Load animation once page starts
    $(document).ready(function() 
    {
        loadStoredAnimations();
    });
    
    // Function to view selected animation
    $("#viewkMC").click(function() {
        let selectedId = $("#kMCList").val();
        if (selectedId) {
            $("#kMCframe").attr(`src`, `${myURL}/kMC_${selectedId}`);
        } else {
            alert("Please select an animation to view.");
        }
    });

    //Function to generate animation with selected frequency
    $("#generateButton").click(function() 
    {
        let frequency = $("#frequencyInput").val();
        let jobname = $("#nameInput").val();
        if (!jobname) {
            alert("Please enter a job name.");
            return;
        }
        if (!frequency) 
        {
            alert("Please enter a frequency value.");
            return;
        }
        $.ajax({
            url: `${myURL}/generate_animation?name=${jobname}&freq=${frequency}`,
            type: "GET",
            success: function(response) {
                alert("Animation generated successfully!");
                loadStoredAnimations(); // Reload stored animations
            },
            error: function() {
                alert("Error generating animation.");
            }
        });
    });
</script>