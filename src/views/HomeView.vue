<template>
<div class="drop"
  @dragover.prevent
    @drop="uploadBooks">

    <div class="header">
      <p>Simple reader</p>
<div @click="triggerUpload" style="cursor: pointer;" class="upload-button">
<input type="file" @change="uploadBooks" id="bookInput" multiple style="display: none" />

    </div>
    </div>
    <div class="content">
    <h1>Library</h1>

      <div v-if="books.length === 0" class="placeholder"><p><strong>Your library is empty.</strong> Add .epub books by dropping them on this window or clicking the plus icon above.</p><p class="xs">Please note: Books are stored locally in your browser's cache, so clearing your browsing data will also remove your books.</p></div>

    <div id="bookCatalog" class="book-grid">
      <div 
        class="book-item" 
        v-for="(book, index) in books" 
        :key="index" 
        @click="openBook(book.fileName)"
      >
        <img :src="book.coverUrl" alt="Book cover" class="book-cover" />
          <button @click.stop="deleteBook(book.fileName)" class="delete-icon">
    Delete
  </button>

      </div>
    </div>
  </div>

</div>

</template>

<script>
import ePub from 'epubjs'
import localforage from 'localforage'

export default {
  name: 'HomeView',
  data() {
    return {
      books: [] // Store book data here
    }
  },
  methods: {
   createDefaultCover(title) {
    const canvas = document.createElement('canvas');
    canvas.width = 220; // set the dimensions you want
    canvas.height = 320;
    const context = canvas.getContext('2d');
    context.fillStyle = '#666'; // choose a default color
    context.fillRect(0, 0, canvas.width, canvas.height);
    context.font = '18px Gentium Book Plus'; // choose a font
    context.fillStyle = 'white';
    context.textAlign = 'center';
    context.fillText(title, canvas.width / 2, canvas.height / 2);
    return canvas.toDataURL();
  },

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

      let coverUrl;
      if (book.cover) {
        try {
          coverUrl = await book.archive.createUrl(book.cover);
        } catch (error) {
          console.error(`Error creating URL for book cover: ${error}`);
          coverUrl = this.createDefaultCover(metadata.title);
        }
      } else {
        console.error(`Cover for book ${file.name} not found.`);
        coverUrl = this.createDefaultCover(metadata.title);
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

    if (book.cover) {
      book.archive.createUrl(book.cover)
        .then(url => {
          this.books.push({
            fileName: key,
            title: metadata.title,
            coverUrl: url
          });
        })
        .catch(error => {
          console.error(`Error creating URL for book cover: ${error}`);
        });
    } else {
      // Default cover and title when book doesn't have its cover
      this.books.push({
        fileName: key,
        title: metadata.title,
        coverUrl: this.createDefaultCover(metadata.title) // use createDefaultCover method
      });
    }
  }
}  


}
</script>

<style>
/* Add styles for your grid here */
</style>
