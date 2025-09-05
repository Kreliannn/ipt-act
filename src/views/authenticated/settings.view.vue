<script setup>
import { ref, watch, onMounted } from 'vue'
import { getAuth, onAuthStateChanged } from 'firebase/auth'
import { doc, getDoc, setDoc } from 'firebase/firestore'
import { db } from '@/config/firebaseConfig'


const isDark = ref(false)

const toggleDark = () => {
  isDark.value = !isDark.value
  document.documentElement.classList.toggle('dark', isDark.value)
}

// keep preference in localStorage
onMounted(() => {
  if (localStorage.getItem('theme') === 'dark') {
    isDark.value = true
    document.documentElement.classList.add('dark')
  }
})

watch(isDark, (val) => {
  localStorage.setItem('theme', val ? 'dark' : 'light')
})

// Refs for settings
const darkMode = ref(false)
const language = ref('en')
const localProfile = ref(false)
const activeStatus = ref('online')
const muteNotifications = ref(false)
const deactivateAccount = ref(false)

const saveLoading = ref(false)


const languages = ['en', 'es', 'fr', 'de']
const statuses = ['online', 'away', 'busy', 'offline']


const auth = getAuth()
const userId = ref(null)

const isLoading = ref(true)

const getSettingsById = async (uid) => {
  try {
    const docRef = doc(db, 'setting', uid)
    const docSnap = await getDoc(docRef)
    if (docSnap.exists()) {
      return { id: docSnap.id, ...docSnap.data() }
    } else {
      return null
    }
  } catch (error) {
    console.error('Error getting document:', error)
    return null
  }
}


onAuthStateChanged(auth, async (user) => {
  if (user) {
    userId.value = user.uid
    const setting = await getSettingsById(user.uid)
    if (setting) {
      darkMode.value = setting.isDarkMode ?? false
      language.value = setting.language ?? 'en'
      localProfile.value = setting.islockProfile ?? false
      activeStatus.value = setting.activeStatus ?? 'online'
      muteNotifications.value = setting.isNotificationMuted ?? false
      deactivateAccount.value = setting.isAccountDeact ?? false
    }
  }
  isLoading.value = false
})

watch([darkMode, language, localProfile, activeStatus, muteNotifications, deactivateAccount], async () => {
  if (!userId.value) {
    console.warn('User not logged in')
    return
  }

  const settingsToSave = {
    isDarkMode: darkMode.value,
    language: language.value,
    islockProfile: localProfile.value,
    activeStatus: activeStatus.value,
    isNotificationMuted: muteNotifications.value,
    isAccountDeact: deactivateAccount.value,
  }

  try {
    console.log(settingsToSave)
    await setDoc(doc(db, 'setting', userId.value), settingsToSave)
    console.log('Settings saved successfully!')
  } catch (error) {
    console.error('Error saving settings:', error)
    console.log('Failed to save settings.')
  }
})


watch(darkMode, async () => {
toggleDark()
})

</script>

<template>
  <div class="min-h-screen  dark:from-gray-900 dark:to-gray-800  dark:text-white  text-gray-900 ">
    <div class="container mx-auto max-w-4xl px-4 py-8">
      
      <div class="mb-8">
        <h1 class="text-3xl font-bold text-gray-800 dark:text-white mb-2">Settings</h1>
        <p class="text-gray-600 dark:text-gray-300">Manage your account preferences and settings</p>
      </div>

      <div v-if="isLoading" class="flex items-center justify-center py-12">
        <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-600"></div>
        <span class="ml-3 text-gray-500 dark:text-gray-400">Loading settings...</span>
      </div>

      <div v-else class="space-y-6">
   
        <div class="bg-white dark:bg-gray-800 rounded-xl shadow-sm border border-gray-200 dark:border-gray-700 p-6">
          <h2 class="text-lg font-semibold text-gray-800 dark:text-white mb-4 flex items-center">
            <svg class="w-5 h-5 mr-2 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M7 21a4 4 0 01-4-4V5a2 2 0 012-2h4a2 2 0 012 2v12a4 4 0 01-4 4zM7 3H5a2 2 0 00-2 2v12a4 4 0 004 4h2M9 3h2a2 2 0 012 2v12a4 4 0 01-2 2H9m8-10V9a2 2 0 00-2-2H5"></path>
            </svg>
            Appearance
          </h2>
          
      
          <div class="flex items-center justify-between py-4 border-b border-gray-100 dark:border-gray-700 last:border-b-0">
            <div class="flex-1">
              <label for="darkMode" class="text-sm font-medium text-gray-700 dark:text-gray-300 cursor-pointer">
                Dark Mode
              </label>
              <p class="text-xs text-gray-500 dark:text-gray-400 mt-1">Toggle between light and dark theme</p>
            </div>
            <div class="relative inline-block w-12 h-6">
              <input 
                type="checkbox" 
                id="darkMode" 
                v-model="darkMode" 
                class="sr-only peer"
              />
              <label for="darkMode" class="block w-full h-full bg-gray-300 peer-focus:outline-none peer-focus:ring-4 peer-focus:ring-blue-300 dark:peer-focus:ring-blue-800 rounded-full cursor-pointer peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-0.5 after:left-0.5 after:bg-white after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-blue-600"></label>
            </div>
          </div>

   
          <div class="flex items-center justify-between py-4">
            <div class="flex-1">
              <label for="language" class="text-sm font-medium text-gray-700 dark:text-gray-300">Language</label>
              <p class="text-xs text-gray-500 dark:text-gray-400 mt-1">Choose your preferred language</p>
            </div>
            <select 
              id="language" 
              v-model="language" 
              class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block px-3 py-2 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
            >
              <option v-for="lang in languages" :key="lang" :value="lang">
                {{ lang.toUpperCase() }}
              </option>
            </select>
          </div>
        </div>

  
        <div class="bg-white dark:bg-gray-800 rounded-xl shadow-sm border border-gray-200 dark:border-gray-700 p-6">
          <h2 class="text-lg font-semibold text-gray-800 dark:text-white mb-4 flex items-center">
            <svg class="w-5 h-5 mr-2 text-green-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"></path>
            </svg>
            Privacy & Security
          </h2>

     
          <div class="flex items-center justify-between py-4 border-b border-gray-100 dark:border-gray-700">
            <div class="flex-1">
              <label for="localProfile" class="text-sm font-medium text-gray-700 dark:text-gray-300 cursor-pointer">
                Local Profile
              </label>
              <p class="text-xs text-gray-500 dark:text-gray-400 mt-1">Keep profile data locally</p>
            </div>
            <div class="relative inline-block w-12 h-6">
              <input 
                type="checkbox" 
                id="localProfile" 
                v-model="localProfile" 
                class="sr-only peer"
              />
              <label for="localProfile" class="block w-full h-full bg-gray-300 peer-focus:outline-none peer-focus:ring-4 peer-focus:ring-green-300 dark:peer-focus:ring-green-800 rounded-full cursor-pointer peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-0.5 after:left-0.5 after:bg-white after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-green-600"></label>
            </div>
          </div>


          <div class="flex items-center justify-between py-4">
            <div class="flex-1">
              <label for="activeStatus" class="text-sm font-medium text-gray-700 dark:text-gray-300">Active Status</label>
              <p class="text-xs text-gray-500 dark:text-gray-400 mt-1">Show your availability to others</p>
            </div>
            <select 
              id="activeStatus" 
              v-model="activeStatus" 
              class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-green-500 focus:border-green-500 block px-3 py-2 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-green-500 dark:focus:border-green-500"
            >
              <option v-for="status in statuses" :key="status" :value="status">
                <span class="flex items-center">
                  {{ status.charAt(0).toUpperCase() + status.slice(1) }}
                </span>
              </option>
            </select>
          </div>
        </div>

   
        <div class="bg-white dark:bg-gray-800 rounded-xl shadow-sm border border-gray-200 dark:border-gray-700 p-6">
          <h2 class="text-lg font-semibold text-gray-800 dark:text-white mb-4 flex items-center">
            <svg class="w-5 h-5 mr-2 text-yellow-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 17h5l-5 5v-5zM15 17H9a6 6 0 01-6-6V9a6 6 0 016-6h6a6 6 0 016 6v2"></path>
            </svg>
            Notifications
          </h2>

          
          <div class="flex items-center justify-between py-4">
            <div class="flex-1">
              <label for="muteNotifications" class="text-sm font-medium text-gray-700 dark:text-gray-300 cursor-pointer">
                Mute Notifications
              </label>
              <p class="text-xs text-gray-500 dark:text-gray-400 mt-1">Disable all notification sounds and alerts</p>
            </div>
            <div class="relative inline-block w-12 h-6">
              <input 
                type="checkbox" 
                id="muteNotifications" 
                v-model="muteNotifications" 
                class="sr-only peer"
              />
              <label for="muteNotifications" class="block w-full h-full bg-gray-300 peer-focus:outline-none peer-focus:ring-4 peer-focus:ring-yellow-300 dark:peer-focus:ring-yellow-800 rounded-full cursor-pointer peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-0.5 after:left-0.5 after:bg-white after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-yellow-600"></label>
            </div>
          </div>
        </div>

   
        <div class="bg-white dark:bg-gray-800 rounded-xl shadow-sm border border-gray-200 dark:border-gray-700 p-6">
          <h2 class="text-lg font-semibold text-gray-800 dark:text-white mb-4 flex items-center">
            <svg class="w-5 h-5 mr-2 text-red-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"></path>
            </svg>
            Account Management
          </h2>

  
          <div class="flex items-center justify-between py-4">
            <div class="flex-1">
              <label for="deactivateAccount" class="text-sm font-medium text-gray-700 dark:text-gray-300 cursor-pointer">
                Deactivate Account
              </label>
              <p class="text-xs text-gray-500 dark:text-gray-400 mt-1">Temporarily disable your account</p>
            </div>
            <div class="relative inline-block w-12 h-6">
              <input 
                type="checkbox" 
                id="deactivateAccount" 
                v-model="deactivateAccount" 
                class="sr-only peer"
              />
              <label for="deactivateAccount" class="block w-full h-full bg-gray-300 peer-focus:outline-none peer-focus:ring-4 peer-focus:ring-red-300 dark:peer-focus:ring-red-800 rounded-full cursor-pointer peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-0.5 after:left-0.5 after:bg-white after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-red-600"></label>
            </div>
          </div>
        </div>


        <div class="text-center py-4">
          <p class="text-xs text-gray-500 dark:text-gray-400 flex items-center justify-center">
            <svg class="w-4 h-4 mr-1 text-green-500" fill="currentColor" viewBox="0 0 20 20">
              <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd"></path>
            </svg>
            Settings are automatically saved
          </p>
        </div>
      </div>
    </div>
  </div>
</template>