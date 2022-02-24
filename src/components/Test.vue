/* eslint-disable */
<template>
  <div class="root">
    <input type="text" id="event" v-model="eventnum">
    <button v-on:click="draw()">submit</button>
    <div id="map"></div>
    <div id="map2"></div>
  </div>
</template>

<script>
  import * as d3 from 'd3'
  import onMounted from 'vue'

  export default {
    name: 'Test',
    data() {
      return {
        eventnum: 1
      }
    },
    methods: {
      get_features(eventx) {
        let myVars = []
        let clips = []
        let ids = []
        let scores = []
        let avgVars = []
        let data = require('../assets/feature_part.json');
        for (var i = 0; i < data.length; i++) {
          if (data[i].event_order == eventx) {
            if (myVars.includes(data[i]['new_clipi-j']) == false) {
              myVars.push(data[i]['new_clipi-j'])
            }
            clips.push(data[i]['new_clipi-j'])
            ids.push(data[i]['id'])
            scores.push(data[i]['score'])
          }
        }
        for (var i = 0; i < myVars.length; i++) {
          var sum = 0
          var count = 0
          for (var j = 0; j < clips.length; j++) {
            if (clips[j] == myVars[i]) {
              sum += parseFloat(scores[j])
              count += 1
            }
          }
          if (count == 0) {
            avgVars.push(0)
          } else {
            avgVars.push((sum / count).toFixed(3))
          }
        }
        var info = {
          "clips": clips,
          "ids": ids,
          "scores": scores,
          "myVars": myVars,
          "avgVars": avgVars
        };
        return info
      },
      heatMap(myGroups, myVars, mySvg, data) {
        const margin = {
            top: 30,
            right: 30,
            bottom: 30,
            left: 60
          },
          width = myVars.length * 100 - margin.left - margin.right,
          height = 450 - margin.top - margin.bottom;

        // append the svg object to the body of the page
        const svg = d3.select(mySvg)
          .append("svg")
          .attr("width", width + margin.left + margin.right)
          .attr("height", height + margin.top + margin.bottom)
          .append("g")
          .attr("transform", `translate(${margin.left},${margin.top})`);

        // Build X scales and axis:
        const x = d3.scaleBand()
          .range([1, width])
          .domain(myVars)
          .padding(0.5);
        svg.append("g")
          .attr("transform", `translate(0, ${height})`)
          .call(d3.axisBottom(x))

        // Build Y scales and axis:
        const y = d3.scaleBand()
          .range([height, 0])
          .domain(myGroups)
          .padding(0.5);
        svg.append("g")
          .call(d3.axisLeft(y));

        // Build color scale
        const myColor = d3.scaleLinear()
          .range(["white", "#a80000"])
          .domain([0, 0.6])

        //Read the data
        svg.selectAll()
          .data(data, function(d) {
            return d.group + ':' + d.variable;
          })
          .join("rect")
          .attr("x", function(d) {
            return x(d.group)
          })
          .attr("y", function(d) {
            return y(d.variable)
          })
          .attr("width", x.bandwidth())
          .attr("height", y.bandwidth())
          .style("fill", function(d) {
            return myColor(d.value)
          })

      },
      lineChart(dataset, mySvg) {
        var min = d3.min(dataset, function(d) {
          return d[1];
        })
        var max = d3.max(dataset, function(d) {
          return d[1];
        })
        var xmax = d3.max(dataset, function(d) {
          return d[0];
        })

        const margin = {
            top: 30,
            right: 10,
            bottom: 30,
            left: 60
          },
          width = dataset.length * 100 - margin.left - margin.right,
          height = 250 - margin.top - margin.bottom;

        // 预留给轴线的距离
        var padding = {
          top: 30,
          right: 30,
          bottom: 30,
          left: 90
        };
        var xScale = d3.scaleBand()
          .domain(dataset.map(datum => datum[0]))
          .range([0, width - padding.left - padding.right]);
        var yScale = d3.scaleLinear()
          .domain([0, max])
          .range([height - padding.top - padding.bottom, 0]);
        var svg = d3.select(mySvg)
          .append('svg')
          .attr('width', width + margin.left + margin.right)
          .attr('height', height + margin.top + margin.bottom);
        var xAxis = d3.axisBottom()
          .scale(xScale)
          .ticks(xmax)
        var yAxis = d3.axisLeft()
          .scale(yScale);
        svg.append('g')
          .attr('class', 'axis')
          .attr('transform', 'translate(' + padding.left + ',' + (height - padding.bottom) + ')')
          .call(xAxis);
        svg.append('g')
          .attr('class', 'axis')
          .attr('transform', 'translate(' + padding.left + ',' + padding.top + ')')
          .call(yAxis);
        var linePath = d3.line()
          .x(function(d) {
            return xScale(d[0])
          })
          .y(function(d) {
            return yScale(d[1])
          });
        svg.append('g')
          .append('path')
          .attr('class', 'line-path')
          .attr('transform', 'translate(' + (padding.left + (width - padding.left - padding.right) / dataset.length /
            2) + ',' + padding.top + ')')
          .attr('d', linePath(dataset))
          .attr('fill', 'none')
          .attr('stroke-width', 3)
          .attr('stroke', 'green');
        svg.append('g')
          .selectAll('circle')
          .data(dataset)
          .enter()
          .append('circle')
          .attr('r', 5)
          .attr('transform', function(d) {
            return 'translate(' + (xScale(d[0]) + padding.left + +(width - padding.left - padding.right) / dataset
              .length / 2) + ',' + (yScale(d[1]) + padding.top) + ')'
          })
          .attr('fill', 'green');
      },
      draw() {
        //d3.select("#map").selectAll('*').remove()
        let eventx = this.eventnum
        let info = this.get_features(eventx)
        let myGroups = ["0", "1", "2", "3", "4", "5"]
        let data = []
        let data2 = []
        for (var i = 0; i < info.clips.length; i++) {
          data.push({
            "group": info.clips[i],
            "variable": info.ids[i],
            "value": info.scores[i]
          })
        }
        for (var i = 0; i < info.avgVars.length; i++) {
          data2.push([info.myVars[i], info.avgVars[i]])
        }
        this.heatMap(myGroups, info.myVars, "#map", data)
        this.lineChart(data2, "#map2")
      },
    },
    mounted() {
      //this.draw(2)
    }
  }
</script>

<style>
  .root {
    overflow-x: auto;
    overflow-y: hidden;
    white-space: nowrap;
  }
  .map {
    display: inline-block;
  }
  .map2 {
    display: inline-block;
  }
</style>
