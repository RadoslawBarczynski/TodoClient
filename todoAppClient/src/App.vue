<script setup lang="ts">
</script>

<template>
  <div class="container py-4">
    <h2 class="text-center mb-4 text-white">Lista zadań</h2>

    <div class="mb-5">
      <i class="fas fa-calendar text-white"></i>
      <span class="font-weight-bold text-white">
        Na dzień
      </span>
        <input
          v-model="filterDate"
          @change="filterTodos"
          type="date"
          class="form-control"
          required
        >
    </div>

    <h5 class="text-center mb-4 text-white">Dodaj zadanie</h5>
    <form @submit.prevent="addTodo" class="row g-3 mb-4">
      <div class="col-md-6">
        <input
          v-model="newDesc"
          class="form-control"
          placeholder="Opis zadania..."
          required
        >
      </div>

      <div class="col-md-4">
        <input
          v-model="newDate"
          type="date"
          class="form-control"
          required
        >
      </div>

      <div class="col-md-2">
        <button class="btn btn-success w-100" type="submit">
          <i class="fa-solid fa-plus"></i>
        </button>
      </div>
    </form>

    <ul class="list-group">
      <li
        v-for="todo in todos"
        :key="todo.id"
        class="list-group-item d-flex justify-content-between align-items-center"
      >

      <!-- Edit template -->
      <template v-if="editId === todo.id">
          <div class="flex-grow-1 me-3">
            <input v-model="editDesc" class="form-control mb-2" required>
            <input v-model="editDate" type="date" class="form-control" required>
          </div>

          <div class="btn-group">
            <button class="btn btn-primary btn-sm" @click="updateTodo(todo.id)">
              <i class="fa-solid fa-floppy-disk"></i>
            </button>
            <button class="btn btn-secondary btn-sm" @click="cancelEdit">
              <i class="fa-solid fa-x"></i>
            </button>
          </div>
        </template>
      <!-- Select template -->
        <template v-else>
          <div class="d-flex gap-2">
            <div class="form-check">
              <input class="form-check-input" @change="checkTodo(todo.id)" type="checkbox" v-model="todo.completedP" id="flexCheckDefault">
            </div>
            <span>
              <strong>{{ todo.description }}</strong>
              <small class="text-muted"> ({{ formatDate(todo.date) }})</small>
            </span>
          </div>

          <div class="btn-group">
            <button class="btn btn-outline-primary btn-sm" @click="startEdit(todo)">
              <i class="fa-solid fa-pen-to-square"></i>
            </button>
            <button class="btn btn-outline-danger btn-sm" @click="deleteTodo(todo.id)">
              <i class="fa-solid fa-trash-can"></i>
            </button>
          </div>
        </template>
      </li>
    </ul>

    <p v-if="todos.length === 0" class="text-center text-muted mt-3">
      Brak zadań tego dnia.
    </p>
  </div>
</template>

<script lang="ts">
interface Todo{
  id: string,
  description: string,
  date: string,
  completedP: boolean
}

interface NewTodo {
  id: string | null;
  description: string;
  date: string;
}

import { defineComponent, ref, onMounted } from "vue";
import { createToaster } from "@meforma/vue-toaster";
import axios from "axios";
const API_URL = "https://todoapii-dzc6etd8drb4a0gx.polandcentral-01.azurewebsites.net";

export default defineComponent({
  name: "App",
  methods:{
    formatDate(dateStr: string): string {
      const [year, month, day] = dateStr.split('-');
      return `${day}-${month}-${year}`;
    },
  },
  setup() {
    const todos = ref<Todo[]>([]);
    const filterDate = ref(new Date().toISOString().slice(0, 10));

    const completedP = ref(false);

    const newDesc = ref("");
    const newDate = ref("");

    const editId = ref<string | null>();
    const editDesc = ref("");
    const editDate = ref("");

    const toaster = createToaster({
      position: "top-right",
      duration: 3000,
    });

    const getAllTodos = async (filterDate: string): Promise<void> => {
      try {
        const response = await axios.get<Todo[]>(`${API_URL}/api/Todo/GetTodos`, {
          params:{
            filterDate: filterDate,
          },
        });

        todos.value = response.data;
      } catch (error) {
        console.error("Błąd pobierania danych:", error);
        toaster.error("Błąd pobierania danych"); 
      }
    };

    const addTodo = async (): Promise<void> => {
      try {
        const jsonTodo: NewTodo = {
          id: null,
          description: newDesc.value,
          date: newDate.value
        };

        const formData = new FormData();
        const jsonString = JSON.stringify(jsonTodo);
        formData.append("jsonTodo", jsonString);

        const response = await axios.post(`${API_URL}/api/Todo/AddTodo`, formData);

        toaster.success("Dodano zadanie"); 

        getAllTodos(filterDate.value);
        
        newDesc.value = "";
        newDate.value = "";

      } catch (error) {
        if (axios.isAxiosError(error)) {
          console.error("Axios error:", error.response?.data);
        } else {
          console.error("Unexpected error:", error);
        }
        toaster.error("Wystąpił błąd podczas dodawania"); 
      }
    };

    const deleteTodo = async (id: string):Promise<void> =>{
        const formData = new FormData();
        const jsonId = id;
        formData.append("jsonId", jsonId)

        const response = await axios.post(`${API_URL}/api/Todo/DeleteTodo`, formData);

        if(response.status == 200){
          toaster.success("Usunięto zadanie");  

          getAllTodos(filterDate.value);
        }else{
          toaster.error("Wystąpił błąd podczas usuwania"); 
        }
    }

    const updateTodo = async(id:string):Promise<void> =>{

      if(editDate.value == "" || editDesc.value == ""){
        toaster.error("Pola nie mogą być puste"); 
        return;
      }

      const jsonTodo: NewTodo = {
          id: id,
          description: editDesc.value,
          date: editDate.value
        };

      const formData = new FormData();
      const jsonString = JSON.stringify(jsonTodo);
      formData.append("jsonTodo", jsonString);

      const response = await axios.post(`${API_URL}/api/Todo/UpdateTodo`, formData);

      if(response.status == 200){
        toaster.success("Zapisano zmiany"); 

        cancelEdit();
        getAllTodos(filterDate.value);
      }else{
        toaster.error("Wystąpił błąd podczas edycji"); 
      }
    }

    const filterTodos = async ():Promise<void> =>{
        getAllTodos(filterDate.value);
    }

    const checkTodo = async (id: string):Promise<void> =>{
        const formData = new FormData();
        const jsonId = id;
        formData.append("jsonId", jsonId)

        const response = await axios.post(`${API_URL}/api/Todo/CheckTodo`, formData);

        if(response.status == 200){
          toaster.success("Wykonano");  
        }else{
          toaster.error("Wystąpił błąd podczas zaznaczania"); 
        }
    }

    //#region Edit view togglers
    const startEdit = (todo: Todo): void => {
      editId.value = todo.id;
      editDesc.value = todo.description;

      const date = new Date(todo.date);
      editDate.value = date.toISOString().split('T')[0] ?? "";
    };

    const cancelEdit = (): void => {
      editId.value = null;
      editDesc.value = "";
      editDate.value = "";
    };
    //#endregion Edit view togglers
    

    onMounted(() =>{
      getAllTodos(filterDate.value);
    })

    return { todos, filterDate, completedP, newDesc, newDate, editId, editDesc, editDate, getAllTodos, addTodo, deleteTodo, updateTodo,
      filterTodos, checkTodo, startEdit, cancelEdit };
  }
});
</script>

<style scoped>
.counter {
  text-align: center;
  margin-top: 50px;
}
button {
  padding: 8px 16px;
  font-size: 16px;
}
</style>