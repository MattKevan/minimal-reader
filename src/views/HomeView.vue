<template>
  <AppHeader @upload-books="uploadBooks" />

  <div class="drop" @dragover.prevent @drop="uploadBooks">
    <div class="content w-full px-4 sm:px-6 lg:px-8">
      <h1 class="md:my-12 my-8 text-4xl md:text-5xl font-bold">Library</h1>

      

      <div id="bookCatalog" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-6 gap-4 sm:gap-6">
        <div v-if="books.length === 0" class="placeholder">
        <p><strong>Your library is empty.</strong> Add .epub books by dropping them on this window or clicking the plus icon above.</p>
        <p class="xs">Please note: Books are stored locally in your browser's cache, so clearing your browsing data will also remove your books.</p>
      </div>
        <div 
          class="book-item overflow-hidden relative hover:cursor-pointer hover:shadow-lg transition-shadow rounded-r-xl border border-gray-300" 
          v-for="(book, index) in books" 
          :key="index" 
          @click="openBook(book.fileName)"
        >
          <img v-if="book.coverUrl" :src="book.coverUrl" alt="Book cover" class="book-cover aspect-[3/4] w-full rounded-r-xl hover:shadow-lg duration-500 object-cover" />
          <div v-else class="default-cover w-full aspect-[3/4] rounded-r-xl flex p-4 hover:shadow-lg duration-500 bg-gray-50 overflow-hidden">
            <span class="font-bold text-2xl tracking-tight">{{ book.title }}</span>
          </div>
          <button @click.stop="deleteBook(book.fileName)" class="delete-icon absolute bottom-2 right-2 p-2 rounded-full">
            Delete
          </button>
        </div>
      </div>
    </div>
  </div>

  <AppFooter />
</template>

<script>
import ePub from 'epubjs'
import localforage from 'localforage'
import AppHeader from '@/components/common/Header.vue'
import AppFooter from '@/components/common/Footer.vue'

export default {
  name: 'HomeView',
  components: {
    AppHeader,
    AppFooter
  },
  data() {
    return {
      books: [] // Store book data here
    }
  },
  methods: {
    async uploadBooks(event) {
      event.preventDefault();
      const files = Array.from(event.dataTransfer ? event.dataTransfer.files : event.target.files);

      for (let file of files) {
        if (file.type !== "application/epub+zip") {
          alert("Please select an EPUB file.");
          continue;
        }

        // Read the book file
        const bookData = await new Response(file).arrayBuffer();

        // Store the book in localForage with filename as the key
        await localforage.setItem(file.name, bookData);

        // Parse and display the book metadata
        const book = ePub(bookData);
        const metadata = await book.loaded.metadata;

        let coverUrl = null;
        if (book.cover) {
          try {
            coverUrl = await book.archive.createUrl(book.cover);
          } catch (error) {
            console.error(`Error creating URL for book cover: ${error}`);
          }
        }

        this.books.push({
          fileName: file.name,
          title: metadata.title,
          coverUrl: coverUrl
        });
      }
    },
    openBook(fileName) {
      this.$router.push({ name: 'BookReader', params: { fileName } });
      console.log(fileName);
    },
    triggerUpload() {
      document.getElementById('bookInput').click();
    },
    async deleteBook(fileName) {
      await localforage.removeItem(fileName);
      this.books = this.books.filter(book => book.fileName !== fileName);
    },
  },
  async created() {
    // Load books from local storage
    const keys = await localforage.keys();
    for (const key of keys) {
      const bookData = await localforage.getItem(key);
      const book = ePub(bookData);
      const metadata = await book.loaded.metadata;

      let coverUrl = null;
      if (book.cover) {
        try {
          coverUrl = await book.archive.createUrl(book.cover);
        } catch (error) {
          console.error(`Error creating URL for book cover: ${error}`);
        }
      }

      this.books.push({
        fileName: key,
        title: metadata.title,
        coverUrl: coverUrl
      });
    }
  }
}
</script>

<style scoped>
.default-cover {
  
}
</style>