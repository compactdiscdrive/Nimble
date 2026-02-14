<script lang="ts">
  import { onMount } from "svelte";
  import { EditorState } from "@codemirror/state";
  import { EditorView, keymap, placeholder } from "@codemirror/view";
  import { defaultKeymap } from "@codemirror/commands";
  import { markdown } from "@codemirror/lang-markdown";
  import { oneDark } from "@codemirror/theme-one-dark";

  let container: HTMLDivElement;
  let view: EditorView;

  export function getMarkdown() {
    return view.state.doc.toString();
  }

  onMount(() => {
    const state = EditorState.create({
      doc: "",
      extensions: [
        placeholder("nimble v0.1. use ctrl+s to save. just start typing, this is a placeholder."),
        keymap.of(defaultKeymap),
        markdown(),
        oneDark,
        EditorView.lineWrapping,
        EditorView.theme({
          "&": {
            height: "100%",
            fontSize: "16px",
            fontFamily: "0xProto Nerd Font",
            backgroundColor: "#222226"
          },
          ".cm-content": {
            padding: "2rem",
            backgroundColor: "#222226"
          },
            ".cm-gutters": {
                backgroundColor: "#222226",
                border: "none"
            },
            ".cm-editor": {
              transform: "none",
              lineHeight: 1.4
            }
        })
      ]
    });

    requestAnimationFrame(() => {
      view.focus();
    });

    view = new EditorView({
      state,
      parent: container
    });
  });
</script>

<div bind:this={container} class="w-full h-full"></div>
