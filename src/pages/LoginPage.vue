<template>
  <div class="login-wrapper">
    <div class="login-container">
      <q-card class="login-card shadow-12">
        <!-- Logo Header -->
        <q-card-section class="text-center q-pt-lg q-pb-none">
          <div class="logo-container q-mb-md">
            <img src="~assets/uce_logo.png" alt="Logo UCE" class="institution-logo" />
          </div>
          <div class="text-h5 text-bold text-grey-9">Red Meteorológica</div>
          <div class="text-caption text-grey-7 q-mt-xs">
            Consola de Monitoreo Ambiental y Telemetría
          </div>
        </q-card-section>

        <!-- Form Section -->
        <q-card-section class="q-px-lg q-pt-md">
          <q-form class="q-gutter-y-md" @submit.prevent="handleLogin">
            <div>
              <label class="input-label">Usuario o Correo Electrónico</label>
              <q-input
                v-model="username"
                outlined
                dense
                placeholder="Ingrese su usuario"
                class="custom-input"
                :rules="[(val) => !!val || 'El usuario es obligatorio']"
              >
                <template #prepend>
                  <q-icon name="person_outline" color="primary" />
                </template>
              </q-input>
            </div>

            <div>
              <div class="flex justify-between items-center q-mb-xs">
                <label class="input-label">Contraseña</label>
                <a href="#" class="forgot-link" @click.prevent="showForgotNotify">¿Olvidó su contraseña?</a>
              </div>
              <q-input
                v-model="password"
                :type="showPassword ? 'text' : 'password'"
                outlined
                dense
                placeholder="Ingrese su contraseña"
                class="custom-input"
                :rules="[(val) => !!val || 'La contraseña es obligatoria']"
              >
                <template #prepend>
                  <q-icon name="lock_outline" color="primary" />
                </template>
                <template #append>
                  <q-icon
                    :name="showPassword ? 'visibility_off' : 'visibility'"
                    class="cursor-pointer"
                    color="grey-6"
                    @click="showPassword = !showPassword"
                  />
                </template>
              </q-input>
            </div>

            <q-btn
              type="submit"
              color="primary"
              size="lg"
              no-caps
              class="full-width q-mt-md text-bold submit-btn"
              :loading="loading"
            >
              Iniciar Sesión
            </q-btn>
          </q-form>
        </q-card-section>

        <!-- Footer -->
        <q-card-section class="text-center q-pt-none q-pb-md text-caption text-grey-6">
          Sistema de Monitoreo Meteorológico
        </q-card-section>
      </q-card>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { useQuasar } from 'quasar';
import { api } from 'src/boot/axios';
import { useRouter } from 'vue-router';
import { useAuthStore } from 'src/stores/auth-store';

const router = useRouter();
const $q = useQuasar();
const username = ref('');
const password = ref('');
const showPassword = ref(false);
const loading = ref(false);

const authStore = useAuthStore();

const showForgotNotify = () => {
  $q.notify({
    type: 'warning',
    message: 'Por favor contacte al administrador del sistema para restablecer su clave.',
    icon: 'help_outline',
  });
};

const handleLogin = () => {
  if (!username.value || !password.value) {
    $q.notify({
      type: 'negative',
      message: 'Por favor, complete todos los campos.',
    });
    return;
  }

  loading.value = true;

  api
    .post('/auth/login', {
      username: username.value,
      password: password.value,
    })
    .then((response) => {
      authStore.setJWT(response.data.access_token);
      authStore.setUser(response.data.usuario);
      authStore.setLogin(true);
      $q.notify({
        type: 'positive',
        message: `Sesión iniciada. Bienvenido, ${response.data.usuario.nombre}.`,
        icon: 'check_circle',
      });
      router.push('/');
    })
    .catch((error) => {
      console.error('Error de login:', error);
      $q.notify({
        type: 'negative',
        message: 'Credenciales incorrectas. Verifique su usuario y contraseña.',
        icon: 'error_outline',
      });
    })
    .finally(() => {
      loading.value = false;
    });
};
</script>

<style scoped>
.login-wrapper {
  width: 100vw;
  height: 100vh;
  background: #f1f5f9;
  display: flex;
  justify-content: center;
  align-items: center;
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
}

.login-container {
  width: 100%;
  max-width: 420px;
  padding: 16px;
}

.login-card {
  background: #ffffff;
  border-radius: 16px;
  border: 1px solid #e2e8f0;
}

.logo-container {
  display: flex;
  justify-content: center;
  align-items: center;
}

.institution-logo {
  max-height: 110px;
  width: auto;
  object-fit: contain;
}

.input-label {
  font-size: 0.85rem;
  font-weight: 600;
  color: #334155;
  display: block;
  margin-bottom: 4px;
}

.forgot-link {
  color: #0284c7;
  font-size: 0.8rem;
  text-decoration: none;
}

.forgot-link:hover {
  text-decoration: underline;
}

.custom-input :deep(.q-field__control) {
  border-radius: 8px;
}

.submit-btn {
  border-radius: 8px;
}
</style>
