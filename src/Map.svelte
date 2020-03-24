<script>
  import { onMount } from "svelte"
  import { feature } from "topojson"

  import { drag } from 'd3-drag'
  import { scaleRadial } from "d3-scale"
  import { geoOrthographic, geoPath, geoProjection, geoOrthographicRaw, geoGraticule, geoCircle } from "d3-geo";
  import { geoInertiaDrag, geoInertiaDragHelper } from 'd3-inertia'
  import { select, mouse } from 'd3-selection'
  import { timer } from 'd3-timer'

  const dpi = window.devicePixelRatio || 1;
  let velocity = 0.02;

  const handleMouseEnter = () => {
    velocity = 0.005;
  };
  const handleMouseLeave = () => {
    velocity = 0.02;
  };

  const getSize = () => {
    const width = document.body.clientWidth;
    const height = document.body.clientHeight;
    return Math.min(width, 0.8 * height)
  };

  let backData = '';
  let isDragging = false;
  let canvas;
  let size = getSize();

  const colorMap = {
    Confirmed: "#B00020",
    Treatment: "#dc8c50",
    Recovered: "#90EE02",
    Deaths: "#000",
  };

  export let data;
  export let active;
  export let world;

  onMount(async function() {
    // fix dpi blur problem
    size = getSize() * dpi;

    // console.log(data.colum);
    const land = feature(world, world.objects.countries);

    const projection = geoOrthographic().rotate([-180, -20, 0]).precision(0.1)
            .fitExtent([[5, 5], [size - 5, size - 5]], {type: "Sphere"});

    const backProjection = geoProjection((a,b) => geoOrthographicRaw(-a,b))
            .clipAngle(90)
            .translate(projection.translate())
            .scale(projection.scale());

    const context = canvas.getContext('2d');

    const min = 1;
    const max = 100000;

    const radiusScale = scaleRadial().domain([0, 1, max]).range([0, 0.5, 13]);

    const svg = select('svg');


    const render = function() {
      // fix dpi blur problem
      size = getSize() * dpi;

      const path = geoPath(projection, context);
      const rotate = projection.rotate();
      const backPath = geoPath(backProjection.rotate([rotate[0]+180, -rotate[1], -rotate[2]]), context);
      context.clearRect(0, 0, size, size);

      context.beginPath(), path({type:"Sphere"}),
              context.fillStyle = '#505050', context.fill(); // fcfcfc

      context.beginPath(), backPath(land),
              context.fillStyle = '#6002EE', context.fill(); // d0ddfa

      context.font = "bold 90pt Oxanium", context.fillStyle = "#000",
              context.textAlign = "center", context.textBaseline = "middle", context.fillText('COVID-19', size/2, size/2);

      context.beginPath(), backPath(geoGraticule()()),
              context.lineWidth = .1, context.strokeStyle = '#97b3f6', context.stroke(); // 97b3f6

      context.beginPath(), path(geoGraticule()()),
              context.lineWidth = .1, context.strokeStyle = '#BB86FC', context.stroke(); // 1046c6

      context.beginPath(), path(land),
              context.globalAlpha = 0.9,
              context.fillStyle = '#D4BFF9', context.fill(), // 5c88ee
              context.lineWidth = 1, context.strokeStyle = '#7335D8', context.stroke(), // 1046c6
              context.globalAlpha = 1;

      data.forEach(data => {
        const number = parseInt(data[active])
        if (number) {
          const circle = geoCircle().center([data.Longitude, data.Latitude]).radius(radiusScale(number));
          context.beginPath(), path(circle()), context.globalAlpha = 0.7,
                  context.fillStyle = colorMap[active], context.fill(), context.globalAlpha = 1;
        }
      });


      context.beginPath(), path({type: "Sphere"}),
              context.lineWidth = .1, context.strokeStyle = '#3700B3', context.stroke(); // 1046c6
    };



    // no need to specify render() here, it's taken care of by the loop below.
    // context.canvas.inertia = geoInertiaDrag(select(context.canvas), null, projection);

    const inertia = geoInertiaDragHelper({
      projection,
      render: (rotation) => {
        projection.rotate([rotation[0], -20, 0]);
      },
      start: () => {
        isDragging = true
      },
      end: () => {
        isDragging = false
      }
    });

    svg.call(
      drag().on("start", inertia.start).on("drag", inertia.move).on("end", inertia.end)
    );

    context.canvas.inertia = inertia;


    timer(function() {
      const rotate = projection.rotate();
      rotate[0] += velocity * 20;
      projection.rotate(rotate);
      render()
    });
  });
</script>

<div class="map" class:drag={isDragging}>
  <canvas width={size} height={size} bind:this={canvas}></canvas>
  <svg width={size/dpi} height={size/dpi}>
    <circle id="placeholder" cx={size/2/dpi} cy={size/2/dpi} r={size/2/dpi} on:mouseenter={handleMouseEnter} on:mouseleave={handleMouseLeave}/>
  </svg>
</div>

<style>
.map {
  cursor: grab;
  position: relative;
  width: 100%;
  height: 80vh;
}


@media (max-width: 500px) {
  .map {
    height: 60vh;
  }
}

.map.drag {
  cursor: grabbing;
}

canvas, svg {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  max-width: 100%;
  max-height: 80vh;
}

#placeholder {
  opacity: 0;
}
</style>
