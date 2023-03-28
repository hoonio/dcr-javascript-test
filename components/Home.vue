<template>
	<span>
		<h1><slot /></h1>
		<h4>By country</h4>
		<v-btn @click="criterion = 'population'">Population size</v-btn>
		<v-btn @click="criterion = 'borders'">Number of borders</v-btn>
		<v-btn @click="criterion = 'timezones'">Number of timezones</v-btn>
		<v-btn @click="criterion = 'languages'">Number of languages</v-btn>
		<h4>By region</h4>
		<v-btn @click="criterion = 'countries'">Number of countries</v-btn>
		<v-btn @click="criterion = 'regiontimezone'">Number of unique timezones</v-btn>
		<svg id="bubble-chart"></svg>
		<div class="tooltip">
			<h2></h2>
			<span></span>
		</div>
    <DataTable :dataset='renderData' :label='criterion' />
	</span>
</template>

<script>
	import * as d3 from "d3";
  import DataTable from './DataTable.vue'
	import countries from "../data/countries.json";

	const displayFigure = (criterion, data) => {
		switch (criterion) {
			case "population":
				return data.population;
			case "borders":
				return data.borders.length;
			case "timezones":
				return data.timezones.length;
			case "languages":
				return data.languages.length;
			case "countries":
				return data.population;
			case "regiontimezone":
				return data.population;
			default:
				return data.population;
		}
	};

	export default {
    components: {
      DataTable,
    },
		data: () => ({
			criterion: "population",
			renderData: [],
			width: 1200,
			height: 600,
		}),
		watch: {
			criterion(newCrit) {
				// console.log("criterion set", newCrit);
        this.renderData = this.groupByContinent(newCrit)
      },
      renderData(newDataSet) {
        d3.selectAll("svg > *").remove();
				this.renderBubbles(newDataSet);
      }
		},
		methods: {
			groupByContinent: (criterion) => {
        if (criterion === 'countries'){
          const regions = countries.reduce((rv, x) => {
            (rv[x['region']] = rv[x['region']] || []).push(x['name'])
            return rv
          }, {})
          const withCount = Object.keys(regions).map(key => ({
            alpha3Code: key,
            name: key,
            display: regions[key].length,
          }))
          return withCount
        } else if  (criterion === 'regiontimezone'){
          const regions = countries.reduce((rv, x) => {
            (rv[x['region']] = rv[x['region']] || []).push(x['timezones'])
            return rv
          }, {})
          const withCount = Object.keys(regions).map(key => ({
            alpha3Code: key,
            name: key,
            display: (regions[key].flat().filter((value, index, arr) => arr.indexOf(value) === index)).length,
          }))
          return withCount
        } else {
          const dataToRender = countries.map((cou) => ({
            alpha3Code: cou.alpha3Code,
            name: cou.name,
            display: displayFigure(criterion, cou),
          }));
          return dataToRender;
        }

			},
			renderBubbles(dataToPlot) {
				const bubble = (data) =>
					d3.pack().size([this.width, this.height]).padding(2)(
						d3.hierarchy({ children: data }).sum((d) => d.display)
					);

				const svg = d3
					.select("#bubble-chart")
					.style("width", this.width)
					.style("height", this.height);

				const root = bubble(dataToPlot);
				const tooltip = d3.select(".tooltip");

				const node = svg
					.selectAll()
					.data(root.children)
					.enter()
					.append("g")
					.attr("transform", `translate(${this.width / 2}, ${this.height / 2})`);

				const circle = node
					.append("circle")
					.style("fill", (d) => "#333")
					.on("mouseover", function (e, d) {
						tooltip.select("h2").attr("href", d.data.region).text(d.data.name);
						tooltip.select("span").text(d.data.capital);
						tooltip.style("visibility", "visible");
						d3.select(this).style("stroke", "#222");
					})
					.on("mousemove", (e) =>
						tooltip
							.style("top", `${e.pageY}px`)
							.style("left", `${e.pageX + 10}px`)
							.style("position", "absolute")
					)
					.on("mouseout", function () {
						d3.select(this).style("stroke", "none");
						return tooltip.style("visibility", "hidden");
					});

				const label = node
					.append("text")
					.attr("dy", 2)
					.attr("dx", 2)
					.text((d) => d.data.alpha3Code);
				// .append("text")
				// .attr("dy", 2)
				// .text((d) => d.data.population);

				node
					.transition()
					.ease(d3.easeExpInOut)
					.duration(1000)
					.attr("transform", (d) => `translate(${d.x}, ${d.y})`);

				circle
					.transition()
					.ease(d3.easeExpInOut)
					.duration(1000)
					.attr("r", (d) => d.r);

				label
					.transition()
					.delay(700)
					.ease(d3.easeExpInOut)
					.duration(1000)
					.style("opacity", 1);
			},
		},
		mounted() {
      this.renderData = this.groupByContinent('population')
		},
	};
</script>