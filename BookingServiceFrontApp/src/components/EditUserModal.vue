<template>
    <transition name="fade">
      <div v-if="visible" class="modal-mask">
        <div class="modal-wrapper">
          <div class="modal-container">
            <button class="close-modal" @click="$emit('close')">×</button>
            <h2>Редактирование пользователя</h2>
            <form id="editUserForm" @submit.prevent="onSubmit">
              <label for="editName">Имя пользователя:</label>
              <input type="text" id="editName" v-model="editUser.name" required>
              <label for="editSurname">Фамилия пользователя:</label>
              <input type="text" id="editSurname" v-model="editUser.surname" required>
              <label for="editEmail">Email:</label>
              <input type="email" id="editEmail" v-model="editUser.email" required>
              <label for="editPhone">Телефон:</label>
              <input type="tel" id="editPhone" v-model="editUser.phone" required>
              <button type="submit">Сохранить</button>
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
      editUser: Object,
    },
    methods: {
      onSubmit() {
        this.$emit('update', this.editUser);
      },
    },
  };
  </script>
  
