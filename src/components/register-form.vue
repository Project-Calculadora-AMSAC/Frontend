<template>
  <div class="container fondo">
    <div class="register-box">
      <div class="register">
        <img src="@/assets/cropped-logo-amsac.png" alt="Logo Activos Mineros" class="logo">
        <h3 class="title">ACTIVOS MINEROS</h3>
          <form @submit.prevent="handleRegister">
          <div class="text-input">
            <i class="ri-user-fill"></i>
            <input id="email" v-model="formData.email" type="text" placeholder="usuario@dominio" required>
          </div>
          <div class="text-input">
            <i class="ri-lock-fill"></i>
            <input class="icx" id="password" v-model="formData.password" type="password" placeholder="Contraseña" required>
          </div>
          <div class="text-input">
            <i class="ri-user-fill"></i>
            <input id="area" v-model="formData.registerArea" type="text" placeholder="Area / Invitado" required>
            <button type="button" @click="setAreaInvitado" class="btn-invitado">Invitado</button>
          </div>
          <button type="submit" class="register-btn">Registrarse</button>
          <div class="create">
          <a @click.prevent="goToLogin" href="#">¿Ya tienes cuenta? Inicia Sesion</a>
          <i class="ri-arrow-right-fill"></i>
        </div>
        </form>

        <transition name="fade">
  <div v-if="error" class="alert-container show">
    <p class="error-message">{{ error }}</p>
  </div>
</transition>       
      </div>
    </div>
  </div>
</template>

<script>
import userService from "@/main/services/userservice";

export default {
  name: "RegisterFormComponent",
  data() {
    return {
      formData: {
        email: "",
        password: "",
        registerArea: "", 
      },
      error: null,
    };
  },
  methods: {
    async handleRegister() {
      try {
        this.error = null; 
        await userService.registerUser(this.formData);
        console.log("Registration successful");
        this.$router.push("/login"); 
      } catch (err) {
        this.error =  "Error al registrase, intente nuevamente";
        this.clearErrorAfterTimeout();
      }
    },
    async goToLogin() {
      this.$router.push("/login");
    },
    clearErrorAfterTimeout() {
      setTimeout(() => {
        this.error = null;
      }, 4000); 
    },
    setAreaInvitado() {
      this.formData.registerArea = "Invitado";
    },
  },
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Nunito:wght@400;600;700&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@400;600;700&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
html, body {
  height: 100%;
  background-size: cover;
}

body {
  font-family: 'Poppins', sans-serif;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0;
  padding: 0;
  background: none; /* Quita cualquier fondo en body */
}

.logo {
  display: block;
  width: 100px;  /* Ajusta según el tamaño deseado */
  margin: 0 auto 10px; /* Centra y da espacio debajo */
  filter: drop-shadow(2px 2px 5px rgba(0, 0, 0, 0.2)); /* Sombra sutil */
}


.container {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center; /* Centrará el modal horizontalmente */
  align-items: center; /* Centrará el modal verticalmente */
  position: relative;
}

.fondo {
  background: url("https://www.amsac.pe/wp-content/uploads/2025/01/NP-Catalogo-de-especies-02-scaled.jpg") no-repeat center center fixed;
  background-size: cover;
  background-position: center;
  background-attachment: fixed;
}



.create i {
  color: #9a9a9a;
  margin-left: 12px;
}



.register-box {
  background: rgba(255, 255, 255);
  padding: 40px;
  border-radius: 15px;
  box-shadow: 0 0 25px 5px rgb(0 0 0 / 20%);
  z-index: 10;
  width: 350px;
  text-align: center;
}


.text-input {
  background: #e6e6e6;
  height: 45px;
  display: flex;
  width: 100%;
  align-items: center;
  border-radius: 10px;
  margin-bottom: 10px;
}

.register h3.title {
  margin: 20px 0;
  font-size: 2.4rem; /* Ajusta según necesites */
  font-weight: 10; /* Negrita */
  letter-spacing: 1px; /* Para mejorar legibilidad */
  text-transform: uppercase; /* Para mantener mayúsculas */

  font-family: 'Nunito', sans-serif; /* Asegura la fuente */
}

.text-input input {
  background: none;
  border: none;
  outline: none;
  width: 100%;
  height: 100%;
  margin-left: 10px;
  font-size: 1rem;
}



.btn-invitado {
  right: 10px;
  background: #74c905;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 5px 10px;
  cursor: pointer;
  transition: transform 0.3s;
}

.btn-invitado:hover {
  transform: scale(1.1);
}

::placeholder {
  color: #9a9a9a;
  font-size: 1.2rem;
}

.register-btn {
  padding: 12px 60px;
  color: #ffffff;
  font-size: 1.2rem;
  background: rgba(62, 189, 36, 0.75);
  border: none;
  border-radius: 15px;
  cursor: pointer;
  margin-top: 15px;
  transition: transform 0.3s;
}


.register-btn:hover {
  transform: scale(1.05);
}

a.forgot {
  margin-top: 15px;
}

.create {
  margin-top: 15px;
}

.create a {
  font-size: 14px;
  color: #555;
  cursor: pointer;
  text-decoration: none;
}
.create i {
  color: #9a9a9a;
  margin-left: 12px;
}


.alert-container {
  position: fixed;  /* Fija el mensaje en la pantalla */
  top: 20px;
  right: 20px;
  background: rgba(255, 0, 0, 0.9); /* Color rojo más visible */
  padding: 12px 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
  z-index: 9999; /* Asegura que esté por encima de otros elementos */
  transition: opacity 0.3s ease-in-out, transform 0.3s ease-in-out;
  opacity: 0; /* Inicia oculto */
  transform: translateY(-10px); /* Pequeña animación */
}

.alert-container.show {
  opacity: 1;  /* Se muestra cuando hay un error */
  transform: translateY(0);
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter, .fade-leave-to {
  opacity: 0;
}

.error-message {
  color: white;
  font-size: 0.9rem;
}
</style>