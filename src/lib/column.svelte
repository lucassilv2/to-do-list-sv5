<script>
  // @ts-nocheck
  import { Plus, X } from "lucide-svelte";
  import Task from './task.svelte';

  let lastValue = $state("");
  let { count = $bindable(), name, id, columns, tasks = $bindable(), deleteColumn } = $props();
  let filteredTasks = $derived(tasks.filter(task => task.columnId === id));
  let editingName = $state(false);
  let newColunmName = $state(name);

  /**
   * Atualiza o nome da coluna
   */
  const updateName = () => {
    name = newColunmName;
    editingName = false;
    columns = columns.map(column => column.id === id ? { ...column, name: newColunmName } : column);
    if (typeof window !== undefined) {
      localStorage.setItem("columns", JSON.stringify(columns));
    }
  };

  /**
   * Criação de uma nova task
   */
  const createTask = (text) => {
    count++;
    return { id: count - 1, text: text, columnId: id };
  };

  /**
   * Adiciona uma nova task
   */
  const addTasks = (text) => {
    if (text.trim() === "") return;
    tasks.push(createTask(text));
    if (typeof window !== undefined) {
      localStorage.setItem("tasks", JSON.stringify(tasks));
    }
    lastValue = "";
  };

  /**
   * Remove uma task
   */
  const removeTask = (id) => {
    tasks = tasks.filter(task => task.id !== id);
    if (typeof window !== undefined) {
      localStorage.setItem("tasks", JSON.stringify(tasks));
    }
  };

  /**
   * Atualiza uma task
   */
  const updateTask = (id, newText) => {
    tasks = tasks.map(task => task.id === id ? { ...task, text: newText } : task);
    if (typeof window !== undefined) {
      localStorage.setItem("tasks", JSON.stringify(tasks));
    }
  };

  /**
   * Inicia o arrasto da task
   */
  const handleDragStart = (event, task) => {
    event.dataTransfer.setData("application/json", JSON.stringify(task));
  };

  /**
   * Permite soltar a task na nova coluna
   */
  const handleDragOver = (event) => {
    event.preventDefault();
  };

  /**
   * Solta a task em uma nova coluna
   */
   const handleDrop = (event) => {
    event.preventDefault();
    let taskData = JSON.parse(event.dataTransfer.getData("application/json"));

    if (taskData.columnId !== id) {
      tasks = tasks.map(task => 
        task.id === taskData.id ? { ...task, columnId: id } : task
      );

      if (typeof window !== undefined) {
        localStorage.setItem("tasks", JSON.stringify(tasks));
      }
    }
  };

</script>

<main class="bg-gray-100 p-6 rounded-lg shadow-md flex flex-col flex-shrink-0 w-80 max-h-100 overflow-y-auto" 
      ondragover={handleDragOver} 
      ondrop={handleDrop}>
  <div class="flex justify-between items-center mb-4">
    {#if editingName}
      <input 
        type="text" 
        bind:value={newColunmName} 
        class="border border-gray-300 p-2 rounded flex-grow"
        onkeydown={(e) => { if (e.key === 'Enter') updateName() }}
      />
    {:else}
      <h2 
        class="text-xl font-bold text-gray-700 cursor-pointer w-full" 
        ondblclick={() => editingName = true}
      >
        {name}
      </h2>
    {/if}
    <button class="bg-red-500 text-white p-2 rounded hover:bg-red-700" onclick={() => deleteColumn(id)}><X size={18} /></button>
  </div>
  <hr class="border-t border-gray-300 mb-4" />
  <div class="flex space-x-2 mb-4">
    <input type="text" bind:value={lastValue} class="border border-gray-300 p-2 rounded flex-grow" placeholder="Nova tarefa..." />
    <button class="bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600" onclick={() => addTasks(lastValue)}><Plus size={18} /></button>
  </div>

  <!-- Lista de tarefas arrastáveis -->
  <div class="space-y-3 flex-grow">
    {#each filteredTasks as task}
      <div role="listitem" draggable="true" ondragstart={(event) => handleDragStart(event, task)}>
        <Task bind:text={task.text} removeTask={removeTask} updateTask={updateTask} bind:columnId={id} />
      </div>
    {/each}
  </div>
</main>
