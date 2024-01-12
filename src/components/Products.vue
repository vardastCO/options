<template>
  <div>
    <h1>Product Search</h1>
    <input v-model="searchQuery" @input="searchProducts" placeholder="Search products..." />

    <div class="product-container">
      <div v-for="product in products" :key="product.id" class="product-item">
        <div class="product-info">
          <label>
            <input type="checkbox" v-model="selectedProducts[product.id]" />
            <h2>{{ product.name }}</h2>
          </label>
        </div>

        <div class="attribute-container">
          <div v-for="attribute in product.attributeValues" :key="attribute.attribute.id" class="attribute-item">
            <label>
              <input type="checkbox" v-model="selectedAttributes[attribute.attribute.id]" @change="updateSelectedAttributes" />
              <strong>{{ attribute.attribute.name }}:</strong>
              <hr/>
              <ul>
              <label>
                <input type="checkbox" v-model="selectedValues[attribute.value.id]" @change="updateSelectedValues" />
                {{ attribute.value.value }}
              </label>

              </ul>
              <hr/>
            </label>
          </div>
        </div>
      </div>
    </div>

    <div>
      <h2>Selected Product IDs:</h2>
      <ul>
        <li v-for="productId in selectedProductIds" :key="productId">{{ productId }}</li>
      </ul>
      <h2>Selected Attribute Values:</h2>
      <ul>
        <li v-for="selectedValue in selectedAttributeValues" :key="selectedValue.valueId">
          {{ selectedValue.productId }} - {{ selectedValue.attributeId }} - {{ selectedValue.valueId }}
        </li>
      </ul>
      <button @click="createOption">Create Option</button>
    </div>
  </div>
</template>

<style>
.product-container {
  display: flex;
  flex-wrap: wrap;
}

.product-item {
  border: 1px solid #ccc;
  margin: 10px;
  padding: 10px;
  width: 300px;
}

.attribute-container {
  margin-top: 10px;
}

.attribute-item {
  margin-bottom: 10px;
}

.attribute-item strong {
  display: block;
  margin-bottom: 5px;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  margin-bottom: 5px;
}
</style>

<script>
import { request } from 'graphql-request';

export default {
  data() {
    return {
      searchQuery: '',
      products: [],
      loading: false,
      isHovered: false,
      selectedProducts: {},  // For selecting products
      selectedProductIds: [],
      selectedAttributeValues: [],
      selectedAttributes: {},
      selectedValues: {},
    };
  },
  methods: {
    async searchProducts() {
      this.loading = true;

      const query = `
        query GetSuggestQuery($suggestInput: SuggestInput!) {
          suggestV2(suggestInput: $suggestInput) {
            products {
              id
              name
              attributeValues {
                attribute {
                  id
                  name
                }
                value {
                  id
                  value
                }
              }
            }
          }
        }
      `;

      const variables = {
        suggestInput: {
          query: this.searchQuery,
        },
      };

      try {
        const data = await request('http://78.46.124.237:3080/graphql', query, variables);
        this.products = data.suggestV2.products;
      } catch (error) {
        console.error('Error fetching products:', error);
      } finally {
        this.loading = false;
      }
    },
    updateSelectedAttributes() {
      // Logic to update selectedAttributeValues based on selectedAttributes
      // Iterate through selectedAttributes and update selectedAttributeValues
    },
    updateSelectedValues() {
      // Logic to update selectedAttributeValues based on selectedValues
      // Iterate through selectedValues and update selectedAttributeValues
    },
    createOption() {
      const selectedProductIds = Object.keys(this.selectedProducts)
        .filter(productId => this.selectedProducts[productId]);

      if (selectedProductIds.length === 0) {
        console.warn('Please select at least one product.');
        return;
      }

      const selectedAttributeId = Object.keys(this.selectedAttributes)[0]; // Assuming only one attribute can be selected
      const selectedValueIds = Object.keys(this.selectedValues)[0];

      if (selectedAttributeId && selectedValueIds.length > 0) {
        const mutation = `
          mutation CreateOption($createOptionInput: CreateOptionInput!) {
            createOption(createOptionInput: $createOptionInput) {
              // Include the fields you want to retrieve after the mutation
              // For example, you might want to get the ID of the created option
              // Adjust this based on your GraphQL schema
              id
              // ... (other fields you want to retrieve)
            }
          }
        `;

        const variables = {
          createOptionInput: {
            productIds: selectedProductIds.map(id => parseInt(id)),
            attributeId: parseInt(selectedAttributeId),
            valueId: parseInt(selectedValueIds) // Assuming only one value can be selected
          },
        };

        request('http://78.46.124.237:3080/graphql', mutation, variables)
          .then(data => {
            // Handle the response as needed
            console.log('Option created:', data.createOption);
            // Optionally, you can reset selectedProductIds and selectedAttributeValues
            this.selectedProductIds = [];
            this.selectedAttributeValues = [];
          })
          .catch(error => {
            console.error('Error creating option:', error);
          });
      } else {
        console.warn('Please select at least one attribute and one value.');
      }
    },
  },
};
</script>
