<script>
  import { onMount } from "svelte";

  let videoStream;
  let isRecording = false;
  let isPaused = false;

  let stream;
  let mediaRecorder;

  let ready = false;

  let speed = 100;
  let script = "";
  let scriptElement;
  let step = 0;
  $: reverseScript = script
    .split("\n")
    .flatMap((t) => t.split(" "))
    .map((t) => `<span>${t}</span>`)
    .reverse()
    .join("");
  onMount(async () => {
    stream = await navigator.mediaDevices.getUserMedia({
      video: true,
      audio: true,
    });
    videoStream.srcObject = stream;
    if (!MediaRecorder.isTypeSupported("video/webm")) {
      console.warn("video/webm is not supported");
    }
    mediaRecorder = new MediaRecorder(stream, {
      mimeType: "video/webm",
    });

    ready = true;
    scriptElement.style.top = `-100%`;

    mediaRecorder.addEventListener("dataavailable", (event) => {
      const blob = new Blob([event.data], { type: "video/webm" });
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = "video.webm";
      a.click();
      URL.revokeObjectURL(url);
    });
  });

  const animate = (speed) => {
    return setInterval(() => {
      if (!isRecording) return;
      step++;
      scriptElement.style.top = `${-scriptElement.clientHeight + step}px`;
    }, 1000 / speed);
  };

  let interval = animate(speed);

  $: {
    if (isRecording) {
      clearInterval(interval);
      if (!isPaused) {
        interval = animate(speed);
      }
    }
  }

  const record = () => {
    isPaused = false;
    if (isRecording) {
      mediaRecorder.stop();
      isRecording = false;
      step = 0;
    } else {
      mediaRecorder.start();
      isRecording = true;
    }
  };

  const pause = () => {
    if (isRecording) {
      if (isPaused) {
        mediaRecorder.resume();
        isPaused = false;
      } else {
        mediaRecorder.pause();
        isPaused = true;
      }
    }
  };
</script>

<main class="p-2 min-h-screen bg-slate-950 text-white">
  <section id="script" class="flex flex-col justify-center items-center">
    <button popovertarget="setting" class="underline">Setup</button>
    <div id="setting" popover="auto" class="w-2/3 rounded-md p-2 bg-slate-500">
      <input
        type="range"
        bind:value={speed}
        min="0"
        max="1000"
        class="w-full"
      />
      <textarea
        bind:value={script}
        class="p-2 rounded-sm w-full resize-y text-slate-800"
        placeholder="Enter your script here..."
      ></textarea>
    </div>
  </section>

  <section id="camera" class="flex flex-col justify-center items-center">
    {#if ready}
      <div class="flex gap-2">
        <button on:click={record} class="bg-slate-50 text-slate-800 px-2 py-1"
          >{isRecording ? "Stop" : "Record"}</button
        >
        {#if isRecording}
          <button on:click={pause} class="bg-slate-50 text-slate-800 px-2 py-1"
            >{isPaused ? "Resume" : "Pause"}</button
          >
        {/if}
      </div>
    {:else}
      <p class="text-center">Loading Camera...</p>
    {/if}
    <div
      class="{!ready &&
        'hidden'} w-full h-[90vh] flex items-center justify-center bg-black relative"
    >
      <div
        class="{!isRecording &&
          'hidden'} flex justify-center w-full h-full bg-slate-950 bg-opacity-65 relative z-50 overflow-hidden"
      >
        <pre
          bind:this={scriptElement}
          class="absolute w-1/2 text-6xl font-semibold flex flex-row-reverse flex-wrap justify-center gap-[1rem]"
          bind:innerHTML={reverseScript}
          contenteditable="false"></pre>
      </div>
      <video
        class="absolute top-0 left-0 object-contain w-full h-full"
        bind:this={videoStream}
        autoplay
        muted
        playsinline
      >
        <track kind="captions" />
      </video>
    </div>
  </section>
</main>
