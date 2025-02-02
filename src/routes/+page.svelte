<script>
  import Column from "$lib/column.svelte";
  import { Plus } from "lucide-svelte";
  import { onMount } from "svelte";

  /** @type {Array<{ id: number, name: string }>} */
  let columns = $state([
    { id: 1, name: "Start" },
    { id: 2, name: "Incoming" },
    { id: 3, name: "Done" }
  ]);

  /** @type {Array<{ id: number, text: string, columnId: number }>} */
  let tasks = $state([]);

  /** Contador global de tarefas */
  let count = $state(0);

  /** Contador para IDs das colunas */
  // svelte-ignore state_referenced_locally
    let columnsCount = $state(columns.length + 1);
  
  /**
   * Adiciona uma nova coluna
   * @returns {void}
   */
  const addColumn = () => {
    columns = [...columns, { id: columnsCount, name: "New Column" }];
    columnsCount++;
    
    if (typeof window !== undefined) {
      localStorage.setItem("columns", JSON.stringify(columns));
      localStorage.setItem("columnsCount", JSON.stringify(columnsCount));
    }
  };

  /**
   * Remove uma coluna e suas tarefas associadas
   * @param {number} id - O ID da coluna a ser removida
   * @returns {void}
   */
  const deleteColumn = (id) => {
    columns = columns.filter((column) => column.id !== id);
    tasks = tasks.filter((task) => task.columnId !== id);
    
    if (typeof window !== undefined) {
      localStorage.setItem("columns", JSON.stringify(columns));
      localStorage.setItem("tasks", JSON.stringify(tasks));
    }
  };
  
  /**
   * Carrega colunas e tarefas do localStorage ao montar o componente.
   * @returns {void}
   */
  onMount(() => {
    if (typeof window !== undefined) {
      let localTasks = localStorage.getItem("tasks");
      if (localTasks) tasks = JSON.parse(localTasks);
      
      let localColumns = localStorage.getItem("columns");
      if (localColumns) columns = JSON.parse(localColumns);

      let localCount = localStorage.getItem("count");
      if (localCount) count = JSON.parse(localCount);

      let localColumnsCount = localStorage.getItem("columnsCount");
      if (localColumnsCount) columnsCount = JSON.parse(localColumnsCount);
    }
  });
</script>

<main class="h-screen w-screen bg-gray-50">
  <div class="p-6 h-full overflow-y-auto overflow-x-auto">
    <div class="flex space-x-4 h-full overflow-x-auto overflow-y-auto">
      {#each columns as column}
        <Column 
          bind:count 
          bind:tasks 
          bind:columns={columns} 
          id={column.id} 
          bind:name={column.name} 
          deleteColumn={deleteColumn} 
        />
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
