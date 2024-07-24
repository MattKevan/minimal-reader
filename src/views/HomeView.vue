<template>
  <div class="flex flex-col min-h-screen">
  <AppHeader @upload-books="uploadBooks" />

  <div class="drop flex-grow" @dragover.prevent @drop="uploadBooks">
    <div class="content w-full px-4 sm:px-6 lg:px-8">
      <h1 class="md:my-12 my-8 text-4xl md:text-5xl font-bold">Library</h1>
      <div v-if="books.length === 0" class="placeholder">
        <p class="text-xl mb-2"><strong>Your library is empty.</strong></p>
        <p>Add DRM-free Epubs by dropping them on this window or by clicking the plus icon above.</p>
        <p class="xs">Books are stored in your browser's cache, so clearing your browsing data will also empty your library.</p>
      </div>
      

      <div id="bookCatalog" class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-6 gap-4 sm:gap-6">
        
        <BookThumbnail
          v-for="(book, index) in books"
          :key="index"
          :book="book"
          @open-book="openBook"
          @delete-book="deleteBook"
        />
      </div>
    </div>
  </div>
  <AppFooter class="mt-auto" />
</div>
</template>

<script>
import ePub from 'epubjs'
import localforage from 'localforage'
import AppHeader from '@/components/common/Header.vue'
import AppFooter from '@/components/common/Footer.vue'
import BookThumbnail from '@/components/common/BookThumbnail.vue'
export default {
  name: 'HomeView',
  components: {
    AppHeader,
    AppFooter,
    BookThumbnail
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
        try {
          const bookData = await new Response(file).arrayBuffer();
          const book = ePub(bookData);
          await book.loaded.metadata;

          await localforage.setItem(file.name, bookData);

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
            title: metadata.title || 'Untitled',
            author: metadata.creator || '', // Changed here
            coverUrl: coverUrl
          });

          console.log(`Successfully added ${file.name} to the library.`);
        } catch (error) {
          console.error(`Error processing file "${file.name}":`, error);
          alert(`Unable to process "${file.name}". It may not be a valid EPUB file.`);
        }
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
        title: metadata.title || 'Untitled',
        author: metadata.creator || '', // Changed here
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