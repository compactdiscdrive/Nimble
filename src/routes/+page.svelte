<script lang="ts">
  import { onMount, onDestroy } from "svelte";
  import { browser } from "$app/environment";
  import Editor from "$lib/Editor.svelte";

  let currentTime = new Date();

  let editorRef: InstanceType<typeof Editor>;

  function generateFilename(md: string) {
    const lines = md.split("\n");

    // Find first non-empty line
    const firstContentLine = lines.find(line => line.trim().length > 0);

    // If it starts with "# "
    if (firstContentLine && firstContentLine.startsWith("# ")) {
      const title = firstContentLine
        .replace(/^#\s+/, "")
        .trim()
        .replace(/[<>:"/\\|?*\x00-\x1F]/g, "") // remove illegal filename chars
        .replace(/\s+/g, "-") // spaces → dashes
        .toLowerCase();

      if (title.length > 0) {
        return `${title}.md`;
      }
    }

    // fallback
    const currentTime = new Date();
    return `note-${currentTime.toISOString().slice(0, 10)}.md`;
  }

  function saveMarkdown() {
    if (!editorRef) return;

    const md = editorRef.getMarkdown();
    const blob = new Blob([md], { type: "text/markdown" });

    const a = document.createElement("a");
    a.href = URL.createObjectURL(blob);
    a.download = generateFilename(md);
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(a.href);
  }

  function handleKeydown(e: KeyboardEvent) {
    if ((e.ctrlKey || e.metaKey) && e.key.toLowerCase() === "s") {
      e.preventDefault();
      saveMarkdown();
    }
  }

  onMount(() => {
    if (!browser) return;

    window.addEventListener("keydown", handleKeydown);
  });

  onDestroy(() => {
    if (!browser) return;

    window.removeEventListener("keydown", handleKeydown);
  });
</script>

<svelte:head>
  <title>Nimble</title>
  <utf-8 charset="UTF-8"></utf-8>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
</svelte:head>

<div class="w-screen h-screen bg-neutral-900 margin-0 p-0">
  <Editor bind:this={editorRef} />
</div>