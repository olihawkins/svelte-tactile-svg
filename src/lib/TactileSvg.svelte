<script lang="ts">

  // Imports ------------------------------------------------------------------

  import type { Snippet } from "svelte";

  // Constants ----------------------------------------------------------------

  const ZOOM_FACTOR = 1.1;
  const MIN_ZOOM = 5;
  const MAX_ZOOM = 4000;

  // Props --------------------------------------------------------------------

  interface Props {
    key?: string;
    children?: Snippet;
  }

  let { children }: Props = $props();

  // Graphical elements -------------------------------------------------------

  let svgEl;
  let viewBox = $state({ x: 0, y: 0, width: 100, height: 100 });
  let viewBoxString = $derived(`${viewBox.x} ${viewBox.y} ${viewBox.width} ${viewBox.height}`);

  // Controls -----------------------------------------------------------------

  let isPanning = false;
  let start = { x: 0, y: 0 };

  // Functions ----------------------------------------------------------------

  function toSVGCoords(e) {
    const pt = svgEl.createSVGPoint();
    pt.x = e.clientX;
    pt.y = e.clientY;
    return pt.matrixTransform(svgEl.getScreenCTM().inverse());
  }

  function onPointerDown(e) {
    isPanning = true;
    const pt = toSVGCoords(e);
    start = { x: e.clientX, y: e.clientY };
    svgEl.setPointerCapture(e.pointerId);
  }

  function onPointerMove(e) {
    if (!isPanning) return;

    const dxScreen = e.clientX - start.x;
    const dyScreen = e.clientY - start.y;

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

    start = { x: e.clientX, y: e.clientY }; // update screen coord
  }

  function onPointerUp(e) {
    isPanning = false;
    svgEl.releasePointerCapture(e.pointerId);
  }

  function onWheel(e) {
    e.preventDefault();


    const direction = e.deltaY > 0 ? 1 : -1;
    const scale = direction > 0 ? ZOOM_FACTOR : 1 / ZOOM_FACTOR;

    const newWidth = viewBox.width * scale;
    const newHeight = viewBox.height * scale;

    // Clamp the zoom level
    if (newWidth < MIN_ZOOM || newWidth > MAX_ZOOM) return;

    const mouse = toSVGCoords(e);

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

  {#if children}
    {@render children()}
  {/if}
</svg>
