<script lang="ts">
  import { onMount } from "svelte";
  import { EditorState } from "@codemirror/state";
  import { EditorView, keymap, placeholder } from "@codemirror/view";
  import { defaultKeymap } from "@codemirror/commands";
  import { markdown } from "@codemirror/lang-markdown";
  import { oneDark } from "@codemirror/theme-one-dark";
  import { BaseDirectory, readTextFile } from "@tauri-apps/plugin-fs";
  

  let container: HTMLDivElement;
  let view: EditorView;

  export function getMarkdown() {
    return view.state.doc.toString();
  }

  export let isDirty = false;
  onMount(async () => {
  let initialContent = "";
  try {
    initialContent = await readTextFile("nimble_temp.md", { baseDir: BaseDirectory.AppData });
  } catch {
    // no temp file yet
  }

  const state = EditorState.create({
    doc: initialContent,
    extensions: [
      placeholder("nimble v0.2.0 - use ctrl+s to save. just start typing, this is a placeholder."),
      keymap.of(defaultKeymap),
      markdown(),
      oneDark,
      EditorView.lineWrapping,
      EditorView.updateListener.of((v) => {
        if (v.docChanged) isDirty = true;
      }),
      EditorView.theme({
        "&": { height: "100%", fontSize: "16px", fontFamily: "0xProto Nerd Font", backgroundColor: "#222226" },
        ".cm-content": { padding: "2rem", backgroundColor: "#222226" },
        ".cm-gutters": { backgroundColor: "#222226", border: "none" },
        ".cm-editor": { transform: "none", lineHeight: 1.4 }
      })
    ]
  });

  view = new EditorView({ state, parent: container });
  view.focus();
});
</script>

<div bind:this={container} class="w-full h-full"></div>