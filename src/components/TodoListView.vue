<!--TodoListView.vue-->
<script setup>
import { ref } from "vue";
import { statuses } from "../const/statuses";

const today = new Date();
let items = ref(JSON.parse(localStorage.getItem("items"))) || [];
let inputContent = ref(); //タスクの内容
let inputLimit = ref(); //タスクの期限
let inputState = ref(); //タスクのステータス
let errorMessage = ref("");

function onEdit(id) {
  let isOnEditOther = false;
  items.value.map((item) => {
    if (item.onEdit) {
      isOnEditOther = true;
      return;
    }
  });
  if (isOnEditOther) {
    errorMessage.value = "他に編集中のタスクがあります";
    isErrorMessage.value = true;
    return;
  }
  inputContent.value = items.value[id].content;
  inputLimit.value = items.value[id].limit;
  inputState.value = items.value[id].state;
  items.value[id].onEdit = true;
}

let isErrorMessage = ref(false);

function onUpdate(id) {
  if (inputContent.value == "" || inputLimit.value == "") {
    errorMessage.value = "タスクの内容と期限を入力してください。";
    isErrorMessage.value = true;
    return;
  }

  const newItem = {
    id: id,
    content: inputContent.value,
    limit: inputLimit.value,
    state: inputState.value,
    onEdit: false,
  };

  items.value.splice(id, 1, newItem);

  localStorage.setItem("items", JSON.stringify(items.value));

  isErrorMessage.value = false;
}

let isShowModal = ref(false);
let deleteItemId = ""; //削除対象のItemのID
let deleteItemContent = ref(); //削除対象のItemの内容

function showDeleteModal(id) {
  isShowModal.value = true;
  deleteItemId = id;
  deleteItemContent.value = items.value[id].content;
}

function onDeleteItem() {
  items.value.splice(deleteItemId, 1);
  isShowModal.value = false;

  items.value = items.value.map((item, index) => ({
    id: index,
    content: item.content,
    limit: item.limit,
    state: item.state,
    onEdit: item.onEdit,
  }));

  localStorage.setItem("items", JSON.stringify(items.value));
}

function onHideModal() {
  isShowModal.value = false;
}
</script>

<template>
  <div class="w-full">
    <div v-if="isErrorMessage" class="">
      <div class="inline-flex text-red-500 my-2 p-2">
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
        <p>{{ errorMessage }}</p>
      </div>
    </div>
    <table class="table-auto w-full border-separate border-spacing-1">
      <thead>
        <tr>
          <th class="border-b-2 border-sky-500">ID</th>
          <th class="border-b-2 border-sky-500">やること</th>
          <th class="border-b-2 border-sky-500">期限</th>
          <th class="border-b-2 border-sky-500">状態</th>
          <th class="border-b-2 border-sky-500">編集</th>
          <th class="border-b-2 border-sky-500">削除</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="item in items"
          :key="item.id"
          :class="{ 'text-red-500': new Date(item.limit) < today }"
          class="text-center"
        >
          <td class="px-1 py-3 m-1">{{ item.id }}</td>
          <td class="rounded px-2 py-1">
            <span v-if="!item.onEdit">{{ item.content }}</span>
            <input
              v-else
              v-model="inputContent"
              type="text"
              class="w-full p-1 border-2 border-sky-200 rounded"
            />
          </td>
          <td class="px-2 py-1">
            <span v-if="!item.onEdit">{{ item.limit }}</span>
            <input
              v-else
              v-model="inputLimit"
              type="date"
              class="w-full p-1 border-2 border-sky-200 rounded"
            />
          </td>
          <td class="px-2 py-1">
            <span
              v-if="!item.onEdit"
              :class="{
                'bg-gray-300 ': item.state.id === 0, // 未実施
                'bg-yellow-300': item.state.id === 1, // 実行中
                'bg-green-300 px-4': item.state.id === 2, // 完了
              }"
              class="p-2 rounded"
            >
              {{ item.state.value }}
            </span>
            <select
              v-else
              v-model="inputState"
              class="w-full p-1 border-2 border-sky-200 rounded"
            >
              <option
                v-for="state in statuses"
                :key="state.id"
                :value="state"
                :selected="state.id == item.state.id"
              >
                {{ state.value }}
              </option>
            </select>
          </td>
          <td
            class="bg-sky-500 text-white font-semibold shadow-sm px-1 py-1 rounded-lg"
          >
            <button v-if="!item.onEdit" @click="onEdit(item.id)">編集</button>
            <button v-else @click="onUpdate(item.id)">完了</button>
          </td>
          <td
            class="bg-white text-gray-700 font-semibold shadow-sm px-1 py-1 ring-1 rounded-lg"
          >
            <button @click="showDeleteModal(item.id)">削除</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>

  <div
    v-if="isShowModal"
    class="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity flex items-center justify-center"
  >
    <div class="bg-white rounded-lg shadow-lg p-6 text-center max-w-md w-full">
      <div>
        <div class="inline-flex">
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
          <p class="ml-2 mb-4">
            {{ deleteItemContent }}を削除してもよろしいですか？
          </p>
        </div>
        <div class="inline-flex justify-center">
          <button
            @click="onDeleteItem()"
            class="rounded-md bg-sky-500 px-3 py-2 mr-2 text-sm font-semibold text-white shadow-sm hover:bg-sky-400"
          >
            はい
          </button>
          <button
            @click="onHideModal()"
            class="rounded-md bg-white px-3 py-2 text-sm font-semibold text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 hover:bg-gray-50"
          >
            キャンセル
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
