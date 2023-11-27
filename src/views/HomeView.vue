<template>
  <div class="home">
    <!-- Hero Section -->
    <header class="hero-section">
      <div class="container text-center">
        <h1>Welcome to DealDetector</h1>
        <p>Find the best deals on electronics across the top e-commerce sites from Romania!</p>
        <!-- Search Bar -->
        <div class="search-bar mt-4">
          <input type="text" class="form-control" placeholder="Search for category..." v-model="searchQuery">
          <button class="btn search-btn">Search</button>
        </div>
      </div>
    </header>

    <!-- Featured Categories -->
    <section class="featured-categories" v-if="filteredCategories.length">
      <div class="container">
        <h2>Featured Categories</h2>
        <div class="row">
          <div class="col-md-4" v-for="category in filteredCategories" :key="category._id">
            <div class="category-card">
              <img :src="category.imageUrl" class="card-img-top" :alt="category.name">
              <div class="card-body">
                <h5 class="card-title">{{ category.name }}</h5>
                <router-link :to="`/categories/${category.name}`" class="btn shop-btn">Shop Now</router-link>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
    <div v-else class="text-center">
      <p>No categories found....</p>
    </div>
  </div>
</template>




<script>

import { ref, computed, onMounted } from 'vue';


export default {
  name: 'HomeView',
  setup() {
    const categories = ref([]);
    const searchQuery = ref("");

    onMounted(async () => {
      try {
        const response = await fetch('http://localhost:3000/api/categories');
        if (!response.ok) {
          throw new Error('Server responded with an error');
        }
        categories.value = await response.json();
      } catch (error) {
        console.error('Failed to load categories:', error);
      }
    });

    const filteredCategories = computed(() => {
      return categories.value.filter(category =>
        category.name.toLowerCase().includes(searchQuery.value.toLowerCase())
      );
    });

    return {
      categories,
      searchQuery,
      filteredCategories
    };
  }
};
</script>

<style scoped>

.featured-categories h2 {
    text-align: center;
    margin-bottom: 2rem;
}
.category-card {
    margin-bottom: 2rem;
    border: none;
    transition: transform .2s, box-shadow .2s;
    box-shadow: 0px 6px 15px rgba(0, 0, 0, 0.1);
    border-radius: 15px;
    overflow: hidden;
}
.card-title {
    font-size: 1.2rem;
    color: #333;
    text-align: center;
    margin-bottom: 1rem;
    font-weight: bold;
    text-transform: uppercase;
}
.category-card:hover {
    transform: translateY(-5px);
    box-shadow: 0px 12px 20px rgba(0, 0, 0, 0.2);
}

.shop-btn {
    display: inline-block;
    background-color: #ffffff;
    color: #0d6efd;
    padding: 10px 20px;
    font-size: 0.9rem;
    border: 2px solid #0d6efd;
    border-radius: 25px;
    transition: all 0.3s ease;
    text-transform: uppercase;
    font-weight: bold;
    text-decoration: none;
    cursor: pointer;
    box-shadow: 0 2px 5px rgba(0,0,0,0.2);
    margin-top: 10px;
}

.shop-btn:hover {
    background-color: #0d6efd;
    color: #ffffff;
    box-shadow: 0 4px 8px rgba(0,0,0,0.3);
}

.card:hover {
    transform: scale(1.05);
}

.hero-section {
    padding: 200px 0;
    background-image: url('../assets/images/hero-image.png');
    background-size: cover;
    background-position: center;
    color: white;
    text-shadow: 0px 0px 3px rgba(0, 0, 0, 0.6);
}

.hero-section h1 {
    font-size: 3.5rem;
    margin-bottom: 1rem;
}

h2 {
  text-shadow: 0px 0px 3px rgba(0, 0, 0, 0.6);
  font-size: 3.5rem;
  margin-bottom: 1rem;
}

.search-bar {
    max-width: 500px;
    margin: 1rem auto;
    display: flex;
    border-radius: 25px;
    overflow: hidden;
}

.search-bar input {
    flex: 1;
    border: none;
    padding: 10px 20px;
}

.search-btn {
    background-color: #2256a4 !important;
    border: none;
    padding: 10px 20px;
    color: white  !important;
}

.featured-categories {
    padding: 3rem 0;
    background-color: #f4f4f4;
}

.category-card {
    margin-bottom: 2rem;
    border: none;
    transition: transform .2s;
    box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1);
}

.category-card:hover {
    transform: scale(1.05);
}

.card-body {
    padding: 1.5rem;
    background-color: white;
    border-radius: 0 0 10px 10px;
}

.shop-btn {
    background-color: #0d6efd;
    border-color: #0d6efd;
    color: white;
}

@media (max-width: 768px) {
    .hero-section {
        padding: 60px 0;
    }
    .hero-section h1 {
        font-size: 2rem;
    }
    .search-bar {
        flex-direction: column;
    }
    .search-bar input, .search-bar button {
        width: 100%;
    }
    .category-card {
        margin-bottom: 1rem;
    }
}
</style>
