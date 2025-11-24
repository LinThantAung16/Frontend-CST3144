<script setup>
import { ref, computed, onMounted } from 'vue';
const lessons = ref([]);
const sortAttribute = ref("subject");
const sortOrder = ref("asc");
const searchQuery = ref("");

const cart = ref([]);
const showCart = ref(false);
const toggleCheckout = () => {
  showCart.value = !showCart.value;
};

//get lessons data (mocked for now)
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

// Search and Sort Logic
const filteredLessons = computed(() => {
  let tempLessons = lessons.value.filter((lesson) => {
    return (
      lesson.subject.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
      lesson.location.toLowerCase().includes(searchQuery.value.toLowerCase())
    );
  });

  tempLessons = tempLessons.sort((a, b) => {
    if (sortOrder.value === "asc") {
      return a[sortAttribute.value] > b[sortAttribute.value] ? 1 : -1;
    } else {
      return a[sortAttribute.value] < b[sortAttribute.value] ? 1 : -1;
    }
  });

  return tempLessons;
});

const addToCart = (lesson) => {
  if (lesson.spaces > 0) {
    lesson.spaces--;
    cart.value.push(lesson);
  }
};

const removeFromCart = (cartItem, index) => {
  cartItem.spaces++;
  cart.value.splice(index, 1);
};


onMounted(() => {
  fetchLessons();
});
</script>

<template>
  <div class="container mx-auto p-6 bg-gray-100 min-h-screen font-sans">
    <header class="flex justify-between items-center mb-8 bg-white p-4 rounded shadow">
        <h1 class="text-3xl font-bold text-blue-600">After School Club</h1>
        <button @click="toggleCheckout" :disabled="cart.length === 0 && !showCart"
            class="bg-blue-500 text-white px-6 py-2 rounded hover:bg-blue-600 disabled:bg-gray-300 disabled:cursor-not-allowed transition">
            <span v-if="showCart">Back to Lessons</span>
            <span v-else>
                <i class="fas fa-shopping-cart mr-2"></i> Cart ({{ cart.length }})
            </span>
        </button>
    </header>
    <div v-if="!showCart">
        <div class="bg-white p-4 rounded shadow mb-6 flex flex-wrap gap-4 items-end">
            <div class="flex-1">
                <label class="block font-bold mb-1">Search:</label>
                <input v-model="searchQuery" type="text" placeholder="Subject or Location..."
                    class="border p-2 rounded w-full">
            </div>

            <div>
                <label class="block font-bold mb-1">Sort By:</label>
                <select v-model="sortAttribute" class="border p-2 rounded">
                    <option value="subject">Subject</option>
                    <option value="location">Location</option>
                    <option value="price">Price</option>
                    <option value="spaces">Availability</option>
                </select>
            </div>

            <div>
                <label class="block font-bold mb-1">Order:</label>
                <div class="flex gap-2">
                    <label><input type="radio" value="asc" v-model="sortOrder"> Ascending</label>
                    <label><input type="radio" value="desc" v-model="sortOrder"> Descending</label>
                </div>
            </div>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
            <div v-for="lesson in filteredLessons" :key="lesson.id"
                class="bg-white p-6 rounded shadow hover:shadow-lg transition">
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

                <button @click="addToCart(lesson)" :disabled="lesson.spaces === 0"
                    class="w-full mt-4 bg-green-500 text-white py-2 rounded hover:bg-green-600 disabled:bg-gray-300 transition">
                    Add to Cart
                </button>
            </div>
        </div>
    </div>
    <div v-else class="flex flex-col md:flex-row gap-6">
      <!-- Cart Items -->
      <div class="flex-1">
        <h2 class="text-2xl font-bold mb-4">Your Basket</h2>
        <div v-if="cart.length === 0" class="text-gray-500">Cart is empty.</div>
        <div v-for="(item, index) in cart" :key="index" class="bg-white p-4 rounded shadow mb-4 flex justify-between items-center">
          <div>
            <h3 class="font-bold text-lg">{{ item.subject }}</h3>
            <p class="text-gray-600">{{ item.location }}</p>
            <p class="font-bold">£{{ item.price }}</p>
          </div>
          <button @click="removeFromCart(item, index)" class="text-red-500 hover:text-red-700 underline">Remove</button>
        </div>
      </div>
      </div>
</div>
</template>