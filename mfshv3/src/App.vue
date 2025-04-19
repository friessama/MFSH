<script setup>
import { ref, onMounted } from 'vue'
import PocketBase from 'pocketbase'

const pb = new PocketBase('http://127.0.0.1:8090')
const records = ref([])
const todoInput = ref('')

onMounted(async () => {
  try {
    records.value = await pb.collection('Todo').getFullList()
  } catch (error) {
    console.error('Error fetching records:', error)
  }
})

const hi = "My name is Jeff"
let number = ref(0)
let name = ref("")
let capitalCities = ref(["London", "Paris", "Berlin", "Madrid"])
let prayerTime = ref({})

function increment() {
  number.value++
}

function startInterval() {
  setInterval(increment, 1)
}

function addCity() {
  if (name.value.trim()) {
    capitalCities.value.push(name.value)
    name.value = ""
  }
}

const API_LINK = "https://www.e-solat.gov.my/index.php?r=esolatApi/takwimsolat&period=today&zone=SGR01"

function getPrayerTimes() {
  fetch(API_LINK)
    .then(result => result.json())
    .then(data => {
      prayerTime.value = data.prayerTime[0]
    })
}

function createTodo() {
  const newTodo = {
    item: todoInput.value,
    isDone: false
  }
  pb.collection('Todo').create(newTodo).then(res => {
    records.value.push(res)
    todoInput.value = ''
  })
}

function toggleTodo(record) {
  pb.collection('Todo').update(record.id, { isDone: record.isDone })
    .then(() => {
      console.log('Todo updated successfully')
    })
    .catch(error => {
      console.error('Error updating todo:', error)
    })
}
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-purple-100 via-white to-blue-100 py-12 px-6 flex flex-col items-center gap-10">

    <!-- Counter Card -->
    <div class="bg-white shadow-xl rounded-2xl w-full max-w-md p-6 space-y-4 text-center">
      <h1 class="text-4xl font-extrabold text-indigo-700">Counter</h1>
      <p class="text-5xl font-bold text-gray-800">{{ number }}</p>
      <button @click="startInterval" class="bg-indigo-600 text-white px-6 py-2 rounded-xl hover:bg-indigo-700 transition-all">
        Start Incrementing
      </button>
    </div>

    <!-- Name & Cities -->
    <div class="bg-white shadow-xl rounded-2xl w-full max-w-md p-6 space-y-6">
      <h2 class="text-2xl font-semibold text-gray-800 flex items-center gap-2">👋 {{ hi }}</h2>
      <input type="text" v-model="name" placeholder="Enter a city" class="w-full border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-purple-500" />
      <button @click="addCity" class="w-full bg-green-600 text-white px-4 py-2 rounded-lg hover:bg-green-700 transition">Add City</button>
      <ol class="list-decimal list-inside text-gray-700 space-y-1 pl-4">
        <li v-for="city in capitalCities" :key="city">{{ city }}</li>
      </ol>
    </div>

    <!-- Prayer Times -->
    <div class="bg-white shadow-xl rounded-2xl w-full max-w-xl p-6 space-y-6">
      <div class="text-center space-y-2">
        <h1 class="text-3xl font-bold text-gray-800">📿 Prayer Times</h1>
        <button @click="getPrayerTimes" class="bg-blue-600 text-white px-6 py-2 rounded-lg hover:bg-blue-700 transition">
          Load Prayer Times
        </button>
      </div>
      <div class="grid grid-cols-2 gap-4 text-lg text-gray-700 font-medium">
        <div>Fajr</div><div class="text-right">{{ prayerTime.fajr || '--:--' }}</div>
        <div>Dhuha</div><div class="text-right">{{ prayerTime.dhuha || '--:--' }}</div>
        <div>Dhuhr</div><div class="text-right">{{ prayerTime.dhuhr || '--:--' }}</div>
        <div>Asr</div><div class="text-right">{{ prayerTime.asr || '--:--' }}</div>
        <div>Maghrib</div><div class="text-right">{{ prayerTime.maghrib || '--:--' }}</div>
        <div>Isha</div><div class="text-right">{{ prayerTime.isha || '--:--' }}</div>
      </div>
    </div>

    <!-- Todo List -->
    <div class="bg-white shadow-xl rounded-2xl w-full max-w-md p-6 space-y-4">
      <h1 class="text-3xl font-bold text-gray-800 text-center">📝 Todo List</h1>
      <div class="flex items-center gap-2">
        <input type="text" v-model="todoInput" placeholder="Add a new todo" class="flex-1 border border-gray-300 rounded-lg px-4 py-2 focus:outline-none focus:ring-2 focus:ring-blue-500" />
        <button @click="createTodo" class="bg-blue-600 text-white px-4 py-2 rounded-lg hover:bg-blue-700 transition">Add</button>
      </div>
        <ul class="space-y-3">
          <li v-for="record in records" :key="record.id" class="flex items-start gap-3 bg-gray-50 p-3 rounded-lg shadow-sm">
            <div class="flex items-center pt-1">
              <input type="checkbox" v-model="record.isDone" @change="toggleTodo(record)" class="h-5 w-5 text-blue-600 rounded border-gray-300 focus:ring-blue-500" />
            </div>
            <div :class="{ 'line-through text-gray-400': record.isDone }" class="text-lg">
              {{ record.item }}
            </div>
          </li>
        </ul>

    </div>

  </div>
</template>

<style scoped>
</style>
