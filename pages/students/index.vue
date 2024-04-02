<script setup lang="ts">
useHead({
  title: 'Students'
});

const columns = [

  { key: 'firstName', label: 'Name', sortable: true },
  { key: 'lastName', label: 'LastName', sortable: true },
  { key: 'group', label: 'Group', sortable: true }
];

const students = [
  {
    id: 1,
    firstName: 'Іван',
    lastName: 'Петров',
    group: 'КН-22'
  },
  {
    id: 2,
    firstName: 'Марина',
    lastName: 'Сидорова',
    group: 'КН-22'
  },
  {
    id: 3,
    firstName: 'Олег',
    lastName: 'Михайлов',
    group: 'КН-21'
  },
  {
    id: 4,
    firstName: 'Михайло',
    lastName: 'Лук’янчук',
    group: 'КН-22'
  },
  {
    id: 5,
    firstName: 'Наталя',
    lastName: 'Терещук',
    group: 'КН-22'
  },
  {
    id: 6,
    firstName: 'Віталій',
    lastName: 'Поліщук',
    group: 'КН-22'
  },
  {
    id: 7,
    firstName: 'Ірина',
    lastName: 'Гнатюк',
    group: 'КН-22'
  },
  {
    id: 8,
    firstName: 'Андрій',
    lastName: 'Лавренюк',
    group: 'КН-21'
  },
  {
    id: 9,
    firstName: 'Марія',
    lastName: 'Ковальчук',
    group: 'КН-21'
  },
  {
    id: 10,
    firstName: 'Сергій',
    lastName: 'Васильєв',
    group: 'КН-23'
  },
  {
    id: 11,
    firstName: 'Анастасія',
    lastName: 'Сидоренко',
    group: 'КН-21'
  },
  {
    id: 12,
    firstName: 'Ігор',
    lastName: 'Мельник',
    group: 'КН-23'
  },
  {
    id: 13,
    firstName: 'Тетяна',
    lastName: 'Бойко',
    group: 'КН-24'
  },
  {
    id: 14,
    firstName: 'Олексій',
    lastName: 'Шевченко',
    group: 'КН-24'
  },
  {
    id: 15,
    firstName: 'Юлія',
    lastName: 'Соловйова',
    group: 'КН-23'
  },
  {
    id: 16,
    firstName: 'Марк',
    lastName: 'Козлов',
    group: 'КН-24'
  },
  {
    id: 17,
    firstName: 'Софія',
    lastName: 'Павленко',
    group: 'КН-24'
  },
  {
    id: 18,
    firstName: 'Євген',
    lastName: 'Григоров',
    group: 'КН-23'
  },
  {
    id: 19,
    firstName: 'Ірина',
    lastName: 'Петренко',
    group: 'КН-24'
  },
  {
    id: 20,
    firstName: 'Олена',
    lastName: 'Кузнецова',
    group: 'КН-23'
  }
];

const sort = ref({ column: 'title', direction: 'asc' as const })
const sortedRows = computed(() => {
  const sortedProducts = [...students]
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

const page = ref(1);
const pageCount = 4;

const rows = computed(() => {
  let filteredProducts = [...sortedRows.value]

  if (q.value) {
    filteredProducts = filteredProducts.filter(product => {
      return Object.values(product).some(value => {
        return String(value).toLowerCase().includes(q.value.toLowerCase())
      })
    })
  }

  const startIndex = (page.value - 1) * pageCount
  const endIndex = startIndex + pageCount

  return filteredProducts.slice(startIndex, endIndex)
})

const q = ref('');
const filteredRows = computed(() => {
  if (!q.value) {
    return students
  }

  return students.filter((person) => {
    return Object.values(person).some((value) => {
      return String(value).toLowerCase().includes(q.value.toLowerCase())
    })
  })
});


watch(q, () => {
  page.value = 1
});

</script>

<template>
  <div>
    <div class="flex justify-center px-3 py-3.5 border-b border-gray-200 dark:border-gray-700">
      <UInput class="w-1/4" v-model="q" placeholder="Пошук..."/>
    </div>

    <UTable :rows="rows" , :columns="columns" v-model:sort="sort"/>

    <div class="flex justify-center px-3 py-3.5 border-t border-gray-200 dark:border-gray-700">
      <UPagination v-model="page" :page-count="pageCount" :total="filteredRows.length"/>
    </div>
  </div>
</template>