<script setup>
import axios from 'axios'
import { onMounted, ref } from 'vue'
import { useToast } from 'vue-toast-notification'
import 'vue-toast-notification/dist/theme-sugar.css'

const $toast = useToast()
const url = 'http://localhost:1337/api'
const token =
  'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwiaWF0IjoxNzIxNDYwMzg1LCJleHAiOjE3MjQwNTIzODV9.n-OKkPpLJO5S1IgR0p4QeT2kOTvDhbDpkZFhZOeO348'

const dataArticles = ref([])
const dataCategories = ref([])
const formMode = ref('create')
const currentArticleId = ref(null)
const showModal = ref(false)

const getArticles = async () => {
  try {
    const res = await axios.get(url + '/blog-posts?populate=*', {
      headers: {
        Authorization: 'Bearer ' + token,
        'Content-Type': 'application/json'
      }
    })
    const data = await res.data
    dataArticles.value = data.data
  } catch (error) {
    console.log(error)
  }
}

const getCategories = async () => {
  try {
    const res = await axios.get(url + '/categories?populate=*', {
      headers: {
        Authorization: 'Bearer ' + token,
        'Content-Type': 'application/json'
      }
    })
    const data = await res.data
    dataCategories.value = data.data
  } catch (error) {
    console.log(error)
  }
}

const formData = ref({
  data: {
    title: '',
    body: '',
    author: '',
    is_saved: false,
    categories: {
      data: []
    }
  }
})

const handleEdit = (id) => {
  const article = dataArticles.value.find((article) => article.id === id)
  formData.value.data.title = article.attributes.title
  formData.value.data.body = article.attributes.body
  formData.value.data.author = article.attributes.author
  formData.value.data.is_saved = article.attributes.is_saved
  formData.value.data.categories = article.attributes.categories
  formMode.value = 'edit'
  currentArticleId.value = id
}

const handleOpenModal = (id) => {
  showModal.value = true
  currentArticleId.value = id
}

const handleDelete = () => {
  axios
    .delete(url + '/blog-posts/' + currentArticleId.value, {
      headers: {
        Authorization: 'Bearer ' + token,
        'Content-Type': 'application/json'
      }
    })
    .then(function () {
      formData.value.data.title = ''
      formData.value.data.body = ''
      formData.value.data.author = ''
      formData.value.data.is_saved = false
      formData.value.data.categories = {
        data: []
      }

      currentArticleId.value = null
      showModal.value = false

      getArticles()
      $toast.error('Article deleted!', {
        position: 'top-right'
      })
    })
    .catch(function (error) {
      console.log(error)
    })
}

const handleCancelUpdate = () => {
  formData.value.data.title = ''
  formData.value.data.body = ''
  formData.value.data.author = ''
  formData.value.data.is_saved = false
  formData.value.data.categories = {
    data: []
  }

  formMode.value = 'create'
  currentArticleId.value = null
}

const handleSubmit = (event) => {
  event.preventDefault()
  if (!formData.value.data.title || !formData.value.data.body || !formData.value.data.author) {
    alert('Please fill in all required fields')
    return
  }

  if (formMode.value === 'edit') {
    axios
      .put(url + '/blog-posts/' + currentArticleId.value, formData.value, {
        headers: {
          Authorization: 'Bearer ' + token,
          'Content-Type': 'application/json'
        }
      })
      .then(function () {
        formData.value.data.title = ''
        formData.value.data.body = ''
        formData.value.data.author = ''
        formData.value.data.is_saved = false
        formData.value.data.categories = {
          data: []
        }

        formMode.value = 'create'
        currentArticleId.value = null

        $toast.success('Article updated!', {
          position: 'top-right'
        })
        getArticles()
      })
      .catch(function (error) {
        console.log(error)
      })
  } else {
    axios
      .post(url + '/blog-posts', formData.value, {
        headers: {
          Authorization: 'Bearer ' + token,
          'Content-Type': 'application/json'
        }
      })
      .then(function () {
        formData.value.data.title = ''
        formData.value.data.body = ''
        formData.value.data.author = ''
        formData.value.data.is_saved = false
        formData.value.data.categories = {
          data: []
        }

        $toast.success('Article created!', {
          position: 'top-right'
        })
        getArticles()
      })
      .catch(function (error) {
        console.log(error)
      })
  }
}

// mounted
onMounted(async () => {
  getArticles()
  getCategories()
})
</script>

<template>
  <div>
    <div class="flex flex-col xl:flex-row justify-center gap-1">
      <div class="w-full px-4 mb-12 xl:mb-0">
        <h1 class="text-2xl font-bold text-center mb-8">List Article</h1>
        <div class="overflow-x-auto">
          <table class="min-w-full divide-y divide-gray-200">
            <thead>
              <tr>
                <th
                  class="px-6 py-3 bg-gray-50 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
                >
                  Title
                </th>
                <th
                  class="px-6 py-3 bg-gray-50 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
                >
                  Body
                </th>
                <th
                  class="px-6 py-3 bg-gray-50 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
                >
                  Author
                </th>
                <th
                  class="px-6 py-3 bg-gray-50 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
                >
                  Categories
                </th>
                <th
                  class="px-6 py-3 bg-gray-50 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
                >
                  Is Saved?
                </th>
                <th
                  class="px-6 py-3 bg-gray-50 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
                >
                  Actions
                </th>
              </tr>
            </thead>
            <tbody class="divide-y divide-gray-200">
              <tr v-for="article in dataArticles" :key="article.id">
                <td class="px-6 py-4 whitespace-nowrap">{{ article.attributes.title }}</td>
                <td class="px-6 py-4 whitespace-nowrap">{{ article.attributes.body }}</td>
                <td class="px-6 py-4 whitespace-nowrap">{{ article.attributes.author }}</td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <span
                    v-if="
                      article.attributes.categories && article.attributes.categories.data.length > 0
                    "
                  >
                    <span
                      v-for="category in article.attributes.categories.data"
                      :key="category.id"
                      class="p-1 bg-slate-600 text-white inline-block mr-1 rounded"
                    >
                      {{ category.attributes.title }}
                    </span>
                  </span>
                  <span class="p-1 bg-black text-white inline-block rounded" v-else
                    >No categories</span
                  >
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <span
                    class="p-1 bg-blue-600 text-white inline-block rounded"
                    v-if="article.attributes.is_saved"
                    >{{ article.attributes.is_saved }}</span
                  >
                  <span class="p-1 bg-purple-600 text-white inline-block rounded" v-else>{{
                    article.attributes.is_saved
                  }}</span>
                </td>
                <td class="px-6 py-4 whitespace-nowrap">
                  <button
                    @click="handleEdit(article.id)"
                    class="bg-green-500 hover:bg-green-700 text-white font-bold py-1 px-3 rounded ml-2"
                  >
                    Edit
                  </button>
                  <button
                    @click="handleOpenModal(article.id)"
                    class="bg-red-500 hover:bg-red-700 text-white font-bold py-1 px-3 rounded ml-2"
                  >
                    Delete
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
      <div class="w-full">
        <h1 class="text-2xl font-bold text-center mb-8">
          {{ formMode === 'create' ? 'Create Article' : 'Update Article' }}
        </h1>
        <div class="w-full bg-white rounded px-8 pt-6 pb-8 mb-4">
          <form action="" @submit.prevent="handleSubmit" class="w-full">
            <div class="mb-3">
              <label for="title" class="block text-gray-700 text-sm font-bold mb-2">Title</label>
              <input
                type="text"
                id="title"
                placeholder="Title"
                v-model="formData.data.title"
                class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
              />
            </div>
            <div class="mb-3">
              <label for="body" class="block text-gray-700 text-sm font-bold mb-2">Body</label>
              <textarea
                id="body"
                rows="4"
                placeholder="Body"
                v-model="formData.data.body"
                class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
              ></textarea>
            </div>
            <div class="mb-3">
              <label for="author" class="block text-gray-700 text-sm font-bold mb-2">Author</label>
              <input
                type="text"
                id="author"
                placeholder="Author"
                v-model="formData.data.author"
                class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
              />
            </div>
            <div class="mb-3">
              <label for="author" class="block text-gray-700 text-sm font-bold mb-2"
                >Is Saved?</label
              >
              <select
                class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                name="is_saved"
                id="is_saved"
                v-model="formData.data.is_saved"
              >
                <option value="false">false</option>
                <option value="true">true</option>
              </select>
            </div>
            <div class="mb-3">
              <label for="categories" class="block text-gray-700 text-sm font-bold mb-2"
                >Categories</label
              >
              <select
                class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
                name="categories"
                id="categories"
                v-model="formData.data.categories"
              >
                <option v-for="category in dataCategories" :key="category.id" :value="category.id">
                  {{ category.attributes.title }}
                </option>
              </select>
            </div>
            <div class="flex items-center gap-4">
              <button
                class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-1 px-3 rounded focus:outline-none focus:shadow-outline"
                type="submit"
                :class="{
                  'opacity-50 cursor-not-allowed':
                    !formData.data.title || !formData.data.body || !formData.data.author
                }"
                :disabled="!formData.data.title || !formData.data.body || !formData.data.author"
              >
                {{ formMode === 'create' ? 'Create' : 'Update' }}
              </button>
              <button
                :class="{ hidden: formMode === 'create' }"
                @click="handleCancelUpdate"
                type="button"
                class="bg-red-500 hover:bg-red-700 text-white font-bold py-1 px-3 rounded focus:outline-none focus:shadow-outline"
              >
                Cancel
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>
    <!-- modal delete -->
    <div>
      <div
        v-if="showModal"
        class="fixed z-10 inset-0 overflow-y-auto"
        aria-labelledby="modal-title"
        role="dialog"
        aria-modal="true"
      >
        <div
          class="flex items-end justify-center min-h-screen pt-4 px-4 pb-20 text-center sm:block sm:p-0"
        >
          <div
            class="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity"
            aria-hidden="true"
          ></div>
          <span class="hidden sm:inline-block sm:align-middle sm:h-screen" aria-hidden="true"
            >&#8203;</span
          >
          <div
            class="inline-block align-bottom bg-white rounded-lg text-left overflow-hidden shadow-xl transform transition-all sm:my-8 sm:align-middle sm:max-w-lg sm:w-full"
          >
            <div class="bg-white px-4 pt-5 pb-4 sm:p-6 sm:pb-4">
              <div class="sm:flex sm:items-start">
                <div
                  class="mx-auto flex-shrink-0 flex items-center justify-center h-12 w-12 rounded-full bg-red-100 sm:mx-0 sm:h-10 sm:w-10"
                >
                  <svg
                    class="h-6 w-6 text-red-600"
                    xmlns="http://www.w3.org/2000/svg"
                    fill="none"
                    viewBox="0 0 24 24"
                    stroke="currentColor"
                    aria-hidden="true"
                  >
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z"
                    />
                  </svg>
                </div>
                <div class="mt-3 text-center sm:mt-0 sm:ml-4 sm:text-left">
                  <h3 class="text-lg leading-6 font-medium text-gray-900" id="modal-title">
                    Delete Article
                  </h3>
                  <div class="mt-2">
                    <p class="text-sm text-gray-500">
                      Are you sure you want to delete this article?
                    </p>
                  </div>
                </div>
              </div>
            </div>
            <div class="bg-gray-50 px-4 gap-4 py-3 sm:px-6 sm:flex sm:flex-row-reverse">
              <button
                class="bg-red-500 hover:bg-red-700 text-white font-bold py-1 px-3 rounded focus:outline-none focus:shadow-outline"
                @click="handleDelete"
                type="button"
              >
                Yes
              </button>
              <button
                class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-1 px-3 rounded focus:outline-none focus:shadow-outline"
                @click="showModal = false"
                type="button"
              >
                Cancel
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
