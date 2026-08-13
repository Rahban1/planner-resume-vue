<script setup>
import { computed, onMounted, ref } from 'vue'

const API_URL = import.meta.env.VITE_API_BASE_URL || 'http://localhost:8080'

const profile = ref(null)
const projects = ref([])
const apiState = ref('checking')
const apiMessage = ref('Checking API')
const visits = ref(null)
const selectedTag = ref('All')
const openProject = ref(null)
const currentTime = ref('')
const form = ref({ name: '', email: '', message: '' })
const formState = ref('idle')
const formMessage = ref('')

const tags = computed(() => {
  const values = new Set(projects.value.flatMap((project) => project.tags))
  return ['All', ...values]
})

const filteredProjects = computed(() => {
  if (selectedTag.value === 'All') return projects.value
  return projects.value.filter((project) => project.tags.includes(selectedTag.value))
})

function updateClock() {
  currentTime.value = new Intl.DateTimeFormat('en', {
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
    hour12: false,
  }).format(new Date())
}

async function request(path, options = {}) {
  const response = await fetch(`${API_URL}${path}`, {
    headers: { 'Content-Type': 'application/json', ...options.headers },
    ...options,
  })
  const body = await response.json()
  if (!response.ok) throw new Error(body.error || 'The request failed.')
  return body
}

async function loadResume() {
  try {
    const [health, profileData, projectData, visitData] = await Promise.all([
      request('/api/health'),
      request('/api/profile'),
      request('/api/projects'),
      request('/api/visits', { method: 'POST' }),
    ])
    profile.value = profileData
    projects.value = projectData
    visits.value = visitData.visits
    apiState.value = 'online'
    apiMessage.value = `${health.service} online`
  } catch (error) {
    apiState.value = 'offline'
    apiMessage.value = error instanceof Error ? error.message : 'API unavailable'
  }
}

async function sendMessage() {
  formState.value = 'sending'
  formMessage.value = ''
  try {
    const result = await request('/api/contact', {
      method: 'POST',
      body: JSON.stringify(form.value),
    })
    formState.value = 'sent'
    formMessage.value = result.message
    form.value = { name: '', email: '', message: '' }
  } catch (error) {
    formState.value = 'error'
    formMessage.value = error instanceof Error ? error.message : 'The message failed.'
  }
}

onMounted(() => {
  updateClock()
  window.setInterval(updateClock, 1000)
  loadResume()
})
</script>

<template>
  <main>
    <header class="topbar">
      <a class="monogram" href="#top" aria-label="Go to top">MRG<span>.</span></a>
      <div class="system-status" :data-state="apiState" :title="apiMessage">
        <span class="status-light" aria-hidden="true"></span>
        API {{ apiState }}
      </div>
      <nav aria-label="Page sections">
        <a href="#work">Work</a>
        <a href="#contact">Contact</a>
      </nav>
      <time>{{ currentTime }}</time>
    </header>

    <section id="top" class="hero">
      <p class="eyebrow">Profile / 001</p>
      <div v-if="profile" class="hero-grid">
        <div>
          <p class="availability"><span></span> Available for thoughtful work</p>
          <h1>{{ profile.name }}</h1>
        </div>
        <div class="hero-copy">
          <p class="role">{{ profile.role }}</p>
          <p>{{ profile.summary }}</p>
          <a href="#work" class="text-link">See selected work <span>↘</span></a>
        </div>
      </div>
      <div v-else class="loading-card">
        <p>Start the Go API at <code>localhost:8080</code>.</p>
        <button type="button" @click="loadResume">Try again</button>
      </div>
      <p class="watermark" aria-hidden="true">ENGINEER</p>
    </section>

    <section v-if="profile" class="facts" aria-label="Profile facts">
      <div><span>Based</span><strong>{{ profile.location }}</strong></div>
      <div><span>Focus</span><strong>Useful software</strong></div>
      <div><span>Stack</span><strong>{{ profile.skills.slice(0, 3).join(' / ') }}</strong></div>
      <div><span>Demo visits</span><strong>{{ visits ?? '—' }}</strong></div>
    </section>

    <section id="work" class="work-section">
      <div class="section-heading">
        <div>
          <p class="eyebrow">Index / 002</p>
          <h2>Selected work</h2>
        </div>
        <div v-if="tags.length > 1" class="filters" aria-label="Filter projects">
          <button
            v-for="tag in tags"
            :key="tag"
            type="button"
            :class="{ active: selectedTag === tag }"
            @click="selectedTag = tag"
          >
            {{ tag }}
          </button>
        </div>
      </div>

      <div class="project-list">
        <article v-for="(project, index) in filteredProjects" :key="project.id">
          <button
            type="button"
            class="project-row"
            :aria-expanded="openProject === project.id"
            @click="openProject = openProject === project.id ? null : project.id"
          >
            <span class="project-number">0{{ index + 1 }}</span>
            <span class="project-name">{{ project.name }}</span>
            <span class="project-tags">{{ project.tags.join(' · ') }}</span>
            <span class="project-year">{{ project.year }}</span>
            <span class="project-action">{{ openProject === project.id ? '−' : '+' }}</span>
          </button>
          <div v-if="openProject === project.id" class="project-detail">
            <p>{{ project.description }}</p>
            <span>API record #{{ project.id }}</span>
          </div>
        </article>
      </div>
    </section>

    <section id="contact" class="contact-section">
      <div class="contact-intro">
        <p class="eyebrow">Channel / 003</p>
        <h2>Have a precise problem?</h2>
        <p>Send a test message to the Go API. This demo validates the message, but it does not store or send it.</p>
      </div>

      <form @submit.prevent="sendMessage">
        <label>
          <span>Name</span>
          <input v-model="form.name" name="name" autocomplete="name" required placeholder="Your name" />
        </label>
        <label>
          <span>Email</span>
          <input v-model="form.email" name="email" type="email" autocomplete="email" required placeholder="you@example.com" />
        </label>
        <label class="message-field">
          <span>Message</span>
          <textarea v-model="form.message" name="message" minlength="10" required placeholder="Tell me what you want to build."></textarea>
        </label>
        <button class="send-button" type="submit" :disabled="formState === 'sending'">
          {{ formState === 'sending' ? 'Sending…' : 'Send to API' }} <span>↗</span>
        </button>
        <p v-if="formMessage" class="form-message" :data-state="formState" role="status">{{ formMessage }}</p>
      </form>
    </section>

    <footer>
      <span>© 2026 Mohd Rahban Ghani</span>
      <span>Vue interface / Go service</span>
      <a href="#top">Back to top ↑</a>
    </footer>
  </main>
</template>
