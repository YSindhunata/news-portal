<script setup>
import { ref, onMounted, computed } from 'vue'
import axios from 'axios'

// ref() digunakan untuk membuat variabel reaktif
const articles = ref([]) // Untuk menampung semua artikel berita
const searchQuery = ref('') // Untuk menampung input dari kotak pencarian
const isLoading = ref(true) // Untuk menampilkan status loading
const error = ref(null) // Untuk menampilkan pesan error jika ada
const isSearchVisible = ref(false)

// API FETCHING
onMounted(() => {
  fetchNews()
})

// Fungsi untuk mengambil data dari berbagai API
const fetchNews = async () => {
  isLoading.value = true
  error.value = null

  // Ambil API keys dari .env file (API disimpan di file .env.local)
  const newsApiKey = import.meta.env.VITE_NEWSAPI_KEY
  const gnewsApiKey = import.meta.env.VITE_GNEWS_KEY
  const newsdataApiKey = import.meta.env.VITE_NEWSDATA_KEY

  // endpoint API yang akan dipanggil. Mencari dengan kata kunci "artificial intelligence"
  const url1 = `https://newsapi.org/v2/everything?q=artificial%20intelligence&apiKey=${newsApiKey}&pageSize=20`
  const url2 = `https://gnews.io/api/v4/search?q=artificial%20intelligence&l&apikey=${gnewsApiKey}&max=20`
  const url3 = `https://newsdata.io/api/1/latest?apikey=${newsdataApiKey}&q=artificial%20intelligence`

  try {
    // Promise.all untuk memanggil semua API secara paralel agar lebih cepat
    const responses = await Promise.all([axios.get(url1), axios.get(url2), axios.get(url3)])

    // Menggabungkan dan memformat data dari semua response
    let combinedArticles = []
    responses.forEach((response) => {
      // Sesuaikan 'response.data.articles' dengan struktur data dari API
      if (response.data.articles) {
        combinedArticles = combinedArticles.concat(response.data.articles)
      }
    })

    // Menghapus duplikat berdasarkan URL artikel
    const uniqueArticles = Array.from(
      new Map(combinedArticles.map((item) => [item['url'], item])).values(),
    )

    // mensortir artikel dari yang terbaru
    articles.value = uniqueArticles.sort(
      (a, b) => new Date(b.publishedAt) - new Date(a.publishedAt),
    )
  } catch (e) {
    console.error('Gagal mengambil data berita:', e)
    error.value = 'Maaf, terjadi kesalahan saat mengambil berita. Silakan coba lagi nanti.'
  } finally {
    isLoading.value = false
  }
}

// FITUR PENCARIAN
// computed property untuk memfilter berita secara reaktif berdasarkan searchQuery
const filteredArticles = computed(() => {
  if (!searchQuery.value) {
    return articles.value
  }
  return articles.value.filter((article) =>
    article.title.toLowerCase().includes(searchQuery.value.toLowerCase()),
  )
})

// Fungsi helper untuk memformat tanggal
const formatDate = (dateString) => {
  return new Date(dateString).toLocaleString('id-ID', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
  })
}
</script>

<template>
  <div class="bg-gray-100 min-h-screen font-sans">
    <nav class="bg-gray-900 sticky top-0 z-10">
      <div class="container mx-auto px-4 sm:px-6">
        <div class="flex items-center justify-between h-16">
          <div class="flex-shrink-0">
            <a href="/" class="flex items-center space-x-3">
              <span class="text-2xl font-semibold text-white">Portal AI News</span>
            </a>
          </div>

          <div class="hidden md:flex items-center">
            <div class="relative">
              <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                <svg class="w-5 h-5 text-gray-400" fill="currentColor" viewBox="0 0 20 20">
                  <path
                    fill-rule="evenodd"
                    d="M8 4a4 4 0 100 8 4 4 0 000-8zM2 8a6 6 0 1110.89 3.476l4.817 4.817a1 1 0 01-1.414 1.414l-4.816-4.816A6 6 0 012 8z"
                    clip-rule="evenodd"
                  ></path>
                </svg>
              </div>
              <input
                type="text"
                v-model="searchQuery"
                class="block w-full pl-10 pr-4 py-2 border border-transparent rounded-lg leading-5 bg-gray-700 text-gray-300 placeholder-gray-400 focus:outline-none focus:bg-white focus:text-gray-900 sm:text-sm transition duration-150 ease-in-out"
                placeholder="Cari berita..."
              />
            </div>
          </div>

          <div class="md:hidden flex items-center">
            <button
              @click="isSearchVisible = !isSearchVisible"
              class="p-2 rounded-md text-gray-400 hover:text-white hover:bg-gray-700 focus:outline-none focus:ring-2 focus:ring-inset focus:ring-white"
            >
              <svg
                class="w-6 h-6"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
                xmlns="http://www.w3.org/2000/svg"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
                ></path>
              </svg>
            </button>
          </div>
        </div>
      </div>

      <div v-show="isSearchVisible" class="md:hidden px-4 pb-3">
        <div class="relative">
          <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
            <svg class="w-5 h-5 text-gray-400" fill="currentColor" viewBox="0 0 20 20">
              <path
                fill-rule="evenodd"
                d="M8 4a4 4 0 100 8 4 4 0 000-8zM2 8a6 6 0 1110.89 3.476l4.817 4.817a1 1 0 01-1.414 1.414l-4.816-4.816A6 6 0 012 8z"
                clip-rule="evenodd"
              ></path>
            </svg>
          </div>
          <input
            type="text"
            v-model="searchQuery"
            class="block w-full pl-10 pr-4 py-2 border border-transparent rounded-lg leading-5 bg-gray-700 text-gray-300 placeholder-gray-400 focus:outline-none focus:bg-white focus:text-gray-900 sm:text-sm transition duration-150 ease-in-out"
            placeholder="Cari berita..."
          />
        </div>
      </div>
    </nav>

    <main class="container mx-auto p-4 sm:p-6">
      <div v-if="isLoading" class="text-center py-20">
        <p class="text-xl text-gray-600">Sedang memuat berita terbaru...</p>
      </div>

      <div
        v-else-if="error"
        class="text-center py-20 bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded relative"
      >
        <p class="font-bold">Error!</p>
        <p>{{ error }}</p>
      </div>

      <div
        v-else-if="filteredArticles.length > 0"
        class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6"
      >
        <div
          v-for="article in filteredArticles"
          :key="article.url"
          class="bg-white rounded-lg shadow-lg overflow-hidden transform hover:-translate-y-1 transition-transform duration-300 flex flex-col"
        >
          <img
            :src="
              article.urlToImage || 'https://placehold.co/600x400/2d3748/ffffff?text=Portal+AI+News'
            "
            alt="News Image"
            class="h-48 w-full object-cover"
          />
          <div class="p-6 flex flex-col flex-grow">
            <h2 class="font-bold text-xl mb-2 text-gray-800 line-clamp-3">{{ article.title }}</h2>
            <p class="text-gray-600 text-sm mb-4">
              Oleh {{ article.author || 'Tidak diketahui' }} | {{ article.source.name }}
            </p>
            <p class="text-gray-500 text-xs mb-4">{{ formatDate(article.publishedAt) }}</p>
            <a
              :href="article.url"
              target="_blank"
              class="inline-block bg-gray-900 hover:bg-gray-500 text-white font-bold py-2 px-4 rounded-full transition-colors duration-300 mt-auto self-start"
            >
              Baca Selengkapnya
            </a>
          </div>
        </div>
      </div>

      <div v-else class="text-center py-20">
        <p class="text-xl text-gray-600">Tidak ada berita yang cocok dengan pencarian Anda.</p>
      </div>
    </main>
  </div>
</template>
