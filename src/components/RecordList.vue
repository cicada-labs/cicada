<script setup>
import { ref, onMounted, defineEmits } from 'vue'
import { LIST_API, DELETE_API, SEARCH_API } from '../utils/config.js';
import { getParmams } from '../utils/helpers.js';

const props = defineProps({
  token: {
    type: String,
    required: true
  },
  service: {
    type: String,
    required: true
  },
  footer: {
    type: Boolean,
    required: true
  },
  search: {
    type: String,
    required: false
  }
})

const records = ref({result: []});
const location = window.location.href;
const pagination = ref([]);

const emit = defineEmits(['update:footer', 'update:search']);

const getRecords = async ({page, keywords} = {}) => {
  let qureyStr = '?';
  let api = LIST_API;
  const params = getParmams();
  page = page || params.get('page');
  keywords = keywords || params.get('keywords');

  if(page) {
    qureyStr += `page=${page}&`;
  }

  if(keywords) {
    qureyStr += `keywords=${keywords}&`;
    api = SEARCH_API;
  }

  const res = await fetch(`${props.service}${api}${qureyStr}`, {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
      'x-cicada-token': props.token,
    },
  });
  const data = await res.json();
  records.value = data;

  const {page: currentPage, pageCount} = data;
  const list = [];
  let expendLeft = false;
  let expendRight = false;
  if(currentPage !== 1) {
    list.push({
      text: 'Prev',
      page: currentPage - 1,
      active: false,
      disabled: false,
    });
  }
  for(let i = 1; i <= pageCount; i++) {
    if(i === 1) {
      list.push({
        page: i,
        text: i,
        active: i === currentPage,
        disabled: i === currentPage,
      });
    } else if(i > 1 && currentPage - i > 4 && !expendLeft) {
      list.push({
        text: '...',
        active: false,
        disabled: true,
      });
      expendLeft = true;
    } else if(currentPage - i < 4 && currentPage > i) {
      list.push({
        text: i,
        page: i,
        active: false,
        disabled: false,
      });
    } else if(currentPage === i) {
      list.push({
        page: i,
        text: i,
        active: true,
        disabled: true,
      });
    } else if(i > currentPage && i - currentPage < 4) {
      list.push({
        page: i,
        text: i,
        active: false,
        disabled: false,
      });
    } else if(i > currentPage && i - currentPage > 4 && !expendRight) {
      list.push({
        text: '...',
        active: false,
        disabled: true,
      });
      expendRight = true;
    } else if(i === pageCount) {
      list.push({
        page: i,
        text: i,
        active: false,
        disabled: false,
      });
    }
  }
  if(currentPage < pageCount) {
    list.push({
      text: 'Next',
      page: currentPage + 1,
      active: false,
      disabled: false,
    });
  }

  pagination.value = list;

  console.log(pagination.value);
  emit('update:footer', true);
};

onMounted(async () => {
  await getRecords();
  history.pushState(JSON.stringify(records.value), null, location.href);
})

const deleteRecord = async (event, id) => {
  event.preventDefault();
  if(confirm('Are you sure to delete?')) {
    const res = await fetch(`${props.service}${DELETE_API}?id=${id}`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-cicada-token': props.token,
      },
      body: JSON.stringify({
        id: id
      }),
    });
    await res.json();
    window.location.reload();
  }
};

const seachByTag = async (event, tag) => {
  event.preventDefault();
  const keywords =`tag:${tag}`;
  emit('update:search', keywords);
  await getRecords({keywords});
  history.pushState(JSON.stringify(records.value), null, `/?keywords=${encodeURIComponent(keywords)}`);
};

addEventListener("popstate", async (event) => {
  // console.log(event, event.state);
  records.value = JSON.parse(event.state);
});
</script>

<template>
  <div class="cicada-list">
    <div v-for="record in records.result" :key="record._id" class="article-item">
      <h2><a :href="record.url" target="_blank">{{ record.title }}</a></h2>
      <p class="datetime">Updated At <time>{{ new Date(record.updatedAt).toLocaleString() }}</time></p>
      <p class="summary">{{ record.summary.replace(/\n/g, '<br />') }}</p>
      <div v-if="record.tags.length > 0" class="tags">Tags: 
        <p class="tags-wrap" v-for="tag in record.tags" :key="tag">
          <a href="###" v-on:click="seachByTag($event, tag)">{{ tag }}</a>
        </p>
      </div>
      <p class="operate">
        <a :href="`add.html?url=${encodeURIComponent(record.url)}&title=${encodeURIComponent(record.title)}&summary=${encodeURIComponent(record.summary)}&tag=${encodeURIComponent(record.tags.join())}&redirect=${encodeURIComponent(location)}`">Edit</a> | <a href="###" v-on:click="deleteRecord($event, record._id)">Delete</a>
      </p>
    </div>
  </div>
  <div v-if="records.pageCount > 1" class="pagination">
    <ul>
      <li v-for="p in pagination" :key="p" :class="{'active': p.active, 'disabled': p.disabled}">
        <a v-if="!p.disabled" :href="`?page=${p.page}`" v-on:click="getRecords({page: p})">{{ p.text }}</a>
        <div v-else>{{ p.text }}</div>
      </li>
    </ul>
  </div>
</template>

<style scoped>
.article-item {
  padding-bottom: 10px;
  border-bottom: 1px solid #ddd;
}
.tags-wrap {
  display: inline-block;
}
p a {
  border-bottom: 1px dashed #888;
  padding-bottom: 1px;
}
time {
  font-style: italic;
  color: #999;
}
.pagination {
  height: 60px;
}

.pagination ul {
  float: right;
  margin-top: 20px;
}
.pagination li {
  position: relative;
  float: left;
  padding: 6px 12px;
  line-height: 1.42857143;
  text-decoration: none;
  color: #337ab7;
  background-color: #ffffff;
  border: 1px solid #dddddd;
  margin-left: -1px;
  list-style: none;
}

.pagination a {
  color:#337ab7;
}

.disabled a,
.disabled div {
  color: #777777;
  background-color: #ffffff;
  border-color: #dddddd;
  cursor: not-allowed;
}
</style>
