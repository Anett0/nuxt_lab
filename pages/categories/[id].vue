<script setup lang="ts">

import dayjs from "dayjs";

interface Category {
  id: number;
  title: string;
  slug: string;
  description: string;
  created_at: string;
  updated_at: string;
  deleted_at: string;
  parent_id: number;
}

const route = useRoute();
const categoryId = ref<number | null>(null);
const category = ref<Category | null>(null);

const getCategory = async (id: number) => {
  try {
    const response = await $fetch<Category>(`http://127.0.0.1:8000/api/blog/categories/${id}`);
    category.value = {
      ...response,
      created_at: dayjs(response.created_at).format('YYYY-MM-DD HH:mm:ss'),
      updated_at: dayjs(response.updated_at).format('YYYY-MM-DD HH:mm:ss'),
      deleted_at: response.deleted_at ? dayjs(response.deleted_at).format('YYYY-MM-DD HH:mm:ss') : ''
    };
  } catch (error) {
    console.error('Error fetching category:', error);
  }
};

onMounted(() => {
  const id = parseInt(route.params.id as string, 10);
  if (!isNaN(id)) {
    categoryId.value = id;
    getCategory(id);
  } else {
    console.error('Invalid Category ID');
  }
});
</script>

<template>
  <div class="text-center m-auto mt-10 w-1/2">
    <div class="p-5">
      <nuxt-link class="mt-20 bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded" to="/categories">Back</nuxt-link>
    </div>
    <div class="">
      <UCard>
        <template #header>
          <p>Category ID: {{ categoryId }}</p>
        </template>
        <ul>
          <li class="border-b-2 mb-3 pb-3">Title: {{category?.title}}</li>
          <li class="border-b-2 my-3 pb-3">Slug: {{category?.slug}}</li>
          <li class="border-b-2 my-3 pb-3">Description {{category?.description ? category?.description : 'none'}}</li>
          <li class="border-b-2 my-3 pb-3">Created at: {{category?.created_at ? category?.created_at : 'none'}}</li>
          <li class="border-b-2 my-3 pb-3">Updated at: {{category?.updated_at ? category?.updated_at : 'none'}}</li>
          <li class="mt-3 pb-3">Deleted at: {{category?.deleted_at ? category?.deleted_at : 'none'}}</li>
        </ul>
        <template #footer>
          <p>Parent id: {{category?.parent_id ? category?.parent_id : 'none'}}</p>
        </template>
      </UCard>
    </div>
  </div>
</template>