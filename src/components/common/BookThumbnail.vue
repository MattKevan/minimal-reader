<template>
  <div
    class="book-item overflow-hidden relative hover:cursor-pointer hover:shadow-lg transition-shadow rounded-r-xl border border-gray-300 dark:border-neutral-700"
    @click="$emit('open-book', book.fileName)"
  >
    <img
      v-if="book.coverUrl"
      :src="book.coverUrl"
      alt="Book cover"
      class="book-cover aspect-[3/4.5] w-full rounded-r-xl hover:shadow-lg duration-500 object-cover"
    />
    <div
      v-else
      class="default-cover w-full aspect-[3/4.5] rounded-r-xl flex flex-col p-4 hover:shadow-lg duration-500 bg-gray-200 dark:bg-gray-800 overflow-hidden"
    >
      <span class="font-bold text-xl tracking-tight mb-2">{{ book.title }}</span>
      <span v-if="book.author && book.author !== 'Unknown Author'" class="text-sm text-gray-600 dark:text-gray-400">{{ book.author }}</span>
    </div>
    <button
      @click.stop="$emit('delete-book', book.fileName)"
      class="delete-icon absolute bottom-2 right-2 p-2 rounded-full"
    >
      Delete
    </button>
  </div>
</template>

<script>
export default {
  name: 'BookThumbnail',
  props: {
    book: {
      type: Object,
      required: true
    }
  }
}
</script>