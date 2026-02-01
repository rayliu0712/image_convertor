<script lang="ts">
  import JSZip from "jszip";

  const FORMATS = new Map([
    ["image/jpeg", "jpg"],
    ["image/png", "png"],
    ["image/webp", "webp"],
  ]);

  const images = $state<
    { url: string; blob: Blob; name: string; ext: string }[]
  >([]);
  let outputType = $state("image/jpeg");
  let quality = $state(100);

  let dialog: HTMLDialogElement;
  let viewIndex = $state(-1);

  function addNewImages(fileList: FileList) {
    const newImages = Array.from(fileList)
      .filter((file) => file.type.startsWith("image/"))
      .map((file) => {
        const filename = file.name;
        const i = filename.lastIndexOf(".");
        const name = i === -1 ? filename : filename.slice(0, i);
        const ext = i === -1 ? "" : filename.slice(i + 1);

        return {
          url: URL.createObjectURL(file),
          blob: file,
          name: name,
          ext: ext,
        };
      });

    images.splice(0, 0, ...newImages);
  }

  function inputImages() {
    const input = document.createElement("input");
    input.type = "file";
    input.accept = "image/*";
    input.multiple = true;

    input.onchange = () => {
      const fileList = input.files;
      if (fileList) {
        addNewImages(fileList);
      }
    };
    input.click();
  }

  function onPaste(e: ClipboardEvent) {
    const fileList = e.clipboardData?.files;
    if (fileList) {
      addNewImages(fileList);
    }
  }

  function removeImage(i: number) {
    const url = images[i].url;
    URL.revokeObjectURL(url);
    images.splice(i, 1);
  }

  function onDrop(e: DragEvent) {
    e.preventDefault();

    const fileList = e.dataTransfer?.files;
    if (fileList) {
      addNewImages(fileList);
    }
  }

  function download(blob: Blob, filename: string) {
    const url = URL.createObjectURL(blob);

    const a = document.createElement("a");
    a.href = url;
    a.download = filename;
    a.click();

    URL.revokeObjectURL(url);
  }

  async function convertImageFile(
    blob: Blob,
    name: string,
    i: number,
  ): Promise<{ converted: Blob; filename: string }> {
    const bitmap = await createImageBitmap(blob);
    const { width, height } = bitmap;

    const canvas = document.createElement("canvas");
    canvas.width = width;
    canvas.height = height;

    const ctx = canvas.getContext("2d")!;
    ctx.fillStyle = "white";
    ctx.fillRect(0, 0, width, height);
    ctx.drawImage(bitmap, 0, 0, width, height);
    bitmap.close();

    return new Promise((resolve) =>
      canvas.toBlob(
        (blob) => {
          const ext = FORMATS.get(blob!.type);
          const filename = `${name}_${i + 1}.${ext}`;

          resolve({ converted: blob!, filename: filename });
        },
        outputType,
        quality / 100, // for image/png, quality param is ignored
      ),
    );
  }

  async function clickDownloadButton() {
    const count = images.length;

    if (count <= 4) {
      await Promise.all(
        images.map(async ({ blob, name }, i) => {
          const { converted, filename } = await convertImageFile(blob, name, i);
          download(converted, filename);
        }),
      );

      return;
    }

    // images > 4
    const zip = JSZip();
    await Promise.all(
      images.map(async ({ blob, name }, i) => {
        const { converted, filename } = await convertImageFile(blob, name, i);
        zip.file(filename, converted);
      }),
    );
    const zipBlob = await zip.generateAsync({ type: "blob" });
    download(zipBlob, `converted_${count}_images.zip`);
  }

  async function viewImage(i: number) {
    viewIndex = i;
    dialog.showModal();
    await new Promise<void>((resolve) => {
      dialog.onclose = () => resolve();
    });
    viewIndex = -1;
  }
</script>

<!-- svelte-ignore a11y_no_static_element_interactions -->
<div
  class="h-screen flex flex-row p-1"
  onpaste={onPaste}
  ondragover={(e) => e.preventDefault()}
  ondrop={onDrop}
>
  <!-- thumbnails -->
  <div
    class="flex-3 grid grid-cols-5 gap-1 content-start overflow-y-scroll pr-1"
  >
    <button
      class="w-full aspect-square bg-neutral-600 cursor-pointer text-5xl hover:bg-neutral-500 duration-150"
      onclick={inputImages}
    >
      +
    </button>

    {#each images as { url: imageUrl }, i}
      <div class="w-full aspect-square relative group">
        <img alt="thumbnail" src={imageUrl} class="size-full object-cover" />

        <div
          class="absolute inset-0 duration-300 bg-black opacity-0 group-hover:opacity-70 flex items-center justify-center"
        >
          <button
            class="absolute top-1 right-1 cursor-pointer"
            onclick={() => removeImage(i)}>✕</button
          >
          <button onclick={() => viewImage(i)} class="cursor-pointer"
            >View</button
          >
        </div>
      </div>
    {/each}
  </div>

  <!-- panel -->
  <div class="flex-1 p-5 flex flex-col gap-10">
    <button
      class="font-bold cursor-pointer border-2 border-solid py-2 hover:bg-neutral-600 duration-150"
      onclick={() => location.reload()}>Image Convertor</button
    >

    <div class="flex flex-col gap-2">
      <p>Output Format</p>
      <select
        class="bg-neutral-600 rounded-lg cursor-pointer px-4 py-2"
        bind:value={outputType}
      >
        {#each FORMATS as [type, ext]}
          <option value={type}>{ext}</option>
        {/each}
      </select>
    </div>

    {#if outputType !== "image/png"}
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
      onclick={clickDownloadButton}
      class="self-end rounded-full duration-150 bg-cyan-700 cursor-pointer hover:bg-cyan-600 px-8 py-2"
      >Download All</button
    >

    <div class="flex-1"></div>

    <a
      class="self-end cursor-pointer text-3xl hover:animate-spin"
      href="https://github.com/rayliu0712/image_convertor"
      target="_blank">🍹</a
    >
  </div>
</div>

<dialog
  bind:this={dialog}
  class="max-h-none max-w-none size-full bg-transparent border-none backdrop:backdrop-blur-md backdrop:bg-black/70"
>
  {#if viewIndex !== -1}
    {@const { url, name, ext } = images[viewIndex]}
    <div class="size-full flex flex-col">
      <div class="flex flex-row">
        <p class="flex-1">{name} / {ext}</p>
        <button
          class="cursor-pointer text-white text-2xl"
          onclick={() => dialog.close()}>✕</button
        >
      </div>

      <div class="flex-1 min-h-0 relative flex">
        <img src={url} alt="view" class="size-full object-contain" />

        {#if viewIndex > 0}
          <button
            class="absolute left-2 cursor-pointer text-white text-3xl self-center"
            onclick={() => viewIndex--}>🡨</button
          >
        {/if}

        {#if viewIndex < images.length - 1}
          <button
            class="absolute right-2 cursor-pointer text-white text-3xl self-center"
            onclick={() => viewIndex++}>🡪</button
          >
        {/if}
      </div>
    </div>
  {/if}
</dialog>
