```js
const width = 100%;
const height = auto;
const color = d3.scaleOrdinal(d3.schemeObservable10);

// Copy the data to protect against mutation by d3.forceSimulation.
const links = data.links.map((d) => Object.create(d));
const nodes = data.nodes.map((d) => Object.create(d));

const forceX = d3.forceX(width / 2).strength(0.2)
const forceY = d3.forceY(height / 2).strength(0.2)

const simulation = d3.forceSimulation(nodes)
    .force("link", d3.forceLink(links).id((d) => d.id))
    .force("charge", d3.forceManyBody())
    .force("center", d3.forceCenter(width / 2, height / 2))
    .force('x', forceX)
  .force('y',  forceY)
    .on("tick", ticked);

const svg = d3.create("svg")
    .attr("width", width)
    .attr("height", height)
    .attr("viewBox", [0, 0, width, height])
    .attr("style", "max-width: 100%; height: auto;");

const link = svg.append("g")
    .attr("stroke", "var(--theme-foreground-faint)")
    .attr("stroke-opacity", 0.6)
  .selectAll("line")
  .data(links)
  .join("line")
    .attr("stroke-width", (d) => Math.sqrt(d.value));

const node = svg.append("g")
    .attr("stroke", "var(--theme-background)")
    .attr("stroke-width", 1.5)
  .selectAll("circle")
  .data(nodes)
  .join("circle")
    .attr("r", (d) => ("Obras" == d.group) ? 5 : 8)
    .attr("fill", (d) => color(d.group))
    .call(drag(simulation));

node.append("title")
    .text((d) => d.title);

function ticked() {
  link
      .attr("x1", (d) => d.source.x)
      .attr("y1", (d) => d.source.y)
      .attr("x2", (d) => d.target.x)
      .attr("y2", (d) => d.target.y);

  node
      .attr("transform", function (d) {
        checkBounds(d);
        return "translate(" + d.x + ", " + d.y + ")";
      })
      .on('dblclick', connectedNodes);
      // .attr("cx", (d) => d.x)
      // .attr("cy", (d) => d.y);
}

function checkBounds(d){
  if (d.x < 0) d.x = 0;
  if (d.x > width) d.x = width;
  if (d.y < 0) d.y = 0;
  if (d.y > width) d.y = height;
}

//Toggle stores whether the highlighting is on
var toggle = 0;

//Create an array logging what is connected to what
var linkedByIndex = {};
for (var i = 0; i < nodes.length; i++) {
    linkedByIndex[i + "," + i] = 1;
};
links.forEach(function (d) {
    linkedByIndex[d.source.index + "," + d.target.index] = 1;
});

//This function looks up whether a pair are neighbours
function neighboring(a, b) {
    return linkedByIndex[a.index + "," + b.index];
}

function connectedNodes() {

    if (toggle == 0) {
        //Reduce the opacity of all but the neighbouring nodes
        var d = d3.select(this).node().__data__;
        node.style("opacity", function (o) {
            return neighboring(d, o) | neighboring(o, d) ? 1 : 0.1;
        });

        link.style("opacity", function (o) {
            return d.index==o.source.index | d.index==o.target.index ? 1 : 0.1;
        });

        //Reduce the op

        toggle = 1;
    } else {
        //Put them back to opacity=1
        node.style("opacity", 1);
        link.style("opacity", 1);
        toggle = 0;
    }

}

display(svg.node());
```

```js
function drag(simulation) {
  function dragstarted(event) {
    if (!event.active) simulation.alphaTarget(0.3).restart();
    event.subject.fx = event.subject.x;
    event.subject.fy = event.subject.y;
  }

  function dragged(event) {
    event.subject.fx = event.x;
    event.subject.fy = event.y;
  }

  function dragended(event) {
    if (!event.active) simulation.alphaTarget(0);
    event.subject.fx = null;
    event.subject.fy = null;
  }

  return d3
    .drag()
    .on("start", dragstarted)
    .on("drag", dragged)
    .on("end", dragended);
}
```

```js
const data = FileAttachment("miserables.json").json();
```
