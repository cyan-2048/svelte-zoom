<script>
  export let alt = "svelte-zoom";
  import Matrix from "./matrix";

  import { calculateAspectRatioFit } from "./other";

  let smooth = true;

  let xY = {
    initX: 0,
    initY: 0,
    newX: 0,
    newY: 0,
  };

  let ratio, img;

  let matrix = new Matrix();
  let contain = null;

  export let maxScale = 11;
  export let scaleValue = 1;

  let scaling = false;

  export function fireManualZoom(dir) {
    const { innerHeight, innerWidth } = window;

    const xFactor = 1 + 0.2 * dir;
    const yFactor = (xFactor * innerHeight) / innerWidth;

    let in_x = (innerWidth - ratio.width * matrix.vtm.a) / 2;
    let in_y = (innerHeight - ratio.height * matrix.vtm.a) / 2;

    const origin = {
      x: innerWidth / 2,
      y: innerHeight / 2,
    };

    const mat = matrix.scale(xFactor, yFactor, origin, in_x, in_y, ratio, maxScale, scaleValue * xFactor, dir);
    img.style.transform = `translate(${mat.e}px,${mat.f}px) scale(${mat.d})`;
    scaleValue = mat.d;
  }

  export const zoomIn = () => fireManualZoom(1);

  export const zoomOut = () => fireManualZoom(-1);

  function onResize() {
    onLoad();
    fireDown(0, 0);
    fireMove(0, 0);
    fireUp();
  }

  function onLoad() {
    const { naturalWidth, naturalHeight } = img;
    const { innerHeight, innerWidth } = window;

    contain = naturalWidth > innerWidth || naturalHeight > innerHeight;

    ratio = calculateAspectRatioFit(naturalWidth, naturalHeight, innerWidth, innerHeight);
  }

  function fireDown(x, y) {
    xY.initX = x;
    xY.initY = y;

    matrix.x = matrix.vtm.e;
    matrix.y = matrix.vtm.f;
  }

  function fireMove(x, y) {
    if (scaling) return;
    let in_x = (window.innerWidth - ratio.width * matrix.vtm.a) / 2;
    let in_y = (window.innerHeight - ratio.height * matrix.vtm.a) / 2;

    xY.newX = xY.initX - x;
    xY.newY = xY.initY - y;
    const mat = matrix.move(in_x >= 0 ? 0 : xY.newX, in_y >= 0 ? 0 : xY.newY, in_x, in_y, ratio);

    img.style.transform = `matrix(${mat.a},${mat.b},${mat.c},${mat.d},${mat.e}, ${mat.f})`;
  }

  function fireUp() {
    matrix.x -= xY.newX;
    matrix.y -= xY.newY;

    scaling = false;
    smooth = true;
  }

  export function moveImage(x, y) {
    const { innerWidth, innerHeight } = window;

    const [a, b] = [innerWidth / 2, innerHeight / 2];

    fireDown(a, b);
    smooth = false;
    fireMove(a + x, b + y);
    fireUp();
  }
</script>

<svelte:window on:resize={onResize} />

<img
  {alt}
  class="c-svelteZoom"
  class:c-svelteZoom--contain={contain}
  class:c-svelteZoom--no-contain={!contain}
  class:c-svelteZoom--transition={smooth}
  class:c-svelteZoom--visible={contain}
  class:c-svelteZoom--hidden={contain === null}
  bind:this={img}
  on:load={onLoad}
  draggable={false}
  {...$$props}
/>

<style>
  .c-svelteZoom {
    width: 100%;
    height: 100%;
    position: absolute;
    transform-origin: 0 0;
    backface-visibility: hidden;
    -moz-backface-visibility: hidden;
  }

  .c-svelteZoom--contain {
    object-fit: contain;
  }

  .c-svelteZoom--no-contain {
    object-fit: contain;
  }

  .c-svelteZoom--transition {
    transition: transform 0.2s;
  }

  .c-svelteZoom--visible {
    visibility: visible;
  }

  .c-svelteZoom--hidden {
    visibility: hidden;
  }
</style>
