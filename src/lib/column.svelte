<script>
  import { Plus, X } from "lucide-svelte";
  import Task from './task.svelte';

  /** Último valor digitado no input de nova tarefa */
  let lastValue = $state("");

  /** 
   * Propriedades do componente 
   * @type {{ count: number, name: string, id: number, columns: Array<{ id: number, name: string }>, tasks: Array<{ id: number, text: string, columnId: number }>, deleteColumn: (id: number) => void }} 
   */
  let { count = $bindable(), name = $bindable(), id, columns = $bindable(), tasks = $bindable(), deleteColumn } = $props();

  /** Lista de tarefas filtradas por coluna */
  let filteredTasks = $derived(tasks.filter(task => task.columnId === id));

  /** Indica se o nome da coluna está em edição */
  let editingName = $state(false);

  /** Novo nome da coluna */
  let newColunmName = $state(name);

  /**
   * Atualiza o nome da coluna
   * @returns {void}
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
   * Cria uma nova task
   * @param {string} text - Texto da tarefa
   * @returns {{ id: number, text: string, columnId: number }} - Objeto da nova tarefa
   */
  const createTask = (text) => {
    count++;
    return { id: count - 1, text: text, columnId: id };
  };

  /**
   * Adiciona uma nova task à lista
   * @param {string} text - Texto da tarefa a ser adicionada
   * @returns {void}
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
   * Remove uma tarefa da lista
   * @param {number} id - O ID da tarefa a ser removida
   * @returns {void}
   */
  const removeTask = (id) => {
    tasks = tasks.filter(task => task.id !== id);
    if (typeof window !== undefined) {
      localStorage.setItem("tasks", JSON.stringify(tasks));
    }
  };

  /**
   * Inicia o arrasto da task
   * @param {DragEvent} event - Evento de arrastar
   * @param {{ id: number, text: string, columnId: number }} task - Tarefa sendo arrastada
   * @returns {void}
   */
  const handleDragStart = (event, task) => {
    // @ts-ignore
    event.dataTransfer.setData("application/json", JSON.stringify(task));
  };

  /**
   * Permite soltar a tarefa na nova coluna
   * @param {DragEvent} event - Evento de arrastar sobre a coluna
   * @returns {void}
   */
  const handleDragOver = (event) => {
    event.preventDefault();
  };

  /**
   * Solta a tarefa em uma nova coluna
   * @param {DragEvent} event - Evento de soltar
   * @returns {void}
   */
  const handleDrop = (event) => {
    event.preventDefault();
    // @ts-ignore
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
  <div class="space-y-3 flex-grow">
    {#each filteredTasks as task}
      <div role="listitem" draggable="true" ondragstart={(event) => handleDragStart(event, task)}>
        <Task bind:text={task.text} id={task.id} removeTask={removeTask}  />
      </div>
    {/each}
  </div>
</main>
