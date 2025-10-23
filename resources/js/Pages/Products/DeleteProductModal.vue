<template>
  <div
    v-if="show"
    class="fixed inset-0 z-50 flex items-center justify-center bg-black/50"
  >
    <div
      class="bg-white rounded-2xl shadow-lg w-full max-w-md p-6 text-center animate-fade-in"
    >
      <h2 class="text-xl font-bold text-gray-800 mb-2">Delete Product</h2>
      <p class="text-gray-600 mb-6">
        Are you sure you want to delete
        <span class="font-semibold text-red-600">{{ product?.name }}</span>?
        This action cannot be undone.
      </p>

      <div class="flex justify-center gap-3">
        <button
          @click="$emit('close')"
          class="bg-gray-300 hover:bg-gray-400 text-gray-800 px-4 py-2 rounded-lg"
        >
          Cancel
        </button>
        <button
          @click="confirmDelete"
          class="bg-red-600 hover:bg-red-700 text-white px-4 py-2 rounded-lg"
        >
          Delete
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import { toast } from "vue3-toastify";
import "vue3-toastify/dist/index.css";

export default {
  props: {
    show: Boolean,
    product: Object,
  },
  methods: {
    async confirmDelete() {
      try {
        await axios.delete(`/api/products/${this.product.id}`);
        toast.success("🗑️ Product deleted successfully!");
        this.$emit("deleted");
        this.$emit("close");
      } catch (error) {
        console.error(error);
        toast.error("❌ Failed to delete product.");
      }
    },
  },
};
</script>

<style scoped>
.animate-fade-in {
  animation: fade-in 0.2s ease-in-out;
}
@keyframes fade-in {
  from {
    opacity: 0;
    transform: scale(0.95);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}
</style>
