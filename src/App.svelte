<script>
  import { onMount } from "svelte";
  import {
    CameraIcon,
    CheckCircleIcon,
    GithubIcon,
    MaximizeIcon,
    MinimizeIcon,
    VideoIcon,
    GridIcon
  } from "svelte-feather-icons";

  let cameraSelectorActive;
  let gridActive;
  let deviceId;
  let video;
  let stream;
  let canvas;
  let downloadLink;
  let videoDevices;
  let permissionDenied;
  let isFullscreen;

  onMount(async () => {
    try {
      stream = await navigator.mediaDevices.getUserMedia({
        video: {
          width: { min: 1280 },
        }
      });
    } catch (err) {
      console.log(err);
      if (err.name !== "AbortError") {
        permissionDenied = true;
        return;
      }
    }
    const devices = await navigator.mediaDevices.enumerateDevices();
    videoDevices = devices.filter(({ kind }) => kind == "videoinput");
    if (videoDevices[0]) {
      deviceId = videoDevices[0].deviceId;
      video.srcObject = stream;
    }
  });

  async function switchStream(id) {
    deviceId = id;
    try {
      stream = await navigator.mediaDevices.getUserMedia({
        video: {
          deviceId,
          width: { min: 1280 },
        }
      });
      cameraSelectorActive = false;
      video.srcObject = stream;
    } catch (err) {
      console.log(err);
      if (err.name !== "AbortError") {
        permissionDenied = true;
      }
    }
  }

  async function toggleFullscreen() {
    if (document.fullscreenElement) {
      document.exitFullscreen();
      isFullscreen = false;
    } else {
      try {
        await document.children[0].requestFullscreen();
        isFullscreen = true;
      } catch (err) {
        isFullscreen = false;
      }
    }
  }

  function toggleVideoSelector() {
    cameraSelectorActive = !cameraSelectorActive;
  }

  function toggleGrid() {
    gridActive = !gridActive;
  }

  function goToGithub() {
    location.href = "https://github.com/dotcypress/camera";
  }

  async function takePhoto() {
    canvas.width = video.videoWidth;
    canvas.height = video.videoHeight;

    const context = canvas.getContext("2d");
    context.drawImage(video, 0, 0, canvas.width, canvas.height);
    downloadLink.download = `Screen Shot ${new Date().toLocaleString()}.png`;
    downloadLink.href = canvas.toDataURL("image/png");
    downloadLink.click();
  }

  function handleKey(event) {
    if (event.keyCode == 32) {
      takePhoto();
    } else if (event.keyCode == 70) {
      toggleFullscreen();
    } else if (event.keyCode == 71) {
      toggleGrid();
    }
  }
</script>

<style>
  :global(html),
  :global(body) {
    position: relative;
    width: 100%;
    height: 100%;
    margin: 0;
    padding: 0;
    background-color: black;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
      Oxygen-Sans, Ubuntu, Cantarell, "Helvetica Neue", sans-serif;
    color: #333;
  }

  h1 {
    margin: 4rem auto;
    text-align: center;
  }

  video {
    position: fixed;
    height: 100%;
    width: 100%;
    object-fit: cover;
    margin: auto;
  }

  header {
    display: flex;
    flex-direction: row-reverse;
    top: 0;
    right: 0;
    left: 0;
    position: fixed;
  }

  footer {
    align-items: center;
    flex-direction: column;
    display: flex;
    bottom: 1rem;
    right: 0;
    left: 0;
    position: fixed;
  }

  nav {
    display: flex;
  }

  nav button {
    cursor: pointer;
    color: whitesmoke;
    background: transparent;
    border: none;
    width: 2.5rem;
    height: 2.5rem;
    filter: drop-shadow(0px 0px 2px rgba(0, 0, 0, 0.5));
  }

  footer nav {
    align-items: flex-end;
  }

  footer nav button.large {
    width: 5rem;
    height: 3.5rem;
  }

  nav button:hover,
  nav button:active {
    color: orangered;
  }

  #download {
    display: none;
  }

  .selector-popup {
    position: relative;
  }

  .selector-popup > div {
    position: absolute;
    display: flex;
    flex-direction: column;
    right: 0;
    top: 2.5rem;
  }

  .selector-popup > div button {
    display: flex;
    align-items: center;
    width: auto;
    white-space: nowrap;
  }

  .selector-popup > div button span {
    width: 1rem;
    margin-right: 0.5rem;
    display: inline-block;
  }

  .grid {
    position: fixed;
    height: 100%;
    width: 100%;
    margin: -1px;
    background-size: 20% 33.34%;
    background-image: linear-gradient(
        to right,
        rgb(70, 70, 70) 1px,
        transparent 1px
      ),
      linear-gradient(to bottom, rgb(70, 70, 70) 1px, transparent 1px);
  }
</style>

<svelte:window on:keyup={handleKey} on:dblclick={toggleFullscreen} />
<a id="download" bind:this={downloadLink} href="/">
  <canvas bind:this={canvas} />
</a>
<video autoplay playsinline bind:this={video} />
{#if permissionDenied}
  <h1>Can't access camera 😫</h1>
{:else if videoDevices}
  {#if gridActive}
    <div class="grid" />
  {/if}
  <header>
    <nav>
      <button title="Select camera" on:click={toggleVideoSelector}>
        <VideoIcon />
      </button>
      {#if cameraSelectorActive}
        <div class="selector-popup">
          <div>
            {#each videoDevices as videoDevice}
              <button on:click={() => switchStream(videoDevice.deviceId)}>
                <span>
                  {#if deviceId === videoDevice.deviceId}
                    <CheckCircleIcon />
                  {/if}
                </span>
                {videoDevice.label}
              </button>
            {/each}
          </div>
        </div>
      {/if}
      <button title="Camera source code" on:click={goToGithub}>
        <GithubIcon />
      </button>
    </nav>
  </header>
  <footer>
    <nav>
      <button title="Toggle grid (G)" on:click={toggleGrid}>
        <GridIcon />
      </button>
      <button title="Take photo (Space)" class="large" on:click={takePhoto}>
        <CameraIcon />
      </button>
      <button title="Toggle fullscreen (F)" on:click={toggleFullscreen}>
        {#if isFullscreen}
          <MinimizeIcon />
        {:else}
          <MaximizeIcon />
        {/if}
      </button>
    </nav>
  </footer>
{:else}
  <h1>Connecting to camera...</h1>
{/if}
