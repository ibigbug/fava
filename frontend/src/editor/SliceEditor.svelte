<script lang="ts">
  import { put } from "../api";
  import { initBeancountEditor } from "../codemirror/setup";
  import { notify } from "../notifications";
  import router from "../router";
  import { closeOverlay } from "../stores";

  import SaveButton from "./SaveButton.svelte";

  export let slice: string;
  export let entry_hash: string;
  export let sha256sum: string;

  let currentSlice = slice;
  $: changed = currentSlice !== slice;

  let saving = false;

  async function save() {
    saving = true;
    try {
      sha256sum = await put("source_slice", {
        entry_hash,
        source: currentSlice,
        sha256sum,
      });
      router.reload();
      closeOverlay();
    } catch (error) {
      if (error instanceof Error) {
        notify(error.message, "error");
      }
    } finally {
      saving = false;
    }
  }
  const [, useEditor] = initBeancountEditor(
    slice,
    (state) => {
      currentSlice = state.doc.toString();
    },
    [
      {
        key: "Control-s",
        mac: "Meta-s",
        run: () => {
          save();
          return true;
        },
      },
    ]
  );
</script>

<form on:submit|preventDefault={save}>
  <div use:useEditor />
  <SaveButton {changed} {saving} />
</form>

<style>
  div {
    margin-bottom: 0.5rem;
    border: 1px solid var(--sidebar-border);
  }
</style>
