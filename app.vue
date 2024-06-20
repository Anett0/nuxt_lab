<template>
  <div>
    <nav class="flex justify-around p-5 mb-5">
      <a href="/categories" class="inline-flex items-center font-medium rounded-md text-lg px-5 py-1 bg-gray-50 dark:bg-gray-400 dark:bg-opacity-10 text-white dark:text-white ring-1 ring-inset ring-gray-500 dark:ring-gray-400 ring-opacity-25 dark:ring-opacity-25">Categories</a>
      <a href="/posts" class="inline-flex items-center font-medium rounded-md text-lg px-5 py-1 bg-gray-50 dark:bg-gray-400 dark:bg-opacity-10 text-white dark:text-white ring-1 ring-inset ring-gray-500 dark:ring-gray-400 ring-opacity-25 dark:ring-opacity-25">Posts</a>
    </nav>
    <hr class="mb-5">
    <NuxtPage />
  </div>
</template>