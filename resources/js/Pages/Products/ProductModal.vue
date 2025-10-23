<template>
  <div
    v-if="show"
    class="fixed inset-0 z-50 flex items-center justify-center bg-black/50"
  >
    <div
      class="bg-white rounded-2xl shadow-lg w-full max-w-lg p-6 relative animate-fade-in"
    >
   
      <div class="flex justify-between items-center mb-4">
        <h2 class="text-xl font-bold">
          {{ mode === 'edit' ? 'Edit Product' : 'Create Product' }}
        </h2>
        <button
          @click="closeModal"
          class="text-gray-500 hover:text-gray-700 text-lg font-bold"
        >
          ✕
        </button>
      </div>

   
      <form @submit.prevent="submitForm">
  
        <div class="mb-4">
          <label class="block font-semibold mb-1">Name</label>
          <input
            v-model.trim="form.name"
            type="text"
            class="border rounded-lg p-2 w-full focus:ring focus:ring-blue-300 outline-none"
            placeholder="Enter product name"
            required
          />
        </div>


        <div class="mb-4">
          <label class="block font-semibold mb-1">Category</label>
          <select
            v-model="form.category"
            class="border rounded-lg p-2 w-full focus:ring focus:ring-blue-300 outline-none"
            required
          >
            <option disabled value="">Select a category</option>
            <option v-for="cat in categories" :key="cat" :value="cat">
              {{ cat }}
            </option>
          </select>
        </div>

        
        <div class="mb-4">
          <label class="block font-semibold mb-1">Description</label>
          <textarea
            v-model.trim="form.description"
            class="border rounded-lg p-2 w-full focus:ring focus:ring-blue-300 outline-none"
            rows="3"
            placeholder="Enter product description"
            required
          ></textarea>
        </div>

  
        <div class="mb-4">
          <label class="block font-semibold mb-1">Date and Time</label>
          <input
            v-model="form.date_time"
            type="datetime-local"
            class="border rounded-lg p-2 w-full focus:ring focus:ring-blue-300 outline-none"
            required
          />
        </div>

      
        <div class="mb-4">
          <label class="block font-semibold mb-1">Image</label>
          <input
            type="file"
            accept="image/*"
            @change="handleFileChange"
            ref="fileInput"
            class="border rounded-lg p-2 w-full"
          />

          <div v-if="preview" class="mt-3">
            <img
              :src="preview"
              alt="Preview"
              class="h-24 w-24 rounded-lg object-cover border"
            />
          </div>

          <div v-else-if="mode === 'edit' && form.image_url" class="mt-3">
            <img
              :src="form.image_url"
              alt="Current"
              class="h-24 w-24 rounded-lg object-cover border"
            />
          </div>
        </div>

    
        <div class="flex justify-end gap-2 mt-6">
          <button
            type="button"
            @click="closeModal"
            class="bg-gray-300 hover:bg-gray-400 text-gray-800 px-4 py-2 rounded-lg"
          >
            Cancel
          </button>

          <button
            type="submit"
            class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg"
          >
            {{ mode === 'edit' ? 'Update' : 'Create' }}
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  props: {
    show: Boolean,
    categories: Array,
    mode: { type: String, default: "create" },
    productData: { type: Object, default: () => ({}) },
  },
  data() {
    return {
      form: {
        id: null,
        name: "",
        category: "",
        description: "",
        date_time: "",
        image: null,
        image_url: null,
      },
      preview: null,
    };
  },
  watch: {
    productData: {
      immediate: true,
      handler(newVal) {
        if (this.mode === "edit" && newVal) {
          this.form = {
            id: newVal.id,
            name: newVal.name,
            category: newVal.category,
            description: newVal.description,
            date_time: this.formatDateTime(newVal.date_time),
            image: null,
            image_url: newVal.image_url || null,
          };
          this.preview = null;
        } else if (this.mode === "create") {
          this.resetForm();
        }
      },
    },
  },
  methods: {
    handleFileChange(e) {
      const file = e.target.files[0];
      this.form.image = file;
      if (file) {
        this.preview = URL.createObjectURL(file);
      }
    },
    formatDateTime(datetime) {
      if (!datetime) return "";
      const d = new Date(datetime);
      return new Date(d.getTime() - d.getTimezoneOffset() * 60000)
        .toISOString()
        .slice(0, 16);
    },
    async submitForm() {
      try {
        const formData = new FormData();
        formData.append("name", this.form.name);
        formData.append("category", this.form.category);
        formData.append("description", this.form.description);
        formData.append("date_time", this.form.date_time);
        if (this.form.image) {
          formData.append("images[0]", this.form.image);
        }

        const config = { headers: { "Content-Type": "multipart/form-data" } };

        if (this.mode === "create") {
          await axios.post("/api/products", formData, config);
          alert("✅ Product created successfully!");
        } else {
          await axios.post(
            `/api/products/${this.form.id}?_method=PUT`,
            formData,
            config
          );
          alert("✅ Product updated successfully!");
        }

        this.$emit("updated");
        this.closeModal();
      } catch (error) {
        if (error.response?.status === 422) {
          console.error("Validation errors:", error.response.data.errors);
          alert("⚠️ Validation failed. Please check your inputs.");
        } else {
          console.error(error);
          alert("❌ Something went wrong while saving the product.");
        }
      }
    },
    closeModal() {
      this.$emit("close");
      this.resetForm();
    },
    resetForm() {
      this.form = {
        id: null,
        name: "",
        category: "",
        description: "",
        date_time: "",
        image: null,
        image_url: null,
      };
      this.preview = null;
      if (this.$refs.fileInput) this.$refs.fileInput.value = "";
    },
  },
};
</script>

<style scoped>
.animate-fade-in {
  animation: fade-in 0.25s ease-in-out;
}

@keyframes fade-in {
  from {
    opacity: 0;
    transform: translateY(-10px) scale(0.98);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}
</style>
