<template>
    <div class="category-page">
        <h1 class="category-title">{{ capitalizedCategoryName }}</h1>
        <div class="search-container">
            <input type="text" v-model="searchQuery" class="search-bar" placeholder="Search products...">
        </div>
        <div class="product-grid">
            <div v-for="product in filteredProducts" :key="product.id" class="product-card">
                <router-link :to="`/product/${categoryName}/${product._id}`">
                    <div class="product-site-tag">{{ product.site }}</div>
                    <div class="product-image">
                        <img :src=product.img  alt="Product" />
                    </div>
                    <div class="product-info">
                        <p class="product-brand uppercase">{{ product.title }}</p>
                        <p class="product-price">{{ product.price }} LEI</p>
                    </div>
                </router-link>
            </div> 
        </div>
    </div>
</template>

<script>
import { onMounted, ref, computed } from 'vue';

export default {
    props: {
        categoryName: String,
    },
    setup(props) {
        const products = ref([]);
        const searchQuery = ref('');

        const capitalizedCategoryName = computed(() => {
            if (!props.categoryName) return '';
            return props.categoryName.charAt(0).toUpperCase() + props.categoryName.slice(1);
        });

        const filteredProducts = computed(() => {
            return products.value.filter(product =>
                product.title.toLowerCase().includes(searchQuery.value.toLowerCase())
            );
        });

        const fetchProductsForCategory = async (categoryName) => {
            try {
                const response = await fetch(`http://localhost:3000/api/products/${categoryName}`);
                if (!response.ok) throw new Error('Failed to fetch products');
                const data = await response.json();
                products.value = data;
            } catch (error) {
                console.error(error);
            }
        };

        onMounted(() => {
            fetchProductsForCategory(props.categoryName);
        });

        return { products, capitalizedCategoryName, searchQuery, filteredProducts };
    },
};
</script>

<style scoped>

a {
  text-decoration: none;
}
.category-page {
    padding: 2rem;
    background-color: #f7f7f7;
}

.product-card a {
  text-decoration: none;
}

.category-title {
    text-align: center;
    margin-bottom: 2rem;
    font-size: 2rem;
    color: #333;
    text-transform: uppercase;
}

.product-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
    gap: 1rem;
    justify-content: center;
}

.product-card {
    position: relative;
    background-color: #fff;
    border: none;
    border-radius: 1rem;
    overflow: hidden;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    transition: transform 0.3s ease-in-out;
    cursor: pointer;
}

.product-site-tag {
  position: absolute;
  top: 0;
  right: 0;
  background-color: #007bff;
  color: #ffffff;
  padding: 0.5rem 1rem;
  border-bottom-left-radius: 1rem;
  font-size: 0.8rem;
  text-transform: uppercase;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2); 
  z-index: 10;
}

/* .product-image img {
  border-top-left-radius: 1rem;
  border-top-right-radius: 1rem;
} */

.product-card:hover {
    transform: scale(1.03);
}

.product-image {
    display: flex;
    justify-content: center;
    align-items: center;
}
.product-image img {
    width: 80%;
    height: auto;
    display: block;
    object-fit: cover;
}

.product-info {
    padding: 1rem;
    text-align: center;
}

.product-brand {
    font-size: 0.9rem;
    font-weight: bold;
    color: #333;
    margin: 0;
}

.product-model {
    font-size: 1rem;
    color: #555;
    margin: 0;
}

.product-price {
    font-size: 1rem;
    color: #1a8917;
    font-weight: bold;
    margin-top: 0.5rem;
    margin-bottom: 1px;
}

.uppercase {
    text-transform: uppercase;
}

.search-container {
    display: flex;
    justify-content: center;
    margin-bottom: 2rem;
}

.search-bar {
    width: 100%;
    max-width: 500px;
    padding: 10px 15px;
    border-radius: 20px;
    border: 2px solid #007bff;
    font-size: 1rem;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
    outline: none;
    transition: all 0.3s ease;
}

.search-bar:focus {
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}
</style>
