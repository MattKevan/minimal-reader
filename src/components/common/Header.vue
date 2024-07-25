<!-- Header.vue -->
<template>
  <header class="flex sticky top-0 justify-start flex-nowrap z-50 w-full bg-white/90  backdrop-blur-md text-sm py-3 dark:bg-black/90 ">
    <nav class="relative w-full px-4 items-center justify-between sm:px-6" aria-label="Global">
      <div class="flex items-center justify-between">
        <a class="text-lg font-semibold" href="#" aria-label="Brand"><img src="text-logo.svg" alt="Minimal Reader" class="w-[80px]"></a>
        <div @click="triggerUpload" style="cursor: pointer;" class="upload-button flex items-center font-medium text-gray-500 hover:text-blue-600 sm:ms-4  sm:ps-6 dark:border-neutral-700 dark:text-neutral-400 dark:hover:text-blue-500">
            <input type="file" @change="handleFileChange" id="bookInput" multiple style="display: none" />
            <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-4">
              <path stroke-linecap="round" stroke-linejoin="round" d="M12 4.5v15m7.5-7.5h-15" />
            </svg>

            Add books
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