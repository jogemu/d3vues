# d3vues

Reactive D3.js visualizations encoded in declarative HTML without any build steps. Decluttered examples without compromising flexibility. Copy into static websites or learn advanced D3 with ease.

```html
<!-- Pie chart -->
<svg v-scope="{
  data: Array.from({ length: 4 }, Math.random),
  radius: Math.min($el.clientWidth, $el.clientHeight)/2 }"
  :viewBox="[-radius, -radius, 2*radius, 2*radius].join(' ')">
  <path v-for="d, i in d3.pie().padAngle(.015)(data)"
    :d="d3.arc().innerRadius(1).outerRadius(radius)(d)"
    :fill="d3.schemeSet2[i%8]"></path>
</svg>
<script src="https://cdn.jsdelivr.net/npm/d3"></script>
<script src="https://cdn.jsdelivr.net/npm/@jogemu/petite-vue" defer init></script>
```

Fetch external data asynchronously with ease. Visualizations update automatically on data changes.

```html
<svg v-scope="{
  json: d3.json('path/to.json'),
  csv: d3.csv('path/to.csv') }">
  ...
  <circle r="10" @click="json.push(4)"></circle>
</svg>
```

Statistical analysis and axes can be placed inline and reused throughout the entire scope.

```html
<svg v-scope="{ 
  points: Array.from({ length: 66 }, ()=>Array.from({ length: 2 }, Math.random)),
  x: d3.scaleLinear().range([0, $el.clientWidth-60]),
  y: d3.scaleLinear().range([0, -$el.clientHeight+40]),
  bins: (points, axis) => d3.bin().value(p=>p[axis])(points).map(bin => {
    bin.v = d3.mean([bin.x0, bin.x1]);
    bin.vs = bin.map(p=>p[(axis+1)%2]);
    bin.extent = d3.extent(bin.vs);
    bin.q1 = d3.quantile(bin.vs, .25);
    bin.q2 = d3.quantile(bin.vs, .50);
    bin.q3 = d3.quantile(bin.vs, .75);
    bin.iqr = bin.q3 - bin.q1;
    bin.r0 = Math.max(bin.extent[0], bin.q1 - bin.iqr * 1.5);
    bin.r1 = Math.min(bin.extent[1], bin.q3 + bin.iqr * 1.5);
    bin.outliers = bin.vs.filter(v => v < bin.r0 || v > bin.r1);
    return bin; }) }"
  :viewBox="[-40, 30-$el.clientHeight, $el.clientWidth, $el.clientHeight].join(' ')">
  <g v-effect="d3.select($el).call(d3.axisBottom(x=x.domain(d3.extent(points, p=>p[0])).nice()));x.bandwidth??=()=>8"></g>
  <g v-effect="d3.select($el).call(d3.axisLeft(y=y.domain(d3.extent(points, p=>p[1])).nice()))"></g>

  <!-- Scatter plot -->
  <g :fill="d3.schemeSet2[0]"><circle v-for="[cx, cy] in points" :cx="x(cx)" :cy="y(cy)" r="2"></circle></g>

  <!-- Line plot -->
  <path :d="d3.line().curve(d3.curveCatmullRom).x(d=>x(d.v)).y(d=>y(d.q1))(bins(points, 0))"
    :stroke="d3.schemeSet2[1]" fill="none"></path>

  <!-- Area plot -->
  <path :d="d3.area().curve(d3.curveCatmullRom).x(d=>x(d.v)).y0(d=>y(d.q1)).y1(d=>y(d.q3))(bins(points, 0))"
    :fill="d3.schemeSet2[2]"></path>

  <!-- Bar plot -->
  <g :stroke="d3.schemeSet2[3]" :stroke-width="x.bandwidth?.()"><line v-for="d in bins(points, 0)"
    :y1="y(0)" :y2="y(d.q1)" :x1="x(d.v)" :x2="x(d.v)"></line></g>

  <!-- Boxplot -->
  <g stroke="black" :stroke-width="x.bandwidth?.()"><g v-for="bin in bins(points, 0)"
    :transform="`translate(${x(bin.v)})`">
    <line :y1="y(bin.r0)" :y2="y(bin.r1)" stroke-width="1"></line>
    <line :y1="y(bin.q1)" :y2="y(bin.q3)" stroke="gray"></line>
    <line :y1="y(bin.q2)-1/2" :y2="y(bin.q2)+1/2"></line>
    <circle v-for="v in bin.outliers" :cy="y(v)" r="1" stroke="none"></circle>
  </g></g>
</svg>
<script src="https://cdn.jsdelivr.net/npm/d3"></script>
<script src="https://cdn.jsdelivr.net/npm/@jogemu/petite-vue" defer init></script>
```

Check the end of the document for [horizontal](#horizontal-plots) and [radar](#radarspider-plot) variations of the data above.

## Hierarchies: Treemap, Icicle, Sunburst, Circle packing

```html
<!-- Treemap -->
<svg v-scope="{
  data: { children: [...Array(4)].map(()=>({ children: [...Array(4)].map(Math.random) }) ) },
  treemap: d3.treemap().size([$el.clientWidth, $el.clientHeight]).padding(2) }"
  :viewBox="[-0, 0, $el.clientWidth, $el.clientHeight].join(' ')">
  <rect v-for="d, i in treemap(d3.hierarchy(data).sum(d=>d)).descendants()"
    :x="d.x0" :y="d.y0" :width="d.x1-d.x0" :height="d.y1-d.y0"
    :fill="d3.schemeSet2[i%8]"></rect>
</svg>
<script src="https://cdn.jsdelivr.net/npm/d3"></script>
<script src="https://cdn.jsdelivr.net/npm/@jogemu/petite-vue" defer init></script>
```

```html
<!-- Icicle -->
<svg v-scope="{
  data: { children: [...Array(4)].map(()=>({ children: [...Array(4)].map(Math.random) }) ) },
  icicle: d3.partition().size([$el.clientHeight, $el.clientWidth]).padding(2) }"
  :viewBox="[-0, 0, $el.clientWidth, $el.clientHeight].join(' ')">
  <rect v-for="d, i in icicle(d3.hierarchy(data).sum(d=>d)).descendants()"
    :x="d.y0" :y="d.x0" :width="d.y1-d.y0" :height="d.x1-d.x0"
    :fill="d3.schemeSet2[i%8]"></rect>
</svg>
<script src="https://cdn.jsdelivr.net/npm/d3"></script>
<script src="https://cdn.jsdelivr.net/npm/@jogemu/petite-vue" defer init></script>
```

```html
<!-- Sunburst -->
<svg v-scope="{
  data: { children: [...Array(4)].map(()=>({ children: [...Array(4)].map(Math.random) }) ) },
  radius: Math.min($el.clientWidth, $el.clientHeight)/2,
  arc: d3.arc().startAngle(d => d.x0).endAngle(d => d.x1).innerRadius(d => d.y0).outerRadius(d => d.y1-2).padAngle(.015) }"
  :viewBox="[-radius, -radius, 2*radius, 2*radius].join(' ')"><g v-scope="{
  sunburst: d3.partition().size([2 * Math.PI, radius]) }">
  <path v-for="d, i in sunburst(d3.hierarchy(data).sum(d=>d)).descendants()"
    :d="arc(d)"
    :fill="d3.schemeSet2[i%8]"></path>
</g></svg>
<script src="https://cdn.jsdelivr.net/npm/d3"></script>
<script src="https://cdn.jsdelivr.net/npm/@jogemu/petite-vue" defer init></script>
```

```html
<!-- Circle packing -->
<svg v-scope="{
  data: { children: [...Array(4)].map(()=>({ children: [...Array(4)].map(Math.random) }) ) },
  pack: d3.pack().size([$el.clientHeight, $el.clientWidth]).padding(2) }"
  :viewBox="[0, $el.clientHeight/2, $el.clientWidth, $el.clientHeight].join(' ')">
  <circle v-for="d, i in pack(d3.hierarchy(data).sum(d=>d)).descendants()"
    :cx="d.x" :cy="d.y" :r="d.r"
    :fill="d3.schemeSet2[i%8]"></circle>
</svg>
<script src="https://cdn.jsdelivr.net/npm/d3"></script>
<script src="https://cdn.jsdelivr.net/npm/@jogemu/petite-vue" defer init></script>
```

## Networks

```html
<!-- Chord -->
<svg v-scope="{
  data: Array.from({ length: 4 }, ()=>Array.from({ length: 4 }, Math.random)),
  radius: Math.min($el.clientWidth, $el.clientHeight)/2,
  chords: d3.chord().padAngle(.015) }"
  :viewBox="[-radius, -radius, 2*radius, 2*radius].join(' ')"><g v-scope="{
  arc: d3.arc().innerRadius(radius*.8).outerRadius(radius),
  ribbon: d3.ribbonArrow().radius(radius*.75) }">
  <path v-for="group in chords(data).groups"
    :d="arc(group)"
    :fill="d3.schemeSet2[group.index%8]"></path>
  <path v-for="chord in chords(data)"
    :d="ribbon(chord)"
    :fill="d3.schemeSet2[chord.target.index%8]"></path>
</g></svg>
<script src="https://cdn.jsdelivr.net/npm/d3"></script>
<script src="https://cdn.jsdelivr.net/npm/@jogemu/petite-vue" defer init></script>
```

```html
<!-- Force-directed graph -->
<svg v-scope="{
  nodes: Array.from({ length: 66 }, (v,i)=>({ index: i })),
  links: Array.from({ length: 65 }, (v,i)=>({ source: i+1, target: d3.randomInt(i)() })) }"
  :viewBox="[-$el.clientWidth/2, -$el.clientHeight/2, $el.clientWidth, $el.clientHeight].join(' ')"><g v-scope="{
  simulation: d3.forceSimulation(nodes)
    .force('link', d3.forceLink(links))
    .force('charge', d3.forceManyBody())
    .force('x', d3.forceX())
    .force('y', d3.forceY()) }">
  <line v-for="link in links" stroke="gray"
    :x1="link.source.x" :x2="link.target.x"
    :y1="link.source.y" :y2="link.target.y"></line>
  <circle v-for="node in nodes"
    :cx="node.x" :cy="node.y" r="2"></circle>
</g></svg>
<script src="https://cdn.jsdelivr.net/npm/d3"></script>
<script src="https://cdn.jsdelivr.net/npm/@jogemu/petite-vue" defer init></script>
```

<details><summary>Force-directed graph interaction</summary>

```html
<!-- Move node and restart simulation -->
<circle v-for="node in nodes"
  @click="node.x=0;simulation.alpha(1).restart()"></circle>
```

</details>

```html
<!-- Arc diagram -->
<svg v-scope="{
  nodes: Array.from({ length: 66 }, (v,i)=>({ index: i })),
  links: Array.from({ length: 65 }, (v,i)=>({ source: i+1, target: d3.randomInt(i)() })),
  y: d3.scalePoint().range([0, $el.clientHeight-20]) }"
  :viewBox="[-40, -10, $el.clientWidth, $el.clientHeight].join(' ')">
  <g v-effect="d3.select($el).call(d3.axisLeft(y=y.domain(nodes.map(d=>d.index))))"></g>
  <path v-for="link in links"
    :d="`M 0 ${y(link.source)} A 1 1 0 0 ${+(link.target>link.source)} 0 ${y(link.target)}`"
    :stroke="d3.schemeSet2[link.target%8]" fill="none"></path>
  <circle v-for="node in nodes"
    :cy="y(node.index)" r="2"
    :fill="d3.schemeSet2[node.index%8]"></circle>
</svg>
<script src="https://cdn.jsdelivr.net/npm/d3"></script>
<script src="https://cdn.jsdelivr.net/npm/@jogemu/petite-vue" defer init></script>
```

```html
<!-- Sankey diagram -->
<svg v-scope="{
  nodes: Array.from({ length: 32 }, (v,i)=>({ index: i })),
  links: Array.from({ length: 31 }, (v,i)=>({ source: i+1, target: d3.randomInt(i)(), value: Math.random() })),
  sankey: d3.sankey().extent([[0, 0], [$el.clientWidth, $el.clientHeight]]),
  linkH: d3.sankeyLinkHorizontal() }"
  :viewBox="[0, 0, $el.clientWidth, $el.clientHeight].join(' ')">
  <g stroke="gray" stroke-opacity=".5" fill="none"><path v-for="link in sankey({nodes, links}).links"
    :d="linkH(link)"
    :stroke-width="link.width"></path></g>
  <rect v-for="node in nodes"
    :x="node.x0" :y="node.y0"
    :height="node.y1-node.y0" :width="node.x1-node.x0"
    :fill="d3.schemeSet2[node.index%8]"></rect>
</svg>
<script src="https://cdn.jsdelivr.net/npm/d3"></script>
<script src="https://cdn.jsdelivr.net/npm/d3-sankey"></script>
<script src="https://cdn.jsdelivr.net/npm/@jogemu/petite-vue" defer init></script>
```

## Maps

GeoJSON types can be added to any data that isn't GeoJSON already. Try [azimuthal](https://d3js.org/d3-geo/azimuthal), [conic](https://d3js.org/d3-geo/conic) and [cylindrical](https://d3js.org/d3-geo/cylindrical) projections.

```html
<svg v-scope="{
  points: Array.from({ length: 66 }, ()=>Array.from({ length: 2 }, d3.randomUniform(-90, 90))),
  geoPath: d3.geoPath(d3.geoMercator().translate([0, 0]).scale($el.clientHeight/8)) }"
  :viewBox="[-$el.clientWidth/2, -$el.clientHeight/2, $el.clientWidth, $el.clientHeight].join(' ')">
  <path :d="geoPath({type: 'Sphere'})" :fill="d3.schemeSet2[2]"></path>
  <path :d="geoPath(d3.geoGraticule10())" stroke="gray" fill="none"></path>
  <g :fill="d3.schemeSet2[1]"><path v-for="point in points"
    :d="geoPath({ type: 'Point', coordinates: point })"></path></g>
  <path :stroke="d3.schemeSet2[0]" fill="none"
    :d="geoPath({ type: 'LineString', coordinates: points })"></path>
  <!-- Geodetic circle corrects map distortion to show true distance -->
  <path :fill="d3.schemeSet2[3]"
    :d="geoPath(d3.geoCircle().center([-90, 60]).radius(25)())"></path>
</svg>
<script src="https://cdn.jsdelivr.net/npm/d3"></script>
<script src="https://cdn.jsdelivr.net/npm/@jogemu/petite-vue" defer init></script>
```

## Analysis: Delaunay, Voronoi, Quadtree, Contours

```html
<svg v-scope="{
  points: Array.from({ length: 66 }, ()=>Array.from({ length: 2 }, Math.random)),
  x: d3.scaleLinear().range([0, $el.clientWidth-60]),
  y: d3.scaleLinear().range([0, -$el.clientHeight+40]),
  geoPath: d3.geoPath(d3.geoIdentity().clipExtent([[0, 0], [$el.clientWidth-60, $el.clientHeight-40]])) }"
  :viewBox="[-40, 30-$el.clientHeight, $el.clientWidth, $el.clientHeight].join(' ')">
  <g v-effect="d3.select($el).call(d3.axisBottom(x=x.domain(d3.extent(points, p=>p[0])).nice()))"></g>
  <g v-effect="d3.select($el).call(d3.axisLeft(y=y.domain(d3.extent(points, p=>p[1])).nice()))"></g>

  <!-- Delaunay & Voronoi -->
  <g v-for="delaunay in [d3.Delaunay.from(points)]" v-scope="{
    line: d3.line().x(p=>x(p[0])).y(p=>y(p[1])) }"
    fill="none" stroke="black">
    <path v-for="triangle in Array.from(delaunay.trianglePolygons())"
      :d="line(triangle)" ></path>
    <path :d="line(delaunay.hullPolygon())"></path>
    <path v-for="cell in Array.from(delaunay.voronoi(d3.zip(x.domain(), y.domain()).flat()).cellPolygons())"
      :d="line(cell)"></path>
  </g>

  <!-- Quadtree -->
  <g v-scope="{ rects: [] }" v-effect="rects = [];
    d3.quadtree(points).visit((node, x1, y1, x2, y2) => !rects.push({x1, y1, x2, y2}))">
    <rect v-for="rect in rects"
      :x="x(rect.x1)" :y="y(rect.y2)"
      :width="x(rect.x2)-x(rect.x1)" :height="y(rect.y1)-y(rect.y2)"
      fill="none" stroke="black"></rect>
  </g>

  <!-- Contours -->
  <g fill="none" stroke="black" transform="scale(1 -1)">
    <path v-for="contour in d3.contourDensity()(points.map(p=>[x(p[0]), -y(p[1])]))"
      :d="geoPath(contour)"></path>
  </g>

  <!-- Scatter plot -->
  <circle v-for="[cx, cy] in points" :cx="x(cx)" :cy="y(cy)" r="2"></circle>
</svg>
<script src="https://cdn.jsdelivr.net/npm/d3"></script>
<script src="https://cdn.jsdelivr.net/npm/@jogemu/petite-vue" defer init></script>
```

```html
<!-- Heatmap -->
<svg v-scope="{
  points: Array.from({ length: 66 }, ()=>Array.from({ length: 2 }, Math.random)),
  x: d3.scaleLinear().range([0, $el.clientWidth-60]),
  y: d3.scaleLinear().range([0, -$el.clientHeight+40]),
  color: d3.scaleQuantize().range(d3.schemePurples[7]),
  binX: d3.bin().value(p=>p[0]),
  binY: d3.bin().value(p=>p[1]),
  int: ([bx, by]) => ({ x0: bx.x0, x1: bx.x1, y0: by.x0, y1: by.x1, size: d3.intersection(bx, by).size }) }"
  :viewBox="[-40, 30-$el.clientHeight, $el.clientWidth, $el.clientHeight].join(' ')">
  <g v-effect="d3.select($el).call(d3.axisBottom(x=x.domain(d3.extent(points, p=>p[0])).nice()))"></g>
  <g v-effect="d3.select($el).call(d3.axisLeft(y=y.domain(d3.extent(points, p=>p[1])).nice()))"></g>
  <g v-effect="color=color.domain([0, d3.max(int.cache, d=>d.size)])"></g>

  <rect v-for="bin in int.cache=d3.cross(binX(points), binY(points)).map(int)"
    :x="x(bin.x0)" :width="x(bin.x1)-x(bin.x0)"
    :y="y(bin.y1)" :height="y(bin.y0)-y(bin.y1)"
    :fill="color(bin.size)"></rect>

  <!-- Scatter plot -->
  <circle v-for="[cx, cy] in points" :cx="x(cx)" :cy="y(cy)" r="2"></circle>
</svg>
<script src="https://cdn.jsdelivr.net/npm/d3"></script>
<script src="https://cdn.jsdelivr.net/npm/@jogemu/petite-vue" defer init></script>
```

```html
<!-- Hexbin -->
<svg v-scope="{
  points: Array.from({ length: 66 }, ()=>Array.from({ length: 2 }, Math.random)),
  x: d3.scaleLinear().range([0, $el.clientWidth-60]),
  y: d3.scaleLinear().range([0, -$el.clientHeight+40]),
  color: d3.scaleQuantize().range(d3.schemePurples[7]),
  hexbin: d3.hexbin().radius(10) }"
  :viewBox="[-40, 30-$el.clientHeight, $el.clientWidth, $el.clientHeight].join(' ')">
  <g v-effect="d3.select($el).call(d3.axisBottom(x=x.domain(d3.extent(points, p=>p[0])).nice()))"></g>
  <g v-effect="d3.select($el).call(d3.axisLeft(y=y.domain(d3.extent(points, p=>p[1])).nice()))"></g>
  <g v-effect="color=color.domain([0, d3.max(hexbin.cache, d=>d.length)])"></g>
  
  <path v-for="d in hexbin.cache=hexbin.x(d=>x(d[0])).y(d=>y(d[1]))(points)"
    :d="hexbin.hexagon()"
    :transform="`translate(${d.x},${d.y})`"
    :fill="color(d.length)"></path>
  
  <!-- Scatter plot -->
  <circle v-for="[cx, cy] in points" :cx="x(cx)" :cy="y(cy)" r="2"></circle>
</svg>
<script src="https://cdn.jsdelivr.net/npm/d3"></script>
<script src="https://cdn.jsdelivr.net/npm/d3-hexbin"></script>
<script src="https://cdn.jsdelivr.net/npm/@jogemu/petite-vue" defer init></script>
```

## Horizontal plots

```html
<svg v-scope="{ 
  points: Array.from({ length: 66 }, ()=>Array.from({ length: 2 }, Math.random)), x: i=>i, y: i=>i,
  bins: (points, axis) => d3.bin().value(p=>p[axis])(points).map(bin => {
    bin.v = d3.mean([bin.x0, bin.x1]);
    bin.vs = bin.map(p=>p[(axis+1)%2]);
    bin.extent = d3.extent(bin.vs);
    bin.q1 = d3.quantile(bin.vs, .25);
    bin.q2 = d3.quantile(bin.vs, .50);
    bin.q3 = d3.quantile(bin.vs, .75);
    bin.iqr = bin.q3 - bin.q1;
    bin.r0 = Math.max(bin.extent[0], bin.q1 - bin.iqr * 1.5);
    bin.r1 = Math.min(bin.extent[1], bin.q3 + bin.iqr * 1.5);
    bin.outliers = bin.vs.filter(v => v < bin.r0 || v > bin.r1);
    return bin; }) }"
  :viewBox="[-40, 30-$el.clientHeight, $el.clientWidth, $el.clientHeight].join(' ')">
  <g v-effect="d3.select($el).call(d3.axisBottom(x=d3.scalePow().domain(d3.extent(points, p=>p[0])).range([0, $el.parentElement.viewBox.baseVal.width-60]).nice()));x.bandwidth??=()=>8"></g>
  <g v-effect="d3.select($el).call(d3.axisLeft(y=d3.scalePow().domain(d3.extent(points, p=>p[1])).range([0, -$el.parentElement.viewBox.baseVal.height+40]).nice()))"></g>

  <!-- Scatter plot -->
  <g :fill="d3.schemeSet2[0]"><circle v-for="[cx, cy] in points" :cx="x(cx)" :cy="y(cy)" r="2"></circle></g>

  <!-- Line plot -->
  <path :d="d3.line().curve(d3.curveCatmullRom).y(d=>y(d.v)).x(d=>x(d.q1))(bins(points, 1))" :stroke="d3.schemeSet2[1]" fill="none"></path>

  <!-- Area plot -->
  <path :d="d3.area().curve(d3.curveCatmullRom).y(d=>y(d.v)).x0(d=>x(d.q1)).x1(d=>x(d.q3))(bins(points, 1))" :fill="d3.schemeSet2[2]"></path>

  <!-- Bar plot -->
  <g :stroke="d3.schemeSet2[3]" :stroke-width="x.bandwidth?.()"><line v-for="d in bins(points, 1)" :x1="x(0)" :x2="x(d.q1)" :y1="y(d.v)" :y2="y(d.v)"></line></g>

  <!-- Boxplot -->
  <g stroke="black" :stroke-width="x.bandwidth?.()"><g v-for="bin in bins(points, 1)" :transform="`translate(0 ${y(bin.v)})`">
    <line :x1="x(bin.r0)" :x2="x(bin.r1)" stroke-width="1"></line>
    <line :x1="x(bin.q1)" :x2="x(bin.q3)" stroke="gray"></line>
    <line :x1="x(bin.q2)-1/2" :x2="x(bin.q2)+1/2"></line>
    <circle v-for="v in bin.outliers" :cx="x(v)" r="1" stroke-width="1"></circle>
  </g></g>
</svg>
<script src="https://cdn.jsdelivr.net/npm/d3"></script>
<script src="https://cdn.jsdelivr.net/npm/@jogemu/petite-vue" defer init></script>
```

## Radar/Spider plot

```html
<svg v-scope="{ 
  points: Array.from({ length: 66 }, ()=>Array.from({ length: 2 }, Math.random)),
  radius: Math.min($el.clientWidth, $el.clientHeight)/2, x: i=>i, y: i=>i,
  bins: (points, axis) => d3.bin().value(p=>p[axis])(points).map(bin => {
    bin.v = d3.mean([bin.x0, bin.x1]);
    bin.vs = bin.map(p=>p[(axis+1)%2]);
    bin.extent = d3.extent(bin.vs);
    bin.q1 = d3.quantile(bin.vs, .25);
    bin.q2 = d3.quantile(bin.vs, .50);
    bin.q3 = d3.quantile(bin.vs, .75);
    bin.iqr = bin.q3 - bin.q1;
    bin.r0 = Math.max(bin.extent[0], bin.q1 - bin.iqr * 1.5);
    bin.r1 = Math.min(bin.extent[1], bin.q3 + bin.iqr * 1.5);
    bin.outliers = bin.vs.filter(v => v < bin.r0 || v > bin.r1);
    return bin; }),
  px: (a, r) => r*Math.cos(a-Math.PI/2),
  py: (a, r) => r*Math.sin(a-Math.PI/2) }"
  :viewBox="[-radius, -radius, 2*radius, 2*radius].join(' ')"><g v-scope="{
  xy: (x, y) => ({ x: px(x, y), y: py(x, y) }),
  xyc: (x, y) => ({ cx: px(x, y), cy: py(x, y) }),
  xyy: (x, y1, y2) => ({ x1: px(x, y1), y1: py(x, y1), x2: px(x, y2), y2: py(x, y2) }) }">
  <g v-effect="x=d3.scaleLinear().domain(d3.extent(points, p=>p[0])).range([0, 2*Math.PI]).nice();x.bandwidth??=()=>8"></g>
  <g v-effect="d3.select($el).call(d3.axisLeft(y=d3.scaleLinear().domain(d3.extent(points, p=>p[1])).range([10, radius-30]).nice()))"></g>
  <g font-size="10" font-family="sans-serif" text-anchor="middle" dominant-baseline ="middle">
    <circle :r="y(x.domain?.()[1])" stroke="black" fill="none"></circle>
    <circle :r="y(x.domain?.()[0])" stroke="black" fill="none"></circle>
    <g v-for="tick in x.ticks?.()">
      <line v-bind="xyy(x(tick), y(x.domain?.()[1]), y(x.domain?.()[1])+6)" stroke="black"></line>
      <text v-bind="xy(x(tick), y(x.domain?.()[1])+15)">{{tick}}</text>
    </g>
  </g>

  <!-- Scatter plot -->
  <g :fill="d3.schemeSet2[0]"><circle v-for="[cx, cy] in points" v-bind="xyc(x(cx), y(cy))"  r="2"></circle></g>

  <!-- Line plot -->
  <path :d="d3.lineRadial().curve(d3.curveCatmullRomClosed).angle(d=>x(d.v)).radius(d=>y(d.q1))(bins(points, 0))" :stroke="d3.schemeSet2[1]" fill="none"></path>

  <!-- Area plot -->
  <path :d="d3.areaRadial().curve(d3.curveCatmullRomClosed).angle(d=>x(d.v)).innerRadius(d=>y(d.q1)).outerRadius(d=>y(d.q3))(bins(points, 0))" :fill="d3.schemeSet2[2]"></path>

  <!-- Bar plot -->
  <g :fill="d3.schemeSet2[3]"><path v-for="d in bins(points, 0)" :d="d3.arc().startAngle(d=>x(d.x0)).endAngle(d=>x(d.x1)).innerRadius(d=>y(0)).outerRadius(d=>y(d.q1))(d)" :fill="d3.schemeSet2[3]"></path></g>

  <!-- Boxplot -->
  <g stroke="black" :stroke-width="x.bandwidth?.()"><g v-for="bin in bins(points, 0)">
    <line v-bind="xyy(x(bin.v), y(bin.r0), y(bin.r1))" stroke-width="1"></line>
    <line v-bind="xyy(x(bin.v), y(bin.q1), y(bin.q3))" stroke="gray"></line>
    <line v-bind="xyy(x(bin.v), y(bin.q2)-1/2, y(bin.q2)+1/2)"></line>
    <circle v-for="v in bin.outliers" v-bind="xyc(x(bin.v), y(v))" r="1" stroke="none"></circle>
  </g></g>
</g></svg>
<script src="https://cdn.jsdelivr.net/npm/d3"></script>
<script src="https://cdn.jsdelivr.net/npm/@jogemu/petite-vue" defer init></script>
```
