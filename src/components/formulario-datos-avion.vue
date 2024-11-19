<script setup>
import { ref, computed ,reactive,watch } from 'vue';
import aeropuertos from '../../aeropuertos.json';
// Estados
const ciudadOrigen = ref('');
const ciudadDestino = ref('');
const aeropuertoOrigen = ref('');
const aeropuertoDestino = ref('');
const fechaSalida = ref('');
const fechaRegreso = ref('');
const claseVuelo = ref('');
const numeroBoletos = ref("");
const estado = reactive({
  boletos: { error: '', valid: false, touched: false},
  fechaSalida: { error: '', valid: false,  touched: false },
  fechaRegreso: { error: '', valid: false,  touched: false },
});

const props = defineProps({
  onFormularioCompletado: Function,
});

// Esados para habilitar o deshabilitar el boton
const esFormularioCompleto = computed(() => {
  return (
    estado.boletos.valid &&
    estado.fechaSalida.valid &&
    estado.fechaRegreso.valid
  );
});

// Agrupacion de aeropuertos por ciudad
const aeropuertosPorCiudad = computed(() => {
  const agrupados = {};
  aeropuertos.forEach((aeropuerto) => {
    if (!agrupados[aeropuerto.city]) {
      agrupados[aeropuerto.city] = [];
    }
    agrupados[aeropuerto.city].push(aeropuerto);
  });
  return agrupados;
});


const aeropuertosOrigen = computed(() => aeropuertosPorCiudad.value[ciudadOrigen.value] || []);
const aeropuertosDestino = computed(() => aeropuertosPorCiudad.value[ciudadDestino.value] || []);

// Validacion de fecha
const esFechaSalidaValida = computed(() => {
  const hoy = new Date();
  const salida = new Date(fechaSalida.value);
  return salida > hoy;
});

const esFechaRegresoValida = computed(() => {
  if (!fechaRegreso.value) return true
  const salida = new Date(fechaSalida.value);
  const regreso = new Date(fechaRegreso.value);
  return regreso > salida;
});

//Fecha de salida
watch(fechaSalida, () => {
  const hoy = new Date();
  const salida = new Date(fechaSalida.value);
  if (salida > hoy) {
    estado.fechaSalida.valid = true;
    estado.fechaSalida.error = '';
  } else {
    estado.fechaSalida.valid = false;
    estado.fechaSalida.error = 'La fecha de salida debe ser posterior a hoy.';
  }
});

// Fecah de regreso
watch([fechaSalida, fechaRegreso], () => {
  const salida = new Date(fechaSalida.value);
  const regreso = new Date(fechaRegreso.value);
  if (!fechaRegreso.value || regreso > salida) {
    estado.fechaRegreso.valid = true;
    estado.fechaRegreso.error = '';
  } else {
    estado.fechaRegreso.valid = false;
    estado.fechaRegreso.error = 'La fecha de regreso debe ser posterior a la de salida.';
  }
});


// Clases de vuelo
const clasesDeVuelo = ['Económica', 'Ejecutiva', 'Primera Clase'];

// Validacion de la cantidad de boletos
const validarboletos=() =>{
  if (numeroBoletos.value < 1 || numeroBoletos.value > 10) {
    estado.boletos.error = 'Minimo 1 maximo 10';
    estado.boletos.valid = false;
  } else {
    estado.boletos.error = '';
    estado.boletos.valid = true;
  }
}

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
      <h3 class="sub-title">Complete el formulario de vuelo</h3>
      <div class="form-group">
        <label for="ciudadOrigen">Ciudad de origen</label>
        <select id="ciudadOrigen" v-model="ciudadOrigen" class="select" required>
          <option value="" disabled>Selecciona una ciudad</option>
          <option v-for="(aeropuertos, ciudad) in aeropuertosPorCiudad" :key="ciudad" :value="ciudad">
            {{ ciudad }}
          </option>
        </select>
      </div>

      <div class="form-group" v-if="ciudadOrigen">
        <label for="aeropuertoOrigen">Aeropuerto de origen</label>
        <select id="aeropuertoOrigen" v-model="aeropuertoOrigen" class="select" required>
          <option value="" disabled>Selecciona un aeropuerto</option>
          <option v-for="aeropuerto in aeropuertosOrigen" :key="aeropuerto.code" :value="aeropuerto.name">
            {{ aeropuerto.name }}
          </option>
        </select>
      </div>

      <div class="form-group">
        <label for="ciudadDestino">Ciudad de destino</label>
        <select id="ciudadDestino" v-model="ciudadDestino" class="select" required>
          <option value="" disabled>Selecciona una ciudad</option>
          <option v-for="(aeropuertos, ciudad) in aeropuertosPorCiudad" :key="ciudad" :value="ciudad"
                  :disabled="ciudad === ciudadOrigen">
            {{ ciudad }}
          </option>
        </select>
      </div>

      <div class="form-group" v-if="ciudadDestino">
        <label for="aeropuertoDestino">Aeropuerto de destino</label>
        <select id="aeropuertoDestino" v-model="aeropuertoDestino" class="select" required>
          <option value="" disabled>Selecciona un aeropuerto</option>
          <option v-for="aeropuerto in aeropuertosDestino" :key="aeropuerto.code" :value="aeropuerto.name">
            {{ aeropuerto.name }}
          </option>
        </select>
      </div>

      <div class="form-group">
        <label for="fechaSalida">Fecha de salida</label>
        <input type="date" id="fechaSalida" v-model="fechaSalida" class="input" required />
        <p v-if="fechaSalida && !esFechaSalidaValida" class="text-error">
          La fecha de salida debe ser posterior a la fecha actual.
        </p>
      </div>

      <div class="form-group">
        <label for="fechaRegreso">Fecha de regreso (opcional)</label>
        <input type="date" id="fechaRegreso" v-model="fechaRegreso" class="input" />
        <p v-if="fechaRegreso && !esFechaRegresoValida" class="text-error">
          La fecha de regreso debe ser posterior a la fecha de salida.
        </p>
      </div>

      <div class="form-group">
        <label for="claseVuelo">Clase de vuelo</label>
        <select id="claseVuelo" v-model="claseVuelo" class="select" required>
          <option value="" disabled>Selecciona una clase</option>
          <option v-for="clase in clasesDeVuelo" :key="clase" :value="clase">
            {{ clase }}
          </option>
        </select>
      </div>

      <div class="form-group">
        <label for="numeroBoletos">Número de boletos</label>
        <input type="number" id="numeroBoletos" v-model="numeroBoletos"  :class="{ 'input-error': estado.boletos.error, 'input-success': estado.boletos.valid }"
        @input="validarboletos" />
        <p class="feedback" :class="{ 'text-error': estado.boletos.error, 'text-success': estado.boletos.valid }">
          {{ estado.boletos.error }}
        </p>
      </div>

      <button type="submit" class="submit-btn" :disabled="!esFormularioCompleto">Continuar</button>
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
  border-color: black;
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
}
</style>
