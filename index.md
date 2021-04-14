<!DOCTYPE html>
<meta charset="utf-8">
<head>
    <title>The World Happiness Score in 2020</title>
</head>
<style>
    body {
        width: 940px;
        margin: 0 auto;
        margin-top: 2em;
    }
    svg {
        font: 10px sans-serif;
    }

    .axis path,
    .axis line {
        fill: none;
        stroke: black;
        shape-rendering: crispEdges;
    }
</style>
<body>
<h1>The World Happiness Score in 2020</h1>
<script src="d3.js"></script>
<script src="topojson.v1.min.js"></script>

<script>
    /**
     * copied from the following resources:
     * http://bl.ocks.org/micahstubbs/281d7b7a7e39a9b59cf80f1b8bd41a72
     * http://bl.ocks.org/msbarry/9911363
     * http://bl.ocks.org/weiglemc/6185069
     *
    **/

    const margin = {top: 0, right: 0, bottom: 0, left: 0};
	const width = 1000 - margin.left - margin.right;
	const height = 2000 - margin.top - margin.bottom;

    const color = d3.scaleThreshold()
        .domain([3, 3.5, 4, 4.5, 5, 5.5, 6, 6.5, 7, 7.5, 8])
        .range( d3.schemeRdYlGn[11] )
        .unknown(d3.rgb(255,200,200));

    const size = d3.scaleSqrt()
        .domain([200000, 1310000000])
        .range([1, 25]);

    const svg = d3.select('body')
			.append('svg')
			.attr('width', width)
			.attr('height', height);

    const map = svg
        .append('g')
        .attr('class', 'map');

    const scatterplot = svg
        .append('g')
        .attr('class', 'scatterplot')
        .attr("transform", "translate(100,500)");

    const scatterplot2 = svg
        .append('g')
        .attr('class', 'scatterplot2')
        .attr("transform", "translate(100,690)");

    const scatterplot3 = svg
        .append('g')
        .attr('class', 'scatterplot3')
        .attr("transform", "translate(100,880)");

    const scatterplot4 = svg
        .append('g')
        .attr('class', 'scatterplot4')
        .attr("transform", "translate(100, 1070)");

    const scatterplot5 = svg
        .append('g')
        .attr('class', 'scatterplot5')
        .attr("transform", "translate(100,1260)");

    const projection = d3.geoMercator()
			.scale(100)
			.translate( [width / 2, 1000 / 3.5]);

	const path = d3.geoPath().projection(projection);


    Promise.all([
        d3.csv('happiness.csv'),
        d3.json('world_countries.json')

    ]).then(function(data) {
		const fertilityById = {};
        let happiness = data[0];
        let countries = data[1];

        let fieldColor = 'Ladderscore';
        let fieldYAxis = "Ladderscore";
        let fieldSize = "Population"
        let fieldCountry = "Country"

        let Logged = 'Logged GDP per capita';
        let Social = "Social support";
        let Healthy = "Healthy life expectancy";
        let Freedom = "Freedom to make life choices";
        let Perceptions = "Perceptions of corruption";

        happiness.forEach(d => {
            if(d[fieldColor] == '') {
                d[fieldColor] = undefined;
            };
        });
        happiness.forEach(d => {
            if(d[fieldSize] == '') {
                d[fieldSize] = undefined;
            };
        });

        happiness.forEach(d => {
            if(d[fieldCountry] == '') {
                d[fieldCountry] = undefined;
            };
        });
        happiness.forEach(d => { fertilityById[d.id] = +d[fieldColor]; });
        happiness.forEach(d => { fertilityById[d.id] = +d[fieldSize]; });
        happiness.forEach(d => { fertilityById[d.id] = +d[fieldCountry]; });
        countries.features.forEach( d => { d[fieldColor] = fertilityById[d.id]});

        svg.append('g')
				.attr('class', 'countries')
				.selectAll('path')
				.data(countries.features)
				.enter().append('path')
                    .attr("class", d => { return "COUNTRY-CODE-"+d.id;} )
				.attr('d', path)
                .style("fill", function(d) { return color(d[fieldColor]);})
				.style('stroke', 'white')
				.style('opacity', 0.8)
				.style('stroke-width', 0.3)
				.on('mouseover',function(d){
                    d3.selectAll("."+d3.select(this).attr("class"))
                        .transition()
                        .duration(200)
                        .style("opacity", 1.0)
                        .style('stroke-width', 1.0)
                        .style("stroke", "black")
                        .style("fill", color(happiness[fieldColor]));
                    d3.select("."+d3.select(this).attr("class"))
                        .append("title")
                        .attr('country', function(d){return d.id})
                        .text(function(d){return d.Country;});
                })
				.on('mouseout', function(d){
                    d3.selectAll("."+d3.select(this).attr("class"))
                        .transition()
                        .duration(200)
                        .style('opacity', 0.8)
                        .style('stroke-width', 0.3)
                        .style("stroke", "white")
                });

		svg.append('path')
				.datum(topojson.mesh(countries.features, (a, b) => a.id !== b.id))
				.attr('class', 'names')
				.attr('d', path);

        // setup x
        var xValue = function(d) { return d[Logged];}, // data -> value
            xScale = d3.scaleLinear().range([0, 2000/2-150]), // value -> display
            xMap = function(d) { return xScale(xValue(d));}, // data -> display
            xAxis = d3.axisBottom().scale(xScale);

        // setup y
        var yValue = function(d) { return d[fieldYAxis];}, // data -> value
            yScale = d3.scaleLinear().range([500/2-100, 0]), // value -> display
            yMap = function(d) { return yScale(yValue(d));}, // data -> display
            yAxis = d3.axisLeft().scale(yScale);

        // don't want dots overlapping axis, so add in buffer to data domain
        xScale.domain([6.3, 12]);
        yScale.domain([2.2, 8]);

        scatterplot.append("text")
            .attr("x", 10)
            .attr("y", 0 - (margin.top / 2))
            .style("font-size", "10px")
            .style("text-decoration", "underline")
            .text("The Happiness Score vs Logged GDP per Capita");
        // x-axis
        scatterplot.append("g")
            .attr("class", "x axis")
            .attr("transform", "translate(0," + (500/2-100) + ")")
            .call(xAxis)
            .append("text")
            .attr("class", "label")
            .attr("x", xScale(8))
            .attr("y", -6)
            .style("text-anchor", "end")
            .text(Logged.replace(/_/g, " "));

        // y-axis
        scatterplot.append("g")
            .attr("class", "y axis")
            .call(yAxis)
            .append("text")
            .attr("class", "label")
            .attr("transform", "rotate(-90)")
            .attr("x", 0)
            .attr("y", yScale(100))
            .attr("dy", "1.5em")
            .style("text-anchor", "end")
            .text(fieldYAxis.replace(/_/g, " "));

        // draw dots
        scatterplot.selectAll(".dot")
            .data(happiness)
            .enter().append("circle")
            .attr("class", d => { return "dot COUNTRY-"+d.Country; } )
            .attr("r", function(d) { return size(d[fieldSize]);})
            .attr("cx", xMap)
            .attr("cy", yMap)
            .style("fill", function(d) { return color(d[fieldColor]);})
            .attr('stroke', function(d) { return color(d[fieldColor]);})
            .attr('stroke-width',2)
            .attr("fill-opacity", .6)
            .on("mouseover", function(d) {
                d3.select(this)
                    .transition()
                    .duration(500)
                    .attr('r',35)
                d3.select(this)
                    .append("title")
                    .attr('country', function(d){return d.Country})
                    .text(function(d){return d.Country; })
                d3.selectAll("")
            })
            .on("mouseout", function(d) {
                d3.select(".dot")
                    .transition()
                    .duration(500)
                    .attr("r", function(d) { return size(d[fieldSize])})
                d3.select(this)
                    .transition()
                    .duration(500)
                    .attr("r", function(d) { return size(d[fieldSize]);})
            });


        // setup x
        var xValue2 = function(d) { return d[Social];}, // data -> value
            xScale2 = d3.scaleLinear().range([0, 2000/2-150]), // value -> display
            xMap2 = function(d) { return xScale2(xValue2(d));}, // data -> display
            xAxis2 = d3.axisBottom().scale(xScale2);

        // don't want dots overlapping axis, so add in buffer to data domain
        xScale2.domain([0.3, 1.0]);

        scatterplot2.append("text")
            .attr("x", 10)
            .attr("y", 0 - (margin.top / 2))
            .style("font-size", "10px")
            .style("text-decoration", "underline")
            .text("The Happiness Score vs Social Support");

        // x-axis
        scatterplot2.append("g")
            .attr("class", "x axis2")
            .attr("transform", "translate(0," + (500/2-100) + ")")
            .call(xAxis2)
            .append("text")
            .attr("class", "label")
            .attr("x", xScale2(10))
            .attr("y", -6)
            .style("text-anchor", "end")
            .text(Logged.replace(/_/g, " "));

        // y-axis
        scatterplot2.append("g")
            .attr("class", "y axis")
            .call(yAxis)
            .append("text")
            .attr("class", "label")
            .attr("transform", "rotate(-90)")
            .attr("x", 0)
            .attr("y", yScale(100))
            .attr("dy", "1.5em")
            .style("text-anchor", "end")
            .text(fieldYAxis.replace(/_/g, " "));

        // draw dots
        scatterplot2.selectAll(".dot")
            .data(happiness)
            .enter().append("circle")
            .attr("class", d => { return "dot COUNTRY-"+d.Country; } )
            .attr("r", function(d) { return size(d[fieldSize]);})
            .attr("cx", xMap2)
            .attr("cy", yMap)
            .style("fill", function(d) { return color(d[fieldColor]);})
            .attr('stroke', function(d) { return color(d[fieldColor]);})
            .attr('stroke-width',2)
            .attr("fill-opacity", .6)
            .on("mouseover", function(d) {
                d3.select(this)
                    .transition()
                    .duration(500)
                    .attr('r',35)
                d3.select(this)
                    .append("title")
                    .attr('country', function(d){return d.Country})
                    .text(function(d){return d.Country; });
            })
            .on("mouseout", function(d) {
                d3.select(this)
                    .transition()
                    .duration(500)
                    .attr("r", function(d) { return size(d[fieldSize]);})
            });


        // setup x
        var xValue3 = function(d) { return d[Healthy];}, // data -> value
            xScale3 = d3.scaleLinear().range([0, 2000/2-150]), // value -> display
            xMap3 = function(d) { return xScale3(xValue3(d));}, // data -> display
            xAxis3 = d3.axisBottom().scale(xScale3);

        // don't want dots overlapping axis, so add in buffer to data domain
        xScale3.domain([45, 80]);

        scatterplot3.append("text")
            .attr("x", 10)
            .attr("y", 0 - (margin.top / 2))
            .style("font-size", "10px")
            .style("text-decoration", "underline")
            .text("The Happiness Score vs Healthy life expectancy");

        // x-axis
        scatterplot3.append("g")
            .attr("class", "x axis3")
            .attr("transform", "translate(0," + (500/2-100) + ")")
            .call(xAxis3)
            .append("text")
            .attr("class", "label")
            .attr("x", xScale3(10))
            .attr("y", -6)
            .style("text-anchor", "end")
            .text(Logged.replace(/_/g, " "));

        // y-axis
        scatterplot3.append("g")
            .attr("class", "y axis")
            .call(yAxis)
            .append("text")
            .attr("class", "label")
            .attr("transform", "rotate(-90)")
            .attr("x", 0)
            .attr("y", yScale(100))
            .attr("dy", "1.5em")
            .style("text-anchor", "end")
            .text(fieldYAxis.replace(/_/g, " "));

        // draw dots
        scatterplot3.selectAll(".dot")
            .data(happiness)
            .enter().append("circle")
            .attr("class", d => { return "dot COUNTRY-"+d.Country; } )
            .attr("r", function(d) { return size(d[fieldSize]);})
            .attr("cx", xMap3)
            .attr("cy", yMap)
            .style("fill", function(d) { return color(d[fieldColor]);})
            .attr('stroke', function(d) { return color(d[fieldColor]);})
            .attr('stroke-width',2)
            .attr("fill-opacity", .6)
            .on("mouseover", function(d) {
                d3.select(this)
                    .transition()
                    .duration(500)
                    .attr('r',35)
                d3.select(this)
                    .append("title")
                    .attr('country', function(d){return d.Country})
                    .text(function(d){return d.Country; });
            })
            .on("mouseout", function(d) {
                d3.select(this)
                    .transition()
                    .duration(500)
                    .attr("r", function(d) { return size(d[fieldSize]);})
            });

        // setup x
        var xValue4 = function(d) { return d[Freedom];}, // data -> value
            xScale4 = d3.scaleLinear().range([0, 2000/2-150]), // value -> display
            xMap4 = function(d) { return xScale4(xValue4(d));}, // data -> display
            xAxis4 = d3.axisBottom().scale(xScale4);

        // don't want dots overlapping axis, so add in buffer to data domain
        xScale4.domain([0.35, 1]);

        scatterplot4.append("text")
            .attr("x", 10)
            .attr("y", 0 - (margin.top / 2))
            .style("font-size", "10px")
            .style("text-decoration", "underline")
            .text("The Happiness Score vs Freedom to make life choices");

        // x-axis
        scatterplot4.append("g")
            .attr("class", "x axis4")
            .attr("transform", "translate(0," + (500/2-100) + ")")
            .call(xAxis4)
            .append("text")
            .attr("class", "label")
            .attr("x", xScale4(10))
            .attr("y", -6)
            .style("text-anchor", "end")
            .text(Logged.replace(/_/g, " "));

        // y-axis
        scatterplot4.append("g")
            .attr("class", "y axis")
            .call(yAxis)
            .append("text")
            .attr("class", "label")
            .attr("transform", "rotate(-90)")
            .attr("x", 0)
            .attr("y", yScale(100))
            .attr("dy", "1.5em")
            .style("text-anchor", "end")
            .text(fieldYAxis.replace(/_/g, " "));

        // draw dots
        scatterplot4.selectAll(".dot")
            .data(happiness)
            .enter().append("circle")
            .attr("class", d => { return "dot COUNTRY-"+d.Country; } )
            .attr("r", function(d) { return size(d[fieldSize]);})
            .attr("cx", xMap4)
            .attr("cy", yMap)
            .style("fill", function(d) { return color(d[fieldColor]);})
            .attr('stroke', function(d) { return color(d[fieldColor]);})
            .attr('stroke-width',2)
            .attr("fill-opacity", .6)
            .on("mouseover", function(d) {
                d3.select(this)
                    .transition()
                    .duration(500)
                    .attr('r',35)
                d3.select(this)
                    .append("title")
                    .attr('country', function(d){return d.Country})
                    .text(function(d){return d.Country; });
            })
            .on("mouseout", function(d) {
                d3.select(this)
                    .transition()
                    .duration(500)
                    .attr("r", function(d) { return size(d[fieldSize]);})
            });



        // setup x
        var xValue5 = function(d) { return d[Perceptions];}, // data -> value
            xScale5 = d3.scaleLinear().range([0, 2000/2-150]), // value -> display
            xMap5 = function(d) { return xScale5(xValue5(d));}, // data -> display
            xAxis5 = d3.axisBottom().scale(xScale5);

        // don't want dots overlapping axis, so add in buffer to data domain
        xScale5.domain([0.1, 1]);

        scatterplot5.append("text")
            .attr("x", 10)
            .attr("y", 0 - (margin.top / 2))
            .style("font-size", "10px")
            .style("text-decoration", "underline")
            .text("The Happiness Score vs Perceptions of Corruption");

        // x-axis
        scatterplot5.append("g")
            .attr("class", "x axis4")
            .attr("transform", "translate(0," + (500/2-100) + ")")
            .call(xAxis5)
            .append("text")
            .attr("class", "label")
            .attr("x", xScale4(10))
            .attr("y", -6)
            .style("text-anchor", "end")
            .text(Logged.replace(/_/g, " "));

        // y-axis
        scatterplot5.append("g")
            .attr("class", "y axis")
            .call(yAxis)
            .append("text")
            .attr("class", "label")
            .attr("transform", "rotate(-90)")
            .attr("x", 0)
            .attr("y", yScale(100))
            .attr("dy", "1.5em")
            .style("text-anchor", "end")
            .text(fieldYAxis.replace(/_/g, " "));

        // draw dots
        scatterplot5.selectAll(".dot")
            .data(happiness)
            .enter().append("circle")
            .attr("class", d => { return "dot COUNTRY-"+d.Country; } )
            .attr("r", function(d) { return size(d[fieldSize]);})
            .attr("cx", xMap5)
            .attr("cy", yMap)
            .style("fill", function(d) { return color(d[fieldColor]);})
            .attr('stroke', function(d) { return color(d[fieldColor]);})
            .attr('stroke-width',2)
            .attr("fill-opacity", .6)
            .on("mouseover", function(d) {
                d3.select(this)
                    .transition()
                    .duration(500)
                    .attr('r',35)
                d3.select(this)
                    .append("title")
                    .attr('country', function(d){return d.Country})
                    .text(function(d){return d.Country; });
            })
            .on("mouseout", function(d) {
                d3.select(this)
                    .transition()
                    .duration(500)
                    .attr("r", function(d) { return size(d[fieldSize]);})
            });


        // draw legend
        var legend = scatterplot.append("g").attr("class", "legend-group").selectAll(".legend")
            .data(color.domain())
            .enter().append("g")
            .attr("class", "legend")
            .attr("transform", function(d, i) { return "translate(-100," + (i+1) * 20 + ")"; });

        // draw legend colored rectangles
        legend.append("rect")
            .attr("x", width/1.05 + 4)
            .attr("width", 18)
            .attr("height", 18)
            .style("fill", (d,i)=> color(d-0.0001));

        // draw legend text
        legend.append("text")
            .attr("x", width/1.0)
            .attr("y", 9)
            .attr("dy", ".35em")
            .style("text-anchor", "end")
            .text(function(d) { return "< "+d;});

        scatterplot.select("g.legend-group")
            .append("g")
            .attr("class", "legend")
            .attr("transform", "translate(-100,0)")
            .append("text")
            .attr("x", width/1.03+28)
            .attr("y", 0)
            .attr("dy", "1.5em")
            .style("text-anchor", "end")
            .text(fieldColor);
    });


</script>
</body>
</html>
