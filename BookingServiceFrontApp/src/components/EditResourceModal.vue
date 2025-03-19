<template>
    <transition name="fade">
      <div v-if="visible" class="modal-mask">
        <div class="modal-wrapper">
          <div class="modal-container">
            <button class="close-modal" @click="$emit('close')">×</button>
            <h2>{{ isEditing ? 'Редактирование ресурса' : 'Создание ресурса' }}</h2>
            <form id="editResourceForm" @submit.prevent="onSubmit">
              <label for="editName">Название ресурса:</label>
              <input type="text" id="editName" v-model="editCurrentResource.name" required>
              <label for="editDescription">Описание ресурса:</label>
              <textarea id="editDescription" v-model="editCurrentResource.description" required></textarea>
              <button type="submit">{{ isEditing ? 'Сохранить' : 'Создать' }}</button>
            </form>
          </div>
        </div>
      </div>
    </transition>
  </template>
  
  <script>
  export default {
    props: {
      visible: Boolean,
      isEditing: Boolean,
      editCurrentResource: Object,
    },
    methods: {
      onSubmit() {
        if (this.isEditing) {
          this.$emit('update', this.editCurrentResource);
        } else {
          this.$emit('create', this.editCurrentResource);
        }
      },
    },
  };
  </script>
  