<!--TodoInput.vue-->
<script setup>
import { ref } from "vue";
import { statuses } from "../const/statuses";

const input = ref("");
const inputDate = ref("");
const isErrorMessage = ref(false);

function onSubmitForm() {
  isErrorMessage.value = false;

  if (input.value == "" || inputDate.value == "") {
    isErrorMessage.value = true;
    event.preventDefault();
    return;
  }
  const items = JSON.parse(localStorage.getItem("items")) || [];

  const newItem = {
    id: items.length,
    content: input.value,
    limit: inputDate.value,
    state: statuses.NOT_START,
    onEdit: false,
  };

  items.push(newItem);

  localStorage.setItem("items", JSON.stringify(items));
}

function onHideCxlButton() {
  isErrorMessage.value = false;
  input.value = "";
  inputDate.value = "";
}
</script>

<template>
  <div class="mx-5 mb-5">
    <h1 class="text-xl font-extrabold text-gray-900 pb-10">TODOリスト</h1>

    <div v-if="isErrorMessage">
      <div class="inline-flex text-red-500 pb-2">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="24"
          height="24"
          viewBox="0 0 24 24"
          fill="none"
          stroke="#ef4444"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
        >
          <circle cx="12" cy="12" r="10"></circle>
          <line x1="12" y1="8" x2="12" y2="12"></line>
          <line x1="12" y1="16" x2="12.01" y2="16"></line>
        </svg>
        <p>タスクの内容と期限を入力してください。</p>
      </div>
    </div>

    <form class="w-full" @submit="onSubmitForm">
      <label class="w-full"
        >やること<input
          class="w-full border-2 border-sky-300 rounded p-1 mt-2 mb-3"
          type="text"
          v-model="input"
      /></label>
      <label class="w-full"
        >期限<input
          class="w-full border-2 border-sky-300 rounded p-1 mt-2 mb-3"
          type="date"
          v-model="inputDate"
      /></label>
      <button
        class="bg-sky-500 text-white font-semibold shadow-sm my-2 px-6 py-3 rounded-lg hover:bg-sky-600 mr-2"
      >
        登録
      </button>
      <button
        v-if="isErrorMessage"
        @click="onHideCxlButton()"
        class="bg-white text-gray-700 shadow-sm my-2 font-semibold ring-1 px-6 py-3 rounded-lg hover:bg-sky-600"
      >
        キャンセル
      </button>
    </form>
  </div>
</template>
