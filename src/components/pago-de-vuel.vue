<script setup>
import {ref,computed,reactive} from "vue";
import visaLogo from '../assets/visa.svg';
import mastercardLogo from '../assets/mastercard.svg';
import americanExpresslogo from '../assets/americanexpress.png';

//Estados
const isModalVisible = ref(false);
const numeroDeTarjeta =ref("");
const tipoDeTarjeta = ref("");
const logoTarjeta = ref("");
const fechaVencimineto =ref("");
const cvv =ref("");
const nombreTarjeta =ref("");
const estado = reactive({
  numeroDeTarjeta: { error: '', valid: false, touched: false },
  fechaVencimineto: { error: '', valid: false, touched: false },
  cvv: { error: '', valid: false, touched: false },
  nombreTarjeta: { error: '', valid: false, touched: false },
});

const props = defineProps({
  onFormularioCompletado: Function,
});
const esFormularioCompleto = computed(() => {
  return (
    estado.numeroDeTarjeta.valid &&
    estado.fechaVencimineto.valid &&
    estado.cvv.valid &&
    estado.nombreTarjeta.valid
  );
});

// Función para detectar el tipo de tarjeta
const detectarTipoDeTarjeta = (numeroDeTarjeta) => {
  const Visa = /^4/;
  const MasterCard = /^(5[1-5]|222[1-9]|22[3-9][0-9]|2[3-6][0-9]{2}|27[01][0-9]|2720)/;
  const americanExpress = /^(34|37)/;

  if (Visa.test(numeroDeTarjeta)) {
    return 'Visa';
  } else if (MasterCard.test(numeroDeTarjeta)) {
    return 'MasterCard';
  } else if (americanExpress.test(numeroDeTarjeta)) {
    return 'American Express';
  } else {
    return '';
  }
};

// validacion del numero de tarjeta
const validarNumeroDeTarjeta = () => {
  const tipo = detectarTipoDeTarjeta(numeroDeTarjeta.value);
  tipoDeTarjeta.value = tipo;
  const validardigitos = /^\d{16}$/;

  //Asignacion del logo dependiendo el tipo de tarjeta
  if (tipo === 'Visa') {
  logoTarjeta.value = visaLogo;
} else if (tipo === 'MasterCard') {
  logoTarjeta.value = mastercardLogo;
} else if (tipo === 'American Express') {
  logoTarjeta.value = americanExpresslogo;
}


  // Validar si el número tiene 16 dígitos
  if (!validardigitos.test(numeroDeTarjeta.value)) {
    estado.numeroDeTarjeta.error = 'Número de tarjeta incompleto o inválido.';
    estado.numeroDeTarjeta.valid = false;
  } else {
    estado.numeroDeTarjeta.error = '';
    estado.numeroDeTarjeta.valid = true;
  }
};

// Validacion de vencimiento de tarjeta
const validarVencimiento = () => {
  const venciTarj = /^\d{2}\/\d{2}$/;
  const fecha = fechaVencimineto.value;
  if (!venciTarj.test(fecha)) {
    estado.fechaVencimineto.error = 'La fecha debe tener el formato MM/AA.';
    estado.fechaVencimineto.valid = false;
    return;
  }
  const [mes, año] = fecha.split('/').map(Number);
  if (mes < 1 || mes > 12) {
    estado.fechaVencimineto.error = 'El mes debe estar entre 01 y 12.';
    estado.fechaVencimineto.valid = false;
    return;
  }
  const ahora = new Date();
  const añoActual = ahora.getFullYear() % 100;
  const mesActual = ahora.getMonth() + 1;

  if (año < añoActual || (año === añoActual && mes < mesActual)) {
    estado.fechaVencimineto.error = 'La fecha de vencimiento debe ser futura.';
    estado.fechaVencimineto.valid = false;
  } else {
    estado.fechaVencimineto.error = '';
    estado.fechaVencimineto.valid = true;
  }
};

// Validacion del CVV
const validarCvv =() => {
const cantidadDigitos=/^\d{3,4}$/;
if (!cantidadDigitos.test(cvv.value)) {
  estado.cvv.error = 'El CVV debe tener entre 3 y 4 dígitos numericos';
  estado.cvv.valid = false;
} else {
estado.cvv.error ="";
estado.cvv.valid = true;
}

}

// Validacion del nombre de la tarjeta
const validarNombreTarjeta = () => {
  if (!/^[a-zA-Z\s]+$/.test(nombreTarjeta.value)) {
    estado.nombreTarjeta.error = 'Solo letras y espacios';
    estado.nombreTarjeta.valid = false;
  } else {
    estado.nombreTarjeta.error = '';
    estado.nombreTarjeta.valid = true;
  }
};
// Función de formulario completado
const validarFormulario = () => {
  if (esFormularioCompleto.value) {
    isModalVisible.value = true;
    props.onFormularioCompletado();
  }
};

const cerrarModal = () => {
  isModalVisible.value = false;
  window.location.reload();
};
// Manejo de foco
const manejarFocus = (campo) => {
  if (!estado[campo].touched) {
    estado[campo].touched = true;
  }
};
</script>

<template>
  <div class="form-container">
    <form @submit.prevent="validarFormulario" class="form">
      <h3 class="sub-title">Ultimo paso para resevar su vuelo</h3>
      <!-- Numero de la tarjeta -->
      <div class="form-group">
        <label for="numeroDeTarjeta">Número de la tarjeta</label>
        <div class="input-container">
          <input type="text" id="numeroDeTarjeta" v-model="numeroDeTarjeta" placeholder="Ingrese el número de la tarjeta" class="input"
            :class="{
              'input-error': estado.numeroDeTarjeta.error && estado.numeroDeTarjeta.touched,
              'input-success': estado.numeroDeTarjeta.valid && estado.numeroDeTarjeta.touched,
            }" @focus="manejarFocus('numeroDeTarjeta')" @input="validarNumeroDeTarjeta" />
            <transition name="fade">
            <img v-if="logoTarjeta" :src="logoTarjeta" alt="Logo tarjeta" class="logo-tarjeta" />
          </transition>
        </div>
        <p class="feedback" :class="{ 'text-error': estado.numeroDeTarjeta.error }">
          {{ estado.numeroDeTarjeta.error }}
        </p>
      </div>

      <!-- Fecha de vencimiento-->
      <div class="form-group">
        <label for="fechaVencimineto">Fecha de vencimiento</label>
        <input type="text" id="fechaVencimineto" v-model="fechaVencimineto" placeholder="En formato MM/AA" class="input"
          :class="{ 'input-error': estado.fechaVencimineto.error && estado.fechaVencimineto.touched, 'input-success': estado.fechaVencimineto.valid && estado.fechaVencimineto.touched, }" @focus="manejarFocus('fechaVencimineto')" @input="validarVencimiento" />
        <p class="feedback" :class="{ 'text-error': estado.fechaVencimineto.error }">
          {{ estado.fechaVencimineto.error }}
        </p>
      </div>

      <!-- Ciudad de destino -->
      <div class="form-group">
        <label for="cvv">Ingrese CVV
        </label>
        <input type="text" id="cvv" v-model="cvv" placeholder="Ingrese el codigo de seguridad" class="input" :class="{
          'input-error': estado.cvv.error && estado.cvv.touched,
          'input-success': estado.cvv.valid && estado.cvv.touched,
        }" @focus="manejarFocus('cvv')" @input="validarCvv" />
        <p class="feedback" :class="{ 'text-error': estado.cvv.error }">
          {{ estado.cvv.error }}
        </p>
      </div>

      <!-- Aeropuerto de destino -->
      <div class="form-group">
        <label for="nombreTarjeta">Nombre de la tarjeta</label>
        <input type="text" id="nombreTarjeta" v-model="nombreTarjeta" placeholder="Ingrese el nombre de la tarjeta" class="input" :class="{
          'input-error': estado.nombreTarjeta.error && estado.nombreTarjeta.touched,
          'input-success': estado.nombreTarjeta.valid && estado.nombreTarjeta.touched,
        }" @focus="manejarFocus('nombreTarjeta')" @input="validarNombreTarjeta" />
        <p class="feedback" :class="{ 'text-error': estado.nombreTarjeta.error }">
          {{ estado.nombreTarjeta.error }}
        </p>
      </div>
      <button type="submit" class="submit-btn" :disabled="!esFormularioCompleto">Pagar</button>

      <div v-if="isModalVisible" class="modal-overlay">
      <div class="modal">
        <h3>  ¡Que tengas un buen viaje!</h3>
        <p>Se reservo tu vuelo correctamente. Se te llegara un correo con todos lo detalles de tu vuelo</p>
        <button class="submit-btn" @click="cerrarModal">OK</button>
      </div>
    </div>
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

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal {
  background-color: white;
  padding: 20px;
  border-radius: 5px;
  text-align: center;
  box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1);
}
.logo-tarjeta {
  width: 50px;
  height: auto;
  transition: opacity 0.5s ease-in-out;
}
.fade-enter-active, .fade-leave-active {
  transition: opacity 1s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
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
  border-color: red;
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
