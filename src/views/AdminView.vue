<template>
    <div class="admin-page-container">
        <button class="update-all-btn" @click="updateAllProducts">Update All Products</button>
        <div class="add-category-card card-css">
            <h3>Add category of products</h3>
            <form @submit.prevent="addCategory">
                <div>
                    <label for="category-name">Category Name:</label>
                    <input id="category-name" v-model="newCategory.name" type="text" required />
                </div>
                <div>
                    <label for="category-image">Category Image URL:</label>
                    <input id="category-image" v-model="newCategory.imageUrl" type="url" required />
                </div>
                <div v-for="(site, index) in newCategory.sites" :key="'site-' + index">
                    <div>
                        <label :for="'site-name-' + index">Site Name:</label>
                        <input :id="'site-name-' + index" v-model="site.name" type="text" :required="index === 0" />
                    </div>
                    <div>
                        <label :for="'link-to-crawl-' + index">Link to Crawl:</label>
                        <input :id="'link-to-crawl-' + index" v-model="site.linkToCrawl" type="url"
                            :required="index === 0" />
                    </div>
                    <button type="button" v-if="index === 0" @click="addSiteInput">Add More</button>
                </div>
                <button type="submit">Create Category</button>
            </form>
        </div>
        <div class="edit-product-card card-css">
                <h3>Edit an existing product</h3>
                <form @submit.prevent="searchProduct">
                    <div>
                    <label for="product-id">Product ID:</label>
                    <input id="product-id" v-model="search.productId" type="text" required />
                    </div>
                    <div>
                    <label for="product-category">Category:</label>
                    <input id="product-category" v-model="search.category" type="text" required />
                    </div>
                    <button type="submit">Search</button>
                </form>
                <form class="form-group" v-if="productFound && product" @submit.prevent="editProduct">
                    <div v-for="(value, key) in product" :key="key">
                    <label v-if="!['_id', 'url', 'site'].includes(key)" class="edit-label-css" :for="`product-${key}`">{{ key }}</label>
                    <input v-if="!['_id', 'url', 'site'].includes(key)" class="edit-input-css" :id="`product-${key}`" v-model="product[key]" type="text" />
                    </div>
                    <button class="update-button" type="submit">Update Product</button>
                </form>

                <div v-if="searchAttempted && !productFound">
                    <p>Product not found. Please try again.</p>
                    <button @click="clearSearch">Close</button>
                </div>
        </div>
    </div>
</template>




<script>
export default {
    data() {
        return {
            newCategory: {
                name: '',
                imageUrl: '',
                sites: [
                    {
                        name: '',
                        linkToCrawl: ''
                    }
                ]
            },
            search: {
                productId: '',
                category: ''
            },
            product: null,
            productFound: false,
            searchAttempted: false
        };
    },
    methods: {
        addSiteInput() {
            this.newCategory.sites.push({
                name: '',
                linkToCrawl: ''
            });
        },
        async addCategory() {
            try {
                const response = await fetch('http://localhost:3000/api/add-category', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify(this.newCategory)
                });

                if (!response.ok) {
                    throw new Error('Error when adding category');
                }

                const responseData = await response.json();
                console.log(responseData.message);
            } catch (error) {
                console.error('Failed to add category', error);
            }
        },
        async searchProduct() {
            try {
                console.log(this.search.category, this.search.productId)
                const response = await fetch(`http://localhost:3000/api/products/details/${this.search.category}/${this.search.productId}`);
                this.searchAttempted = true;
                if (!response.ok) {
                    this.productFound = false;
                    throw new Error('Product not found');
                }
                this.product = await response.json();
                this.productFound = true;
                console.log(this.product)
            } catch (error) {
                console.error('Error searching for product', error);
            }
        },
        async editProduct() {
            try {
                const response = await fetch(`http://localhost:3000/api/update-product/${this.search.productId}`, {
                    method: 'PUT',
                    headers: {
                        'Content-Type': 'application/json'
                    },
                    body: JSON.stringify({
                        id: this.search.productId,
                        category: this.search.category, 
                        product: this.product
                    })
                });

                if (!response.ok) {
                    throw new Error('Error when updating product');
                }

                const responseData = await response.text();
                console.log(responseData);

                this.product = {};
                this.search.productId = '';
                this.search.category = '';
            } catch (error) {
                console.error('Failed to update product', error);
            }
        },
        async updateAllProducts() {
            try {
                const response = await fetch('http://localhost:3000/api/update-all', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json'
                    }
                });

                if (!response.ok) {
                    throw new Error('Failed to start the update process');
                }

                const result = await response.json();
                console.log(result.message);
                alert(result.message);
            } catch (error) {
                console.error('Error during update:', error);
                alert('Error during update: ' + error.message);
            }
        },
        clearSearch() {
            this.searchAttempted = false;
            this.product = null;
            this.productFound = false;
        }
    },
};
</script>




<style>
.admin-page-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 20px;
    gap: 20px;
    flex: 1;
    background-color: #cbddd4;
}

.add-category-card, .edit-product-card {
    border-radius: 8px;
    padding: 24px;
    background-color: white;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    width: 50%;
    max-width: 600px;
    margin-bottom: 20px;
    display: flex;
    flex-direction: column;
    gap: 16px;
}

.edit-label-css, .card-css label {
    font-size: 14px;
    color: #333;
    margin-bottom: 4px;
}

.edit-input-css, .card-css input{
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 14px;
    color: #333;
    width: calc(100% - 50px);
}

.card-css button {
    padding: 8px 15px;
    background-color: #5cb85c;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 14px;
    transition: background-color 0.3s ease;
    align-self: start;
    margin-top: 10px;
}
.update-all-btn{
    padding: 8px 15px;
    background-color: #e56c15;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 19px;
    transition: background-color 0.3s ease;
}
.card-css button:hover {
    background-color: #4cae4c;
}
.update-all-btn:hover{
    background-color: #b05412;
}
@media (max-width: 768px) {
    .card-css {
        width: 90%;
        margin-bottom: 10px;
    }
}</style>