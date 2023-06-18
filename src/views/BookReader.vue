<template>

<button @click="goHome" class="menu-button back">Home</button>

<button @click="openSidebar" id="toc-link" class="menu-button contents">Open Sidebar</button>

<div id="book-title"></div>
  <div id="viewer" class="scrolled"></div>
<button id="next" @click="goNext">Next</button>
<button id="prev" @click="goPrev">Previous</button>

  <div id="toc-sidebar">
    <button id="close-sidebar" @click="closeSidebar">Close</button>
    <ul id="toc">
      <li v-for="(chapter, index) in toc" :key="index">
        <a href="#" @click.prevent="displayChapter(chapter.href)">{{ chapter.label }}</a>
      </li>
    </ul>
  </div>
<div id="overlay" @click="closeSidebar"></div>


</template>

<script>

import ePub from 'epubjs'
import localforage from 'localforage'

export default {
  name: 'BookReader',
  props: {
    fileName: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      book: null,
      rendition: null,
      toc: []
    }
  },
  methods: {
   async loadBook() {
  try {
    const bookData = await localforage.getItem(this.fileName);
    if (bookData) {
      this.book = ePub(bookData); 

      this.rendition = this.book.renderTo("viewer", {
        flow: "scrolled-doc",
        width: "100%",
        height: "100%",
        fullsize: true,
        enableSwipe: false
      });

      this.defineHooks();
      await this.loadTOC();
      await this.displayBook();
    } else {
      alert('Book not found');
    }
  } catch (error) {
    console.error('Error loading book:', error);
  }
},

    defineHooks() {
      this.book.rendition.hooks.content.register((contents) => {
        let doc = contents.document;
        let styleEl = doc.createElement("style");
        styleEl.textContent = `
          @import url('https://fonts.googleapis.com/css2?family=Gentium+Book+Plus:ital,wght@0,400;0,700;1,400;1,700&display=swap');

          body, p {
            font-family: 'Gentium Book Plus', sans-serif !important;
            font-size: 18px !important;
            line-height: 1.6em !important;
            text-align: left !important;
          }
          p, p.indent {
            text-indent: 0em !important;
            margin-bottom: 1em !important;
            margin-top: 0 !important;
          }
          a {
            color: #333 !important;
            text-decoration: none;
          }
          a:hover {
            text-decoration: underline !important;
            color: #555 !important;
          }
          h1, h2, h3, h4, h5, h6 {
            font-family: 'Gentium Book Plus', sans-serif !important;
            color:#333 !important;
            margin: 1em 0 !important;
            text-align: left !important;
            line-height: 1.4em !important;

          }
          li, ol {
            font-family: 'Gentium Book Plus', sans-serif !important;
            font-size: 18px !important;
            line-height: 1.6em !important;

          }
          @media (prefers-color-scheme: dark) {
            body {
              background-color: #000;
              color: #999 !important;
            }
            body, p, a,h1, h2, h3, h4, h5, h6, ol, ul, li {
              color: #999 !important;
            }
            a {
              color: #999 !important;
              text-decoration: none;
            }
            a:hover {
              text-decoration: underline;
              color: #777 !important;
            }
          }
      `;
        doc.head.appendChild(styleEl);
      });

      

      this.rendition.on("relocated", (location) => {
        localStorage.setItem(`epub-location-${this.fileName}`, location.start.cfi);
      });
    },
    goNext(event) {
          event.preventDefault();
          this.rendition.next();
        },
        goPrev(event) {
          event.preventDefault();
          this.rendition.prev();
        },
 async loadTOC() {
    const navigation = await this.book.loaded.navigation;
    const tocDiv = document.getElementById('toc');

    const createTOCEntry = (parent, chapter) => {
      var listItem = document.createElement('li');
      var link = document.createElement('a');
      link.textContent = chapter.label;
      link.href = chapter.href;
      link.addEventListener('click', function(e){
        e.preventDefault();
        this.rendition.display(chapter.href);
        this.closeSidebar();  // add this line
      }.bind(this));

      listItem.appendChild(link);
      parent.appendChild(listItem);

      if (chapter.subitems.length > 0) {
        var sublist = document.createElement('ul');
        chapter.subitems.forEach((subitem) => {
          createTOCEntry(sublist, subitem);
        });
        listItem.appendChild(sublist);
      }
    }

    navigation.toc.forEach((chapter) => {
      createTOCEntry(tocDiv, chapter);
    });

    this.toc = navigation.toc;
  },
    async displayBook() {
      try {
      const savedLocation = localStorage.getItem(`epub-location-${this.fileName}`);
      const currentSectionIndex = savedLocation || undefined;
      await this.rendition.display(currentSectionIndex);
        } catch (error) {
    console.error('Error displaying book:', error);
  }
    },
    displayChapter(href) {
      this.rendition.display(href);
    },
    openSidebar() {
      var sidebar = document.getElementById('toc-sidebar');
      var body = document.body;
      var overlay = document.getElementById('overlay');
      sidebar.classList.add('open');
      body.classList.add('no-scroll');
      overlay.style.display = 'block'; // Show the overlay
    },
    closeSidebar() {
      var sidebar = document.getElementById('toc-sidebar');
      var body = document.body;
      var overlay = document.getElementById('overlay');
      sidebar.classList.remove('open');
      body.classList.remove('no-scroll');
      overlay.style.display = 'none'; // Hide the overlay
    },
    handleFileInput(e) {
      var file = e.target.files[0];
      if (file.type !== "application/epub+zip") {
        alert("Please select an EPUB file.");
        return;
    }
  },
 goHome() {
    this.$router.push({ name: 'Home' }); // Assuming 'Home' is the name of your home route
  },
  mounted() {
    //this.loadBook();
  },

},
watch: {
    fileName: {
      handler(newVal, oldVal) {
        if (newVal !== oldVal) {
          this.loadBook();
        }
      },
      immediate: true,
    },
},
};
</script>

<style scoped>
/* Your styles go here */
</style>

