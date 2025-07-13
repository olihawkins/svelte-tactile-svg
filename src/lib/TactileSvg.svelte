<svelte:options namespace="svg" />

<script lang="ts">

  // Imports ------------------------------------------------------------------

  import type { Snippet } from "svelte";

  // Constants ----------------------------------------------------------------

  const ZOOM_FACTOR = 1.1;
  const MIN_ZOOM = 5;
  const MAX_ZOOM = 4000;

  // Props --------------------------------------------------------------------

  interface Props {
    children?: Snippet;
  }

  let { children }: Props = $props();

  // Bound elements -----------------------------------------------------------

  let svgEl;

  // State --------------------------------------------------------------------

  let viewBox = $state({ x: 0, y: 0, width: 100, height: 100 });
  let viewBoxString = $derived(`${viewBox.x} ${viewBox.y} ${viewBox.width} ${viewBox.height}`);

  let touchStartDistance: number | null = $state(null);
  let lastTouchMidpoint: { x: number, y: number } | null = $state(null);

  let pinchInitialDistance: number | null = $state(null);
  let pinchInitialViewBox: typeof viewBox = $state(null);
  let pinchAnchorSVG: { x: number, y: number } | null = $state(null);

  // Controls -----------------------------------------------------------------

  let isPanning = false;
  let start = { x: 0, y: 0 };

  // Functions ----------------------------------------------------------------

  function toSVGCoords(e: Event) {
    const pt = svgEl.createSVGPoint();
    pt.x = e.clientX;
    pt.y = e.clientY;
    return pt.matrixTransform(svgEl.getScreenCTM().inverse());
  }

  function getTouchDistance(e: TouchEvent) {
    const [touch1, touch2] = [e.touches[0], e.touches[1]];
    const dx = touch2.clientX - touch1.clientX;
    const dy = touch2.clientY - touch1.clientY;
    return Math.sqrt(dx * dx + dy * dy);
  }

  function getTouchMidpoint(e: TouchEvent) {
    const [touch1, touch2] = [e.touches[0], e.touches[1]];
    return {
      x: (touch1.clientX + touch2.clientX) / 2,
      y: (touch1.clientY + touch2.clientY) / 2
    };
  }

  // Handlers -----------------------------------------------------------------

  function onPointerDown(e: Event) {
    isPanning = true;
    const pt = toSVGCoords(e);
    start = { x: e.clientX, y: e.clientY };
    svgEl.setPointerCapture(e.pointerId);
  }

  function onPointerMove(e: Event) {

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

  function onPointerUp(e: Event) {
    isPanning = false;
    svgEl.releasePointerCapture(e.pointerId);
  }

  function onWheel(e: Event) {
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

  function onTouchStart(e: TouchEvent) {
    if (e.touches.length === 2) {
      pinchInitialDistance = getTouchDistance(e);
      pinchInitialViewBox = { ...viewBox };

      const midpoint = getTouchMidpoint(e);
      const pt = svgEl.createSVGPoint();
      pt.x = midpoint.x;
      pt.y = midpoint.y;
      pinchAnchorSVG = pt.matrixTransform(svgEl.getScreenCTM().inverse());
    }
  }

  function onTouchMove(e: TouchEvent) {
    if (e.touches.length === 2 && pinchInitialDistance && pinchAnchorSVG && pinchInitialViewBox) {
      
      const newDistance = getTouchDistance(e);
      const zoomRatio = pinchInitialDistance / newDistance;

      const newWidth = pinchInitialViewBox.width * zoomRatio;
      const newHeight = pinchInitialViewBox.height * zoomRatio;

      if (newWidth < MIN_ZOOM || newWidth > MAX_ZOOM) return;

      // Keep the anchor point fixed in SVG space
      viewBox.width = newWidth;
      viewBox.height = newHeight;
      viewBox.x = pinchAnchorSVG.x - (pinchAnchorSVG.x - pinchInitialViewBox.x) * zoomRatio;
      viewBox.y = pinchAnchorSVG.y - (pinchAnchorSVG.y - pinchInitialViewBox.y) * zoomRatio;

      // Prevent default pinch-to-zoom behavior
      e.preventDefault(); 
    }
  }

  function onTouchEnd(evt: TouchEvent) {
    if (evt.touches.length < 2) {
      pinchInitialDistance = null;
      pinchInitialViewBox = null;
      pinchAnchorSVG = null;
    }
  }

</script>

<svg
  bind:this={svgEl}
  viewBox={viewBoxString}
  onpointerdown={onPointerDown}
  onpointermove={onPointerMove}
  onpointerup={onPointerUp}
  onwheel={onWheel}
  ontouchstart={onTouchStart}
  ontouchmove={onTouchMove}
  ontouchend={onTouchEnd}
  style="touch-action: none; width: 100%; height: 100%; border: 1px solid #ccc;"
>

  {#if children}
    {@render children()}
  {/if}
</svg>
