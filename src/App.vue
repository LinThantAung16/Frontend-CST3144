<script setup>
import { ref, computed, onMounted, watch } from 'vue';
const serverURL = "https://cst3144-backend-drwa.onrender.com";
const lessons = ref([]);
const sortAttribute = ref("subject");
const sortOrder = ref("asc");
const searchQuery = ref("");

const cart = ref([]);
const showCart = ref(false);

const checkoutForm = ref({
  name: "",
  phone: ""
});

const toggleCheckout = () => {
  showCart.value = !showCart.value;
};

//fetch lesson from backend
const fetchLessons = async () => {
  try {
     const response = await fetch(`${serverURL}/lessons`);
      const data = await response.json();
      lessons.value = data;
     if(lessons == null){
           lessons.value = [
            { id: 101, subject: "Math", location: "London", price: 100, spaces: 5, icon: "fas fa-calculator" },
            { id: 102, subject: "English", location: "York", price: 80, spaces: 5, icon: "fas fa-book" },
            { id: 103, subject: "Music", location: "Bristol", price: 90, spaces: 5, icon: "fas fa-music" },
            { id: 104, subject: "Science", location: "London", price: 120, spaces: 5, icon: "fas fa-flask" },
            { id: 105, subject: "Art", location: "Oxford", price: 95, spaces: 5, icon: "fas fa-palette" },
            { id: 106, subject: "History", location: "London", price: 75, spaces: 5, icon: "fas fa-landmark" }
        ];
      }
  } catch (error) {
    console.error("Error fetching lessons:", error);
  }
};

//Search logic implemant with backend
const performSearch = async (query) => {
  try {
    const response = await fetch(`${serverURL}/search?q=${query}`);
    if (!response.ok) throw new Error("Failed to search");
    const data = await response.json();
    lessons.value = data; // Update the list with ONLY the matching results
  } catch (error) {
    console.error("Error searching lessons:", error);
  }
};

watch(searchQuery, (newQuery) => {
  if (newQuery.trim().length > 0) {
    // If there is text, go to the backend to filter
    performSearch(newQuery);
  } else {
    // If text is cleared, fetch all lessons again
    fetchLessons();
  }
});

// Sort Logic
const filteredLessons = computed(() => {
  let tempLessons = lessons.value;

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

    const existingItem = cart.value.find(item => item.lesson.id === lesson.id);

    if (existingItem) {
      existingItem.quantity++;
    } else {
      cart.value.push({ lesson: lesson, quantity: 1 });
    }
  }
};

//Increase Qty in Cart
const increaseCartQuantity = (item) => {
  
  if (item.lesson.spaces > 0) {
    item.quantity++;
    item.lesson.spaces--;
  }
};

//Decrease Qty in Cart
const decreaseCartQuantity = (item, index) => {
  item.quantity--;
  item.lesson.spaces++;

  // If quantity drops to 0, remove item from cart
  if (item.quantity <= 0) {
    cart.value.splice(index, 1);
  }
};

const removeFromCart = (cartItem, index) => {
  cartItem.lesson.spaces += cartItem.quantity;
  cart.value.splice(index, 1);
};

//Submit order to back end
const submitOrder = async () => {
  const lessonIDs = [];
  
  cart.value.forEach(item => {
    for (let i = 0; i < item.quantity; i++) {
      // FIX IS HERE: Access 'item.lesson.id' instead of 'item.id'
      lessonIDs.push(item.lesson.id); 
    }
  });

  const newOrder = {
    name: checkoutForm.value.name,
    phone: checkoutForm.value.phone,
    lessonIDs: lessonIDs, // Use the flattened array
    spaces: lessonIDs.length // Use the total count of seats, not just unique items
  };

  console.log("Submitting Order Payload:", newOrder);
  try {
    // UNCOMMENT when Node.js server is ready
    
    await fetch(`${serverURL}/orders`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(newOrder)
    });

    // Update spaces for each lesson (PUT request) [cite: 436]
    for (const item of cart.value) {
       await fetch(`${serverURL}/lessons/${item.lesson.id}`, {
          method: "PUT",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ spaces: item.lesson.spaces }) 
       });
    }
  

    alert("Order Submitted Successfully!");
    // orderSubmitted.value = true;
    cart.value = []; // Clear cart
    checkoutForm.value.name = "";
    checkoutForm.value.phone = "";
    
  } catch (error) {
    console.error("Error submitting order:", error);
  }
};

//name and phone validation
const isFormValid = computed(() => {
  const nameRegex = /^[A-Za-z\s]+$/;
  const phoneRegex = /^[0-9]+$/;
  return nameRegex.test(checkoutForm.value.name) && phoneRegex.test(checkoutForm.value.phone);
});

// Helper function to determine badge color
const getAvailabilityColor = (spaces) => {
    if (spaces === 0) return 'bg-gray-400 text-gray-800'; // Sold Out
    if (spaces < 3) return 'bg-red-400 text-red-900'; // Low Stock
    if (spaces < 5) return 'bg-yellow-400 text-yellow-900'; // Med Stock
    return 'bg-green-400 text-green-900'; // Good Stock
};


onMounted(() => {
  fetchLessons();
});
</script>

<template>
  <!-- Background is bright yellow-orange -->
  <div class="container mx-auto p-6 bg-[#FFFAF0] min-h-screen font-sans border-x-4 border-black max-w-6xl">
    
    <!-- HEADER -->
    <header class="flex flex-col md:flex-row justify-between items-center mb-10 bg-[#FFD700] p-6 border-4 border-black shadow-[8px_8px_0px_0px_rgba(0,0,0,1)] rounded-xl">
      <h1 class="text-4xl font-black text-black tracking-tighter uppercase transform -rotate-2">

        <i class="fas fa-shapes mr-3 text-white text-shadow-black"></i>After School HUB
      </h1>
      
      <button 
        @click="toggleCheckout"
        :disabled="cart.length === 0 && !showCart"
        class="mt-4 md:mt-0 bg-white text-black px-6 py-3 rounded-lg border-4 border-black font-bold shadow-[4px_4px_0px_0px_rgba(0,0,0,1)] hover:translate-x-[2px] hover:translate-y-[2px] hover:shadow-[2px_2px_0px_0px_rgba(0,0,0,1)] active:translate-x-[4px] active:translate-y-[4px] active:shadow-none transition-all disabled:opacity-50 disabled:cursor-not-allowed">
        <span v-if="showCart">
          <i class="fas fa-arrow-left mr-2"></i> GO BACK
        </span>
        <span v-else>
          <i class="fas fa-shopping-basket mr-2"></i> SHOPPING CART ({{ cart.length }})
        </span>
      </button>
    </header>

    <!-- LESSONS VIEW -->
    <div v-if="!showCart">
      
      <!-- CONTROLS BAR -->
      <div class=" p-6 rounded-xl border-4 border-black shadow-[8px_8px_0px_0px_rgba(0,0,0,1)] mb-10">
        <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
          <!-- Search -->
          <div class="flex flex-col">
            <label class="font-black text-black mb-2 uppercase">Search:</label>
            <input 
              v-model="searchQuery" 
              type="text" 
              class="border-4 border-black p-3 rounded-lg focus:outline-none focus:ring-4 focus:ring-yellow-400 font-bold"
              placeholder="Search by Subject or Location...">
          </div>
          
          <!-- Sort -->
          <div class="flex flex-col">
            <label class="font-black text-black mb-2 uppercase">Sort By:</label>
            <select v-model="sortAttribute" class="border-4 border-black p-3 rounded-lg font-bold bg-white cursor-pointer">
              <option value="subject">Subject</option>
              <option value="location">Location</option>
              <option value="price">Price</option>
              <option value="spaces">Availability</option>
            </select>
          </div>

          <!-- Order -->
          <div class="flex flex-col">
            <label class="font-black text-black mb-2 uppercase">Order:</label>
            <div class="flex bg-white border-4 border-black rounded-lg overflow-hidden h-full">
                <label class="flex-1 flex items-center justify-center cursor-pointer hover:bg-gray-100 border-r-4 border-black font-bold">
                    <input type="radio" value="asc" v-model="sortOrder" class="mr-2 accent-black h-5 w-5"> Asc
                </label>
                <label class="flex-1 flex items-center justify-center cursor-pointer hover:bg-gray-100 font-bold">
                    <input type="radio" value="desc" v-model="sortOrder" class="mr-2 accent-black h-5 w-5"> Desc
                </label>
            </div>
          </div>
        </div>
      </div>

      <!-- CARDS GRID -->
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
        <div v-for="lesson in filteredLessons" :key="lesson.id" 
             class="bg-white p-6 rounded-xl border-4 border-black shadow-[8px_8px_0px_0px_rgba(0,0,0,1)] hover:-translate-y-2 transition-transform duration-200 flex flex-col relative overflow-hidden group">
          
          <!-- Decorative Circle Background -->
          <div class="absolute -top-10 -right-10 w-32 h-32 bg-yellow-200 rounded-full border-4 border-black z-0 group-hover:bg-yellow-300 transition-colors"></div>

          <div class="relative z-10 flex justify-between items-start mb-4">
            <h2 class="text-3xl font-black text-black transform -rotate-1">{{ lesson.subject }}</h2>
            <i :class="lesson.icon" class="text-3xl text-black bg-white p-2 border-2 border-black rounded-lg shadow-[3px_3px_0px_0px_rgba(0,0,0,1)]"></i>
          </div>
          
          <div class="relative z-10 space-y-3 mb-6 font-bold text-gray-700">
            <p><i class="fas fa-map-pin mr-2 text-red-500"></i> {{ lesson.location }}</p>
            <p><i class="fas fa-pound-sign mr-2 text-green-600"></i> {{ lesson.price }}</p>
                  <div class="flex items-center mt-2">
                <i class="fas fa-users mr-2 text-blue-500"></i>
                <span class="px-3 py-1 rounded-full border-2 border-black font-black text-sm uppercase shadow-[2px_2px_0px_0px_rgba(0,0,0,1)]"
                      :class="getAvailabilityColor(lesson.spaces)">
                    <span v-if="lesson.spaces === 0">Sold Out</span>
                    <span v-else>{{ lesson.spaces }} Spaces Left</span>
                </span>
            </div>
          </div>

          <button 
            @click="addToCart(lesson)" 
            :disabled="lesson.spaces === 0"
            class="relative z-10 mt-auto w-full py-3 rounded-lg font-black text-white border-4 border-black shadow-[4px_4px_0px_0px_rgba(0,0,0,1)] hover:shadow-[2px_2px_0px_0px_rgba(0,0,0,1)] hover:translate-x-[2px] hover:translate-y-[2px] active:translate-x-[4px] active:translate-y-[4px] active:shadow-none transition-all"
            :class="lesson.spaces > 0 ? 'bg-[#FF4081]' : 'bg-gray-400 cursor-not-allowed'">
            {{ lesson.spaces > 0 ? 'ADD TO BASKET' : 'SOLD OUT!' }}
          </button>
        </div>
      </div>
    </div>

    <!-- CART VIEW -->
    <div v-else class="flex flex-col lg:flex-row gap-8">
      <div class="flex-1 bg-white p-8 rounded-xl border-4 border-black shadow-[8px_8px_0px_0px_rgba(0,0,0,1)]">
        <h2 class="text-3xl font-black mb-6 uppercase border-b-4 border-black pb-4">Your Backpack</h2>
        <div v-if="cart.length === 0" class="text-center font-bold text-xl text-gray-400 py-10">
            It's empty in here!
        </div>
        <div v-for="(item, index) in cart" :key="index" class="flex justify-between items-center mb-4 p-4 bg-blue-50 border-2 border-black rounded-lg">
          <div>
            <h3 class="font-black text-xl">{{ item.lesson.subject }}</h3>
            <p class="font-bold text-sm">{{ item.lesson.location }}</p>
          </div>
          <div class="flex items-center gap-4">
            <span class="font-black text-xl">£{{ item.lesson.price }}</span>
           <!-- QUANTITY CONTROLS -->
          <div class="flex items-center gap-3 mr-6">
            <button 
                @click="decreaseCartQuantity(item, index)" 
                class="bg-gray-200 text-gray-700 w-8 h-8 rounded-full hover:bg-gray-300 font-bold">
                -
            </button>
            <span class="font-bold text-lg">{{ item.quantity }}</span>
            <button 
                @click="increaseCartQuantity(item)" 
                :disabled="item.lesson.spaces === 0"
                class="bg-gray-200 text-gray-700 w-8 h-8 rounded-full hover:bg-gray-300 font-bold disabled:opacity-50 disabled:cursor-not-allowed">
                +
            </button>
          </div>

          <button @click="removeFromCart(item, index)" class="text-red-500 hover:text-red-700">
            <i class="fas fa-trash-alt"></i>
          </button>
        </div>
          </div>
        </div>
      

      <div class="w-full md:w-1/3 bg-[#98FB98] p-6 rounded-xl border-4 border-black shadow-[8px_8px_0px_0px_rgba(0,0,0,1)] h-fit">
        <h2 class="text-2xl font-black mb-6 uppercase">Checkout</h2>
        <div class="mb-4">
          <label class="block font-bold mb-2">Student Name:</label>
          <input v-model="checkoutForm.name" type="text" class="w-full border-4 border-black p-3 rounded-lg focus:outline-none focus:ring-4 focus:ring-white">
        </div>
        <div class="mb-6">
          <label class="block font-bold mb-2">Phone Number:</label>
          <input v-model="checkoutForm.phone" type="text" class="w-full border-4 border-black p-3 rounded-lg focus:outline-none focus:ring-4 focus:ring-white">
        </div>
        <button 
          @click="submitOrder" 
          :disabled="!isFormValid || cart.length === 0"
          class="w-full bg-black text-white py-4 rounded-lg font-black text-xl hover:bg-gray-800 disabled:bg-gray-500 border-4 border-white transition-all">
          CHECKOUT
        </button>
         <p v-if="!isFormValid" class="text-red-600 font-bold text-sm mt-3 bg-white p-2 border-2 border-black rounded">
          * Name: Letters only<br>* Phone: Numbers only
        </p>
      </div>
      </div>
    </div>


</template>