# ADIM 2: VUE TARAFI İŞLEMLERİ

## Vue.js Kurulumu

```bash
npm install vue@latest
```

## Vue Router Kurulumu

```bash
npm install vue-router@4
```

## Temel Vue Bileşenleri Oluşturma

### UserList.vue Bileşeni

```bash
mkdir -p resources/js/components
touch resources/js/components/UserList.vue
```

```vue
<!-- resources/js/components/UserList.vue -->
<template>
  <div>
    <h1>Kullanıcı Listesi</h1>
    <ul>
      <li v-for="user in users" :key="user.id">
        <router-link :to="{ name: 'userDetail', params: { id: user.id } }">
          {{ user.name }}
        </router-link>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'UserList',
  data() {
    return {
      users: [],
    };
  },
  mounted() {
    // API'den kullanıcıları al
    fetch('/api/users')
      .then((response) => response.json())
      .then((data) => (this.users = data));
  },
};
</script>
```

### UserDetail.vue Bileşeni

```bash
touch resources/js/components/UserDetail.vue
```

```vue
<!-- resources/js/components/UserDetail.vue -->
<template>
  <div>
    <h1>Kullanıcı Detayı</h1>
    <div v-if="user">
      <h2>{{ user.name }}</h2>
      <p>Email: {{ user.email }}</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'UserDetail',
  data() {
    return {
      user: null,
    };
  },
  mounted() {
    // Route parametresinden ID'yi al
    const userId = this.$route.params.id;
    // API'den kullanıcı detayını al
    fetch(`/api/users/${userId}`)
      .then((response) => response.json())
      .then((data) => (this.user = data));
  },
};
</script>
```

## Vue Router Yapılandırması

```javascript
// resources/js/router/index.js
import { createRouter, createWebHistory } from 'vue-router';
import UserList from '../components/UserList.vue';
import UserDetail from '../components/UserDetail.vue';

const routes = [
  {
    path: '/',
    name: 'userList',
    component: UserList,
  },
  {
    path: '/user/:id',
    name: 'userDetail',
    component: UserDetail,
  },
];

const router = createRouter({
  history: createWebHistory(),
  routes,
});

export default router;
```

## Ana Vue Uygulaması

```javascript
// resources/js/app.js
import { createApp } from 'vue';
import router from './router';
import App from './App.vue';

const app = createApp(App);
app.use(router);
app.mount('#app');
```

## App.vue Ana Bileşeni

```vue
<!-- resources/js/App.vue -->
<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<script>
export default {
  name: 'App',
};
</script>
```

## Webpack Mix Yapılandırması

```javascript
// webpack.mix.js
const mix = require('laravel-mix');

mix
  .js('resources/js/app.js', 'public/js')
  .vue()
  .postCss('resources/css/app.css', 'public/css', [
    //
  ]);
```

## Derleme ve Çalıştırma

```bash
# Geliştirme için derleme
npm run dev

# Üretim için derleme
npm run build
```

Bu adımları tamamladıktan sonra Vue.js tarafı hazır olacaktır. Bir sonraki adımda Inertia.js entegrasyonunu yapacağız.
