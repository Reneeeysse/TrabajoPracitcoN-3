<script setup>
import { ref } from 'vue';
import Formulario from '../components/Formulario-datos-pasajero.vue';
import FormularioDatosAvion from '@/components/formulario-datos-avion.vue';
import Pagodevuelo from '@/components/pago-de-vuel.vue';

const currentForm = ref(0);
const showForms = ref(false);

const avanzarFormulario = () => {
  if (currentForm.value === 0) {
    currentForm.value = 1;
  } else if (currentForm.value === 1) {
    currentForm.value = 2;
  } else if (currentForm.value === 2) {
    currentForm.value = 3;
  }
};

const reservar = () => {
  showForms.value = true;
  currentForm.value = 1;
};
</script>

<template>
  <div>
    <!-- Header visible siempre -->
    <div class="container">
      <header class="header">
        <h1>Bienvenido a Aerolíneas SSJ</h1>
        <p v-if="!showForms">
          Presione el siguiente botón para reservar un vuelo
        </p>
      </header>

      <button v-if="!showForms" @click="reservar" class="submit-btn">Reservar</button>
    </div>

    <transition name="fade">
      <div v-if="showForms && currentForm === 1" key="form1">
        <Formulario @formularioCompletado="avanzarFormulario" />
      </div>
    </transition>

    <transition name="fade">
      <div v-if="showForms && currentForm === 2" key="form2">
        <FormularioDatosAvion @formularioCompletado="avanzarFormulario" />
      </div>
    </transition>

    <transition name="fade">
      <div v-if="showForms && currentForm === 3" key="form3">
        <Pagodevuelo />
      </div>
    </transition>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.header {
  font-size: 2rem;
  margin-bottom: 20px;
  color: #333;
}

.submit-btn {
  padding: 12px;
  background-color: #007BFF;
  color: white;
  border: none;
  border-radius: 5px;
  font-size: 1.1rem;
  cursor: pointer;
  transition: background-color 0.3s;
  margin-bottom: 20px;
}

.submit-btn:hover {
  background-color: #0056b3;
}

.submit-btn:focus {
  outline: none;
}
</style>
