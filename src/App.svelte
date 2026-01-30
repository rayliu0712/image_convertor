<script lang="ts">
  let quality = $state(100);
  let outputFormat = $state("jpg");
  let imgSrc = $state("");
  let inputFileElement: HTMLInputElement;

  function download(event: Event) {}

  const li: Array<number> = [];
  for (let i = 1; i < 19; i++) {
    li.push(i);
  }
</script>

<input
  class="hidden"
  type="file"
  accept="image/*"
  multiple
  bind:this={inputFileElement}
/>

<div class="h-screen flex flex-row p-1">
  <!-- thumbnails -->
  <div class="flex-3 grid grid-cols-5 gap-1 overflow-y-scroll pr-1">
    <button
      class="w-full aspect-square bg-neutral-600 cursor-pointer text-4xl"
      onclick={() => inputFileElement.click()}
    >
      +
    </button>
    {#each li as i}
      <img
        alt="thumbnail"
        src="assets/{i}.jpg"
        class="w-full aspect-square object-cover"
      />
    {/each}
  </div>

  <!-- panel -->
  <div class="flex-1 p-5 flex flex-col gap-10">
    <button
      class="font-bold cursor-pointer border-2 border-solid py-2"
      onclick={() => location.reload()}>Image Convertor</button
    >

    <div class="flex flex-col gap-2">
      <p>Output Format</p>
      <select
        class="bg-neutral-600 rounded-lg cursor-pointer px-4 py-2"
        bind:value={outputFormat}
      >
        {#each ["jpg", "png", "webp"] as fmt}
          <option>{fmt}</option>
        {/each}
      </select>
    </div>

    {#if outputFormat !== "png"}
      <div class="flex flex-col gap-2">
        <p>Quality {quality}%</p>

        <input
          type="range"
          min="0"
          step="1"
          max="100"
          class="accent-cyan-600 cursor-pointer"
          bind:value={quality}
        />
      </div>
    {/if}

    <button
      onclick={download}
      class="self-end rounded-full bg-cyan-700 cursor-pointer hover:bg-cyan-600 px-8 py-2"
      >Download All</button
    >

    <div class="flex-1"></div>

    <a
      class="self-end cursor-pointer text-3xl"
      href="https://github.com/rayliu0712/image_convertor"
      target="_blank">🍹</a
    >
  </div>
</div>
