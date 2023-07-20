<script setup>
import { ref } from 'vue'
import { SAVE_API } from './utils/config.js';

const params = (new URL(document.location)).searchParams;
const record = ref({
  title: params.get('title'),
  url: params.get('url'),
  summary: params.get('summary'),
  tag: params.get('tag'),
});
const saveForm = ref(null);
const btnState = ref(null);
const infoMsg = ref('');

const service = localStorage.getItem('service');
const token = localStorage.getItem('token');

async function saveRecord(event) {
  const isValid = saveForm.value.checkValidity();
  if(!isValid) {
    saveForm.value.reportValidity();
    return;
  }

  event.preventDefault();
  btnState.value = 'disabled';
  infoMsg.value = 'Saving...';

  const url = `${service}${SAVE_API}?page=1`;

  const res = await fetch(url, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-cicada-token': token,
    },
    body: JSON.stringify(record.value),
  });

  await res.json();
  infoMsg.value = 'Saved!';

  setTimeout(() => {
    if(window.opener) {
      window.close();
      window.opener.location.reload();
    } else {
      const redirectURL = params.get('redirect');
      window.location.href = redirectURL;
    }
  }, 500);
}
</script>

<template>
  <header>
    <h1><a href="/">Add to Favorite</a></h1>
  </header>

  <main>
    <form v-if="service&&token" class="form-horizontal" autocomplete="off" ref="saveForm">
      <div class="form-group">
        <label>Title: </label>
        <input type="text" name="title" class="form-control" required v-model="record.title"/>
      </div>
      <div class="form-group">
        <label>URL: </label>
        <input type="text" name="url" class="form-control" required v-model="record.url"/>
      </div>
      <div class="form-group">
        <label>Summary: </label>
        <textarea name="summary" class="form-control" v-model="record.summary"/>
      </div>
      <div class="form-group">
        <label>Tag: </label>
        <input type="text" name="tag" class="form-control" v-model="record.tag"/>
      </div>
      <div>
        <button type="submit" class="btn btn-primary" v-on:click="saveRecord($event)"
        :disabled="btnState">Submit</button>
        <label class="info">{{ infoMsg }}</label>
      </div>
    </form>
    <div v-else><h2>Invalid service.</h2></div>
  </main>
</template>

<style scoped>
.form-horizontal {
  text-align: center;
  margin-top: -36px;
}
.form-group {
  margin: 6px 0;
}
.form-group label {
  text-align: right;
  margin-bottom: 0;
  padding-top: 7px;
  min-width: 100px;
  display: inline-block;
}

.form-group input,
.form-group textarea {
  margin-left: 10px;
  width: 60vw;
}
.form-horizontal button {
  margin-top: 10px;
}

label.info {
  margin-left: 10px;
  color: #09a;
  display: inline-block;
  vertical-align: middle;
  font-size: 12px;
  font-style: italic;
  position: absolute;
  margin-top: 18px;
}
</style>
