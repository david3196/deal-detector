<template>
  <div class="product-page-container">
    <div v-if="productDetails" :key="productDetails._id" class="product-details-card">
      <img :src="productDetails.img" alt="Product" class="product-image" />
      <div class="product-content">
        <h1>{{ uppercase(productDetails.title) }}</h1>
        <p class="product-price">{{ productDetails.price }}.** LEI</p>
        <p class="product-stock" :class="{ 'in-stock': productDetails.stock > 0, 'out-of-stock': productDetails.stock === 0 }">
        {{ productDetails.stock > 0 ? 'In Stock' : 'Out of Stock' }}
      </p>
        <div class="price-history-graph">
          <canvas ref="priceHistoryCanvas"></canvas>
        </div>
        <a :href="productDetails.url" :target="productDetails.url" class="product-link-button">Go to {{
          uppercase(productDetails.site) }}</a>
        <button @click="addToTrackList(productDetails._id)" class="add-to-list-button">Add to List</button>
      </div>
    </div>
    <div class="price-history-card" v-if="priceHistory.length > 0">
      <h2>Price History</h2>
      <div class="price-history-content">
        <table class="price-history-table">
        <thead>
          <tr>
            <th>Date</th>
            <th>Price</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(entry, index) in priceHistory" :key="index">
            <td>{{ entry.date }}</td>
            <td>{{ entry.price }} LEI</td>
          </tr>
        </tbody>
        </table>
      </div>
    </div>
  </div>
</template>



<script>
import { ref, onMounted, computed, nextTick  } from 'vue';
import { useRoute } from 'vue-router';
import { useStore } from 'vuex';
import { Chart, registerables } from 'chart.js';
Chart.register(...registerables);

export default {
  setup() {
    const productDetails = ref(null);
    const priceHistory = ref([]);
    const priceHistoryCanvas = ref(null);
    const route = useRoute();
    const store = useStore();
    const userEmail = computed(() => store.getters.userEmail);
    const categoryName = computed(() => route.params.categoryName);

    const fetchPriceHistory = async (url) => {
      try {
        const response = await fetch(url);
        if (!response.ok) throw new Error('Failed to fetch price history');
        priceHistory.value = await response.json();
      } catch (error) {
        console.error(error);
      }
    };

    function renderPriceHistoryChart() {
      const ctx = priceHistoryCanvas.value.getContext('2d');
      const prices = priceHistory.value.map(entry => entry.price);
      const dates = priceHistory.value.map(entry => new Date(entry.date).toLocaleDateString());
      new Chart(ctx, {
        type: 'line',
        data: {
          labels: dates,
          datasets: [{
            label: 'Price',
            data: prices,
            fill: true,
            backgroundColor: 'rgba(0, 123, 255, 0.2)',
            borderColor: 'rgba(0, 123, 255, 1)',
            tension: 0.4
          }]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          scales: {
            y: {
              beginAtZero: false
            }
          }
        }
      });
    }

    onMounted(async () => {
      try {
        const { id } = route.params;
        const productResponse = await fetch(`http://localhost:3000/api/products/details/${categoryName.value}/${id}`);
        if (!productResponse.ok) throw new Error('Failed to fetch product details');
        productDetails.value = await productResponse.json();

        const encodedProductUrl = encodeURIComponent(productDetails.value.url);
        await fetchPriceHistory(`http://localhost:3000/api/prices/${categoryName.value}?productUrl=${encodedProductUrl}`);

        nextTick(() => {
          if (priceHistoryCanvas.value) {
            renderPriceHistoryChart();
          }
        });
      } catch (error) {
        console.error(error);
      }
    });

    const addToTrackList = async (productId) => {
      try {
        const response = await fetch(`http://localhost:3000/api/addToList`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({ productId, categoryName: categoryName.value, userEmail: userEmail.value })
        });

        if (!response.ok) {
          throw new Error(`Failed to add product to list: ${response.status}`);
        }

        // Check if the response is JSON
        const contentType = response.headers.get('content-type');
        if (contentType && contentType.includes('application/json')) {
          const result = await response.json();
          console.log('Product added to list:', result);
        } else {
          const textResult = await response.text();
          console.log('Product added to list:', textResult);
        }
      } catch (error) {
        console.error('Error adding product to list:', error);
      }
    };
    
    return { productDetails, userEmail, categoryName, addToTrackList, priceHistory, priceHistoryCanvas };
  },
  methods: {
    uppercase(value) {
      if (!value) return '';
      return value.toUpperCase();
    },
  }
};
</script>


<style scoped>
.product-page-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 20px;
    gap: 20px;
    flex: 1;
    background-color: #f5f5f5;
}
.in-stock {
  color: green; 
  font-weight: bold;
}

.out-of-stock {
  color: red; 
  font-weight: bold;
}
.product-details-card, .price-history-card {
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 16px;
    background-color: white;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    width: 50%;
    margin-bottom: 20px;
}


.price-history-card {
    margin: 20px auto;
    border: none;
    border-radius: 8px;
    padding: 16px;
    background-color: #2c3e50; 
    color: #ffffff; 
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); 
    overflow: hidden;
}

.product-image {
    max-width: 200px;
    height: auto;
    border-radius: 4px;
    display: block;
    margin: 0 auto 10px auto;
}

.product-content h1 {
    font-size: 1.5rem;
    font-weight: bold;
    color: #333;
    margin-bottom: 10px;
}

.product-content .product-price {
    font-size: 1.25rem;
    color: #555;
    margin-bottom: 16px;
}

.product-link-button {
    padding: 10px 15px;
    background-color: #007bff;
    color: white;
    text-decoration: none;
    border-radius: 4px;
    display: inline-block;
    margin-top: 10px;
}

.add-to-list-button {
    padding: 10px 15px;
    background-color: #be8814;
    color: white;
    text-decoration: none;
    border-radius: 4px;
    display: inline-block;
    margin-top: 10px;
    border: none;
    margin-left: 10px;
}

.price-history-content {
    max-height: 400px;
    overflow-y: auto; 
    width: 100%;
}

.price-history-table {
    width: 100%;
    border-collapse: collapse;
    margin: 20px 0;
}

.price-history-table th,
.price-history-table td {
    padding: 12px 8px; 
    border-bottom: 1px solid #4a5c68; 
}

.price-history-table thead {
    background-color: #34495e;
}

.price-history-table th {
    font-weight: bold;
    color: #ecf0f1;
}

.price-history-table tbody tr:hover {
    background-color: #3b4a59; 
}

.price-history-graph {
    width: 80%;
    height: 200px;
    margin: 1rem auto;
    background-color: #1e1e1e; 
    border-radius: 8px;
    padding: 10px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    position: relative;
}

.price-history-graph canvas {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
}

@media (max-width: 768px) {
    .product-details-card, .price-history-card {
        width: 100%;
    }
}


</style>
