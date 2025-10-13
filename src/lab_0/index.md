---
title: "Lab 0: Getting Started"
toc: true
---

# Hello 
This is Erin's lab zero. Testing markdown headers and text content, HTML elements table, list, and image, and one JS input element

<h2>HTML</h2> 
<p> This is a table of things I like </p>
<table border="1">
  <tr>
    <th>Thing</th>
    <th>Preference</th>
  </tr>
  <tr>
    <td>Pretzels</td>
    <td>Salty</td>
  </tr>
  <tr>
    <td>Mini Golf</td>
    <td>With friends</td>
  </tr>
    <tr>
    <td>Gender Expression</td>
    <td>Fluid</td>
  </tr>
      <tr>
    <td>Reading</td>
    <td>In the morning</td>
  </tr>
  
</table>

<p>This is a list of my favorite animals:</p>
<ul>
  <li>Chameleon</li>
  <li>Giraffe</li>
  <li>Beaver</li>
</ul>
<p> a drawing I did :) </p>

<img src="../lab_0/assets/Bannana Slug Drawing.png" alt="Marker Drawing" width="400">
<p> the rescue dog I am adopting xoxo </p>
<img src="../lab_0/assets/gwen.jpeg" alt="dog" width="400">

<h2>JS Input Element</h2>
<div id="colorContainer"></div>

<script>
  const container = document.getElementById("colorContainer");

  // Create input
  const input = document.createElement("input");
  input.type = "text";
  input.placeholder = "Type a color";

  // Create text to change
  const output = document.createElement("p");
  output.textContent = "Watch me change color!";
  output.style.fontSize = "20px";
  output.style.fontWeight = "bold";

  // Add event listener
  input.addEventListener("input", () => {
    output.style.color = input.value;
  });

  // Add to container
  container.appendChild(input);
  container.appendChild(output);
</script>