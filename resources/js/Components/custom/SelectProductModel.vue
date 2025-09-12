<template>
  <TransitionRoot as="template" :show="open">
    <Dialog class="relative z-10" @close="closeModal">
      <!-- Overlay -->
      <TransitionChild
        as="template"
        enter="ease-out duration-300"
        enter-from="opacity-0"
        enter-to="opacity-100"
        leave="ease-in duration-200"
        leave-from="opacity-100"
        leave-to="opacity-0"
      >
        <div class="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity" />
      </TransitionChild>

      <!-- Modal -->
      <div class="fixed inset-0 z-10 flex items-center justify-center">
        <TransitionChild
          as="template"
          enter="ease-out duration-300"
          enter-from="opacity-0 scale-95"
          enter-to="opacity-100 scale-100"
          leave="ease-in duration-200"
          leave-from="opacity-100 scale-100"
          leave-to="opacity-0 scale-95"
        >
          <DialogPanel
            class="bg-white border-1 border-gray-600 rounded-[20px] shadow-xl w-5/6 p-6 text-center min-h-[600px]"
          >
            <div class="flex flex-col items-center justify-start">
              <div class="w-full flex">
                <div class="w-full py-12 space-y-8">
                  <!-- Search (always visible) -->
                  <div class="w-full md:w-3/6 mx-auto">
                    <input
                      v-model="search"
                      @input="() => fetchProducts()"
                      type="text"
                      placeholder="Search ..."
                      class="w-full custom-input"
                    />
                  </div>

                  <!-- Extra UI (hidden in simple mode) -->
                  <div v-if="!simpleMode" class="w-full space-y-4">
                    <!-- Top Categories -->
                    <div class="w-full pt-3">
                      <div class="flex items-center space-x-4">
                        <p class="text-2xl flex-shrink-0">Top Categories:</p>
                        <div class="flex space-x-4 overflow-x-auto whitespace-nowrap max-w-full">
                          <div
                            v-for="category in parentCategories"
                            :key="category.id"
                            class="cursor-pointer flex-shrink-0"
                            @click="selectParentCategory(category)"
                          >
                            <p
                              :class="{
                                'text-2xl text-blue-600': selectedParentCategory?.id !== category.id,
                                'text-2xl text-red-600': selectedParentCategory?.id === category.id,
                              }"
                            >
                              {{ category.name }}
                            </p>
                          </div>
                        </div>
                      </div>
                    </div>

                    <!-- Filters row -->
                    <div class="flex w-full space-x-4">
                      <select
                        v-model="selectedCategory"
                        @change="() => fetchProducts()"
                        class="px-6 py-3 text-xl font-normal tracking-wider text-blue-600 bg-white rounded-lg cursor-pointer custom-select"
                      >
                        <option value="">Filter by Category</option>
                        <option
                          v-for="category in allcategoriesFiltered"
                          :key="category.id"
                          :value="category.id"
                        >
                          {{
                            category.hierarchy_string
                              ? category.hierarchy_string + ' ----> ' + category.name
                              : category.name
                          }}
                        </option>
                      </select>

                      <select
                        v-model="stockStatus"
                        @change="() => fetchProducts()"
                        class="px-6 py-3 text-xl font-normal tracking-wider text-blue-600 bg-white rounded-lg cursor-pointer custom-select"
                      >
                        <option value="">Filter by Stock</option>
                        <option value="in">In Stock</option>
                        <option value="out">Out of Stock</option>
                      </select>

                      <select
                        v-model="sort"
                        @change="() => fetchProducts()"
                        class="px-6 py-3 text-xl font-normal tracking-wider text-blue-600 bg-white rounded-lg cursor-pointer custom-select"
                      >
                        <option value="">Filter by Price</option>
                        <option value="asc">Ascending</option>
                        <option value="desc">Descending</option>
                      </select>

                      <select
                        v-model="color"
                        @change="() => fetchProducts()"
                        class="px-6 py-3 text-xl font-normal tracking-wider text-blue-600 bg-white rounded-lg cursor-pointer custom-select"
                      >
                        <option value="">Filter by Base</option>
                        <option
                          v-for="colorOption in colors"
                          :key="colorOption.id"
                          :value="colorOption.name"
                        >
                          {{ colorOption.name }}
                        </option>
                      </select>

                      <select
                        v-model="size"
                        @change="() => fetchProducts()"
                        class="px-6 py-3 text-xl font-normal tracking-wider text-blue-600 bg-white rounded-lg cursor-pointer custom-select"
                      >
                        <option value="">Filter by Size</option>
                        <option
                          v-for="sizeOption in sizes"
                          :key="sizeOption.id"
                          :value="sizeOption.name"
                        >
                          {{ sizeOption.name }}
                        </option>
                      </select>

                      <span
                        @click="resetFilters"
                        class="px-6 py-3 text-xl font-normal tracking-wider text-white text-center bg-blue-600 rounded-lg custom-select"
                      >
                        Reset
                      </span>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <!-- Product grid -->
            <div class="mt-4">
              <template v-if="loading">
                <div class="col-span-4 text-center text-gray-500">
                  <p class="text-center text-red-500 text-[17px]">Loading...</p>
                </div>
              </template>

              <template v-else>
                <template v-if="products.data.length > 0">
                  <!-- make the grid scroll vertically and fit 6 items per row on xl -->
                  <div class="max-h-[55vh] overflow-y-auto">
                    <div class="grid gap-4 grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 xl:grid-cols-6">
                      <div
                        v-for="product in products.data.filter((p) => !(hidePromotions && p.is_promotion))"
                        :key="product.id"
                        @click="product.stock_quantity > 0 && selectProduct(product)"
                        :class="[
                          'text-white bg-black border-2 rounded-xl shadow-md p-2 transition-transform duration-200',
                          product.stock_quantity > 0
                            ? selectedProducts.find((p) => p.id === product.id)
                              ? 'border-green-600 hover:-translate-y-1 cursor-pointer'
                              : 'border-black hover:-translate-y-1 cursor-pointer'
                            : 'border-red-600 cursor-not-allowed',
                        ]"
                      >
                        <div>
                          <img
                            :src="product.image ? `/${product.image}` : '/images/placeholder.jpg'"
                            alt="Product Image"
                            class="object-cover w-full h-40 md:h-44 rounded-md"
                          />
                        </div>

                        <div class="px-1 pt-2 space-y-2">
                          <div class="flex items-start justify-between gap-2 text-[11px] font-bold tracking-wide">
                            <p class="text-justify line-clamp-2">{{ product.name || 'N/A' }}</p>
                            <p class="px-2 py-1 text-white bg-green-700 rounded-full flex items-center text-xs">
                              {{ product.selling_price || 'N/A' }}
                            </p>
                          </div>

                          <div class="flex justify-center gap-2 items-start w-full text-gray-400 text-xs">
                            <div class="flex gap-1">
                              <p class="font-bold">Color:</p>
                              <p>{{ product.color?.name || 'N/A' }}</p>
                            </div>
                            <div class="flex gap-1">
                              <p class="font-bold">Size:</p>
                              <p>{{ product.size?.name || 'N/A' }}</p>
                            </div>
                          </div>

                          <div>
                            <p
                              v-if="product.stock_quantity > 0"
                              class="text-sm font-bold tracking-wider text-green-500"
                            >
                              <i class="ri-checkbox-blank-circle-fill"></i> In Stock
                            </p>
                            <p
                              v-else
                              class="text-sm font-bold tracking-wider text-red-500"
                            >
                              <i class="ri-checkbox-blank-circle-fill"></i> Out of Stock
                            </p>
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>

                  <div class="flex items-center justify-between mt-6">
                    <div class="pagination flex space-x-4">
                      <!-- <button
                        @click="fetchPage(products.prev_page_url)"
                        :disabled="!products.prev_page_url"
                        class="px-4 py-2 text-[15px] text-white bg-blue-500 rounded-md shadow-md hover:bg-blue-600 disabled:bg-gray-300 disabled:cursor-not-allowed"
                      >
                        Previous
                      </button> -->
                      <!-- <button
                        @click="fetchPage(products.next_page_url)"
                        :disabled="!products.next_page_url"
                        class="px-4 py-2 text-[15px] text-white bg-blue-500 rounded-md shadow-md hover:bg-blue-600 disabled:bg-gray-300 disabled:cursor-not-allowed"
                      >
                        Next
                      </button> -->
                    </div>
                    <button
                      class="px-6 py-2 text-[15px] text-white bg-blue-600 rounded hover:bg-blue-700"
                      @click.prevent="closeModal(true)"
                    >
                      Import
                    </button>
                  </div>
                </template>

                <template v-else>
                  <div class="col-span-4 text-center text-gray-500">
                    <p class="text-center text-red-500 text-[17px]">No Products available</p>
                  </div>
                </template>
              </template>
            </div>

            <PromotionItemModal
              :open="itemModalOpen"
              @update:open="itemModalOpen = $event"
              :product="selectedPromotion"
            />
          </DialogPanel>
        </TransitionChild>
      </div>
    </Dialog>
  </TransitionRoot>
</template>

<script setup>
import { Dialog, DialogPanel, TransitionChild, TransitionRoot } from '@headlessui/vue';
import { ref, watch } from 'vue';
import { useForm } from '@inertiajs/vue3';
import PromotionItemModal from '@/Components/custom/PromotionItemModal.vue';
import axios from 'axios';

const itemModalOpen = ref(false);

const products = ref({ data: [] });
const loading = ref(false);
const search = ref('');
const selectedCategory = ref('');
const stockStatus = ref('');
const sort = ref('');
const color = ref('');
const size = ref('');
const selectedParentCategory = ref(null);
const parentCategories = ref([]);

// events
const emit = defineEmits(['update:open', 'selected-products']);

// props
const {
  open,
  allcategories,
  colors,
  sizes,
  hidePromotions,
  initialCategory,   // category id from the new grid
  simpleMode,        // when true: hide top categories + dropdowns
} = defineProps({
  open: { type: Boolean, required: true },
  allcategories: Array,
  colors: Array,
  sizes: Array,
  hidePromotions: { type: Boolean, default: false },
  initialCategory: { type: [String, Number], default: '' },
  simpleMode: { type: Boolean, default: false },
});

const allcategoriesFiltered = ref(allcategories);
const selectedPromotion = ref(null);
const selectedProducts = ref([]);

const selectProduct = (product) => {
  const idx = selectedProducts.value.findIndex((p) => p.id === product.id);
  if (idx === -1) selectedProducts.value.push(product);
  else selectedProducts.value.splice(idx, 1);
};

const viewPromotion = (product) => {
  selectedPromotion.value = product;
  itemModalOpen.value = true;
};

const resetFilters = () => {
  search.value = '';
  // keep category in simple mode; otherwise clear
  selectedCategory.value = simpleMode ? selectedCategory.value : '';
  stockStatus.value = '';
  sort.value = '';
  color.value = '';
  size.value = '';
  fetchProducts();
};

const form = useForm({});

const closeModal = (triggerImport = false) => {
  emit('update:open', false);
  if (triggerImport) emit('selected-products', selectedProducts.value);
  selectedProducts.value = [];
};

const fetchProducts = async (url = '/api/products') => {
  loading.value = true;
  try {
    const { data } = await axios.post(url, {
      search: search.value,
      selectedCategory: selectedCategory.value, // ✅ filters by category id
      stockStatus: simpleMode ? '' : stockStatus.value,
      sort: simpleMode ? '' : sort.value,
      color: simpleMode ? '' : color.value,
      size: simpleMode ? '' : size.value,
    });
    products.value = data.products || { data: [] };
  } catch (err) {
    console.error('Error fetching products:', err);
    products.value = { data: [] };
  } finally {
    loading.value = false;
  }
};

const fetchParentCategories = async (url = '/api/top-categories') => {
  if (simpleMode) return; // no need when minimal
  try {
    const { data } = await axios.post(url);
    parentCategories.value = data.categories || [];
  } catch (err) {
    console.error('Error fetching categories:', err);
  }
};

const fetchPage = (url) => {
  if (url) fetchProducts(url);
};

const selectParentCategory = (category) => {
  selectedParentCategory.value = category;
  allcategoriesFiltered.value = [category, ...(category.children || [])];
  selectedCategory.value = category.id;
  fetchProducts();
};

// initial load
fetchProducts();
fetchParentCategories();

// preselect category when opening
watch(
  () => open,
  (isOpen) => {
    if (!isOpen) return;
    if (simpleMode) {
      // lock to chosen category; clear other filters
      stockStatus.value = '';
      sort.value = '';
      color.value = '';
      size.value = '';
    }
    selectedCategory.value = initialCategory ? String(initialCategory) : selectedCategory.value;
    fetchProducts();
  }
);

watch(
  () => initialCategory,
  (val) => {
    if (open) {
      selectedCategory.value = val ? String(val) : '';
      fetchProducts();
    }
  }
);
</script>
