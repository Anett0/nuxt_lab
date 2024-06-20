<template>
  <div class="w-1/2 m-auto">
    <div class="p-5">
      <nuxt-link class="mt-20 bg-red-500 hover:bg-red-700 text-white font-bold py-2 px-4 rounded" to="/posts">Back</nuxt-link>
    </div>
    <div class="p-5 border-2 rounded">
      <h1 class="text-center mb-2">Editing post {{post?.title}}</h1>
      <UForm :schema="schema" :state="state" class="space-y-4" @submit="onSubmit">
        <UFormGroup label="Title" name="title">
          <UInput v-model="state.title" />
        </UFormGroup>

        <UFormGroup label="Slug" name="slug">
          <UInput v-model="state.slug" />
        </UFormGroup>

        <UFormGroup label="Content" name="content_raw">
          <UTextarea v-model="state.content_raw" />
        </UFormGroup>

        <UFormGroup label="Excerpt" name="excerpt">
          <UInput v-model="state.excerpt" />
        </UFormGroup>

        <UFormGroup label="Category" name="category_id">
          <USelect v-model="state.category_id" :options="categoryOptions" />
        </UFormGroup>

        <UFormGroup class="flex m-auto justify-center gap-3 items-center" label="Is published" name="is_published">
          <UCheckbox v-model="state.is_published" />
        </UFormGroup>

        <UButton class="flex m-auto mt-5" type="submit">
          Update
        </UButton>
      </UForm>
    </div>
  </div>
</template>

<script setup lang="ts">
import { z } from 'zod';
import { reactive, ref, computed, onMounted } from 'vue';
import axios from 'axios';
import { useRoute } from 'vue-router';

interface Category {
  id: number;
  title: string;
}
interface Post{
  id: number;
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
    console.log(categories.value);
  } catch (error) {
    console.error('Error fetching categories:', error);
  }
};
getCategories();

const route = useRoute();
const postId = ref<number | null>(null);
const post = ref<Post | null>(null);

const getPost = async (id: number) => {
  try {
    const response = await $fetch<Post>(`http://127.0.0.1:8000/api/blog/posts/${id}`);
    post.value = response;
    state.category_id = post.value.category_id.toString();
    state.title = post.value.title;
    state.slug = post.value.slug;
    state.is_published = !!post.value.is_published; // Convert to boolean
    state.excerpt = post.value.excerpt;
    if(post.value.excerpt === null)
    {
      state.excerpt = ''
    }
    state.content_raw = post.value.content_raw;
  } catch (error) {
    console.error('Error fetching post:', error);
  }
};

onMounted(() => {
  const id = parseInt(route.params.id as string, 10);
  if (!isNaN(id)) {
    postId.value = id;
    getPost(id);
  } else {
    console.error('Invalid post ID');
  }
});

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
        return !posts.value.some(post => post.title === value && post.id !== postId.value)
      }, 'Post title must be unique'),
  slug: z.string().max(200, 'Slug must be less than 200 characters').
  refine(value => {
    return !posts.value.some(post => post.slug === value && post.id !== postId.value)
  }, 'Post slug must be unique').optional(),
  content_raw: z.string().min(5, 'Description must be at least 5 characters').max(10000, 'Content must be less than 10000 characters'),
  category_id: z.string().refine(value => {
    const numberValue = Number(value);
    return Number.isInteger(numberValue) && categories.value.some(category => category.id === numberValue);
  }, 'Category ID must be a valid integer and exist in categories'),
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
      category_id: Number(result.category_id),
      is_published: !!result.is_published // Ensure boolean
    };

    // Send the data to the server
    const response = await axios.put(`http://127.0.0.1:8000/api/blog/posts/${postId.value}`, dataToSend);
    alert("Post successfully updated!");
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