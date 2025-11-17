---
title: "Lab 2: Subway Staffing"
toc: true
---
<h2> NYC Transit Authority: 2025 Ridership Trends & 2026 Staffing Intel </h2>
<style>
h3 { background-color: #f0e6f6;
  color: rebeccapurple;
  padding: 15px 20px;
  border-radius: 5px;
  border-left: 5px solid rebeccapurple;
  }
</style>

<!-- Import Data -->
```js
const incidents = FileAttachment("./data/incidents.csv").csv({ typed: true })
const local_events = FileAttachment("./data/local_events.csv").csv({ typed: true })
const upcoming_events = FileAttachment("./data/upcoming_events.csv").csv({ typed: true })
const ridership = FileAttachment("./data/ridership.csv").csv({ typed: true })
```

<!-- Include current staffing counts from the prompt -->

```js
const currentStaffing = {
  "Times Sq-42 St": 19,
  "Grand Central-42 St": 18,
  "34 St-Penn Station": 15,
  "14 St-Union Sq": 4,
  "Fulton St": 17,
  "42 St-Port Authority": 14,
  "Herald Sq-34 St": 15,
  "Canal St": 4,
  "59 St-Columbus Circle": 6,
  "125 St": 7,
  "96 St": 19,
  "86 St": 19,
  "72 St": 10,
  "66 St-Lincoln Center": 15,
  "50 St": 20,
  "28 St": 13,
  "23 St": 8,
  "Christopher St": 15,
  "Houston St": 18,
  "Spring St": 12,
  "Chambers St": 18,
  "Wall St": 9,
  "Bowling Green": 6,
  "West 4 St-Wash Sq": 4,
  "Astor Pl": 7
}
```



  <h3>Part I: Overall Ridership: Impact of Fare Increase & Local Events by Station</h3>

  The chart below shows total daily ridership (entrances + exists) across all NYC subway stations in the dataset. This runs from June through mid-August 2025. The orange dashed line marks July 15th, when fares increased from $2.75 to $3.00. 

  Use the dropdwon to examine individual stations and their corresponding events, which are noted on the ridership line as orange dots. 
  
  **The aggregate view 'All Stations' reveals a clear ridership decline following the fare increase.** 
  
  Note that event level impacts are more subtle, so event dots are removed from the 'All Stations' view. Toggle to specific stations to examine if event dots correspond with peaks in ridership. For example, viewing 86th street ridership peaks coinciding with a major June fireworks display.

<!-- Running an array to get unique values of station names for a dropdown -->
```js
const allStations = ["All Stations", ...Array.from(new Set(ridership.map(d => d.station)))]
const selectedStation = view(Inputs.select(allStations, {label: "Select Station"}))
```
<!-- first simple plot
```js
Plot.plot({
  height: 400, 
  width,
  marks: [
    Plot.line(ridership,{
      x: "date",
      y: "entrances",
      z: "station",
      stroke: "station",
    })

  ]
})
```
-->

```js
const fareUpdate = new Date("2025-07-15")
```
```js
const filteredRidership = selectedStation === "All Stations" 
  ? ridership 
  : ridership.filter(d => d.station === selectedStation);

const filteredEvents = selectedStation === "All Stations"
  ? [] //no events to show for all stations mode
  : local_events.filter(d => d.nearby_station === selectedStation);
```

```js

Plot.plot({
  height: 500,
  width,
  marginLeft: 60,
  x: { label: "Date",
    tickFormat: "%b %d",
   },
  y: { label: "Total Ridership", grid: true},
  style: {
    fontSize: "12px"
  },


  marks: [
    Plot.frame(), 
      // Plot.groupX transform to aggregate ridership from entrances and exits
    Plot.lineY(filteredRidership, 
      Plot.groupX(
        { y: "sum" },
        {x: "date", y: d => d.entrances + d.exits,}
      )
    ), 
    
     // Event dots positioned at ridership values
    Plot.dot(filteredEvents, {
      x: "date",
      y: eventData => {
        // Find the ridership total for this event's date and station
        const matchingRidership = filteredRidership.filter(r => 
          r.date.toDateString() === eventData.date.toDateString() &&
          r.station === eventData.nearby_station
        );
        
        // Sum entrances + exits for this date
        const total = matchingRidership.reduce((sum, r) => sum + r.entrances + r.exits, 0);
        
        return total || 0; // Return 0 if no matching data
      },
      r: 3,
      fill: "orange",
      fillOpacity: 0.7,
      stroke: "black",
      tip: true, 
      channels: {"Event": "event_name"}
    }),


    //mark annotation of fair increase on July 15th
    Plot.ruleX([fareUpdate], {
      stroke: "orange",
      strokeWidth: 2, 
      strokeDasharray: "4,4"
    }), 
    Plot.text([fareUpdate], {
      x: fareUpdate,
      text: ["Fare Increase:\n $2.75 to $3.00"],
      fill: "orange",
      fontSize: 10,
      fontWeight: "bold", 
      dy: -150,
      dx: 40, 
    }), 
  
  ]
})
```

<h3>Station Differences: Week-over-Week Ridership</h3> 

To better understand how fare increase and local events effect individual stations, the heat map below shows week-over-week percentage changes in ridership. 

By binning the data for each week, it's easier to compare ridership trends without the variance between weekdays and weekend traffic peaks. Additionally, a week-over-week metric allows comparison across stations with different volumes of rider traffic and minimizes the variance of higher-use transfer stations. 

*Note: stations are sorted by overall ridership volume, the highest ridership stations at the top. Data in the week of June 02 was ommited from this chart as there's incomplete prior week data for a % change calculation.* 


<!-- First attempt at creating a binned plot to show ridership by week instead
```js
Plot.plot({
  height: 800,
  width,
  marginLeft: 150,
  marginRight: 100,
  x: { 
    label: "Week Starting",
    tickFormat:"%b %d", 
    type: "band"
  },
  y: { 
    label: null,
    domain: allStations.filter(s => s !== "All Stations") // Remove "All Stations" from y-axis
  },
  color: {
    type: "linear",
    scheme: "PuBuGn",
    label: "Total Ridership",
    legend: true, 
    domain: [0, 300000]
  },
  style: {
    fontSize: "12px"
  },
  marks: [
      Plot.cell(ridership,
          Plot.binX(
            { fill: "sum" },
            {
              x: { value: "date", interval: "week" },
              y: "station",
              fill: d => d.entrances + d.exits,
              inset: 0.5,
              tip: true,
            }
          )
        ),
    // Event markers - only on their corresponding station rows
    Plot.dot(local_events, {
      x: "date",
      y: "nearby_station",
      r: 5,
      fill: "black",
      fillOpacity: 0.8,
      tip: true,
      channels: {"Event": "event_name"}
    }),
    
    // Fare increase line
    Plot.ruleX([fareUpdate], {
      stroke: "orange",
      strokeWidth: 3,
      strokeDasharray: "4,4"
    })
  ]
})
``-->

<!-- 
```js
// Pre-bin ridership data by week and station
const weeklyRidership1 = d3.flatRollup(
  ridership,
  v => d3.sum(v, d => d.entrances + d.exits),
  d => d3.utcMonday(d.date), // Start of week (Monday)
  d => d.station
)
.filter(([week]) => week >= new Date("2025-06-02") && week <= new Date("2025-08-11")) // Filter to complete weeks
.map(([week, station, total]) => ({
  week,
  station,
  total_ridership: total
}));
```
-->

<!-- Initial attempt at binning ridership, needed to adjust this approach to a Week-over-week metric to see more meaninful differences and changes within ridership. 
```js
Plot.plot({
  height: 800,
  width,
  marginLeft: 150,
  marginRight: 100,
  x: { 
    label: "Week Starting",
    type: "band", // Discrete band scale like your teacher's example
    tickFormat: d => d3.utcFormat("%b %d")(d)
  },
  y: { 
    label: "",
    domain: allStations.filter(s => s !== "All Stations")
  },
  color: {
    type: "linear",
    scheme: "PuBuGn",
    label: "Total Ridership",
    legend: true,
    domain: [0, 300000]
  },
  style: {
    fontSize: "12px"
  },
  marks: [
    Plot.frame(),
    
    // Heatmap cells with pre-binned data
    Plot.cell(weeklyRidership1, {
      x: "week",
      y: "station",
      fill: "total_ridership",
      inset: 0.5,
      tip: true
    }),
    
    // Event markers (they'll snap to the weekly bins)
    Plot.dot(local_events, {
      x: d => d3.utcMonday(d.date), // Snap to week start
      y: "nearby_station",
      r: 5,
      fill: "black",
      fillOpacity: 0.8,
      tip: true,
      legend: true,
      channels: {"Event": "event_name"}
    }),
    
    // Fare increase line
    Plot.ruleX([d3.utcMonday(fareUpdate)], {
      stroke: "black",
      strokeWidth: 3,
    })
  ]
})
```
-->




```js
// Pre-bin ridership data by week and station, filtering incomplete weeks
const weeklyRidershipRaw = d3.flatRollup(
  ridership,
  v => d3.sum(v, d => d.entrances + d.exits),
  d => d3.utcMonday(d.date),
  d => d.station
)
.filter(([week]) => week >= new Date("2025-06-02") && week <= new Date("2025-08-04"))
.map(([week, station, total]) => ({
  week,
  station,
  total_ridership: total
}));

// Calculate week-over-week percent change
const weeklyRidership = weeklyRidershipRaw
  .sort((a, b) => a.station.localeCompare(b.station) || a.week - b.week) // Sort by station, then week
  .map((d, i, arr) => {
    // Find previous week for the same station
    const prevWeek = arr.find(p => 
      p.station === d.station && 
      p.week.getTime() === d.week.getTime() - 7 * 24 * 60 * 60 * 1000 // Exactly one week before
    );
    
    const wow_change = prevWeek 
      ? ((d.total_ridership - prevWeek.total_ridership) / prevWeek.total_ridership) * 100 
      : null; // First week has no comparison
    
    return {
      ...d,
      prev_week_ridership: prevWeek?.total_ridership,
      wow_change
    };
  })
  .filter(d => d.wow_change !== null); //removing June 2 week since there's not enough data before this for a week over week percentage change

//average ridership per station for sorting the cells to be more readable 
const avgRidershipByStation = d3.rollup(
  weeklyRidershipRaw,
  v => d3.mean(v, d => d.total_ridership),
  d => d.station
);
const stationsSorted = Array.from(avgRidershipByStation.entries())
  .sort((a, b) => b[1] - a[1]) // Sort by ridership descending
  .map(d => d[0]);

const preFareAvg = d3.mean(
  weeklyRidershipRaw.filter(d => d.week < new Date("2025-07-14")),
  d => d.total_ridership
);

const postFareAvg = d3.mean(
  weeklyRidershipRaw.filter(d => d.week >= new Date("2025-07-14")),
  d => d.total_ridership
);

const fareImpact = ((postFareAvg - preFareAvg) / preFareAvg * 100).toFixed(1);

```



```js
Plot.plot({
  height: 800,
  width,
  marginLeft: 150,
  marginRight: 100,
  x: { 
    label: "Week Starting",
    type: "band",
    tickFormat: d => d3.utcFormat("%b %d")(d)
  },
  y: { 
    label: null,
    domain: stationsSorted
  },
  color: {
    type: "diverging",
    scheme: "PuOr", // Purple = decrease, Orange = increase
    label: "Week-over-Week % Change",
    legend: true,
    domain: [-30, 30], // Adjust based on your data
    pivot: 0
  },
  style: {
    fontSize: "12px"
  },
  marks: [
    Plot.frame(),
    
    // Heatmap showing week-over-week change
    Plot.cell(weeklyRidership.filter(d => d.wow_change !== null), { // Skip first week
      x: "week",
      y: "station",
      fill: "wow_change",
      inset: 0.5,
      tip: true,
      title: d => `${d.station}\nWeek of ${d3.utcFormat("%b %d")(d.week)}\nRidership: ${d.total_ridership.toLocaleString()}\nPrevious week: ${d.prev_week_ridership?.toLocaleString()}\nChange: ${d.wow_change?.toFixed(1)}%`
    }),
    
    // Event markers
    Plot.dot(local_events.filter(d => d3.utcMonday(d.date) >= new Date("2025-06-09") && d3.utcMonday(d.date) <= new Date("2025-08-04")), {
      x: d => d3.utcMonday(d.date),
      y: "nearby_station",
      r: 3,
      fill: "black",
      stroke:"white",
      fillOpacity: 0.8,
      tip: {anchor:"top"},
      channels: {"Event": "event_name"}
    }),
    
    // Fare increase line
    Plot.ruleX([d3.utcMonday(fareUpdate)], {
      stroke: "black",
      strokeWidth: 3,
    }),

    // Annotation text for fare impact
    Plot.text([{
      x: new Date("2025-07-21"),
      y: stationsSorted[0], // Top station
      label: `Avg ridership change after fare increase: ${fareImpact}%`
    }], {
      x: "x",
      y: "y",
      text: "label",
      fill: "black",
      fontSize: 11,
      fontWeight: "bold",
      dy: -40,
      dx: 50
    })


  ]
})

```

<h4>Key observations:</h4>

- **Purple cells** indicate ridership decreases compared to the previous week
- **Orange cells** indicate ridership increases
- **Black dots** mark weeks when events occurred as noted in 'nearby station' data
- The **vertical black line** marks the week of July 15th fare increase

Across nearly all stations, the week of July 15th shows purple, demonstrating the immediate impact of decreased ridership corresponding with fare increase. Stations with the largest dips include 14th street (-36%), 72nd street (-31%), and West 4 St-Wash Sq (-23%). The average ridership change following the fare increase was **${fareImpact}%**.

<h4>Local Event Impact on Ridership</h4>

In contrast, weeks with events, especially at high-traffic stations, tend to show orange cells indicating event-driven ridership surges. For example, Chambers St. station is one of the only stations with ridership increases during the new fare roll out, and you can see the Pride Parade took place the same week, which explains the increase. 


<h3>Part II: Incident Data & Response Time by Station</h3>

  The scatter plot below shows average response time to incidents across all 25 subway stations. Use the drop down to filter by incident severity. Each dot represents a station, positioned by total incidents versus avg. response time. The dashed lines serve as a median quadrant lines to benchmark performance. 
  
  Stations situated in the upper right corner, above both dashed lines, face the most challenge, a high number of incidents combined with slow response times. These stations could be priority for increased staffing efforts.

```js
// Create severity filter toggle
const severityFilter = view(Inputs.select(
  ["all", "low", "medium", "high"],
  {label: "Incident Severity", value: "all"}
));
```

```js
// Filter and aggregate data based on severity selection
const filteredIncidents = severityFilter === "all" 
  ? incidents 
  : incidents.filter(d => d.severity === severityFilter);

const stationIncidentStats = d3.rollup(
  filteredIncidents,
  v => ({
    avg_response_time: d3.mean(v, d => d.response_time_minutes),
    incident_count: v.length,
    median_response_time: d3.median(v, d => d.response_time_minutes),
    avg_staffing: d3.mean(v, d => d.staffing_count) // Average staffing across incidents at stations because it varies by incident
  }),
  d => d.station
);

const stationData = Array.from(stationIncidentStats, ([station, stats]) => ({
  station,
  ...stats
}));
```

```js
// Calculate medians for quadrant lines
const medianIncidents = d3.median(stationData, d => d.incident_count);
const meanResponseTime = d3.mean(stationData, d => d.avg_response_time);
```

```js
Plot.plot({
  height: 700,
  width,
  marginLeft: 60,
  marginRight: 120,
  marginTop: 40,
  marginBottom: 60,
  x: {
    label: "Total Incidents",
    grid: true
  },
  y: {
    label: "Average Response Time (minutes)",
    grid: true
  },
  marks: [

    // Quadrant lines
    Plot.ruleY([meanResponseTime], {
      stroke: "gray",
      strokeDasharray: "4,4",
      strokeWidth: 1.5
    }),
    
    Plot.ruleX([medianIncidents], {
      stroke: "gray",
      strokeDasharray: "4,4",
      strokeWidth: 1.5
    }),
    
    // Scatter plot - dot size based on average staffing
    Plot.dot(stationData, {
      x: "incident_count",
      y: "avg_response_time",
      r: 8,
      fill: "orange",
      fillOpacity: 0.6,
      stroke: "darkorange",
      strokeWidth: 1.5,
      tip: true,
      title: d => `${d.station}\nIncidents: ${d.incident_count}\nAvg Response: ${d.avg_response_time.toFixed(1)} min\nMedian Response: ${d.median_response_time.toFixed(1)} min\nAvg Staffing: ${d.avg_staffing.toFixed(1)}`
    }),
    
Plot.text(stationData, {
  x: "incident_count",
  y: "avg_response_time",
  text: d => d.station.split("-")[0], // Abbreviated names
  fontSize: 10,
  dy: -18,
  fill: "black",
  stroke: "white",
  strokeWidth: 2,
  paintOrder: "stroke"
}),
    
    // Label for median incident line
    Plot.text([{x: medianIncidents, y: meanResponseTime}], {
      x: "x",
      y: d => d.y * 0.5,
      text: [`Median Incidents: ${medianIncidents.toFixed(0)}`],
      fontSize: 10,
      fontWeight: "bold",
      fill: "black",
      dx: 5,
      dy: -5
    }),
    
    // Label for mean response time line
    Plot.text([{x: medianIncidents, y: meanResponseTime}], {
      x: d => d.x * 0.3,
      y: "y",
      text: [`Mean Response: ${meanResponseTime.toFixed(1)} min`],
      fontSize: 10,
      fontWeight: "bold",
      fill: "black",
      dx: 5,
      dy: -5
    })
  ]
})
```
<h4>Benchmarking Station Incident Performance</h4> 

Stations with the slowest overall response times are:

- **59 St** - 19.3 min average
- **West 4 St** - 18.9 min average
- **Bowling Green** - 18.5 min average
- **Canal St** - 18.4 min average

Stations with the fastest are:

- **Grand Central-42 St** - 5.1 min average 
- **Times Sq-42 St** - 5.3 min average
- **Herald Sq-34 St** - 5.8 min average  

<h4>Observations by Severity</h4>

  The toggle between severity levels shows that incidents marked 'high' consistenlty show longer response times across stations, with the system-wide average at ~8 minutes for 'low' severity incidents, and ~12 minutes for 'high' severity. 

  Although there's some variance upper quadrent stations, based on severity, these stations deem the most support as *consistentl low performers* across incident levels: 125th St, Canal St, 59th St, Bowling Green, West 4th St.




<h3>Part III: Staffing Support based on 2026 Estimated Event Attendance</h3>

  To identify which stations need additional staffing for the 2026 event calculator this chart examines average event attendance per staff member for each station. This metric is used to help estimate workload while accounting for both the size and frequency of events. Stations with more small events, and those with fewer larger ones are evaluated against one-another based on workload relevent to current staffing levels. 



```js
// Calculate 2026 event metrics per station
const station2026Stats = d3.rollup(
  upcoming_events,
  v => ({
    total_expected_attendance: d3.sum(v, d => d.expected_attendance),
    avg_expected_attendance: d3.mean(v, d => d.expected_attendance),
    event_count: v.length,
    max_event_attendance: d3.max(v, d => d.expected_attendance)
  }),
  d => d.nearby_station
);

// Combine with current staffing
const staffingAnalysis = Array.from(station2026Stats, ([station, stats]) => ({
  station,
  ...stats,
  current_staffing: currentStaffing[station] || 0,
  attendance_per_staff: stats.total_expected_attendance / (currentStaffing[station] || 1)
})).sort((a, b) => b.attendance_per_staff - a.attendance_per_staff); // Sort by workload per staff

// new metric: avg attendance per event, adjusted for staffing, to account for stations with more events
const staffingAnalysisAdjusted = staffingAnalysis.map(d => ({
  ...d,
  avg_attendance_per_staff: d.avg_expected_attendance / d.current_staffing,
  workload_score: (d.total_expected_attendance / d.current_staffing) * (d.event_count / 10) // Balanced metric
}));

const stationEventText = d3.rollup(
  upcoming_events,
  v => v.map(e => e.event_name).join(", "),
  d => d.nearby_station
);
```


```js
Plot.plot({
  height: 600,
  width,
  marginLeft: 150,
  marginRight: 200,
  x: {
    label: "Average Event Attendance per Staff Member",
    grid: true
  },
  y: {
    label: null
  },
  marks: [
    Plot.barX(staffingAnalysisAdjusted, {
      x: "avg_attendance_per_staff",
      y: "station",
      fill: "#4B0082",
      Opacity: 0.6,
      sort: { y: "-x" },
      tip: true,
      title: d => `${d.station}\n2026 Events: ${d.event_count}\nAvg Event Size: ${Math.round(d.avg_expected_attendance).toLocaleString()}\nCurrent Staff: ${d.current_staffing}\nAvg Attendance per Staffer: ${Math.round(d.avg_attendance_per_staff).toLocaleString()}`
    }), 


    Plot.text(
      staffingAnalysisAdjusted.map(d => ({
        ...d,
        events: stationEventText.get(d.station) || ""
      })),
      {
        y: "station",
        text: "events",
        fontSize: 8,
        fill: "black",
        stroke: "white",
        strokeWidth: 3,
        paintOrder: "stroke",
        textAnchor: "start",
        dx: 210, 
        lineWidth: 25
      }
    )
  ]
})
```




<h4>Anticipated Staffing Workloads by Station</h4>

Based on 2026 event projections and current staffing allocations, **the top station requiring increased staffing support is:**

- **${staffingAnalysisAdjusted[0].station}** - Each staff member will manage an average of **${Math.round(staffingAnalysisAdjusted[0].avg_attendance_per_staff).toLocaleString()} attendees per event** across ${staffingAnalysisAdjusted[0].event_count} scheduled events. Current staffing: ${staffingAnalysisAdjusted[0].current_staffing}.

<div class="card">

  Note: This station shows need in all three sections of transit analysis. Canal St. has the most strained workload based on the 2026 event attendance, and is a historically poor performer in incident response time, as identified in Part II. Canal St. station also represents one of the highest volumes of ridership across the 2025 summer event data, indicating a precedent that 2026 may see similar traffic. This puts it at the top of the list for additional support.
  
  </div>

It's also valuable to point out the 2026 event strain on **34th St. Penn Station**. Despite a lower avg. event attendance/staff metric, event notes detail the next highest number of events versus the other stations. It's possible that, despite strain on staff, one temporary high volume event won't merit additional staff year round. 


<h4>Bonus: Distribution of 2026 Event Sizes by Station</h4>

While average attendance per staff member identifies overall workload, understanding the **distribution of event sizes** at each station helps distinguish between stations with consistently large events versus those with occasional spikes. This context ensures staffing recommendations account for sustained demand rather than isolated incidents.

```js
// Focus on top stations for readability
const topStations = staffingAnalysisAdjusted.slice(0, 10).map(d => d.station);
const topStationEvents = upcoming_events.filter(d => topStations.includes(d.nearby_station));
```


```js
Plot.plot({
  height: 500,
  width,
  marginLeft: 150,
  x: {
    label: "Expected Event Attendance",
    grid: true
  },
  y: {
    label: null,
    domain: topStations
  },
  marks: [
    // Box plot showing distribution
    Plot.boxX(topStationEvents, {
      x: "expected_attendance",
      y: "nearby_station",
      fill: "rebeccapurple",
      fillOpacity: 0.3,
      stroke: "rebeccapurple",
      strokeWidth: 2
    }),
    
    // Individual events as dots
    Plot.dot(topStationEvents, {
      x: "expected_attendance",
      y: "nearby_station",
      fill: "rebeccapurple",
      fillOpacity: 0.6,
      r: 4,
      tip: true,
      title: d => `${d.event_name}\nExpected: ${d.expected_attendance.toLocaleString()}`
    })
  ]
})
```
<h4>Event Attendee Distribution by Station </h4>

This box plot reveals the consistency of event sizes at each station:

- **Wide boxes with long whiskers** indicate stations with highly variable event sizes (mix of small and large events), for example Fulton St, which anticipates a small crowd for a Farmers Market, but a larger crowd for Museum Night.
- **Narrow boxes** show stations with more consistently sized events, or like West 4 St, only one. 
- **Outlier dots far from the box** represent one-off high-attendance events

For example, if a station appears high on the average workload chart but shows mostly small events with one outlier in this distribution, it may not need as much sustained staffing as a station with consistently large events. **Stations with consistently high attendance across multiple events warrant priority staffing increases**, as staff will face sustained pressure throughout the summer rather than managing a single large event.

This distribution analysis confirms that, in addition to ${staffingAnalysisAdjusted[0].station}, ${staffingAnalysisAdjusted[1].station}, and ${staffingAnalysisAdjusted[2].station} face sustained high-volume events, not just isolated spikes and could benefit from additional staffing for summer event season. 



