<script setup lang="ts">
import axios from "axios";
import {ref, watch} from "vue";
const columns = [
  {
    key: 'id',
    label: '#'
  },
  {
    key: 'user_name',
    label: 'Автор',
  },
  {
    key: 'category_title',
    label: 'Категорія'
  },
  {
    key: 'title',
    label: 'Заголовок'
  },
  {
    key: 'published_at',
    label: 'Дата публікації'
  },
  {
    key: 'actions',
    label: 'Дії'
  }
]
const items = (post:Post) => [
  [{
    label: 'Редагувати',
    icon: 'i-heroicons-pencil-square-20-solid',
    click: () => window.location.href = (`/posts/update/${post?.id}`)
  }, {
    label: 'Деталі',
    icon: 'i-heroicons-information-circle-20-solid',
    click: () => window.location.href = (`/posts/${post?.id}`)
  }], [{
    label: 'Видалити',
    icon: 'i-heroicons-trash-20-solid',
    click: () => {
      if (confirm(`Ви впевнені, що хочете видалити статтю '${post.title}'?`))
      {
        axios.delete(`http://127.0.0.1:8000/api/blog/posts/${post.id}`)
            .then(res => {
              console.log(res);
              alert('Пост видалено!');
              posts.value = posts.value.filter(p => p.id !== post.id);
            })
            .catch(error => {
              console.error(error);
              alert('Не вдалось видалити пост');
            });
      }
    }
  }]
]
interface User {
  name: string;
}
interface Category {
  title: string;
}
interface Post {
  id: number;
  user: User;
  category: Category;
  title: string;
  published_at: string;
}
const posts = ref<Post[]>([]);
const page = ref(1);
const totalPages = ref(1);
const totalPosts = ref(0);
const getPosts = async (page = 1) => {
  try {
    const response = await axios.get(`http://127.0.0.1:8000/api/blog/posts?page=${page}`);
    posts.value = response.data.data;
    totalPages.value = response.data.last_page;
    totalPosts.value = response.data.total;
  } catch (error) {
    console.error('Error fetching posts:', error);
  }
};
watch(page, () => {
  getPosts(page.value);
});
getPosts();
const rows = computed(() => {
  return posts.value.map((post) => ({
    id: post.id,
    user_name: post.user.name,
    category_title: post.category.title,
    title: post.title,
    published_at: post.published_at,
  }));
});
</script>
<template>
  <div class="flex flex-col">
    <div class="me-5">
      <NuxtLink to="/posts/create">
        <button class="bg-transparent hover:bg-blue-500 float-end text-blue-700 font-semibold hover:text-white py-2 px-4 border border-blue-500 hover:border-transparent rounded">
          Add Post
        </button>
      </NuxtLink>
    </div>
    <div>

      <UTable class="mt-3" :columns="columns" :rows="rows" :total="rows.length">

        <template #actions-data="{ row }">
          <UDropdown :items="items(row)">
            <UButton color="gray" variant="ghost" icon="i-heroicons-ellipsis-horizontal-20-solid" />
          </UDropdown>
        </template>
      </UTable>
      <div class="d-flex mt-5">
        <UPagination v-model="page" :page-count="5" :total="totalPosts" />
      </div>
    </div>
  </div>
</template>