<template>
  <div>
    <header>
      <div>
        <button class="btn add-btn" @click="handleAddWatchLater"><svg style="width: 14px;height: 14px;"
            xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"
            stroke-linecap="round" stroke-linejoin="round" class="feather feather-plus">
            <line x1="12" y1="5" x2="12" y2="19"></line>
            <line x1="5" y1="12" x2="19" y2="12"></line>
          </svg> Watch Later</button>
      </div>
      <div class="more-btn">
        <button @click="cleanSelected" class="btn">清空选中{{ showCount(selectedCount) }}</button>
        <button @click="cleanWatched" class="btn">清空已看{{ showCount(watchedCount) }}</button>
      </div>
      <input placeholder="Click enter to add a custom url" @keyup.enter="handleAddUserInputWebsite" type="text"
        v-model="userInputSiteURL">
    </header>
    <main>
      <ul>
        <li v-for="item in watchLaterWebsites" :key="item.url">
          <input v-model="item.selected" type="checkbox">
          <img :src="item.favIconUrl">
          <a :title="`${item.title} - ${item.created}`" class="site-url" :class="{ watched: item.watched }"
            @click="markWebsiteReaded(item)" target="_blank" :href="item.url">{{
          item.title }}
            <!-- <span class="site-url-popup">{{ item.title }} {{ item.created }}</span> -->
          </a>
          <span class="trash-icon" @click="handleDeleteWebsite(item)">
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor"
              stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-trash-2">
              <polyline points="3 6 5 6 21 6"></polyline>
              <path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path>
              <line x1="10" y1="11" x2="10" y2="17"></line>
              <line x1="14" y1="11" x2="14" y2="17"></line>
            </svg>
          </span>
        </li>
      </ul>
    </main>
  </div>

</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'

const LOCAL_STORAGE_KEY = 'WATCH_LATER_WEBSITES'

const watchLaterWebsites = ref(JSON.parse(localStorage.getItem(LOCAL_STORAGE_KEY) || '[]'))
const userInputSiteURL = ref("")

const siteCount = computed(() => watchLaterWebsites.value.length)
const watchedCount = computed(() => watchLaterWebsites.value.filter(s => s.watched).length)
const selectedCount = computed(() => watchLaterWebsites.value.filter(s => s.selected).length)

onMounted(() => {
  watchLaterWebsites.value.sort((a, b) => {
    // 优先按是否已观看排序
    if (a.watched !== b.watched) {
      return a.watched - b.watched
    } else {
      return new Date(b.created) - new Date(a.created)
    }
  })
})

const saveWatchLaterAfter = (fn) => {
  return (...args) => {
    const ret = fn(...args)
    localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(watchLaterWebsites.value))
    return ret
  }
}

const showCount = (cnt) => {
  return cnt ? `(${cnt})` : ""
}

const createWebsiteItem = (url, title, favIconUrl) => {
  return {
    url,
    title,
    favIconUrl,
    created: new Date(),
    created: new Date(),
    watched: false,
    selected: false
  }
}

const addWatchLater = saveWatchLaterAfter((url, title, favIconUrl) => {
  if (watchLaterWebsites.value.some(item => item.url === url)) {
    return
  }

  watchLaterWebsites.value.unshift(createWebsiteItem(url, title, favIconUrl))
})

const handleAddWatchLater = () => {
  // addWatchLater('url' + new Date(), 'title', 'favIconUrl')

  chrome.tabs.query({ active: true, currentWindow: true }, function (tabs) {
    if (tabs.length > 0) {
      const activeTab = tabs[0];
      // console.log(tabs, 'tabs');

      const { url, title, favIconUrl } = activeTab
      addWatchLater(url, title, favIconUrl)
    }
  });

}

const handleAddUserInputWebsite = () => {
  const url = userInputSiteURL.value
  if (url && URL.canParse(url)) {
    addWatchLater(url, url, "")
    userInputSiteURL.value = ""
  }
}

const findWebsite = (item, index) => {
  const method = index ? 'findIndex' : 'find'
  return watchLaterWebsites.value[method]((s) => s.url === item.url)
}

const markWebsiteReaded = saveWatchLaterAfter((item) => {
  const target = findWebsite(item)
  target.watched = true
})

const handleDeleteWebsite = saveWatchLaterAfter((item) => {
  watchLaterWebsites.value = watchLaterWebsites.value.filter(s => s.url !== item.url)
})

const cleanWatched = saveWatchLaterAfter(() => {
  watchLaterWebsites.value = watchLaterWebsites.value.filter(s => !s.watched)
})

const cleanSelected = saveWatchLaterAfter(() => {
  watchLaterWebsites.value = watchLaterWebsites.value.filter(s => !s.selected)
})
</script>

<style scoped>
header {
  display: flex;
  flex-flow: column;

  /* box-shadow: 0 0 3px rgba(0, 0, 0, .3); */
  padding: 0.4rem 0;

  margin-bottom: 0.5rem;
}

header div {
  margin-bottom: 0.5rem;
}

header input {
  margin-top: 0.5rem;
  line-height: 1.2rem;

  border: 1px solid #ccc;
  box-shadow: 0 0 3px rgba(0, 0, 0, .3);
}

header input:focus {
  outline: 2px solid #00b3ee;
}

header .more-btn {
  display: flex;
  gap: 5px;
}

.more-btn .btn {
  flex: 1;
}

.btn {
  color: #00b3ee;
  background: white;
  border: none;
  box-shadow: 0 0 3px rgba(0, 0, 0, 0.3);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  /* border-radius: 2px; */
}

.btn:hover {
  background-color: #00b3ee;
  color: white;
}

.add-btn {
  width: 100%;
  height: 1.5rem;
}

a,
svg:hover {
  color: #00b3ee;
  transition: all 0.3s ease;
}

a.watched {
  color: #999;
  text-decoration: line-through;
}

ul {
  padding: 0;
  max-height: 220px;
  overflow-y: auto;
  padding: 0.4rem 0.2rem;
  margin: 0;
}

li {
  list-style: none;
  margin-bottom: 0.5rem;
  padding: 0.4rem 0.2rem;
  box-shadow: 0 0 3px rgba(0, 0, 0, 0.3);

  display: flex;
  justify-content: space-between;
  align-items: center;
}

li .trash-icon {
  width: 16px;
  height: 16px;
}

li .site-url {
  flex: 1;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  margin-left: 0.3rem;
  position: relative;
}

/* .site-url>.site-url-popup {
  position: absolute;
  background-color: rgba(0, 0, 0, 0.7);
  top: 2rem;
} */

li img {
  width: 16px;
  height: 16px;
}
</style>
