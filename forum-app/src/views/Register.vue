<template>
  <div class="container d-flex justify-content-center align-items-center" style="min-height: 80vh;">
    <div class="card p-4" style="max-width: 400px; width: 100%;">
      <h3 class="mb-4 text-center">Créer un compte</h3>

      <!-- Erreur -->
      <div v-if="error" class="alert alert-danger alert-dismissible fade show">
        {{ error }}
        <button type="button" class="btn-close" @click="error = ''"></button>
      </div>

      <form @submit.prevent="handleRegister">
        <!-- Email -->
        <div class="mb-3">
          <label class="form-label">Email</label>
          <input
            type="email"
            class="form-control"
            v-model="email"
            required
          />
        </div>

        <!-- Password -->
        <div class="mb-3">
          <label class="form-label">Mot de passe</label>
          <input
            type="password"
            class="form-control"
            v-model="password"
            minlength="6"
            required
          />
        </div>

        <button class="btn btn-primary w-100" :disabled="loading">
          {{ loading ? "Inscription..." : "S'inscrire" }}
        </button>
      </form>

      <div class="mt-3 text-center">
        <router-link to="/login">Déjà un compte ? Se connecter</router-link>
      </div>
    </div>
  </div>
</template>


<script setup>
import { ref } from "vue";
import { useRouter } from "vue-router";
import { useAuth } from "../composables/useAuth";

const email = ref("");
const password = ref("");
const error = ref("");
const loading = ref(false);

const router = useRouter();
const { register } = useAuth();

async function handleRegister() {
  error.value = "";
  loading.value = true;

  try {
    await register(email.value, password.value);
    router.push("/login");

  } catch (err) {
    error.value = err.message || "Erreur lors de l'inscription.";
  } finally {
    loading.value = false;
  }
}
</script>

<style scoped>
.container {
  max-width: 100%;
}
</style>
