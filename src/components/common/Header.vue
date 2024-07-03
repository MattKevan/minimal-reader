<!-- Header.vue -->
<template>
  <header class="flex justify-start flex-nowrap z-50 w-full bg-white border-b border-gray-200 text-sm py-3 sm:py-0 dark:bg-neutral-800 dark:border-neutral-700">
    <nav class="relative w-full px-4 sm:flex items-center justify-between sm:px-6 lg:px-8" aria-label="Global">
      <div class="flex items-center justify-between">
        <a class="flex-none text-xl font-semibold dark:text-white" href="#" aria-label="Brand">Minimal Reader</a>
      </div>
      <div id="navbar-collapse-with-animation" class="overflow-hidden transition-all duration-300 basis-full grow">
        <div class="flex flex-row items-center justify-end">
          <div @click="triggerUpload" style="cursor: pointer;" class="upload-button flex items-center gap-x-2 font-medium text-gray-500 hover:text-blue-600 py-2 sm:py-0 sm:ms-4 sm:my-6 sm:ps-6 dark:border-neutral-700 dark:text-neutral-400 dark:hover:text-blue-500">
            <input type="file" @change="handleFileChange" id="bookInput" multiple style="display: none" />
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-4">
              <path stroke-linecap="round" stroke-linejoin="round" d="M3 16.5v2.25A2.25 2.25 0 0 0 5.25 21h13.5A2.25 2.25 0 0 0 21 18.75V16.5m-13.5-9L12 3m0 0 4.5 4.5M12 3v13.5" />
            </svg>
            Add books
          </div>
        </div>
      </div>
    </nav>
  </header>
</template>

<script>
export default {
  name: 'AppHeader',  
  methods: {
    triggerUpload() {
      document.getElementById('bookInput').click();
    },
    handleFileChange(event) {
      this.$emit('upload-books', event);
    }
  }
}
</script>