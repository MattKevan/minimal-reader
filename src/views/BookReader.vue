<template>
  <div class="">
    <div class="">
      <button @click="goHome" class="menu-button back bg-[url('assets/img/grid.svg')] dark:bg-[url('assets/img/grid-white.svg')]">Home</button>
      <button class="menu-button toc bg-[url('assets/img/list.svg')] dark:bg-[url('assets/img/list-white.svg')]" data-hs-overlay="#hs-overlay-right">Contents</button>    
    </div>

    <div id="book-title" class="font-bold text-center py-4"></div>
    <div id="viewer" class="scrolled max-w-4xl ml-auto mr-auto mb-20"  :class="{ 'hidden': isResizing }"></div>

    <div class="">
      <button id="prev" @click="goPrev" class="bg-[url('assets/img/back.svg')] dark:bg-[url('assets/img/back-white.svg')]">Previous</button>
      <button id="next" @click="goNext" class="bg-[url('assets/img/forward.svg')] dark:bg-[url('assets/img/forward-white.svg')]">Next</button>
    </div>

</div>
    <div id="hs-overlay-right" class="hs-overlay hs-overlay-open:translate-x-0 hidden translate-x-full fixed top-0 end-0 transition-all duration-300 transform h-full max-w-xs w-full z-[2800] bg-white border-s dark:bg-neutral-800 dark:border-neutral-700" tabindex="-1">
      <div class="flex justify-between items-center py-3 px-4 border-b dark:border-neutral-700">
        <h3 class="font-bold text-gray-800 dark:text-white">
          Contents
        </h3>
        <button type="button" class="flex justify-center items-center size-7 text-sm font-semibold rounded-full border border-transparent text-gray-800 hover:bg-gray-100 disabled:opacity-50 disabled:pointer-events-none dark:text-white dark:hover:bg-neutral-700" data-hs-overlay="#hs-overlay-right">
          <span class="sr-only">Close modal</span>
          <svg class="flex-shrink-0 size-4" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M18 6 6 18"></path>
            <path d="m6 6 12 12"></path>
          </svg>
        </button>
      </div>
      <div class="p-4">
        <ul id="toc" class="space-y-2">
          <li v-for="(chapter, index) in toc" :key="index">
            <a href="#" @click.prevent="displayChapter(chapter.href, true)" class="text-blue-600 hover:text-blue-800">
              {{ chapter.label }}
            </a>
          </li>
        </ul>
      </div>
    </div>
  

</template>

<script>
import ePub from 'epubjs'
import localforage from 'localforage'

export default {
  name: 'BookReader',
  components: {
  },
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
      toc: [],
      resizeTimeout: null,
      isResizing: false,

    }
  },
  methods: {
    async loadBook() {
      try {
        const bookItem = await localforage.getItem(this.fileName);
        if (bookItem && bookItem.data) {
          this.book = ePub(bookItem.data);

          this.rendition = this.book.renderTo("viewer", {
            flow: "scrolled-doc",
            width: "100%",
            height: "100%",
            fullsize: true,
            enableSwipe: false,
          });

          this.defineHooks();
          await this.loadTOC();
          await this.displayBook();

          // Set the book title
          document.getElementById('book-title').textContent = await this.book.loaded.metadata.then(metadata => metadata.title);
        } else {
          console.error('Book not found in local storage or invalid book data');
          this.$router.push({ name: 'Home' });
        }
      } catch (error) {
        console.error('Error loading book:', error);
        this.$router.push({ name: 'Home' });
      }
    },
    
    handleKeydown(event) {
      if (event.key === 'ArrowLeft') {
        this.goPrev(event);
      } else if (event.key === 'ArrowRight') {
        this.goNext(event);
      } else if (event.key === 'Escape') {
        this.goHome();
      }
    },

    defineHooks() {
      this.book.rendition.hooks.content.register((contents) => {
        let doc = contents.document;
        let head = doc.querySelector('head');

        // Remove existing stylesheets
        Array.from(head.querySelectorAll('link[rel="stylesheet"], style'))
          .forEach(el => el.remove());

          // Modify links
          const links = doc.querySelectorAll('a');
        links.forEach(link => {
          const href = link.getAttribute('href');
          if (href) {
            if (href.startsWith('http://') || href.startsWith('https://')) {
              // External link: open in new tab
              link.setAttribute('target', '_blank');
            } else {
              // Internal link: remove href and make it non-clickable
              link.removeAttribute('href');
              link.style.textDecoration = 'none';
              link.style.color = 'inherit';
              link.style.cursor = 'text';
            }
          }
        });


        // Inject minimal Tailwind styles
        let style = doc.createElement('style');
        style.textContent = this.getMinimalTailwindStyles();
        head.appendChild(style);

        // Remove empty paragraphs
        var paras = doc.getElementsByTagName('p');
        for (var i = paras.length - 1; i >= 0; i--) {
          if (paras[i].innerHTML.replace(/\s|&nbsp;/g, '').length == 0)
            paras[i].parentNode.removeChild(paras[i]);
        }
        // Convert all-caps headings to sentence case
        this.convertAllCapsHeadings(doc);

        // Add Tailwind-like typography classes to body
        doc.body.classList.add('prose', 'mx-auto', 'px-4');

        // Apply dark mode styles
        if (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches) {
          doc.body.classList.add('dark');
        }
      });

      this.rendition.on("relocated", (location) => {
        localStorage.setItem(`epub-location-${this.fileName}`, location.start.cfi);
      });
    },
    convertAllCapsHeadings(doc) {
      const headings = doc.querySelectorAll('h1, h2, h3, h4, h5, h6');
      headings.forEach(heading => {
        if (this.isAllCaps(heading.textContent)) {
          heading.textContent = this.toSentenceCase(heading.textContent);
        }
      });
    },

    isAllCaps(text) {
      return text === text.toUpperCase() && text !== text.toLowerCase();
    },

    toSentenceCase(text) {
      return text.charAt(0).toUpperCase() + text.slice(1).toLowerCase();
    },

    handleInternalLink(href) {
      if (href.startsWith('#')) {
        // It's an anchor within the current chapter
        this.rendition.display(href);
      } else {
        // It's a link to another chapter
        const item = this.book.spine.get(href);
        if (item) {
          this.rendition.display(item.href);
        } else {
          console.error(`Unable to find item for href: ${href}`);
        }
      }
    },

    getMinimalTailwindStyles() {
      return `
        @import url('https://fonts.googleapis.com/css2?family=Gentium+Book+Plus:ital,wght@0,400;0,700;1,400;1,700&display=swap');
        .prose { 
          font-family: 'Gentium Book Plus', sans-serif; 
          font-size: 20px;
          line-height: 180%;}
        .prose p { margin-bottom: 1em; }
        .prose h1, .prose h2, .prose h3, .prose h4 { margin-top: 1.5em; margin-bottom: 0.5em; font-weight: 700; }
        .prose h1 { font-size: 2.5em; line-height: 120%; }
        .prose h2 { font-size: 1.75em; line-height: 120%;}
        .prose h3 { font-size: 1.5em; line-height: 120%;}
        .prose h4 { font-size: 1.25em; line-height: 120%;}
        .prose strong {font-weight: 700; }
        .prose a { color: #333; text-decoration: none; }
        .prose ul, .prose ol { margin-top: 1em; margin-bottom: 1em; padding-left: 1.5em; }
        .prose li { margin-bottom: 0.25em; }
        .prose li p { margin: 0;}
        .prose img { 
        margin-top: 1em; margin-bottom: 1em; 
        width: 80%;
        height: 100%;
        margin-left:auto;
        margin-right:auto;}
        @media (prefers-color-scheme: dark) {
          body.dark { background-color: #000; color: #e2e8f0; }
          body.dark a { color: #e2e8f0; }
        }
      `;
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

    saveCurrentLocation() {
      if (this.rendition) {
        const currentLocation = this.rendition.currentLocation();
        if (currentLocation && currentLocation.start) {
          localStorage.setItem(`epub-location-${this.fileName}`, currentLocation.start.cfi);
        }
      }
    },

    displayChapter(href) {
      if (this.rendition) {
        this.closeSidebar();  
        this.rendition.display(href);
      }
    },


    openSidebar() {
      document.getElementById('toc-sidebar').classList.remove('translate-x-full');
      document.getElementById('overlay').classList.remove('hidden');
      document.body.classList.add('overflow-hidden');
    },

    closeSidebar() {
   // Use the third-party library to close the sidebar
   if (window.HSOverlay) {
        window.HSOverlay.close(document.querySelector('#hs-overlay-right'));
      } else {
        console.warn('HSOverlay not found. Make sure the library is properly loaded.');
      }
    },

    goHome() {
      this.$router.push({ name: 'Home' });
    },

    handleFileInput(e) {
      var file = e.target.files[0];
      if (file.type !== "application/epub+zip") {
        alert("Please select an EPUB file.");
        return;
      }
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

  mounted() {
    window.addEventListener('keydown', this.handleKeydown);
    window.addEventListener('resize', this.handleResize);
    window.addEventListener('beforeunload', this.saveCurrentLocation);

    if (!this.fileName) {
      const savedFileName = localStorage.getItem('currentBook');
      if (savedFileName) {
        this.$router.replace({ name: 'BookReader', params: { fileName: savedFileName } });
      } else {
        this.$router.push({ name: 'Home' });
      }
    } else {
      localStorage.setItem('currentBook', this.fileName);
      this.loadBook();
    }

  },

  beforeUnmount() {
    window.removeEventListener('keydown', this.handleKeydown);
    window.removeEventListener('resize', this.handleResize);
    window.removeEventListener('beforeunload', this.saveCurrentLocation);

  },
};
</script>

<style scoped>
/* Tailwind classes are used instead of custom styles */
</style>