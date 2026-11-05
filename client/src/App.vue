<template>
  <div class="app" :class="{ 'sidebar-rail': isRail }">
    <aside class="sidebar">
      <div class="logo">
        <svg class="logo-mark" viewBox="0 0 36 36" fill="none" aria-hidden="true">
          <rect width="36" height="36" rx="9" fill="#0f172a"/>
          <path d="M18 8l9 5v10l-9 5-9-5V13l9-5z" stroke="#ffffff" stroke-width="2" stroke-linejoin="round"/>
          <path d="M9 13l9 5 9-5M18 18v10" stroke="#ffffff" stroke-width="2" stroke-linejoin="round"/>
        </svg>
        <div class="logo-text">
          <h1>{{ t('nav.companyName') }}</h1>
          <span class="subtitle">{{ t('nav.subtitle') }}</span>
        </div>
      </div>
      <!-- overflow belongs on this list only: clipping .sidebar would cut off the footer dropdowns -->
      <nav class="sidebar-nav">
        <router-link to="/" :class="{ active: $route.path === '/' }" :title="isRail ? t('nav.overview') : undefined">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <rect x="3" y="3" width="7" height="7" rx="1.5"/><rect x="14" y="3" width="7" height="7" rx="1.5"/><rect x="3" y="14" width="7" height="7" rx="1.5"/><rect x="14" y="14" width="7" height="7" rx="1.5"/>
          </svg>
          <span class="nav-label">{{ t('nav.overview') }}</span>
        </router-link>
        <router-link to="/inventory" :class="{ active: $route.path === '/inventory' }" :title="isRail ? t('nav.inventory') : undefined">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <path d="M21 8l-9-5-9 5v8l9 5 9-5V8z"/><path d="M3 8l9 5 9-5"/><path d="M12 13v8"/>
          </svg>
          <span class="nav-label">{{ t('nav.inventory') }}</span>
        </router-link>
        <router-link to="/orders" :class="{ active: $route.path === '/orders' }" :title="isRail ? t('nav.orders') : undefined">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <rect x="5" y="4" width="14" height="17" rx="2"/><path d="M9 4V3h6v1"/><path d="M9 10h6M9 14h6M9 18h3"/>
          </svg>
          <span class="nav-label">{{ t('nav.orders') }}</span>
        </router-link>
        <router-link to="/spending" :class="{ active: $route.path === '/spending' }" :title="isRail ? t('nav.finance') : undefined">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <rect x="2" y="5" width="20" height="14" rx="2"/><path d="M2 10h20"/><path d="M6 15h4"/>
          </svg>
          <span class="nav-label">{{ t('nav.finance') }}</span>
        </router-link>
        <router-link to="/demand" :class="{ active: $route.path === '/demand' }" :title="isRail ? t('nav.demandForecast') : undefined">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <path d="M3 17l6-6 4 4 8-8"/><path d="M15 7h6v6"/>
          </svg>
          <span class="nav-label">{{ t('nav.demandForecast') }}</span>
        </router-link>
        <router-link to="/reports" :class="{ active: $route.path === '/reports' }" :title="isRail ? 'Reports' : undefined">
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <path d="M14 3H7a2 2 0 00-2 2v14a2 2 0 002 2h10a2 2 0 002-2V8l-5-5z"/><path d="M14 3v5h5"/><path d="M9 13h6M9 17h6"/>
          </svg>
          <span class="nav-label">Reports</span>
        </router-link>
      </nav>
      <div class="sidebar-footer">
        <!-- Below the breakpoint the rail is forced, so a toggle there would have no visible effect -->
        <button
          v-if="!isNarrow"
          type="button"
          class="sidebar-toggle"
          :aria-expanded="!isRail"
          :title="isRail ? toggleLabel : undefined"
          @click="toggleSidebar"
        >
          <svg class="nav-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.75" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <rect x="3" y="4" width="18" height="16" rx="2"/>
            <path d="M9 4v16"/>
            <path :d="isRail ? 'M13 9l3 3-3 3' : 'M16 9l-3 3 3 3'"/>
          </svg>
          <span class="nav-label">{{ toggleLabel }}</span>
        </button>
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </aside>
    <div class="main-column">
      <FilterBar />
      <main class="main-content">
        <router-view />
      </main>
    </div>

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted, computed } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

const SIDEBAR_STORAGE_KEY = 'sidebar-collapsed'
// Keep in step with the max-width: 900px media queries in the style blocks
const NARROW_VIEWPORT_QUERY = '(max-width: 900px)'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    const readCollapsedPreference = () => {
      try {
        return localStorage.getItem(SIDEBAR_STORAGE_KEY) === 'true'
      } catch (err) {
        return false
      }
    }

    // Two separate sources: the saved choice and the viewport. The auto-collapse only
    // changes isNarrow, so shrinking the window never overwrites what the user saved
    // and the expanded sidebar comes back when the window widens again.
    const collapsedPreference = ref(readCollapsedPreference())
    const narrowViewport = window.matchMedia(NARROW_VIEWPORT_QUERY)
    const isNarrow = ref(narrowViewport.matches)
    const isRail = computed(() => isNarrow.value || collapsedPreference.value)

    const toggleLabel = computed(() => {
      return isRail.value ? t('nav.expandSidebar') : t('nav.collapseSidebar')
    })

    const toggleSidebar = () => {
      collapsedPreference.value = !collapsedPreference.value
      try {
        localStorage.setItem(SIDEBAR_STORAGE_KEY, String(collapsedPreference.value))
      } catch (err) {
        // Storage can be blocked. The toggle still works for this session.
      }
    }

    const handleViewportChange = (event) => {
      isNarrow.value = event.matches
    }

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(() => {
      loadTasks()
      narrowViewport.addEventListener('change', handleViewportChange)
    })

    onUnmounted(() => {
      narrowViewport.removeEventListener('change', handleViewportChange)
    })

    return {
      t,
      isNarrow,
      isRail,
      toggleLabel,
      toggleSidebar,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask
    }
  }
}
</script>

<style>
:root {
  /* Slate palette */
  --color-text-strong: #0f172a;
  --color-text: #1e293b;
  --color-text-body: #334155;
  --color-text-label: #475569;
  --color-text-muted: #64748b;
  --color-border-strong: #cbd5e1;
  --color-border: #e2e8f0;
  --color-border-subtle: #f1f5f9;
  --color-surface: #ffffff;
  --color-page: #f8fafc;
  --color-hover: #f1f5f9;

  /* Accent and status */
  --color-accent: #2563eb;
  --color-accent-soft: #eff6ff;
  --color-success: #059669;
  --color-warning: #ea580c;
  --color-danger: #dc2626;

  /* Spacing */
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 0.75rem;
  --space-4: 1rem;
  --space-5: 1.25rem;
  --space-6: 1.5rem;
  --space-8: 2rem;

  /* Radii */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;

  /* Elevation */
  --shadow-card: 0 1px 2px rgba(15, 23, 42, 0.05);
  --shadow-card-hover: 0 4px 12px rgba(15, 23, 42, 0.08);

  /* Stacking order: filter bar, then sidebar. Modals are teleported to <body>
     with their own z-index of 1000 or 2000, so both values stay below them. */
  --z-bar: 90;
  --z-sidebar: 100;

  /* Sidebar */
  --sidebar-width: 256px;
  --sidebar-rail-width: 72px;
  --sidebar-transition: 0.2s ease;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background: var(--color-page);
  color: var(--color-text);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* Sidebar and main column read this one value so they cannot drift apart */
.app {
  --sidebar-current-width: var(--sidebar-width);
  min-height: 100vh;
}

.app.sidebar-rail {
  --sidebar-current-width: var(--sidebar-rail-width);
}

.sidebar {
  position: fixed;
  top: 0;
  bottom: 0;
  left: 0;
  width: var(--sidebar-current-width);
  display: flex;
  flex-direction: column;
  background: var(--color-surface);
  border-right: 1px solid var(--color-border);
  z-index: var(--z-sidebar);
  transition: width var(--sidebar-transition);
}

.logo {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  flex-shrink: 0;
  padding: var(--space-4) var(--space-3);
  border-bottom: 1px solid var(--color-border);
}

.logo-mark {
  width: 36px;
  height: 36px;
  flex-shrink: 0;
  margin-left: 6px;
}

.logo-text {
  min-width: 0;
}

.logo h1 {
  font-size: 1rem;
  font-weight: 700;
  line-height: 1.3;
  color: var(--color-text-strong);
  letter-spacing: -0.025em;
}

.subtitle {
  display: block;
  font-size: 0.75rem;
  line-height: 1.3;
  color: var(--color-text-muted);
  font-weight: 400;
}

.sidebar-nav {
  flex: 1;
  min-height: 0;
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
  padding: var(--space-3);
  overflow-x: hidden;
  overflow-y: auto;
}

.sidebar-nav a,
.sidebar-toggle {
  position: relative;
  display: flex;
  align-items: center;
  flex-shrink: 0;
  gap: var(--space-3);
  width: 100%;
  padding: 0.625rem var(--space-3);
  color: var(--color-text-muted);
  background: none;
  border: none;
  border-radius: var(--radius-md);
  font-family: inherit;
  font-weight: 500;
  font-size: 0.875rem;
  text-align: left;
  text-decoration: none;
  white-space: nowrap;
  cursor: pointer;
  transition: color 0.15s ease, background 0.15s ease;
}

.sidebar-nav a:hover,
.sidebar-toggle:hover {
  color: var(--color-text-strong);
  background: var(--color-hover);
}

.sidebar-nav a.active {
  color: var(--color-accent);
  background: var(--color-accent-soft);
}

.nav-icon {
  width: 20px;
  height: 20px;
  flex-shrink: 0;
}

.sidebar-footer {
  display: flex;
  flex-direction: column;
  flex-shrink: 0;
  gap: var(--space-2);
  padding: var(--space-3);
  border-top: 1px solid var(--color-border);
}

/* Rail mode. Text is hidden visually, not with display: none, so each icon-only
   control keeps its accessible name. */
.app.sidebar-rail .logo-text,
.app.sidebar-rail .nav-label,
.app.sidebar-rail .language-label,
.app.sidebar-rail .profile-name {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip-path: inset(50%);
  white-space: nowrap;
}

.app.sidebar-rail .sidebar-nav a,
.app.sidebar-rail .sidebar-toggle {
  justify-content: center;
}

.app.sidebar-rail .language-button,
.app.sidebar-rail .profile-button {
  justify-content: center;
  padding: 0.375rem;
}

.app.sidebar-rail .language-button {
  padding: 0.625rem 0.375rem;
}

.app.sidebar-rail .language-button .chevron,
.app.sidebar-rail .profile-button .chevron {
  display: none;
}

/* min-width: 0 lets wide tables scroll inside their container instead of pushing
   the page sideways. No overflow here: it would stop the filter bar sticking. */
.main-column {
  display: flex;
  flex-direction: column;
  min-width: 0;
  min-height: 100vh;
  margin-left: var(--sidebar-current-width);
  transition: margin-left var(--sidebar-transition);
}

.main-content {
  flex: 1;
  max-width: 1600px;
  width: 100%;
  margin: 0 auto;
  padding: var(--space-6) var(--space-8);
}

.page-header {
  margin-bottom: var(--space-6);
}

.page-header h2 {
  font-size: 1.75rem;
  font-weight: 700;
  color: var(--color-text-strong);
  margin-bottom: 0.375rem;
  letter-spacing: -0.025em;
}

.page-header p {
  color: var(--color-text-muted);
  font-size: 0.938rem;
}

.stats-grid {
  display: grid;
  /* min() stops a column overflowing when the main column is narrower than 280px */
  grid-template-columns: repeat(auto-fit, minmax(min(280px, 100%), 1fr));
  gap: var(--space-5);
  margin-bottom: var(--space-6);
}

.stat-card {
  background: var(--color-surface);
  padding: var(--space-5);
  border-radius: var(--radius-lg);
  border: 1px solid var(--color-border);
  box-shadow: var(--shadow-card);
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

.stat-card:hover {
  border-color: var(--color-border-strong);
  box-shadow: var(--shadow-card-hover);
}

.stat-label {
  color: var(--color-text-muted);
  font-size: 0.813rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.625rem;
}

.stat-value {
  font-size: 2rem;
  font-weight: 700;
  color: var(--color-text-strong);
  letter-spacing: -0.025em;
}

.stat-card.warning .stat-value {
  color: var(--color-warning);
}

.stat-card.success .stat-value {
  color: var(--color-success);
}

.stat-card.danger .stat-value {
  color: var(--color-danger);
}

.stat-card.info .stat-value {
  color: var(--color-accent);
}

.card {
  background: var(--color-surface);
  border-radius: var(--radius-lg);
  padding: var(--space-5);
  border: 1px solid var(--color-border);
  box-shadow: var(--shadow-card);
  margin-bottom: var(--space-5);
}

/* Fixed-width bars can exceed the main column beside the rail. Scrolling stays inside
   the card so the page itself never scrolls sideways. */
.card:has(.bar-chart) {
  overflow-x: auto;
}

.card-header {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: center;
  gap: var(--space-3);
  margin-bottom: var(--space-4);
  padding-bottom: 0.875rem;
  border-bottom: 1px solid var(--color-border);
}

.card-title {
  font-size: 1.063rem;
  font-weight: 700;
  color: var(--color-text-strong);
  letter-spacing: -0.025em;
}

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: var(--color-page);
  border-top: 1px solid var(--color-border);
  border-bottom: 1px solid var(--color-border);
}

th {
  text-align: left;
  padding: 0.625rem var(--space-3);
  font-weight: 600;
  color: var(--color-text-label);
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: 0.625rem var(--space-3);
  border-top: 1px solid var(--color-border-subtle);
  color: var(--color-text-body);
  font-size: 0.875rem;
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  background: var(--color-page);
}

.badge {
  display: inline-block;
  padding: 0.25rem 0.625rem;
  border-radius: var(--radius-sm);
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.025em;
}

.badge.success {
  background: #d1fae5;
  color: #065f46;
}

.badge.warning {
  background: #fed7aa;
  color: #92400e;
}

.badge.danger {
  background: #fecaca;
  color: #991b1b;
}

.badge.info {
  background: #dbeafe;
  color: #1e40af;
}

.badge.increasing {
  background: #d1fae5;
  color: #065f46;
}

.badge.decreasing {
  background: #fecaca;
  color: #991b1b;
}

.badge.stable {
  background: #e0e7ff;
  color: #3730a3;
}

.badge.high {
  background: #fecaca;
  color: #991b1b;
}

.badge.medium {
  background: #fed7aa;
  color: #92400e;
}

.badge.low {
  background: #dbeafe;
  color: #1e40af;
}

.loading {
  text-align: center;
  padding: 3rem;
  color: var(--color-text-muted);
  font-size: 0.938rem;
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: 1rem;
  border-radius: var(--radius-md);
  margin: 1rem 0;
  font-size: 0.938rem;
}
/* Same breakpoint as NARROW_VIEWPORT_QUERY */
@media (max-width: 900px) {
  .main-content {
    padding: var(--space-4);
  }
}

@media (prefers-reduced-motion: reduce) {
  .sidebar,
  .main-column {
    transition: none;
  }
}
</style>
