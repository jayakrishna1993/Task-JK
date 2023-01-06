<template>
  <div
    class="row"
    style="background-color: #d2d2d2; height: 60px; margin-right: auto"
  >
    <div class="row col-4 col-md-6 col-sm-12 sub-nav">
      <select
        class="select-drop-down"
        id="selectButton"
        type="button"
        data-bs-toggle="dropdown"
        aria-expanded="false"
        aria-label="Select Year"
      ></select>
    </div>
  </div>
  <div id="plot-wrapper">
    <div class="row">
      <div class="col-4 col-md-4 col-sm-12 col-xs-12">
        <div class="world-pop-div">
          <h1 class="pop-title">World Population</h1>
          <div id="selectedYear">({{ 2022 }})</div>
          <div id="popValue">
            <span id="selectedPopulation" class="pop-bold">7.8</span
            ><span class="bn-bold">Bn</span>
          </div>
        </div>
      </div>
      <div
        id="plotSVG"
        class="scatter-css col-8 col-md-8 col-sm-12 col-xs-12"
      ></div>
    </div>
  </div>
</template>
<script>
import * as d3 from "d3";
export default {
  name: "Dashboard-View",
  data() {
    return {
      populationData: [],
      yearsSelection: [],
      selectedYear: "",
    };
  },
  methods: {
    async showPlot() {
      console.log("Window Inner width", window.innerWidth);
      const height = 500;
      // creating a div for tooltip that appends to svg body
      var div = d3
        .select("#plotSVG")
        .append("div")
        .attr("class", "tooltip")
        .style("opacity", 0)
        .style("position", "absolute")
        .style("background-color", "black")
        .style("border-radius", "5px")
        .style("padding", "10px")
        .style("color", "white");
      // Population details from csv file
      this.populationData = await d3.csv("population.csv", function (data) {
        return data;
      });

      // Filtering out the years for dropdown
      let years = this.populationData.map((item) => item.Year);
      // make datacopy to avoid the direct mutation of original reference
      let masterCopy = this.populationData.map((item) => {
        return item;
      });

      // Removing duplicates
      years = [...new Set([...years])].sort();
      years.unshift("Select Year");
      this.yearsSelection = years;
      let legendData = masterCopy.map((data) => data.Country);
      legendData = [...new Set([...legendData])].sort();

      // Dropdown implementation
      d3.select("#selectButton")
        .selectAll("myOptions")
        .data(years)
        .enter()
        .append("option")
        .text(function (d) {
          return d;
        })
        .attr("value", function (d) {
          return d;
        });

      // initialise the svg plotting
      let svg = d3
        .select("#plotSVG")
        .append("svg")
        .style("overflow", "visible")
        .append("g")
        .attr("transform", "translate(50,50)")
        .attr("viewBox", `0 0 600 400`);

      let xScale = d3.scaleLinear().domain([0, 600]).range([0, 600]);
      let yScale = d3.scaleLinear().domain([0, 5]).range([400, 0]);

      // Setting colors for chart
      let chartColors = {
        Angola: "#1AE67D",
        Burundi: "#7f7f7f",
        Chile: "#8c564b",
        Djibouti: "#1f77b4",
        Eritrea: "#ff7f0e",
        Ethiopia: "#2ca02c",
        Kenya: "#d62728",
      };

      //Setting axis of x & y
      svg
        .append("g")
        .attr("id", "yAxis")
        .call(
          d3
            .axisLeft(yScale)
            .ticks(5)
            .tickFormat(d3.format("d"))
            .tickSizeOuter(0)
        );
      svg
        .append("g")
        .attr("transform", "translate(0,400)")
        .attr("id", "xAxis")
        .call(
          d3
            .axisBottom(xScale)
            .ticks(5)
            .tickFormat(d3.format("d"))
            .tickSizeOuter(0)
        );
      // Drawing bubbles on chart
      svg
        .selectAll(".bubble")
        .data(this.populationData)
        .join("circle")
        .attr("class", "bubble")
        .attr("cx", (d) => xScale(d.Population_Density))
        .attr("cy", (d) => yScale(d.Population_Growth_Rate))
        .attr("r", (d) => Math.sqrt(d.Population / 5))
        .attr("stroke", (d) => chartColors[d.Country])
        .attr("fill", (d) => chartColors[d.Country])
        .attr("fill-opacity", 0.5)
        .on("mouseover", function (event, d) {
          div.transition().duration(200).style("opacity", 0.9);
          div
            .html(d.Country + "<br/>" + d.Population)
            .style("left", event.pageX + "px")
            .style("top", event.pageY + "px");
        })
        .on("mousemove", function (event, d) {
          div.transition().duration(200).style("opacity", 0.9);
          div
            .html(d.Country + "<br/>" + d.Population)
            .style("left", event.pageX + "px")
            .style("top", event.pageY + "px");
        })
        .on("mouseout", function (d) {
          console.log(d);
          div.transition().duration(500).style("opacity", 0);
        });

      // When year changes updating the chart with new values
      async function update(selectedGroup) {
        let dataFilter = [];
        if (selectedGroup === "" || selectedGroup === "Select Year") {
          dataFilter = masterCopy;
        } else {
          dataFilter = masterCopy?.filter(
            (item) => item.Year === selectedGroup
          );
        }
        svg
          .selectAll(".bubble")
          .data(dataFilter)
          .join("circle")
          .attr("class", "bubble")
          .attr("cx", (d) => xScale(d.Population_Density))
          .attr("cy", (d) => yScale(d.Population_Growth_Rate))
          .attr("r", (d) => Math.sqrt(d.Population / 5))
          .attr("stroke", (d) => chartColors[d.Country])
          .attr("fill", (d) => chartColors[d.Country])
          .attr("fill-opacity", 0.5)
          .on("mouseover", function (event, d) {
            div.transition().duration(200).style("opacity", 0.9);
            div
              .html(d.Country + "<br/>" + d.Population)
              .style("left", event.pageX + "px")
              .style("top", event.pageY + "px")
              .style("class", "tooltip");
          })
          .on("mousemove", function (event, d) {
            div.transition().duration(200).style("opacity", 0.9);
            div
              .html(d.Country + "<br/>" + d.Population)
              .style("left", event.pageX + "px")
              .style("top", event.pageY + "px")
              .style("class", "tooltip");
          })
          .on("mouseout", function (d) {
            console.log(d);
            div.transition().duration(500).style("opacity", 0);
          });
      }
      // Adding titles of axis and legends
      svg
        .append("text")
        .attr("x", 250)
        .attr("y", height - 50)
        .attr("class", "label")
        .text("Population Density");
      svg
        .append("text")
        .attr("y", -25)
        .attr("x", -height / 2)
        .attr("transform", "rotate(-90)")
        .attr("class", "label")
        .text("Population Growth");
      svg
        .selectAll("legendSymbols")
        .data(legendData)
        .enter()
        .append("circle")
        .attr("cx", height + 60)
        .attr("cy", (d, i) => 150 + i * 25)
        .attr("r", 5)
        .attr("fill", (d) => chartColors[d])
        .attr("fill-opacity", 0.5)
        .style("z-index", 1);

      svg
        .selectAll("legendTexts")
        .data(legendData)
        .enter()
        .append("text")
        .text((d) => d)
        .attr("x", height + 70)
        .attr("y", (d, i) => 150 + i * 25 + 5)
        .attr("class", "textbox");
      // Detect the year selection change
      d3.select("#selectButton").on("change", function (d) {
        const selectedOption = d.target.value;
        const randomPopulation =
          Math.floor(Math.random() * (1000 - 100 + 100)) / 100;
        update(selectedOption);
        d3.select("#selectedYear").html("(" + d.target.value + ")" + "<br/>");
        d3.select("#selectedPopulation").html(randomPopulation);
      });
    },
  },

  mounted() {
    this.showPlot();
  },
};
</script>

<style scoped>
select {
  font-size: 16px;
}

#xAxis {
  font-size: 16px;
}

#yAxis {
  font-size: 16px;
}

.scatter-css {
  height: 37.5rem;
  width: 50%;
  text-align: left;
}

svg {
  height: 37.5rem;
  width: 100%;
}

.select-drop-down {
  width: 250px;
  height: 45px;
  margin-top: 5px;
  border-radius: 6px;
  background-color: #3391d2;
  color: white;
  outline: 0;
}

@media only screen and (max-width: 700px) {
  .select-drop-down {
    width: 350px;
  }

  .sub-nav {
    padding-left: 0px;
  }
}
@media only screen and (min-width: 700px) and (max-width: 900px) {
  .select-drop-down {
    width: 350px;
  }

  .sub-nav {
    padding-left: 0;
  }
}
@media only screen and (min-width: 900px) {
  .sub-nav {
    display: flex;
    padding-left: 200px;
  }
}

.world-pop-div {
  border-radius: 6px;
  border: 1px solid #d1d1d1;
  height: 28rem;
  margin-left: 5rem;
  margin-top: 2.5rem;
  margin-right: 5rem;
}

.pop-title {
  padding-top: 30px;
}

#selectedYear {
  font-size: 20px;
  font-weight: 600;
}

#popValue {
  margin-top: 4rem;
}

.pop-bold {
  font-size: 55px;
  font-weight: 900;
  color: #3391d2;
}

.bn-bold {
  font-size: 30px;
  font-weight: 900;
  color: #3391d2;
}
</style>
