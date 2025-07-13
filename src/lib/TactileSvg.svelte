<script lang="ts">

  // Constants
  const ZOOM_FACTOR = 1.1;
  const MIN_ZOOM = 5;
  const MAX_ZOOM = 4000;

  // Graphical elements
  let svgEl;
  let viewBox = $state({ x: 0, y: 0, width: 100, height: 100 });
  let viewBoxString = $derived(`${viewBox.x} ${viewBox.y} ${viewBox.width} ${viewBox.height}`);

  // Controls
  let isPanning = false;
  let start = { x: 0, y: 0 };

  function toSVGCoords(evt) {
    const pt = svgEl.createSVGPoint();
    pt.x = evt.clientX;
    pt.y = evt.clientY;
    return pt.matrixTransform(svgEl.getScreenCTM().inverse());
  }

  function onPointerDown(evt) {
    isPanning = true;
    const pt = toSVGCoords(evt);
    start = { x: evt.clientX, y: evt.clientY };
    svgEl.setPointerCapture(evt.pointerId);
  }

  function onPointerMove(evt) {
    if (!isPanning) return;

    const dxScreen = evt.clientX - start.x;
    const dyScreen = evt.clientY - start.y;

    // Apply screen delta to a point and transform to SVG coords
    const svgStart = svgEl.createSVGPoint();
    svgStart.x = 0;
    svgStart.y = 0;
    const svgOrigin = svgStart.matrixTransform(svgEl.getScreenCTM().inverse());

    svgStart.x = dxScreen;
    svgStart.y = dyScreen;
    const svgMoved = svgStart.matrixTransform(svgEl.getScreenCTM().inverse());

    const dxSVG = svgMoved.x - svgOrigin.x;
    const dySVG = svgMoved.y - svgOrigin.y;

    viewBox.x -= dxSVG;
    viewBox.y -= dySVG;

    start = { x: evt.clientX, y: evt.clientY }; // update screen coord
  }

  function onPointerUp(evt) {
    isPanning = false;
    svgEl.releasePointerCapture(evt.pointerId);
  }

  function onWheel(evt) {
    evt.preventDefault();


    const direction = evt.deltaY > 0 ? 1 : -1;
    const scale = direction > 0 ? ZOOM_FACTOR : 1 / ZOOM_FACTOR;

    const newWidth = viewBox.width * scale;
    const newHeight = viewBox.height * scale;

    // Clamp the zoom level
    if (newWidth < MIN_ZOOM || newWidth > MAX_ZOOM) return;

    const mouse = toSVGCoords(evt);

    // Zoom toward cursor
    viewBox.x = mouse.x - (mouse.x - viewBox.x) * scale;
    viewBox.y = mouse.y - (mouse.y - viewBox.y) * scale;
    viewBox.width = newWidth;
    viewBox.height = newHeight;
  }

</script>


<svg
  bind:this={svgEl}
  viewBox={viewBoxString}
  onpointerdown={onPointerDown}
  onpointermove={onPointerMove}
  onpointerup={onPointerUp}
  onwheel={onWheel}
  style="touch-action: none; width: 100%; height: 100%; border: 1px solid #ccc;"
>
  <!-- Example content -->
  <circle cx="50" cy="50" r="5" fill="red" />
  <rect x="20" y="20" width="10" height="10" fill="blue" />
</svg>
