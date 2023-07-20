<script setup>
import RecordList from './components/RecordList.vue'
import { getParmams } from './utils/helpers.js';

const appServer = 'https://cicada.vercel.app';
// const appServer = 'http://localhost:5173';

import { ref } from 'vue'
const service = ref({
  url: localStorage.getItem('service'),
  token: localStorage.getItem('token'),
});

const defaultURL = ref('https://h36x4inrc7.us.aircode.run/auth?token=aircode');

const checkAuth = async () => {
  const url = service.value.url || defaultURL.value;
  const res = await fetch(url, {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
    },
  });
  const data = await res.json();
  if(!data.error) {
    service.value.token = data.token;
    service.value.url = url.replace(/\/[^/]*$/g, '')
    localStorage.setItem('token', data.token);
    localStorage.setItem('service', service.value.url);
  }
}

const showFooter = ref(false);

const searchString = ref(getParmams().get('keywords') || '');

const addToFavorite = `javascript:void function(e, t, n, r, c, i, s, o, u) {\n    n = location.href,\n    r = t.title,\n    c = t.documentElement.outerHTML,\n    i = "" + (e.getSelection ? e.getSelection() : t.getSelection ? t.getSelection() : t.selection.createRange().text);\n    if (!i) {\n        o = t.getElementsByTagName("meta");\n        for (var a = 0; a < o.length; a++) u = o[a],\n        u && u.name.toLowerCase() === "description" && (i = u.content)\n    }\n    s = encodeURIComponent;\n    var f = "${appServer}/add.html?title=" + s(r) + "&url=" + s(n) + "&summary=" + s(i);\n    e.open(f, "_blank", "scrollbars=no,width=800,height=500,left=75,top=20,status=no,resizable=yes")\n} (window, document);`;

</script>

<template>
  <header>
    <h1><a href="/">My Favorite</a></h1>
    <div v-if="service.token" class="wrapper">
      <form class="form-inline" autocomplete="off">
        <div class="form-group">
          <input type="text" name="keywords" placeholder="Search" class="form-control" :value="searchString"/>
        </div>
        <button type="submit" class="btn btn-primary">Search</button>
      </form>
    </div>
    <div v-else class="wrapper login">
      <form class="form-inline" autocomplete="off">
      <div class="form-group">
        <input type="text" :placeholder="defaultURL" class="form-control" 
        v-model="service.url"/>
      </div>
      <button type="button" class="btn btn-primary" v-on:click="checkAuth">Login</button>
      <a href="###">help?</a>
    </form>
    </div>
  </header>

  <main v-if="service.token">
    <RecordList :token="service.token" :service="service.url" v-model:footer="showFooter" v-model:search="searchString"/>
  </main>

  <footer v-show="showFooter">
    Powered by Cicada & <a href="https://aircode.io">AirCode</a>. <a :href="addToFavorite">Add to Favorite <svg xmlns="http://www.w3.org/2000/svg" width="12" height="12" fill="currentColor" class="bi bi-arrow-up-right" viewBox="0 0 16 16">
  <path fill-rule="evenodd" d="M14 2.5a.5.5 0 0 0-.5-.5h-6a.5.5 0 0 0 0 1h4.793L2.146 13.146a.5.5 0 0 0 .708.708L13 3.707V8.5a.5.5 0 0 0 1 0v-6z"/>
</svg></a>
  </footer>
</template>

<style scoped>
header {
  line-height: 1.5;
  padding-bottom: 40px;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

.form-group input {
  width: 60vw;
  max-width: 480px;
}

.login a {
  padding-left: 4px;
  /* vertical-align: bottom; */
}
</style>
