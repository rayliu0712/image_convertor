<script lang="ts">
  let inputFileElement: HTMLInputElement;
  let quality = $state(100);
  let outputFormat = $state("jpg");
  let imageUrls = $state<string[]>([]);

  function download(e: Event) {}

  function createNewUrls(array: Blob[]) {
    const newUrls = array.map((blob: Blob) => URL.createObjectURL(blob));
    imageUrls.splice(0, 0, ...newUrls);

    inputFileElement.value = "";
  }

  function onInputFiles(e: Event) {
    const fileList = inputFileElement.files;
    if (fileList) {
      createNewUrls(Array.from(fileList));
    }
  }

  function onPaste(e: ClipboardEvent) {
    const fileList = e.clipboardData?.files;
    if (fileList) {
      createNewUrls(Array.from(fileList));
    }
  }

  async function onReadClipboard(e: Event) {
    const clipboardItems = await navigator.clipboard.read();

    const blobArray: Blob[] = [];

    for (const item of clipboardItems) {
      const type = item.types.find((type: String) => type.startsWith("image/"));

      if (type) {
        const blob = await item.getType(type);
        blobArray.push(blob);
      }
    }

    createNewUrls(blobArray);
  }

  function remove(i: number) {
    const url = imageUrls[i];
    URL.revokeObjectURL(url);
    imageUrls.splice(i, 1);
  }
</script>

<input
  class="hidden"
  type="file"
  accept="image/*"
  multiple
  bind:this={inputFileElement}
  onchange={onInputFiles}
/>

<div class="h-screen flex flex-row p-1" onpaste={onPaste}>
  <!-- thumbnails -->
  <div
    class="flex-3 grid grid-cols-5 gap-1 content-start overflow-y-scroll pr-1"
  >
    <button
      class="w-full aspect-square bg-neutral-600 cursor-pointer text-5xl"
      onclick={() => inputFileElement.click()}
    >
      +
    </button>

    <button
      class="w-full aspect-square bg-neutral-600 cursor-pointer text-5xl"
      onclick={onReadClipboard}
    >
      📋
    </button>

    {#each imageUrls as url, i}
      <div class="w-full aspect-square relative group">
        <img alt="thumbnail" src={url} class="size-full object-cover" />

        <div
          class="absolute inset-0 duration-300 bg-black opacity-0 group-hover:opacity-70 flex items-center justify-center"
        >
          <button
            class="absolute top-1 right-1 cursor-pointer"
            onclick={() => remove(i)}>✕</button
          >
          <button onclick={() => {}} class="cursor-pointer">View</button>
        </div>
      </div>
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
