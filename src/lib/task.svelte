<script>
  import { X, Pencil, Check } from "lucide-svelte";
  /** @type {{ text: string, id: number, removeTask:  (id: number) => void} }*/
  let { text = $bindable(), id, removeTask } = $props();
  let editing = $state(false);
</script>

<div class="rounded border border-black p-3 flex items-center justify-between bg-white shadow-md w-full max-w-md">
  {#if !editing}
    <div class="flex items-center w-full gap-2">
      <span class="text-lg text-gray-800 break-words max-w-[80%]">{text}</span>
      <div class="flex flex-col gap-2 flex-shrink-0 ml-auto">
        <button class="bg-red-500 text-white p-2 rounded hover:bg-red-700" onclick={() => removeTask(id)}>
          <X size={18} />
        </button>
        <button class="bg-blue-500 text-white p-2 rounded hover:bg-blue-700" onclick={() => { editing = true;  }}>
          <Pencil size={18} />
        </button>
      </div>
    </div>
  {:else}
    <div class="flex items-center gap-2 w-full">
      <textarea bind:value={text} class="border border-gray-300 rounded p-2 flex-grow"></textarea>
      <button class="bg-green-500 text-white p-2 rounded hover:bg-green-700" onclick={() => { editing = false }}><Check size={18} /></button>
    </div>
  {/if}  
</div>
