---
title: "Lab 1: Passing Pollinators"
toc: true
---
<!-- 
This page is where you can iterate. Follow the lab instructions in the [readme.md](./README.md).-->

<h2>Key Questions to Answer</h2> 

  1. What is the body mass and wing span distribution of each pollinator species observed?
  2. What is the ideal weather (conditions, temperature, etc) for pollinating?
  3. Which flower has the most nectar production?

Loading in the data, 

```js
const pollinatorData = await FileAttachment("data/pollinator_activity_data.csv").csv({ typed: true })
```
```js
Inputs.table(pollinatorData)
```


<!--Attempt to plot with PollinatorData based on class note structure, Body Mass on X axis, Wingspan on Y axis -->

<h3>Pollinator Body Mass vs. Wingspan</h3>

```js
Plot.plot({
    height: 600, 
    y: {
        grid:true
    },  x: { label: "Average Body Mass (g)" },
  y: { label: "Average Wing Span (mm)" },
  color: { legend: true },
    marks: [
        Plot.frame(),
        Plot.dot(pollinatorData, {
            x: "avg_body_mass_g",
            y: "avg_wing_span_mm",
            fill: "pollinator_species",
            title: "species"
        })
    ]
})
```
<h3>Ideal weather conditions for pollination</h3>

```js
Plot.plot({
  marks: [
    Plot.barY(pollinatorData, 
      Plot.groupX({y: "sum"},
      {
         x: "weather_condition",
         y: "nectar_production"}
         )
    )
  ],
  y: {label: "Total Nectar Production"},
  x: {label: "Weather Condition"},
  color: { legend: false }
})
```
Total nectar production by weather conditions shows a slight lead for cloudy weather. However, using the average instead of sum of nectar_production values helps account for any unequal sampling, like observing more often on sunny days.  
```js
Plot.plot({
  marks: [
    Plot.barY(pollinatorData, 
      Plot.groupX({y: "mean"},
      {
         x: "weather_condition",
         y: "nectar_production"}
         )
    )
  ],
  y: {label: "Average Nectar Production"},
  x: {label: "Weather Condition"},
  color: { legend: false }
})
```
Average nectar production shows a smaller difference across weather categories. 


<h3>Flower with the most nectar production</h3>

```js
Plot.plot({
  marks: [
    Plot.barY(pollinatorData, 
      Plot.groupX({y: "sum"},
      {
         x: "flower_species",
         y: "nectar_production", 
         fill: "flower_species",}
         )
    )
  ],
  y: {label: "Total Nectar Production"},
  x: {label: ""},
  color: { legend: false }
})
```
Sunflowers show the highest total nectar production across the dataset. 
