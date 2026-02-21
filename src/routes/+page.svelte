<script lang="ts">
  import { onMount, onDestroy } from "svelte";
  import { browser } from "$app/environment";
  import Editor from "$lib/Editor.svelte";
  import { writeTextFile, BaseDirectory, mkdir } from "@tauri-apps/plugin-fs";

  let isDirty = false;
  let editorRef: InstanceType<typeof Editor> | undefined;

  onMount(async () => {
    if (!browser) return;
    window.addEventListener("keydown", handleKeydown);
  });

  onDestroy(() => {
    if (!browser) return;
    window.removeEventListener("keydown", handleKeydown);
  });

  function generateFilename(md: string) {
    const lines = md.split("\n");
    const firstContentLine = lines.find(line => line.trim().length > 0);
    if (firstContentLine && firstContentLine.startsWith("# ")) {
      const title = firstContentLine
        .replace(/^#\s+/, "")
        .trim()
        .replace(/[<>:"/\\|?*\x00-\x1F]/g, "")
        .replace(/\s+/g, "-")
        .toLowerCase();
      if (title.length > 0) return `${title}.md`;
    }
    return `note-${new Date().toISOString().slice(0, 10)}.md`;
  }

  async function saveMarkdown(name?: string) {
  if (!editorRef) return;
  const md = editorRef.getMarkdown();
  const filename = name ?? `note-${new Date().toISOString().slice(0, 10)}.md`;
  try {
    await mkdir("Nimble", { baseDir: BaseDirectory.Document, recursive: true });
    await writeTextFile(`Nimble/${filename}`, md, { baseDir: BaseDirectory.Document });
    prompt("Filename: ~/Documents/Nimble/", filename);
  } catch (e) {
    alert("save failed: " + e);
  }
}

  async function saveTempMarkdown() {
    if (!editorRef) return;
    const md = editorRef.getMarkdown();
    try {
      await writeTextFile("nimble_temp.md", md, { baseDir: BaseDirectory.AppData });
    } catch (e) {
      alert("autosave failed: " + e);
    }
  }

  async function handleKeydown(e: KeyboardEvent) {
    if ((e.ctrlKey || e.metaKey) && e.key.toLowerCase() === "s") {
      e.preventDefault();
      await saveMarkdown(generateFilename(editorRef!.getMarkdown()));
    } else if (isDirty) {
      await saveTempMarkdown();
    }
  }
</script>

<svelte:head>
  <title>Nimble</title>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
</svelte:head>

<div class="w-screen h-screen bg-neutral-900 m-0 p-0">
  <Editor bind:this={editorRef} bind:isDirty />
</div>