<script>
  // @ts-nocheck
  import Column from "$lib/column.svelte";
    import Task from "$lib/task.svelte";
  import { Plus } from "lucide-svelte";
  import { onMount } from "svelte";

  let tasks = $state([]);
  let count = $state(0);
  let columns = $state([
    { id: 1, name: "Start" },
    { id: 2, name: "Incoming" },
    { id: 3, name: "Done" }
  ]);
  
  /**
   *  Adiciona uma nova coluna
   * @returns {void}
   */
  const addColumn = () => {
    const newId = columns.length ? columns[columns.length - 1].id + 1 : 1;
    columns = [...columns, { id: newId, name: "New Column" }];
    if (typeof window !== undefined) localStorage.setItem("columns", JSON.stringify(columns));
  };

  const deleteColumn = (id) => {
    columns = columns.filter((column) => column.id !== id);
    tasks = tasks.filter((task) => task.columnId !== id)
    if (typeof window !== undefined) localStorage.setItem("columns", JSON.stringify(columns));
    if (typeof window !== undefined) localStorage.setItem("tasks", JSON.stringify(tasks));
  };
  
  onMount(() => {
    if (typeof window !== undefined) {
      let localTasks = localStorage.getItem("tasks");
      if (localTasks) tasks = JSON.parse(localTasks);
      
      let localColumns = localStorage.getItem("columns");
      if (localColumns) columns = JSON.parse(localColumns);

      let localCount = localStorage.getItem("count");
      if (localCount) count = JSON.parse(localCount);
    }
  });
</script>

<main class="h-screen w-screen bg-gray-50">
  <div class="p-6 h-full overflow-y-auto overflow-x-auto">
    <div class="flex space-x-4 h-full overflow-x-auto overflow-y-auto">
      {#each columns as column}
        <Column bind:count bind:tasks id={column.id} name={column.name} bind:columns={columns} deleteColumn={deleteColumn} />
      {/each}
      <div class="mb-4">
        <button 
          onclick={addColumn} 
          class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600"
        >
          <Plus size={18} />
        </button>
      </div>
    </div>
  </div>
</main>
