<template>
  <div class="home-page">
    <h1>Welcome to My Shop</h1>
    <input type="text" v-model="searchQuery" placeholder="Search products" />
    <select v-model="sortBy">
      <option value="name">Sort by Name</option>
      <option value="price">Sort by Price</option>
    </select>
    <ul class="product-list">
      <li v-for="product in filteredProducts" :key="product.id">
        <img :src="product.image" :alt="product.name" />
        <h2>{{ product.name }}</h2>
        <p>{{ product.price | currency }}</p>
        <button @click="addToWishlist(product)">Add to Wishlist</button>
        <button @click="addToComparisonList(product)">Add to Compare</button>
        <button @click="login">Login</button>
        <button @click="logout">Logout</button>
      </li>
    </ul>
    <router-link to="/wishlist"
      >My Wishlist ({{ totalWishlistedItems }})</router-link
    >
  </div>
</template>

<script>
import { ref, computed, reactive, onMounted } from "vue";
import { useRouter } from "vue-router";
import { useStore } from "vuex";
import { jwtDecode } from "jwt-decode";

export default {
  name: "Home",
  setup() {
    const products = reactive([]); // Initialize products as a reactive array
    const wishlistedProducts = ref([]);
    const comparisonList = ref([]);
    const searchQuery = ref("");
    const sortBy = ref("name");
    const isLoggedIn = ref(!!localStorage.getItem("token"));
    const userId = ref(null);
    const router = useRouter();
    const store = useStore();

    const MAX_COMPARISON_ITEMS = 4;

    onMounted(async () => {
      try {
        const res = await fetch("https://fakestoreapi.com/products");
        if (res.ok) {
          products.value = await res.json();
        } else {
          console.error("Failed to fetch products:", res.statusText);
        }
      } catch (error) {
        console.error("Failed to fetch products:", error);
      }

      const token = localStorage.getItem("token");
      if (token) {
        userId.value = jwtDecode(token).userId;
        if (userId.value) {
          comparisonList.value =
            JSON.parse(localStorage.getItem(`comparison-${userId.value}`)) ||
            [];
        }
      }
    });

    const totalWishlistedItems = computed(
      () => wishlistedProducts.value.length
    );

    const filteredProducts = computed(() => {
      return products.value
        .filter(
          (product) =>
            product &&
            product.name &&
            product.name.toLowerCase().includes(searchQuery.value.toLowerCase())
        )
        .sort((a, b) =>
          (a[sortBy.value] || 0) > (b[sortBy.value] || 0) ? 1 : -1
        );
    });

    const addToWishlist = (product) => {
      if (!wishlistedProducts.value.includes(product)) {
        wishlistedProducts.value.push(product);
      }
    };

    const addToComparisonList = (product) => {
      if (!isLoggedIn.value) {
        router.push("/login");
        return;
      }

      if (comparisonList.value.length < MAX_COMPARISON_ITEMS) {
        const existingItem = comparisonList.value.find(
          (item) => item.id === product.id
        );
        if (!existingItem) {
          comparisonList.value.push(product);
          saveComparisonList();
        }
      } else {
        alert("Comparison list is full. You can only add up to 4 items.");
      }
    };

    const saveComparisonList = () => {
      try {
        if (userId.value) {
          localStorage.setItem(
            `comparison-${userId.value}`,
            JSON.stringify(comparisonList.value)
          );
        }
      } catch (error) {
        console.error("Failed to save comparison list:", error);
      }
    };

    const login = async () => {
      // Use Vuex action for login
      try {
        await store.dispatch("login"); // Example action
        isLoggedIn.value = true;
        router.push("/home");
      } catch (error) {
        console.error("Login failed:", error);
      }
    };

    const logout = async () => {
      // Use Vuex action for logout
      try {
        await store.dispatch("logout"); // Example action
        localStorage.removeItem("token");
        isLoggedIn.value = false;
        router.push("/login");
      } catch (error) {
        console.error("Logout failed:", error);
      }
    };

    return {
      searchQuery,
      sortBy,
      filteredProducts,
      totalWishlistedItems,
      addToWishlist,
      addToComparisonList,
      login,
      logout,
    };
  },
};
</script>

<style scoped>
.home-page {
  background-color: white;
}

.product-list {
  list-style-type: none;
  padding: 0;
}

.product-list li {
  margin-bottom: 20px;
}

.product-list img {
  max-width: 150px;
  height: auto;
}

.product-list h2 {
  font-size: 18px;
  margin: 10px 0;
}

.product-list p {
  color: #888;
}

button {
  margin-right: 10px;
}
</style>
