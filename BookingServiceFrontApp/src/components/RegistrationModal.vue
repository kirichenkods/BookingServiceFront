<template>
    <transition name="fade">
        <div v-if="visible" class="modal-mask">
            <div class="modal-wrapper">
                <div class="modal-container">
                    <button class="close-modal" @click="$emit('close')">×</button>
                    <h2>Регистрация</h2>
                    <form id="registrationForm" @submit.prevent="registerUser">
                        <label for="name">Имя пользователя:</label>
                        <input type="text" id="name" v-model="newUser.name" required>
                        <label for="surname">Фамилия пользователя:</label>
                        <input type="text" id="surname" v-model="newUser.surname" required>
                        <label for="login">Логин:</label>
                        <input type="text" id="login" v-model="newUser.login" required>
                        <label for="password">Пароль:</label>
                        <input type="password" id="password" v-model="newUser.password" required>
                        <label for="email">Email:</label>
                        <input type="email" id="email" v-model="newUser.email" required>
                        <label for="phone">Телефон:</label>
                        <input type="tel" id="phone" v-model="newUser.phone" required>
                        <button type="submit">Зарегистрироваться</button>
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
    },
    data() {
        return {
            newUser: {
                name: '',
                surname: '',
                login: '',
                password: '',
                email: '',
                phone: '',
            },
        };
    },
    methods: {
        registerUser() {
            const data = {
                name: this.newUser.name,
                surname: this.newUser.surname,
                login: this.newUser.login,
                password: this.newUser.password,
                email: this.newUser.email,
                phone: this.newUser.phone,
            };
            fetch('http://localhost:8085/auth/sign-up',
                {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify(data),
                }).then((response) => response.json()).then((data) => {
                    if (data.token) {
                        // Сохраняем полученный JWT-токен 
                        localStorage.setItem('jwtToken', data.token);
                        alert('Вы успешно зарегистрированы!');
                        this.$emit('close');
                    } else {
                        alert('Ошибка регистрации');
                    }
                }).catch((error) => console.error('Error:', error));
        },
    },
};
</script>
