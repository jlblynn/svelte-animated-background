<script>
  import {  fade } from 'svelte/transition';
  import { onMount, tick } from 'svelte';

  const images = Array.from({ length: 82 }, (_, i) => `./frames/${i + 1}.jpg`);
  let currentImage = images[0];
  let isLoading = true;
  let progress = 0;
  let imageCache = {}; // Cache for preloaded images

  onMount(() => {
    window.addEventListener('scroll', handleScroll);

    // Preload images for better performance
    preloadImages().then(() => {
      isLoading = false;
    });

    // Reset scroll position to top
    window.scrollTo(0, 0);
  });

  async function handleScroll() {
    const scrollPosition = window.pageYOffset;
    const imageIndex = Math.floor(scrollPosition / 50);

    if (imageIndex < images.length) {
      const nextImage = images[imageIndex];
      if (currentImage !== nextImage) {
        await fadeOut();
        currentImage = nextImage;
        await tick(); // Wait for Svelte to update DOM
        fadeIn();
        cacheNextImage(imageIndex + 1);
      } else {
        cacheNextImage(imageIndex + 1);
      }
    }
  }

  function fadeOut() {
    return fade(document.getElementById('background-image'), {
      duration: 50,
      opacity: 0,
    });
  }

  function fadeIn() {
    return fade(document.getElementById('background-image'), {
      duration: 50,
      opacity: 1,
    });
  }

  async function preloadImages() {
    try {
      const promises = images.map((image, index) => {
        if (index === 0) {
          return new Promise((resolve, reject) => {
            const firstImage = new Image();
            firstImage.src = image;
            firstImage.onload = resolve;
            firstImage.onerror = reject;
          });
        } else {
          return new Promise((resolve) => {
            const img = new Image();
            img.src = image;
            img.onload = resolve;
            img.onerror = resolve; // Ignore errors for subsequent images
          });
        }
      });

      await Promise.all(promises);
      updateProgress(images.length);
    } catch (error) {
      console.error('Failed to preload images:', error);
    }
  }

  function updateProgress(loadedCount) {
    progress = (loadedCount / images.length) * 100;
  }

  function cacheNextImage(index) {
    if (index >= images.length || imageCache[index]) {
      return;
    }

    const image = new Image();
    image.src = images[index];
    imageCache[index] = image;
  }
</script>

{#if isLoading}
  <div class="loading">
    <div class="progress-bar">
      <div class="progress-bar-inner" style="width: {progress}%"></div>
    </div>
    Loading...
  </div>
{:else}
<div class="background-container">
  <div class="background-image" id="background-image" style="background-image: url({currentImage})"></div>
</div>
{/if}

<style>
    .background-image {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-position: center;
    background-repeat: no-repeat;
    background-size: cover;
    transition: opacity 0.5s ease-in-out; /* Added transition for fading animation */
  }
  
  .loading {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100vh;
    font-size: 24px;
    font-weight: bold;
  }

  .progress-bar {
    width: 100%;
    height: 10px;
    background-color: #f0f0f0;
    border-radius: 5px;
    overflow: hidden;
  }

  .progress-bar-inner {
    height: 100%;
    background-color: #007bff;
    transition: width 0.3s ease-in-out;
  }

  @media only screen and (max-width: 600px) {
    .background-container {
      width: 100%;
      height: 100%;
      overflow: hidden;
      position: fixed;
      top: 200px;
    }
    .background-image {
      width: 100%;
      height: 300px;
      position: relative;
    }
  }
</style>