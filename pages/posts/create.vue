<script setup lang="ts">
import { z } from 'zod';
import { reactive, ref, computed } from 'vue';
import axios from 'axios';

interface Category {
  id: number;
  title: string;
}
interface Post{
  title: string;
  slug: string;
  content_raw: string;
  excerpt: string;
  category_id: number;
  is_published: boolean;
}
const categories = ref<Category[]>([]);

const getCategories = async () => {
  try {
    const response = await $fetch<Category[]>(`http://127.0.0.1:8000/api/blog/categories/get`);
    categories.value = response;
    if (categories.value.length > 0) {
      state.category_id = categories.value[0].id.toString();
    }
  } catch (error) {
    console.error('Error fetching categories:', error);
  }
};
getCategories();

const posts = ref<Post[]>([]);
const getPosts = async () => {
  try {
    const response = await $fetch<Post[]>(`http://127.0.0.1:8000/api/blog/posts/get`);
    posts.value = response;
  } catch (error) {
    console.error('Error fetching posts:', error);
  }
};
getPosts();

// Computed property for category options
const categoryOptions = computed(() => {
  return categories.value.map(category => ({
    value: category.id.toString(), // Convert id to string
    label: category.title
  }));
});

// Define Zod schema for validation
const schema = z.object({
  title: z.string().min(5, 'Title must be at least 5 characters').max(200, 'Title must be less than 200 characters')
      .refine(value => {
        return !posts.value.some(post => post.title === value)
      }, 'Post title must be unique'),
  slug: z.string().max(200, 'Slug must be less than 200 characters').
  refine(value => {
    return !posts.value.some(post => post.slug === value)
  }, 'Post slug must be unique').optional(),
  content_raw: z.string().min(5, 'Description must be at least 5 characters').max(10000, 'Content must be less than 10000 characters'),
  category_id: z.string().refine(value => {
    const numberValue = Number(value);
    return Number.isInteger(numberValue) && categories.value.some(category => category.id === numberValue);
  }, 'Parent ID must be a valid integer and exist in categories'),
  is_published: z.boolean().optional(),
  excerpt: z.string().optional()
});

type Schema = z.infer<typeof schema>;

const state = reactive<Partial<Schema>>({
  title: undefined,
  slug: undefined,
  content_raw: undefined,
  category_id: undefined,
  is_published: false,
  excerpt: ""
});

async function onSubmit() {
  try {

    const result = schema.parse(state);
    console.log(result);

    // Convert category_id to number before sending to server
    const dataToSend = {
      ...result,
      category_id: Number(result.category_id)
    };

    // Send the data to the server
    const response = await axios.post('http://127.0.0.1:8000/api/blog/posts', dataToSend);
    alert("Post successfully created!");
    window.location.href = '/posts';
  } catch (e) {
    if (e instanceof z.ZodError) {
      // Handle validation errors
      console.error(e.errors);
    } else {
      // Handle other errors
      console.error('Error submitting form:', e);
    }
  }
}
</script>

<template >
  <div class="m-auto w-1/2">

    <div class="p-5">
      <nuxt-link class="mt-20 bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded" to="/posts">Back</nuxt-link>
    </div>
    <UCard class="p-5 border-2">
      <UForm :schema="schema" :state="state" class="space-y-4" @submit="onSubmit">
        <UFormGroup label="Заголовок" name="title">
          <UInput v-model="state.title" />
        </UFormGroup>
        <UFormGroup label="Текст статті" name="content_raw">
          <UTextarea v-model="state.content_raw" />
        </UFormGroup>

        <UFormGroup label="Категорія" name="category_id" >
          <USelect v-model="state.category_id" :options="categoryOptions" />
        </UFormGroup>

        <UFormGroup label="Псевдонім" name="slug" >
          <UInput v-model="state.slug" type="text"  />
        </UFormGroup>
        <UFormGroup label="Короткий текст" name="excerpt" >
          <UTextarea v-model="state.excerpt" type="text"  />
        </UFormGroup>
        <UFormGroup label="Published" name="is_published" >
          <UCheckbox v-model="state.is_published"/>
        </UFormGroup>



        <UButton type="submit" class="flex m-auto mt-5">
          Save
        </UButton>

      </UForm>
    </UCard>
  </div>
</template>