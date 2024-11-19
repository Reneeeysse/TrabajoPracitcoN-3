<script setup>
import paises from '../../paises.json';
import { ref, reactive,computed } from 'vue';

const props = defineProps({
  onFormularioCompletado: Function,
});

// Estados
const nombre = ref('');
const pasaporte = ref('');
const nacimiento = ref('');
const nacionalidad = ref('');
const estado = reactive({
  nombre: { error: '', valid: false, touched: false },
  pasaporte: { error: '', valid: false, touched: false },
  nacimiento: { error: '', valid: false, touched: false },
  nacionalidad: { error: '', valid: false, touched: false },
});

// Esados para habilitar o deshabilitar el boton
const esFormularioCompleto = computed(() => {
  return (
    estado.nombre.valid &&
    estado.pasaporte.valid &&
    estado.nacimiento.valid &&
    estado.nacionalidad.valid
  );
});

// Validacio de nombre
const validarNombre = () => {
  if (!/^[a-zA-Z\s]{3,}$/.test(nombre.value)) {
    estado.nombre.error = 'El nombre debe contener al menos 3 letras';
    estado.nombre.valid = false;
  } else {
    estado.nombre.error = '';
    estado.nombre.valid = true;
  }
};
// Validacio de pasaporte
const validarPasaporte = () => {
  if (!/^[A-Za-z0-9]{9}$/.test(pasaporte.value)) {
    estado.pasaporte.error = 'Número de pasaporte incompleto';
    estado.pasaporte.valid = false;
  } else {
    estado.pasaporte.error = '';
    estado.pasaporte.valid = true;
  }
};
// Validacio de nacimineto
const validarNacimiento = () => {
  const pattern = /^\d{2}\/\d{2}\/\d{4}$/;
  if (!pattern.test(nacimiento.value)) {
    estado.nacimiento.error = 'La fecha debe tener el formato dd/mm/yyyy';
    estado.nacimiento.valid = false;
    return;
  }

  const [dia, mes, año] = nacimiento.value.split('/').map(Number);
  const añoactual = 2024;
  const edad = añoactual - año;

  if (edad < 18 || (edad === 18 && (mes > 11 || (mes === 11 && dia > 15)))) {
    estado.nacimiento.error = 'Debes ser mayor de 18 años';
    estado.nacimiento.valid = false;
  } else {
    estado.nacimiento.error = '';
    estado.nacimiento.valid = true;
  }
};
// Validacio de nacionalidad
const validarNacionalidad = () => {
  if (!nacionalidad.value) {
    estado.nacionalidad.error = 'Debes seleccionar tu nacionalidad';
    estado.nacionalidad.valid = false;
  } else {
    estado.nacionalidad.error = '';
    estado.nacionalidad.valid = true;
  }
};

// Manejo de focus
const manejarFocus = (campo) => {
  if (!estado[campo].touched) {
    estado[campo].touched = true;
  }
};

// Función de formulario completado
const validarFormulario = () => {
  if (esFormularioCompleto.value) {
    props.onFormularioCompletado();
  }
};
</script>

<template>
  <div class="form-container">
    <form @submit.prevent="validarFormulario" class="form">
      <h3 class="sub-title">Complete el siguiente formulario</h3>

      <div class="form-group">
        <label for="nombre">Nombre completo</label>
        <input type="text" id="nombre" v-model="nombre" placeholder="Ingresa tu nombre completo" class="input" :class="{
          'input-error': estado.nombre.error && estado.nombre.touched,
          'input-success': estado.nombre.valid && estado.nombre.touched,
        }" @focus="manejarFocus('nombre')" @input="validarNombre" />
        <p class="feedback" :class="{ 'text-error': estado.nombre.error }">
          {{ estado.nombre.error }}
        </p>
      </div>

      <div class="form-group">
        <label for="pasaporte">Pasaporte</label>
        <input type="text" id="pasaporte" v-model="pasaporte" placeholder="Ingresa tu número de pasaporte" class="input"
          :class="{ 'input-error': estado.pasaporte.error && estado.pasaporte.touched,'input-success': estado.pasaporte.valid && estado.pasaporte.touched, }" @focus="manejarFocus('pasaporte')"  @input="validarPasaporte" />
        <p class="feedback" :class="{ 'text-error': estado.pasaporte.error }">
          {{ estado.pasaporte.error }}
        </p>
      </div>

      <div class="form-group">
        <label for="nacimiento">Fecha de nacimiento</label>
        <input type="text" id="nacimiento" v-model="nacimiento" placeholder="dd/mm/yyyy" class="input"
          :class="{ 'input-error': estado.nacimiento.error && estado.nacimiento.touched, 'input-success': estado.nacimiento.valid && estado.nacimiento.touched, }" @focus="manejarFocus('nacimiento')" @input="validarNacimiento" />
        <p class="feedback" :class="{ 'text-error': estado.nacimiento.error }">
          {{ estado.nacimiento.error }}
        </p>
      </div>

      <select id="nacionalidad" v-model="nacionalidad" class="select" @change="validarNacionalidad">
        <option value="" disabled>Selecciona tu nacionalidad</option>
        <option v-for="pais in paises" :key="pais.iso2" :value="pais.iso2">
          {{ pais.nameES }}
        </option>
      </select>
      <p class="feedback" :class="{ 'text-error': estado.nacionalidad.error }">
        {{ estado.nacionalidad.error }}
      </p>

      <button type="submit" class="submit-btn" :disabled="!esFormularioCompleto">
        Continuar
      </button>

    </form>
  </div>
</template>




<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans:ital,wght@0,100..900;1,100..900&display=swap');

* {
  font-family: "Noto Sans", serif;
  font-optical-sizing: auto;
  font-style: normal;
}

.error {
  color: red;
  font-size: 0.9rem;
  margin-top: 5px;
}

.form-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 60vh;
  font-family: 'Roboto', sans-serif;
}

.form-title {
  font-size: 2rem;
  margin-bottom: 30px;
  color: #333;
}

.sub-title {
  text-align: center;
}

.form {
  width: 100%;
  max-width: 500px;
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin-bottom: 20px;
}

label {
  font-size: 1rem;
  color: #333;
  font-weight: 500;
  margin-bottom: 8px;
  display: block;
}

.input-error {
  box-shadow: 0 0 5px red;
}

.input-success {
  box-shadow: 0 0 5px green;
  border-color: rgb(0, 250, 83);
}


.feedback {
  font-size: 0.9rem;
  margin-top: 5px;
}

.text-error {
  color: red;
}

.text-success {
  color: green;
}

.input,
.select {
  width: 95%;
  padding: 12px;
  border-radius: 5px;
  border: 1px solid #ddd;
  font-size: 1rem;
  color: #333;
  transition: border 0.3s;
}

.input:focus,
.select:focus {
  outline: none;
}

button.submit-btn {
  width: 100%;
  padding: 12px;
  background-color: #007BFF;
  color: white;
  border: none;
  border-radius: 5px;
  font-size: 1.1rem;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
</style>
