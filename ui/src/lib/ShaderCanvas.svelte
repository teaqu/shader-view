<script lang="ts">
  import { onMount } from 'svelte';
  
  export let inputManager: any;
  
  // Callback props instead of event dispatchers
  export let onCanvasReady: (canvas: HTMLCanvasElement) => void = () => {};
  export let onCanvasResize: (data: { width: number, height: number }) => void = () => {};
  
  let glCanvas: HTMLCanvasElement;
  
  onMount(() => {
    // Setup resize handling
    const container = glCanvas.parentElement!;
    
    function resizeCanvasToFit16x9() {
      const container = glCanvas.parentElement!;
      const styles = getComputedStyle(container);
      const paddingX = parseFloat(styles.paddingLeft) + parseFloat(styles.paddingRight);

      const w = container.clientWidth - paddingX;
      const h = container.clientHeight;
      const aspect = 16 / 9;

      let newWidth, newHeight;
      if (w / h > aspect) {
        newHeight = h;
        newWidth = h * aspect;
      } else {
        newWidth = w;
        newHeight = w / aspect;
      }

      glCanvas.style.width = `${newWidth}px`;
      glCanvas.style.height = `${newHeight}px`;
      // Set internal resolution based on devicePixelRatio for high-DPI rendering
      const scaleFactor = window.devicePixelRatio;
      onCanvasResize({ width: newWidth * scaleFactor, height: newHeight * scaleFactor });
    }

    const resizeObserver = new ResizeObserver(resizeCanvasToFit16x9);
    resizeObserver.observe(container);
    resizeCanvasToFit16x9();

    // Setup input handling when canvas is ready
    setupInputHandling();

    // Notify parent that canvas is ready
    onCanvasReady(glCanvas);
  });

  function setupInputHandling() {
    if (inputManager && glCanvas) {
      inputManager.setupEventListeners(glCanvas);
      // Make canvas focusable for keyboard events
      glCanvas.tabIndex = 0;
      glCanvas.focus();
    }
  }

  // Re-setup input handling when inputManager changes
  $: if (inputManager && glCanvas) {
    setupInputHandling();
  }
</script>

<div class="canvas-container">
  <canvas bind:this={glCanvas}></canvas>
</div>
