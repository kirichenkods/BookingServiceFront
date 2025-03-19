<template>
  <div id="app">
    <div class="auth-block">
      <!-- Вход и Регистрация -->
      <template v-if="!isAuthenticated">
        <button @click="showLoginModal = true">Вход</button>
        <button @click="showRegistrationModal = true">Регистрация</button>
      </template>

      <!-- Выход -->
      <template v-else>
        <button @click="logout">Выход</button>
      </template>
    </div>

    <!-- Шапка -->
    <header class="header">
      <h1>Резервирование ресурсов</h1>
    </header>

    <div class="content">
      <div class="tabs">
        <button v-if="!isAdmin" :class="{ active: currentTab === 'byTitle' }"
          @click="currentTab = 'byTitle'; resetResults()">
          Поиск по названию
        </button>
        <button v-if="!isAdmin" :class="{ active: currentTab === 'byDate' }"
          @click="currentTab = 'byDate'; resetResults()">
          Поиск по дате
        </button>
        <button v-if="!isAdmin && isAuthenticated" :class="{ active: currentTab === 'myResources' }"
          @click="currentTab = 'myResources'; resetResults(); loadMyResources()">
          Мои ресурсы
        </button>
        <button v-if="isAdmin" :class="{ active: currentTab === 'manageUsers' }"
          @click="currentTab = 'manageUsers'; loadUsers()">
          Управление пользователями
        </button>
        <button v-if="isAdmin" :class="{ active: currentTab === 'manageResources' }"
          @click="currentTab = 'manageResources'; loadResources()">
          Управление ресурсами
        </button>
      </div>

      <div class="tab-content">
        <div v-if="!isAdmin & currentTab === 'byTitle'" class="search-container">
          <form @submit.prevent="searchByTitle">
            <label for="title">Название:</label>
            <input type="text" id="title" v-model="titleSearch" placeholder="Введите название">
            <button type="submit">Найти</button>
          </form>
        </div>

        <div v-if="currentTab === 'byDate'" class="search-container">
          <form @submit.prevent="searchByPeriod">
            <label for="from">С:</label>
            <input type="datetime-local" id="from" v-model="periodFrom" required>
            <label for="to">По:</label>
            <input type="datetime-local" id="to" v-model="periodTo" required>
            <button type="submit">Найти</button>
          </form>
        </div>

        <div v-if="currentTab === 'myResources'">
          <p>Мои забронированные ресурсы:</p>
          <ul v-if="myBookings.length > 0">
            <li v-for="resource in myBookings" :key="resource.id">
              <div class="result-card">
                <h3>{{ resource.resourceTitle }}</h3>
                <p>Начало бронирования: {{ formatDate(resource.startDate) }}</p>
                <p>Окончание бронирования: {{ formatDate(resource.endDate) }}</p>
                <button @click="releaseResource(resource.id)">Освободить</button>
              </div>
            </li>
          </ul>
          <p v-else>Нет забронированных ресурсов.</p>
        </div>

        <!-- Список пользователей -->
        <div v-if="currentTab === 'manageUsers'" class="list-view">
          <p>Список пользователей:</p>
          <ul v-if="users.length > 0">
            <li v-for="user in users" :key="user.id">
              <div class="item">
                <span>{{ user.name }} {{ user.surname }}</span>
                <span>{{ user.email }}</span>
                <span>{{ user.phone }}</span>
                <button @click="editUser(user.id)">Редактировать</button>
                <button @click="deleteUser(user.id)">Удалить</button>
              </div>
            </li>
          </ul>
          <p v-else>Нет зарегистрированных пользователей.</p>
        </div>

        <div v-if="currentTab === 'manageResources'" class="list-view">
          <button @click="showCreateResourceModal = true">
            Создать ресурс
          </button>
          <p>Список ресурсов:</p>
          <ul v-if="resources.length > 0">
            <li v-for="resource in resources" :key="resource.id">
              <div class="item">
                <span>{{ resource.name }}</span>
                <span>{{ resource.description }}</span>
                <button @click="editCurrentResource(resource.id)">Редактировать</button>
                <button @click="deleteResource(resource.id)">Удалить</button>
              </div>
            </li>
          </ul>
          <p v-else>Нет доступных ресурсов.</p>
        </div>

      </div>
      <div v-if="results.length > 0 && currentTab !== ''">
        <p>Результаты поиска:</p>
        <ul>
          <li v-for="result in results" :key="result.id">
            <div class="result-card">
              <h3>{{ result.title }}</h3>
              <p>{{ result.description }}</p>
              <template v-if="currentTab === 'byDate'">
                <button @click="bookResourceDirectly(result, periodFrom, periodTo)">Забронировать</button>
              </template>
              <template v-else>
                <button @click="openBookingModal(result, periodFrom)">Забронировать</button>
              </template>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>

  <!-- Модальное окно для регистрации -->
  <transition name="fade">
    <div v-if="showRegistrationModal" class="modal-mask">
      <div class="modal-wrapper">
        <div class="modal-container">
          <button class="close-modal" @click="showRegistrationModal = false">×</button>
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

  <!-- Модальное окно для редактирования пользователя -->
  <transition name="fade">
    <div v-if="showEditModal" class="modal-mask">
      <div class="modal-wrapper">
        <div class="modal-container">
          <button class="close-modal" @click="showEditModal = false">×</button>
          <h2>Редактирование пользователя</h2>
          <form id="editUserForm" @submit.prevent="updateUser">
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

  <!-- Модальное окно для входа -->
  <transition name="fade">
    <div v-if="showLoginModal" class="modal-mask">
      <div class="modal-wrapper">
        <div class="modal-container">
          <button class="close-modal" @click="showLoginModal = false">×</button>
          <h2>Вход</h2>
          <form id="loginForm" @submit.prevent="authenticateUser">
            <label for="loginInput">Логин:</label>
            <input type="text" id="loginInput" v-model="credentials.login" required>
            <label for="passwordInput">Пароль:</label>
            <input type="password" id="passwordInput" v-model="credentials.password" required>
            <button type="submit">Войти</button>
          </form>
        </div>
      </div>
    </div>
  </transition>

  <!-- Модальное окно для бронирования -->
  <transition name="fade">
    <div v-if="showBookingModal" class="modal-mask">
      <div class="modal-wrapper">
        <div class="modal-container">
          <button class="close-modal" @click="showBookingModal = false">×</button>
          <h2>Бронирование ресурса</h2>
          <form id="bookingForm" @submit.prevent="sendBookingRequest">
            <label for="durationType">Тип длительности:</label>
            <select id="durationType" v-model="selectedDurationType">
              <option value="MINUTES">Минуты</option>
              <option value="HOURS">Часы</option>
              <option value="DAYS">Дни</option>
            </select>
            <label for="duration">Продолжительность:</label>
            <input type="number" id="duration" v-model="duration" required min="1">
            <label for="dateStart">Дата начала:</label>
            <input type="datetime-local" id="dateStart" v-model="dateStart" required>
            <button type="submit">Забронировать</button>
          </form>
        </div>
      </div>
    </div>
  </transition>

  <!-- Модальное окно для редактирования ресурса -->
  <transition name="fade">
    <div v-if="showEditResourceModal" class="modal-mask">
      <div class="modal-wrapper">
        <div class="modal-container">
          <button class="close-modal" @click="showEditResourceModal = false">×</button>
          <h2>{{ isEditing ? 'Редактирование ресурса' : 'Создание ресурса' }}</h2>
          <form id="editResourceForm" @submit.prevent="isEditing ? updateResource() : createResource()">
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
  data() {
    return {
      currentTab: 'byTitle',
      showLoginModal: false,
      showRegistrationModal: false,
      showEditModal: false,
      newUser: {
        name: '',
        surname: '',
        login: '',
        password: '',
        email: '',
        phone: ''
      },
      credentials: {
        login: '',
        password: ''
      },

      selectedResource: {},
      showBookingModal: false,
      selectedDurationType: 'HOURS',
      duration: 1,
      dateStart: '',

      titleSearch: '',
      periodFrom: '',
      periodTo: '',
      myBookings: [], // Список забронированных ресурсов
      users: [],
      resources: [], // Список ресурсов
      editResource: {}, // Редактируемый ресурс
      showEditResourceModal: false, // Флаг для отображения модального окна редактирования ресурса
      isEditing: false, // Флаг для определения режима редактирования или создания
      showCreateResourceModal: false, // Флаг для отображения модального окна создания ресурса
      results: []
    }
  },
  computed: {
    isAuthenticated() {
      return !!localStorage.getItem('jwtToken');
    },
    isAdmin() {
      return localStorage.getItem('userRole') === 'ROLE_ADMIN';
    }
  },
  methods: {
    openBookingModal(resource) {
      this.selectedResource = resource;
      this.showBookingModal = true;
    },

    
  reloadPage() {
    window.location.reload();
  },

    sendBookingRequest() {
      const bookingData = {
        userId: localStorage.getItem('userId'), 
        resourceId: this.selectedResource.id,
        durationType: this.selectedDurationType,
        duration: parseInt(this.duration),
        dateStart: this.dateStart,
      };

      fetch('http://localhost:8085/bookings', {
        method: 'POST',
        headers: this.addAuthHeader({
          'Content-Type': 'application/json'
        }),
        body: JSON.stringify(bookingData)
      })
        .then(response => {
          if (!response.ok) {
            return response.text(); 
          }
          return response.json(); 
        })
        .then(data => {
          if (typeof data === 'number') { 
            alert(`Ресурс успешно забронирован! Ваш ID бронирования: ${data}`);
            this.showBookingModal = false;
          } else if (typeof data === 'string') { 
            alert(data); 
          } else {
            alert('Не удалось забронировать ресурс.');
          }
        })
        .catch(error => console.error('Error:', error));
    },

    // Метод для открытия модального окна редактирования пользователя
    editUser(id) {
      this.editUser = {};
      const url = `http://localhost:8085/users/${id}`;
      fetch(url, {
        method: 'GET',
        headers: this.addAuthHeader({
          'Content-Type': 'application/json'
        }),
      })
        .then(response => response.json())
        .then(data => {
          this.editUser = data;
          this.showEditModal = true;
        })
        .catch(error => console.error('Error:', error));
      this.showEditModal = false;
    },

    // Метод для отправки POST-запроса на сервер
    updateUser() {
      const url = `http://localhost:8085/users/update`;
      fetch(url, {
        method: 'POST',
        headers: this.addAuthHeader({
          'Content-Type': 'application/json'
        }),
        body: JSON.stringify(this.editUser)
      })
        .then(response => response.json())
        .then(data => {
          if (data) {
            alert('Пользователь успешно обновлен!');
            this.showEditModal = false;
            this.editUser = {};
            this.loadUsers(); 
          } else {
            alert('Ошибка обновления пользователя');
          }
        })
        .catch(error => console.error('Error:', error));
    },

    // Метод для отправки запроса по периоду 
    searchByPeriod() {
      const url = `http://localhost:8085/resources/find-between/${this.periodFrom}/${this.periodTo}`;

      fetch(url, {
        method: 'GET',
        headers: this.addAuthHeader({
          'Content-Type': 'application/json'
        }),
      })
        .then(response => response.json())
        .then(data => {
          if (!data || !data.length) {
            this.results = [];
            alert('Ничего не найдено');
          } else {
            this.results = data.map(item => ({
              id: item.id,
              title: item.name,
              description: item.description
            }));
          }
        })
        .catch(error => console.error('Error:', error));
    },

    // Метод для отправки запроса по названию
    searchByTitle() {
      const url = `http://localhost:8085/resources/${this.titleSearch}`;

      fetch(url, {
        method: 'GET',
        headers: this.addAuthHeader({
          'Content-Type': 'application/json'
        }),
      })
        .then((response) => response.json())
        .then((data) => {
          if (!data || !Object.keys(data).length) {
            this.results = [];
            alert('Ничего не найдено');
          } else if (Array.isArray(data)) {
            this.results = data.map(item => ({
              id: item.id,
              title: item.name,
              description: item.description
            }));
          } else {
            this.results = [{
              id: data.id,
              title: data.name,
              description: description
            }];
          }
        })
        .catch((error) => console.error('Error:', error));
    },

    resetResults() {
      this.results = []; 
    },

    // Метод для регистрации пользователя
    registerUser() {
      const data = {
        name: this.newUser.name,
        surname: this.newUser.surname,
        login: this.newUser.login,
        password: this.newUser.password,
        email: this.newUser.email,
        phone: this.newUser.phone
      };

      fetch('http://localhost:8085/auth/sign-up', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(data)
      })
        .then(response => response.json())
        .then(data => {
          if (data.token) {
            // Сохраняем полученный JWT-токен
            localStorage.setItem('jwtToken', data.token);
            alert('Вы успешно зарегистрированы!');
            this.showRegistrationModal = false;
            this.reloadPage();
          } else {
            alert('Ошибка регистрации');
          }
        })
        .catch(error => console.error('Error:', error));
    },

    // Метод для аутентификации пользователя
    authenticateUser() {
      const credentials = {
        login: this.credentials.login,
        password: this.credentials.password,
        role: this.credentials.role
      };

      fetch('http://localhost:8085/auth/sign-in', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(credentials)
      })
        .then(response => response.json())
        .then(data => {
          if (data.token) {
            // Сохраняем полученный JWT-токен
            localStorage.setItem('jwtToken', data.token);
            // Сохраняем userId в localStorage
            localStorage.setItem('userId', data.userId);
            localStorage.setItem('userRole', data.userRole)
            alert('Вы успешно вошли!');
            this.showLoginModal = false;
            this.reloadPage();
          } else {
            alert('Неверный логин или пароль');
          }
        })
        .catch(error => console.error('Error:', error));
    },

    // Метод для добавления токена в заголовки запросов
    addAuthHeader(headers) {
      const token = localStorage.getItem('jwtToken');
      if (token) {
        headers['Authorization'] = `Bearer ${token}`;
      }
      return headers;
    },

    // Метод для выхода из аккаунта
    logout() {
      localStorage.removeItem('jwtToken'); 
      this.$forceUpdate(); 
      location.reload(); 
    },

    // Загрузка забронированных ресурсов
    loadMyResources() {
      const userId = localStorage.getItem('userId');
      const url = `http://localhost:8085/bookings/find/${userId}`;

      fetch(url, {
        method: 'GET',
        headers: this.addAuthHeader({
          'Content-Type': 'application/json'
        }),
      })
        .then((response) => response.json())
        .then((data) => {
          if (!data || !Array.isArray(data) || !data.length) {
            this.myBookings = [];
            alert('Нет забронированных ресурсов');
          } else {
            this.myBookings = data.map(item => ({
              id: item.id,
              resourceTitle: item.resourceName,
              startDate: item.dateStart,
              endDate: item.dateEnd
            }));
          }
        })
        .catch((error) => console.error('Error:', error));
    },

    // Освобождение ресурса
    releaseResource(id) {
      const url = `http://localhost:8085/bookings/delete/${id}`;

      fetch(url, {
        method: 'GET',
        headers: this.addAuthHeader({
          'Content-Type': 'application/json'
        }),
      })
        .then(() => {
          alert('Ресурс освобожден!');
          this.loadMyResources(); 
        })
        .catch((error) => console.error('Error:', error));
    },

    // Форматирование даты
    formatDate(dateString) {
      const date = new Date(dateString);
      return `${date.toLocaleDateString()} ${date.toLocaleTimeString()}`;
    },

    openBookingModal(result, periodFrom) {
      this.selectedResource = result;
      this.showBookingModal = true;
      this.dateStart = periodFrom; 
      const calcDuration = calculateDuration(periodFrom, this.periodTo);
      this.duration = calcDuration.value;
      this.durationType = calcDuration.type;
    },

    //функция для бронирования при поиске по дате
    bookResourceDirectly(result, periodFrom, periodTo) {
      const durationData = this.calculateDuration(periodFrom, periodTo);
      const bookingData = {
        userId: localStorage.getItem('userId'),
        resourceId: result.id,
        durationType: durationData.type,
        duration: durationData.value,
        dateStart: periodFrom,
      };

      fetch('http://localhost:8085/bookings', {
        method: 'POST',
        headers: this.addAuthHeader({
          'Content-Type': 'application/json'
        }),
        body: JSON.stringify(bookingData)
      })
        .then(response => response.json())
        .then(data => {
          if (data) {
            alert('Ресурс успешно забронирован!');
          } else {
            alert('Не удалось забронировать ресурс.');
          }
        })
        .catch(error => console.error('Error:', error));
    },

    // Метод для расчета продолжительности
    calculateDuration(startDateTime, endDateTime) {
      const MINUTES_IN_HOUR = 60;
      const HOURS_IN_DAY = 24;
      const MILLISECONDS_IN_MINUTE = 1000 * 60;

      // Парсим строки в объекты Date
      const start = new Date(startDateTime);
      const end = new Date(endDateTime);

      // Вычисляем разницу в миллисекундах
      const diffInMs = end.getTime() - start.getTime();

      // Переводим разницу в минуты
      const diffInMinutes = Math.floor(diffInMs / MILLISECONDS_IN_MINUTE);

      if (diffInMinutes % (HOURS_IN_DAY * MINUTES_IN_HOUR) === 0) {
        return {
          value: diffInMinutes / (HOURS_IN_DAY * MINUTES_IN_HOUR),
          type: 'DAYS'
        };
      } else if (diffInMinutes % MINUTES_IN_HOUR === 0) {
        return {
          value: diffInMinutes / MINUTES_IN_HOUR,
          type: 'HOURS'
        };
      } else {
        return {
          value: diffInMinutes,
          type: 'MINUTES'
        };
      }
    },

    loadUsers() {
      const url = 'http://localhost:8085/users';
      fetch(url, {
        method: 'GET',
        headers: this.addAuthHeader({
          'Content-Type': 'application/json'
        }),
      })
        .then(response => response.json())
        .then(data => {
          this.users = data.map(item => ({
            id: item.id,
            name: item.name,
            surname: item.surname,
            email: item.email,
            phone: item.phone
          }));
        })
        .catch(error => console.error('Error:', error));
    },

    deleteUser(id) {
      const url = `http://localhost:8085/users/delete/${id}`;
      fetch(url, {
        method: 'GET',
        headers: this.addAuthHeader({
          'Content-Type': 'application/json'
        }),
      })
        .then(() => {
          alert('Пользователь удалён!');
          loadUsers();
        })
        .catch(error => console.error('Error:', error));
    },

    // Метод для загрузки всех ресурсов
    loadResources() {
      const url = 'http://localhost:8085/resources';
      fetch(url, {
        method: 'GET',
        headers: this.addAuthHeader({
          'Content-Type': 'application/json'
        }),
      })
        .then(response => response.json())
        .then(data => {
          this.resources = data.map(item => ({
            id: item.id,
            name: item.name,
            description: item.description
          }));
        })
        .catch(error => console.error('Error:', error));
    },

    // Метод для удаления ресурса
    deleteResource(id) {
      const url = `http://localhost:8085/resources/delete/${id}`;
      fetch(url, {
        method: 'GET',
        headers: this.addAuthHeader({
          'Content-Type': 'application/json'
        }),
      })
        .then(() => {
          alert('Ресурс удалён!');
          this.loadResources(); 
        })
        .catch(error => console.error('Error:', error));
    },

    // Метод для редактирования ресурса
    editCurrentResource(id) {
      const url = `http://localhost:8085/resources/${id}`;
      fetch(url, {
        method: 'GET',
        headers: this.addAuthHeader({
          'Content-Type': 'application/json'
        }),
      })
        .then(response => response.json())
        .then(data => {
          this.editCurrentResource = data;
          this.showEditResourceModal = true;
        })
        .catch(error => console.error('Error:', error));
    },

    createResource() {
      const url = 'http://localhost:8085/resources/create';
      fetch(url, {
        method: 'POST',
        headers: this.addAuthHeader({
          'Content-Type': 'application/json'
        }),
        body: JSON.stringify(this.editCurrentResource)
      })
        .then(response => response.json())
        .then(data => {
          if (data) {
            alert('Ресурс успешно создан!');
            this.showCreateResourceModal = false;
            this.loadResources(); 
          } else {
            alert('Ошибка создания ресурса');
          }
        })
        .catch(error => console.error('Error:', error));
    },

    showCreateResourceModal() {
      this.isEditing = false;
      this.editCurrentResource = {};
      this.showEditResourceModal = true;
    },

    // Метод для сохранения изменений ресурса
    updateResource() {
      const url = `http://localhost:8085/resources/update`;
      fetch(url, {
        method: 'POST',
        headers: this.addAuthHeader({
          'Content-Type': 'application/json'
        }),
        body: JSON.stringify(this.editCurrentResource)
      })
        .then(response => response.json())
        .then(data => {
          if (data) {
            alert('Ресурс успешно обновлен!');
            this.showEditResourceModal = false;
            this.loadResources(); // Обновляем список ресурсов
          } else {
            alert('Ошибка обновления ресурса');
          }
        })
        .catch(error => console.error('Error:', error));
    },

  }
};
</script>