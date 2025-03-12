<template>
  <div>
    <head>
      <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    </head>
    <div class="container mt-5">
      <div class="row">
        <!-- Registrierungsformular -->
        <div class="col-md-6">
          <h2>Registrierung</h2>
          <form @submit.prevent="register">
            <div class="mb-3">
              <label for="username" class="form-label">Benutzername</label>
              <input type="text" class="form-control" id="username" v-model="registerForm.username" required>
            </div>
            <div class="mb-3">
              <label for="password" class="form-label">Passwort</label>
              <input type="password" class="form-control" id="password" v-model="registerForm.password" required>
            </div>
            <div class="mb-3">
              <label for="confirmPassword" class="form-label">Passwort bestätigen</label>
              <input type="password" class="form-control" id="confirmPassword" v-model="registerForm.confirmPassword" required>
            </div>
            <button type="submit" class="btn btn-primary">Registrieren</button>
            <div v-if="registerMessage.text" :class="['alert mt-3', registerMessage.type]">
              {{ registerMessage.text }}
            </div>
          </form>
        </div>

        <!-- Login-Formular -->
        <div class="col-md-6">
          <h2>Login</h2>
          <form @submit.prevent="login">
            <div class="mb-3">
              <label for="loginUsername" class="form-label">Benutzername</label>
              <input type="text" class="form-control" id="loginUsername" v-model="loginForm.username" required>
            </div>
            <div class="mb-3">
              <label for="loginPassword" class="form-label">Passwort</label>
              <input type="password" class="form-control" id="loginPassword" v-model="loginForm.password" required>
            </div>
            <button type="submit" class="btn btn-primary">Einloggen</button>
            <div v-if="loginMessage.text" :class="['alert mt-3', loginMessage.type]">
              {{ loginMessage.text }}
            </div>
          </form>
        </div>
      </div>
    </div>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { createClient } from '@supabase/supabase-js'

const supabase = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_KEY
)

const registerForm = ref({
  username: '',
  password: '',
  confirmPassword: ''
})

const loginForm = ref({
  username: '',
  password: ''
})

const registerMessage = ref({ text: '', type: '' })
const loginMessage = ref({ text: '', type: '' })

const register = async () => {
  if (registerForm.value.password !== registerForm.value.confirmPassword) {
    registerMessage.value = {
      text: 'Die Passwörter stimmen nicht überein!',
      type: 'alert-danger'
    }
    return
  }

  try {
    const { data, error } = await supabase
      .from('users')
      .insert([
        {
          username: registerForm.value.username,
          password_hash: registerForm.value.password // In der Praxis sollten Sie das Passwort hashen!
        }
      ])

    if (error) throw error

    registerMessage.value = {
      text: 'Erfolgreich registriert!',
      type: 'alert-success'
    }
    registerForm.value = { username: '', password: '', confirmPassword: '' }
  } catch (error) {
    registerMessage.value = {
      text: 'Registrierung fehlgeschlagen: ' + error.message,
      type: 'alert-danger'
    }
  }
}

const login = async () => {
  try {
    const { data, error } = await supabase
      .from('users')
      .select()
      .eq('username', loginForm.value.username)
      .eq('password_hash', loginForm.value.password)
      .single()

    if (error) throw error

    if (data) {
      loginMessage.value = {
        text: 'Erfolgreich eingeloggt!',
        type: 'alert-success'
      }
      loginForm.value = { username: '', password: '' }
    } else {
      throw new Error('Ungültige Anmeldedaten')
    }
  } catch (error) {
    loginMessage.value = {
      text: 'Login fehlgeschlagen: ' + error.message,
      type: 'alert-danger'
    }
  }
}
</script>

<style scoped>
.alert-success {
  background-color: #d4edda;
  border-color: #c3e6cb;
  color: #155724;
}

.alert-danger {
  background-color: #f8d7da;
  border-color: #f5c6cb;
  color: #721c24;
}
</style> 