<script setup lang="ts">
useHead({
  title: 'Products'
})

const columns = [
  { key: 'title', label: 'Title', sortable: true },
  { key: 'description', label: 'label', sortable: true },
  { key: 'price', label: 'Price', sortable: true },
  { key: 'rating', label: 'Rating', sortable: true },
  { key: 'brand', label: 'Brand', sortable: true },
  { key: 'category', label: 'Category', sortable: true },
  { key: 'thumbnail', label: 'Thumbnail'}
];

const { data } = await useFetch<any>('https://dummyjson.com/products');
const products = ref(data.value.products);
const q = ref('');
const page = ref(1);
const pageCount = 5;
const total = ref(products.value.length);

const sort = ref({ column: '', direction: 'asc' as const })
const sortedRows = computed(() => {
  const sortedProducts = [...products.value]
  const { column, direction } = sort.value

  if (column && direction) {
    sortedProducts.sort((a, b) => {
      const aValue = a[column]
      const bValue = b[column]
      if (aValue < bValue) return direction === 'asc' ? -1 : 1
      if (aValue > bValue) return direction === 'asc' ? 1 : -1
      return 0
    })
  }

  return sortedProducts
})

const rows = computed(() => {
  let filteredProducts = [...sortedRows.value]

  if (q.value) {
    filteredProducts = filteredProducts.filter(product => {
      return Object.values(product).some(value => {
        return String(value).toLowerCase().includes(q.value.toLowerCase())
      })
    })
  }

  total.value = filteredProducts.length;
  const startIndex = (page.value - 1) * pageCount
  const endIndex = startIndex + pageCount

  return filteredProducts.slice(startIndex, endIndex)
})

const filteredRows = computed(() => {
  if (!q.value) {
    total.value = products.value.length;
    return products.value.slice((page.value - 1) * pageCount, (page.value) * pageCount);
  }
  page.value = 1;
  let result = products.value.filter((product: any) => {
    return Object.values(product).some(value =>
        String(value).toLowerCase().includes(q.value.toLowerCase())
    );
  });
  total.value = result.length;
  return result.slice((page.value - 1) * pageCount, (page.value) * pageCount);
});
watch(q,()=>{
  page.value = 1;
})

</script>

<template>
  <div>
    <div class="flex px-3 py-3.5 border-b border-gray-200 dark:border-gray-700">
      <UInput color="black"  v-model="q" placeholder="Пошук..." />
    </div>
    <UTable class="w-full" :ui="{ td: { base: 'mx-auto max-w-[0] truncate' } }" :columns="columns" :rows="rows"
            v-model:sort="sort"
            sort-mode="manual">
      <template #thumbnail-data="{ row }">
        <img class="w-[100px] h-[100px]" :src="row.thumbnail" alt="Мініатюра" />
      </template>
      <template #rating-data="{ row }">
        <span :class="row.rating < 4.5 ? 'text-red-700' : 'text-green-700' ">{{ row.rating }}</span>
      </template>
    </UTable>
    <div class="flex justify-center mt-5">
      <UPagination    v-model="page" :page-count="pageCount" :total="total" class="text-black"
                      :class="{ 'text-gray-500': page !== currentPage, 'text-green-700': page === currentPage }" />
    </div>
  </div>
</template>