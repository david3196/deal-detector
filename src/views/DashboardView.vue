<template>
    <div class="dashboard-page-container">
        <div class="price-history-card">
            <h2>Price History</h2>
            <div class="price-history-content">
                <table class="price-history-table">
                    <thead>
                        <tr>
                            <th>Product Name</th>
                            <th>Price</th>
                            <th>Site</th>
                            <th>Remove from List</th>
                        </tr>
                    </thead>
                    <tbody>
                        <template v-for="(products, category) in watchList" :key="category">
                            <tr v-for="product in products" :key="product.id">
                            <td>{{ product.name }}</td>
                            <td>{{ product.price }} LEI</td>
                            <td class="product-site">{{ product.site }}</td>
                            <td>
                                <button @click="removeFromWatchList(category, product.id)" class="remove-from-list-button">
                                Remove
                                </button>
                            </td>
                            </tr>
                        </template>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</template>



<script>
import { ref, onMounted, computed, watch } from 'vue';
import { useStore } from 'vuex';

export default {
    setup() {
        const store = useStore();
        const watchList = ref({});
        const userEmail = computed(() => store.getters.userEmail);

        const fetchWatchList = async (email) => {
            try {
                const response = await fetch(`http://localhost:3000/api/getWatchList?email=${encodeURIComponent(email)}`);
                if (!response.ok) {
                    throw new Error('Failed to fetch watch list');
                }
                watchList.value = await response.json();
            } catch (error) {
                console.error('Error fetching watch list:', error);
            }
        };

        watch(userEmail, (newEmail) => {
            if (newEmail) {
                fetchWatchList(newEmail);
            }
        });

        onMounted(() => {
            if (userEmail.value) {
                fetchWatchList(userEmail.value);
            }
        });

        const removeFromWatchList = async (category, productId) => {
            try {
                const response = await fetch(`http://localhost:3000/api/removeFromWatchList`, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({ userEmail: userEmail.value, category, productId }),
                });

                if (!response.ok) {
                    throw new Error('Failed to remove product from watch list');
                }

                await fetchWatchList(userEmail.value);
            } catch (error) {
                console.error('Error removing product from watch list:', error);
            }
        };


        return { watchList, removeFromWatchList };
    },
};
</script>

<style scoped>
.dashboard-page-container {
  display: flex;
  justify-content: center; 
  align-items: center;
  padding: 40px 20px;
  min-height: calc(100vh - 80px); 
  box-sizing: border-box;
  background-color: #e5e5e5;
}

.price-history-card {
  width: 100%;
  max-width: 50%;
  max-height: 50%;
  margin: auto;
  background-color: #ffffff;
  border-radius: 10px;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.price-history-content {
  max-height: 100%;
  overflow-y: auto;
  padding: 0;
}

.price-history-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 0;
  margin-bottom: 0;
}
.product-site {
  text-transform: uppercase;
}

.price-history-table th,
.price-history-table td {
  padding: 12px;
  text-align: left;
  border-bottom: 2px solid #eaeaea;
}

.price-history-table th {
  background-color: #4a4a4a;
  color: white;
  font-size: 1rem;
}

.price-history-table td {
  font-size: 0.9rem;
  color: #333;
}

.price-history-table tr:hover {
  background-color: #f9f9f9;
}

.remove-from-list-button {
  padding: 8px 16px;
  background-color: #ff6b6b;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.remove-from-list-button:hover {
  background-color: #ff3b3b;
}

@media (max-width: 768px) {
  .dashboard-page-container {
    padding: 20px;
  }

  .price-history-card {
    width: 100%;
    margin: 20px 0;
  }
}

</style>
