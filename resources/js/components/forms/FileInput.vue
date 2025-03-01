<template>
  <div class="relative" :class="marginBottom">
    <label v-if="label"
           :class="[theme.default.label,{'uppercase text-xs':uppercaseLabels, 'text-sm':!uppercaseLabels}]"
    >
      {{ label }}
      <span v-if="required" class="text-red-500 required-dot">*</span>
    </label>
    <span class="inline-block w-full rounded-md shadow-sm">
      <button type="button" aria-haspopup="listbox" aria-expanded="true" aria-labelledby="listbox-label"
              class="cursor-pointer relative w-full" :class="theme.default.input"
              :style="inputStyle" @click.prevent="showUploadModal=true"
      >
        <div v-if="currentUrl==null" class="h-6 text-gray-600 dark:text-gray-400">
          Upload file <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 inline" fill="none" viewBox="0 0 24 24"
                           stroke="currentColor"
          >
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                  d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12"
            />
          </svg>
        </div>
        <div v-else class="h-6 text-gray-600 dark:text-gray-400 flex">
          <div class="flex-grow">
            <p><svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 inline mr-2 -mt-1" fill="none" viewBox="0 0 24 24"
                    stroke="currentColor"
            >
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M9 13h6m-3-3v6m5 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"
              />
            </svg>{{ file.name }}</p>
          </div>
          <a href="#" class="hover:text-nt-blue flex" @click.prevent="clearUrl">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24"
                 stroke="currentColor"
            >
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"
              />
            </svg></a>
        </div>
      </button>
    </span>
    <small v-if="help" :class="theme.default.help" v-text="help" />
    <has-error v-if="form" :form="form" :field="name" />

    <!--  Modal  -->
    <modal :portal-order="2" :show="showUploadModal" @close="showUploadModal=false">
      <h2 class="text-lg font-semibold">
        Upload a file
      </h2>

      <div class="max-w-3xl mx-auto lg:max-w-none">
        <div class="sm:mt-5 sm:grid sm:grid-cols-1 sm:gap-4 sm:items-start sm:pt-5">
          <div class="mt-2 sm:mt-0 sm:col-span-2 mb-5">
            <div
              v-cloak
              class="w-full flex justify-center items-center px-6 pt-5 pb-6 border-2 border-gray-300 border-dashed rounded-md h-128"
              @dragover.prevent="onUploadDragoverEvent($event)"
              @drop.prevent="onUploadDropEvent($event)"
            >
              <div v-if="loading" class="text-gray-600 dark:text-gray-400">
                <loader class="h-6 w-6 mx-auto m-10" />
                <p class="text-center mt-6">
                  Uploading your file...
                </p>
              </div>
              <template v-else>
                <div
                  class="absolute rounded-full bg-gray-100 h-20 w-20 z-10 transition-opacity duration-500 ease-in-out"
                  :class="{
                    'opacity-100': uploadDragoverTracking,
                    'opacity-0': !uploadDragoverTracking
                  }"
                />
                <div class="relative z-20 text-center">
                  <input ref="actual-input" class="hidden" type="file" :name="name"
                         @change="manualFileUpload"
                  >
                  <svg xmlns="http://www.w3.org/2000/svg" class="mx-auto h-24 w-24 text-gray-200" fill="none"
                       viewBox="0 0 24 24" stroke="currentColor"
                  >
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                          d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12"
                    />
                  </svg>
                  <p class="mt-5 text-sm text-gray-600">
                    <button
                      type="button"
                      class="font-semibold text-nt-blue hover:text-nt-blue-dark focus:outline-none focus:underline transition duration-150 ease-in-out"
                      @click="openFileUpload"
                    >
                      Upload your file
                    </button>
                    or drag and drop
                  </p>
                  <p class="mt-1 text-xs text-gray-500">
                    Up to {{ mbLimit }}mb
                  </p>
                </div>
              </template>
            </div>
          </div>
        </div>
      </div>
    </modal>
  </div>
</template>

<script>
import Modal from '../Modal'
import Form from 'vform'
import inputMixin from '~/mixins/forms/input'

export default {
  name: 'FileInput',
  components: { Modal },
  mixins: [inputMixin],
  props: {
    endpoint: { type: String, required: true },
    mbLimit: { type: Number, default: 5 },
    /**
     * Key used when sending file to endpoint
     */
    fileFormKey: { type: String, default: 'asset' },
    /**
     * Key of the response body containing what we want to retrieve(url or relative path for instance)
     */
    responseKey: { type: String, default: 'url' }
  },

  data: () => ({
    showUploadModal: false,

    file: [],
    uploadDragoverTracking: false,
    uploadDragoverEvent: false,
    loading: false
  }),

  computed: {
    currentUrl () {
      return this.compVal
    }
  },

  methods: {
    clearUrl () {
      this.compVal = null
    },
    onUploadDragoverEvent (e) {
      this.uploadDragoverEvent = true
      this.uploadDragoverTracking = true
    },
    onUploadDropEvent (e) {
      this.uploadDragoverEvent = false
      this.uploadDragoverTracking = false
      this.droppedFiles(e)
    },
    droppedFiles (e) {
      const droppedFiles = e.dataTransfer.files

      if (!droppedFiles) return

      this.file = droppedFiles[0]
      this.uploadFileToServer()
    },
    openFileUpload () {
      this.$refs['actual-input'].click()
    },
    manualFileUpload (e) {
      this.file = e.target.files[0]
      this.uploadFileToServer()
    },
    uploadFileToServer () {
      const assetForm = new Form({
        [this.fileFormKey]: this.file
      })
      this.loading = true
      assetForm.post(this.endpoint).then((response) => {
        this.compVal = response.data[this.responseKey]
        this.showUploadModal = false
        this.loading = false
      }).catch((error) => {
        this.compVal = null
        this.showUploadModal = false
        this.loading = false
      })
    }
  }
}
</script>
