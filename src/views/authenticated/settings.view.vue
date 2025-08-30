<script setup>
import { ref, watch } from 'vue'
import { getAuth, onAuthStateChanged } from 'firebase/auth'
import { doc, getDoc, setDoc } from 'firebase/firestore'
import { db } from '@/config/firebaseConfig'

// Refs for settings
const darkMode = ref(false)
const language = ref('en')
const localProfile = ref(false)
const activeStatus = ref('online')
const muteNotifications = ref(false)
const deactivateAccount = ref(false)

const saveLoading = ref(false)

// Language and status options
const languages = ['en', 'es', 'fr', 'de']
const statuses = ['online', 'away', 'busy', 'offline']

// Firebase Auth and userId
const auth = getAuth()
const userId = ref(null)

// Loading state
const isLoading = ref(true)

// Load user settings
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

// Watch for auth state and fetch settings
onAuthStateChanged(auth, async (user) => {
  if (user) {
    userId.value = user.uid
    const setting = await getSettingsById(user.uid)
    if (setting) {
      // Populate the form inputs
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

// Save settings to Firestore
const handleSave = async () => {
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
    alert('Settings saved successfully!')
  } catch (error) {
    console.error('Error saving settings:', error)
    alert('Failed to save settings.')
  }
}
</script>

<template>
  <div class="h-screen bg-gray-100 dark:bg-gray-900 text-gray-900 dark:text-white p-6">
    <h1 class="text-2xl font-bold mb-6">Settings</h1>

    <div v-if="isLoading" class="text-center text-gray-500">Loading settings...</div>

    <div v-else class="space-y-6">
      <!-- Dark Mode -->
      <div class="flex items-center justify-between">
        <label for="darkMode">Dark Mode</label>
        <input type="checkbox" id="darkMode" v-model="darkMode" class="w-5 h-5" />
      </div>

      <!-- Language -->
      <div class="flex items-center justify-between">
        <label for="language">Language</label>
        <select id="language" v-model="language" class="rounded px-2 py-1 bg-white text-black">
          <option v-for="lang in languages" :key="lang" :value="lang">
            {{ lang.toUpperCase() }}
          </option>
        </select>
      </div>

      <!-- Local Profile -->
      <div class="flex items-center justify-between">
        <label for="localProfile">Local Profile</label>
        <input type="checkbox" id="localProfile" v-model="localProfile" class="w-5 h-5" />
      </div>

      <!-- Active Status -->
      <div class="flex items-center justify-between">
        <label for="activeStatus">Active Status</label>
        <select id="activeStatus" v-model="activeStatus" class="rounded px-2 py-1 bg-white text-black">
          <option v-for="status in statuses" :key="status" :value="status">
            {{ status }}
          </option>
        </select>
      </div>

      <!-- Mute Notifications -->
      <div class="flex items-center justify-between">
        <label for="muteNotifications">Mute Notifications</label>
        <input type="checkbox" id="muteNotifications" v-model="muteNotifications" class="w-5 h-5" />
      </div>

      <!-- Deactivate Account -->
      <div class="flex items-center justify-between">
        <label for="deactivateAccount">Deactivate Account</label>
        <input type="checkbox" id="deactivateAccount" v-model="deactivateAccount" class="w-5 h-5" />
      </div>

      <!-- Save Button -->
      <div class="pt-6">
        <button
          class="bg-blue-600 hover:bg-blue-700 text-white font-semibold py-2 px-4 rounded shadow transition-all duration-200"
          @click="handleSave"
          :class="{ 'bg-red-500' : isLoading }"
        >
          Save Changes
        </button>
      </div>

      
    </div>
  </div>
</template>
