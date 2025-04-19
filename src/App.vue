<script setup>
import HelloWorld from './components/HelloWorld.vue'
import {ref, onMounted} from 'vue'
import PocketBase from 'pocketbase'
const pb = new PocketBase('http://127.0.0.1:8090')
const records = ref([])
const todoInput = ref("")
const hi = "my name is"
const isTrue = ref(true)
const solatApiLink = "https://www.e-solat.gov.my/index.php?r=esolatApi/takwimsolat&period=today&zone=SGR01"
let name = ref("")
let number = ref(0);
let prayerTime = ref({})
let capitalCities = ref(["England","Paris","Rome","Kuala Lumpur"])
function increment(){
  number.value++;
  console.log(number)
  }
function startInterval(){
  setInterval(increment,1000)
}

function addToList(e){
  if(name.value === ""){
    return
  }
  if(e.keyCode !== 13){
    return
  }
  capitalCities.value.push(name.value)
  name.value = ""
}

function getPrayerTime(){
  fetch(solatApiLink).then(result => result.json()).then(data=>{
    prayerTime.value = data.prayerTime[0]
    console.log(prayerTime.value)
  }
  )
}

function createTodo(){
  const newTodo = {
    item: todoInput.value,
    isDone: false
  }
  pb.collection('todos').create(newTodo).then(res => {
    records.value.push(res)
    todoInput.value = ""
  })
}

onMounted(()=>{
  getPrayerTime()
  pb.collection('todos').getFullList().then(res => {
    records.value = res
  })
})

</script>

<template>
  <div class="max-w-4xl mx-auto p-6 bg-gradient-to-br from-indigo-50 to-blue-50 rounded-xl shadow-lg">
    <!-- Counter section -->
    <div class="mb-8 flex items-center gap-4">
      <h1 class="text-5xl font-bold text-indigo-600">{{ number }}</h1>
      <button 
        @click="startInterval"
        class="px-4 py-2 bg-indigo-600 text-white rounded-lg hover:bg-indigo-700 transition duration-300 shadow-md"
      >
        Increment
      </button>
    </div>
    <div>
      <h1 class="text-2xl font-semibold text-gray-800 mb-4">Todos</h1>
      <input 
        type="text" 
        v-model="todoInput"
        class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition duration-200 mb-4"
        placeholder="Enter text here..."
      >      
      <button @click="createTodo" class="bg-indigo-400">Create Todo</button>
      <ul class="space-y-2">
        <li v-for="record in records" :key="record.id" class="flex items-center justify-between">
          <span class="text-lg font-medium text-gray-800">{{ record.item }}</span>
          <button @click="pb.collection('todos').delete(record.id)" class="text-red-500 hover:text-red-700 transition duration-300">
            Delete
</button></li></ul>            
    </div>
    <!-- Input section -->
    <div class="mb-8">
      <input 
        v-on:keyup="addToList" 
        type="text" 
        v-model="name"
        class="w-full p-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 transition duration-200 mb-4"
        placeholder="Enter text here..."
      >
      <div class="flex gap-4 items-baseline">
        <h1 class="text-2xl font-semibold text-gray-800">{{ hi }}</h1>
        <h2 class="text-xl font-medium text-indigo-500">{{ name }}</h2>
      </div>
    </div>

    <!-- List section -->
    <div class="mb-8 bg-white p-6 rounded-lg shadow-md">
      <h3 class="text-lg font-semibold text-gray-700 mb-3">Capital Cities</h3>
      <ol class="list-decimal pl-6 space-y-2">
        <li 
          v-for="city in capitalCities" 
          class="text-gray-700 hover:text-indigo-600 transition"
        >
          {{ city }}
        </li>
      </ol>
    </div>

    <!-- Toggle section -->
    <div class="mb-8 flex items-center gap-4">
      <h1 
        v-if="isTrue" 
        class="text-xl font-bold text-indigo-600 transition-opacity duration-300"
      >
        England is my city
      </h1>
      <button 
        @click="isTrue = !isTrue"
        class="px-4 py-2 bg-indigo-500 text-white rounded-lg hover:bg-indigo-600 transition duration-300 shadow-md"
      >
        Toggle Text
      </button>
    </div>

    <!-- Prayer times section -->
    <div class="bg-white rounded-xl overflow-hidden shadow-lg">
      <div class="bg-indigo-600 p-4 flex justify-between items-center">
        <h3 class="text-xl font-semibold text-white">Prayer Times</h3>
        <button 
          @click="getPrayerTime"
          class="px-4 py-2 bg-white text-indigo-600 font-medium rounded-lg hover:bg-gray-100 transition duration-300 shadow-md"
        >
          Refresh Times
        </button>
      </div>
      
      <div class="p-4">
        <table class="w-full">
          <thead>
            <tr class="border-b border-gray-200">
              <th class="py-3 text-indigo-600 font-semibold">Fajr</th>
              <th class="py-3 text-indigo-600 font-semibold">Dhuha</th>
              <th class="py-3 text-indigo-600 font-semibold">Dhuhr</th>
              <th class="py-3 text-indigo-600 font-semibold">Asr</th>
              <th class="py-3 text-indigo-600 font-semibold">Maghrib</th>
              <th class="py-3 text-indigo-600 font-semibold">Isha</th>
            </tr>
          </thead>
          <tbody>
            <tr class="text-center hover:bg-indigo-50 transition duration-200">
              <td class="py-4 px-2">{{ prayerTime.fajr }}</td>
              <td class="py-4 px-2">{{ prayerTime.dhuha }}</td>
              <td class="py-4 px-2">{{ prayerTime.dhuhr }}</td>
              <td class="py-4 px-2">{{ prayerTime.asr }}</td>
              <td class="py-4 px-2">{{ prayerTime.maghrib }}</td>
              <td class="py-4 px-2">{{ prayerTime.isha }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
