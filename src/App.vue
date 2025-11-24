<script setup>
    import { ref, onMounted } from 'vue';
    const lessons = ref([]);
    const fetchLessons = async () => {
  try {
    lessons.value = [
      { id: 101, subject: "Math", location: "London", price: 100, spaces: 5, icon: "fas fa-calculator" },
      { id: 102, subject: "English", location: "York", price: 80, spaces: 5, icon: "fas fa-book" },
      { id: 103, subject: "Music", location: "Bristol", price: 90, spaces: 5, icon: "fas fa-music" },
      { id: 104, subject: "Science", location: "London", price: 120, spaces: 5, icon: "fas fa-flask" },
      { id: 105, subject: "Art", location: "Oxford", price: 95, spaces: 5, icon: "fas fa-palette" },
      { id: 106, subject: "History", location: "London", price: 75, spaces: 5, icon: "fas fa-landmark" }
    ];
  } catch (error) {
    console.error("Error fetching lessons:", error);
  }
};
onMounted(() => {
  fetchLessons();
});
</script>

<template>
    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
        <div v-for="lesson in lessons" :key="lesson.id" class="bg-white p-6 rounded shadow hover:shadow-lg transition">
          <div class="flex justify-between items-start">
            <div>
              <p class="text-gray-500 text-sm">Subject:</p>
              <h2 class="text-2xl font-bold mb-2">{{ lesson.subject }}</h2>
            </div>
            <i :class="lesson.icon" class="text-4xl text-blue-400"></i>
          </div>
          
          <p class="text-gray-700"><i class="fas fa-map-marker-alt mr-2"></i> {{ lesson.location }}</p>
          <p class="text-gray-700 mt-1"><i class="fas fa-tag mr-2"></i> Price: £{{ lesson.price }}</p>
          <p class="mt-1 font-bold" :class="lesson.spaces > 0 ? 'text-green-600' : 'text-red-600'">
            Spaces: {{ lesson.spaces }}
          </p>

          <button 
            class="w-full mt-4 bg-green-500 text-white py-2 rounded hover:bg-green-600 disabled:bg-gray-300 transition">
            Add to Cart
          </button>
        </div>
      </div>
</template>