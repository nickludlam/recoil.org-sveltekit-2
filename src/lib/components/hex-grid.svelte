<script lang='js'>
  import { onMount } from "svelte";

  const TAU = 2 * Math.PI;

  function getRadianAngle(degreeValue) {
    return degreeValue * Math.PI / 180;
  }

  const main = () => {
    const ctx = document.querySelector('#drawing').getContext('2d');
    ctx.translate(1000, -300);
    ctx.rotate(getRadianAngle(90));
    
    ctx.lineJoin = "round";
    ctx.lineWidth = 30;

    drawGrid(ctx, 1, 1, 26, 26, {
      radius: 60,
      inset: 26,
      lineWidth: 0
    });
  };

  const defaultGridOptions = {
    radius: 30,
    sides: 6,
    inset: 0,
    // Context
    lineWidth: 10,
    fillStyle: 'red',
    strokeStyle: 'black',
    // Other
    randomColors: null
  };

  const lerpBetweenColors = (color1, color2, t) => {
    const [r1, g1, b1, a1] = color1;
    const [r2, g2, b2, a2] = color2;
    return [
      r1 + (r2 - r1) * t,
      g1 + (g2 - g1) * t,
      b1 + (b2 - b1) * t,
      a1 + (a2 - a1) * t
    ];
  };

  const getColorForUV = (u, v) => {
    v = 1 - v;
    const color1 = [.75, .85, 1, 1]; // top left
    const color2 = [1, 1, 1, 1]; // bottom right

    // Make t the hypothenuse of the triangle
    const t = Math.sqrt(u * u + v * v) / Math.sqrt(2);

    return lerpBetweenColors(color1, color2, t);
  };

  const drawGrid = (ctx, x, y, w, h, options = {}) => {
    const opts = { ...defaultGridOptions, ...options };
    const points = createPoly(opts);
    opts.diameter = opts.radius * 2;
    for (let gy = y; gy < y + h; gy++) {
      for (let gx = x; gx < x + w; gx++) {
        const normalisedX = gx / (x + w);
        const normalisedY = gy / (y + h);
        const color = getColorForUV(normalisedX, normalisedY);
        // convert the rgba array to a string
        ctx.fillStyle = `rgba(${color.map(x => x * 255).join(',')})`;
        ctx.strokeStyle = ctx.fillStyle;
        drawPoly(ctx, gridToPixel(gx, gy, opts), points, opts);
      }
    }
  };

  const gridToPixel = (gridX, gridY, opts) => {
    const m = gridMeasurements(opts);
    return toPoint(
      Math.floor(gridX * m.gridSpaceX),
      Math.floor(gridY * m.gridSpaceY + (gridX % 2 ? m.gridOffsetY : 0))
    );
  };

  const drawPoly = (ctx, origin, points, opts) => {
    // ctx.strokeStyle = opts.strokeStyle;
    ctx.save();
    ctx.translate(origin.x, origin.y);
    polyPath3(ctx, points);
    ctx.restore();
    if (opts.lineWidth) ctx.lineWidth = opts.lineWidth;
    if (opts.fillStyle || opts.randomColors) ctx.fill();
    if (opts.strokeStyle) ctx.stroke();
  };

  const createPoly = (opts, points = []) => {
    const
      { inset, radius, sides } = opts,
      size = radius - inset,
      step = TAU / sides;
    for (let i = 0; i < sides; i++) {
      points.push(toPolarCoordinate(0, 0, size, step * i));
    }
    return points;
  };

  const gridMeasurements = (opts) => {
    const
      { diameter, inset, radius, sides } = opts,
      edgeLength = Math.sin(Math.PI / sides) * diameter,
      gridSpaceX = diameter - edgeLength / 2,
      gridSpaceY = Math.cos(Math.PI / sides) * diameter,
      gridOffsetY = gridSpaceY / 2;
    return {
      diameter,
      edgeLength,
      gridSpaceX,
      gridSpaceY,
      gridOffsetY
    };
  };

  const polyPath3 = (ctx, points = []) => {
    const [{ x: startX, y: startY }] = points;
    ctx.beginPath();
    ctx.moveTo(startX, startY);
    points.forEach(({ x, y }) => { ctx.lineTo(x, y); });
    ctx.closePath();
  };

  const pickRandom = (arr) => arr[Math.floor(Math.random() * arr.length)];

  const toPoint = (x, y) => ({ x, y });

  const toPolarCoordinate = (centerX, centerY, radius, angle) => ({
    x: centerX + radius * Math.cos(angle),
    y: centerY + radius * Math.sin(angle)
  });

  const generateColors = (count, saturation = 1.0, lightness = 0.5, alpha = 1.0) =>
    Array.from({ length: count }, (_, i) =>
      `hsla(${[
        Math.floor(i / count * 360),
        `${Math.floor(saturation * 100)}%`,
        `${Math.floor(lightness * 100)}%`,
        alpha
      ].join(', ')})`);

  onMount(() => {
    main();
  });
</script>
<canvas id="drawing" width="1100" height="1100" class="absolute z-0" style="z-index:-1"></canvas>
