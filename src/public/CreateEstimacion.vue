<template>
  <div class="container fondo">
    <div class="estimacion-box" :class="{ 'blur-background': mostrarDetalle }">
      <button @click="goBack" class="btn-back">
        <i class="fas fa-arrow-left"></i> 
      </button>
      <div class="estimacion">
        <h1 class="title">Registro de Estimación</h1>
                  <div class="proyecto-fecha-container">
            <div class="contenedor-proyecto">
              <label class="texto">Proyecto</label>
              <div class="proyecto-input">
                <select v-model="estimacion.proyectoId" @change="cargarTiposPAM" :disabled="proyectoBloqueado" 
                  class="w-full p-2 border rounded input-standard">
                  <option v-for="proyecto in proyectos" :key="proyecto.proyectoId" :value="proyecto.proyectoId">
                    {{ proyecto.name }}
                  </option>
                </select>

                <div class="botones-proyecto">
                  <button @click="mostrarModalNuevoProyecto" class="btn-nuevoproy"></button>
                  <button @click="bloquearProyecto" class="Fijar-desfijar">
                    {{ proyectoBloqueado ? 'Desfijar' : 'Fijar' }}
                  </button>
                </div>
              </div>
            </div>

            <div class="fecha-container">
              <label class="texto">Fecha</label>
              <input type="date" :value="fecha" class="input-fecha" disabled />
            </div>
          </div>
    
          <div v-if="modalNuevoProyecto" class="detalle-overlay show">
          <div class="detalle-box">
            <button @click="cerrarModalNuevoProyecto" class="btn-close">&times;</button> <!-- 🔥 Botón X dentro del modal -->
            <h2 class="texto">Crear Nuevo Proyecto</h2>

            <div class="modal-content">
              <label class="cost-item"><strong>Nombre del Proyecto:</strong></label>
              <input v-model="nuevoProyecto.nombre" type="text" class="modal-input" placeholder="Ingrese el nombre del proyecto" />
            </div>

            <div class="modal-footer">
              <button class="btn-secondary" @click="guardarNuevoProyecto">Guardar</button>
              <button class="btn-secondary" @click="cerrarModalNuevoProyecto">Cancelar</button>
            </div>
          </div>
        </div>

        <div class="mb-4">
          <label class="texto">Identificador PAM</label>
          <div class="flex items-center">
            <input type="text" v-model="estimacion.codPam" :disabled="idPamBloqueado" class="w-full p-2 border rounded input-standard" placeholder="Ingrese el identificador del Pasivo Ambiental Minero" />
            <input type="checkbox" v-model="idPamBloqueado" @change="toggleIdPam" class="ml-2s">
            <span class="checkbox-text"> {{ idPamBloqueado ? 'No requiere' : 'Requiere' }} ID de PAM</span>                  </div>
        </div>
        <div class="mb-4 flex items-center gap-4">
        <div class="w-full">
          <label class="texto">Tipo de PAM</label>
          <select v-model="estimacion.tipoPamId" @change="cargarAtributos" class="w-full p-2 border rounded input-standard">
            <option disabled value="">Ingrese el Pasivo Ambiental Minero</option>
            <option v-for="tipoPam in tiposPAM" :key="tipoPam.id" :value="tipoPam.id">
              {{ tipoPam.name }}
            </option>
          </select>
        </div>
        
        <button v-if="atributos.length" @click="abrirModalAtributos" class="btn-secondary">
          {{ atributosIngresados ? "Ver Atributos" : "Ingresar Atributos" }}
        </button>
      </div>
      <p v-if="costoEstimado" class="flex items-center gap-2 text-lg font-semibold mt-4">
          Costo Estimado:  {{ formatNumero(costoEstimado.totalEstimado) }}
          <Eye @click="toggleDetalle" class="cursor-pointer text-green-600" size="24" />
        </p>
        <div class="flex justify-between mt-4">
          <button v-if="!estimacionGuardada" @click="guardarEstimacion" class="btn-primary">
            Calcular
          </button>
          <button @click="limpiarFormulario" class="btn-primary">
            Nuevo
          </button>
        </div>
    
      </div>
    </div>
    
    <div v-if="costoEstimado" class="detalle-overlay" :class="{ 'show': mostrarDetalle }">
      <div class="detalle-box">
        <p @click="toggleDetalle" class="X">
          X
        </p>
        <h2 class="text-lg font-semibold mb-4">Costo Estimado del PAM</h2>
        <div class="grid grid-cols-2 gap-4 costo-estimado-grid">
      <p class="costo-item"><strong>Costo Directo:</strong> {{ formatNumero(costoEstimado.costoDirecto) }}</p>
      <p class="costo-item"><strong>Gastos Generales:</strong> {{ formatNumero(costoEstimado.gastosGenerales) }}</p>
      <p class="costo-item"><strong>Utilidad:</strong> {{ formatNumero(costoEstimado.utilidades)}}</p>
      <p class="costo-item"><strong>Subtotal:</strong>  {{ formatNumero(costoEstimado.subTotal)}}</p>
      <p class="costo-item"><strong>IGV:</strong>  {{ formatNumero(costoEstimado.igv)}}</p>
      <p class="costo-item"><strong>Subtotal Obra:</strong>  {{ formatNumero(costoEstimado.subTotalObras) }}</p>
      <p class="costo-item"><strong>Expediente Técnico:</strong>  {{ formatNumero(costoEstimado.expedienteTecnico) }}</p>
      <p class="costo-item"><strong>Supervisión:</strong>  {{ formatNumero(costoEstimado.supervision) }}</p>
      <p class="costo-item"><strong>Gestión de Proyectos:</strong> {{ formatNumero(costoEstimado.gestionProyecto) }}</p>
      <p class="costo-item"><strong>Capacitación:</strong>  {{ formatNumero(costoEstimado.capacitacion) }}</p>
      <p class="costo-item"><strong>Contingencias:</strong>  {{ formatNumero(costoEstimado.contingencias) }}</p>
    </div>

    <div class="total-estimado-container">
      <p class="cost-item total-estimado"><strong>Total Estimado:</strong>  {{ formatNumero(costoEstimado.totalEstimado) }}</p>
    </div>

      </div>
    </div>
    <transition name="fade">
      <div v-if="error" class="alert-container">
        <p class="error-message">{{ error }}</p>
      </div>
    </transition>

    <div v-if="mostrarModalCoberturas" class="modal-overlay">
      <div class="modal-box">
        <button @click="cerrarModalCobertura" class="btn-close">&times;</button>
        <h2 class="texto">Seleccionar Tipo de Cobertura</h2>
        <div class="grid grid-cols-2 gap-4">
          <div v-for="(tipo, index) in tiposCobertura" :key="tipo.value" class="cobertura-option" @click="seleccionarCobertura(tipo.value)">
            <img :src="tipo.img" :alt="tipo.label" :class="{'cobertura-img': true, 'cobertura-img-small': index === 0}"/>
            <p>{{ tipo.label }}</p>
          </div>
        </div>
      </div>
    </div>

  </div>
  <div v-if="modalAtributos" class="detalle-overlay show">
  <div class="detalle-box">
    <button @click="cerrarModalAtributos" class="btn-close">&times;</button>
    <h2 class="texto">Atributos del PAM</h2>

    <div class="grid grid-cols-2 gap-4">
      <div v-for="atributo in atributos" :key="atributo.atributoPamId" class="mb-2">
        <label class="block text-sm font-medium">
          {{ obtenerDescripcionAtributo(atributo.nombre) }}
        </label>

        <template v-if="atributo.nombre === 'TipoCierre'">
          <select v-model="valoresAtributos[atributo.atributoPamId]" class="w-full p-2 border rounded input-standard" required>
            <option value="TRASLADO">TRASLADO</option>
            <option value="INSITU">INSITU</option>
          </select>
        </template>

        <template v-else-if="atributo.nombre === 'TipoCobertura'">
          <button @click="mostrarModalCobertura(atributo.atributoPamId)" class="w-full p-2 border rounded input-standard">
            Seleccionar
          </button>
          <p v-if="valoresAtributos[atributo.atributoPamId]">Tipo: {{ valoresAtributos[atributo.atributoPamId] }}</p>
        </template>

        <template v-else-if="atributo.tipoDato === 'bool'">
          <select v-model="valoresAtributos[atributo.atributoPamId]" class="w-full p-2 border rounded input-standard" required>
            <option :value="true">Sí</option>
            <option :value="false">No</option>
          </select>
        </template>

        <template v-else-if="atributo.tipoDato === 'decimal'">
          <input type="number" v-model="valoresAtributos[atributo.atributoPamId]" class="w-full p-2 border rounded input-standard" required />
        </template>

        <template v-else>
          <input type="text" v-model="valoresAtributos[atributo.atributoPamId]" class="w-full p-2 border rounded input-standard" required />
        </template>
      </div>
    </div>

    <div class="modal-footer">
      <button class="btn-secondary" @click="guardarAtributos">Grabar</button>
      <button class="btn-secondary" @click="cerrarModalAtributos">Cancelar</button>
    </div>
  </div>
</div>
</template>



<script>
import bdService from "@/main/services/bdservice";
import { Eye } from 'lucide-vue-next';

export default {
  components: {
    Eye,
  },
  data() {
    return {
      proyectos: [],
      tiposPAM: [],
      atributos: [],
      error: null,
      costosByProyecto: [],
      valoresAtributos: {},
      costoEstimado: null,
      totalProyecto: 0,
      fecha: new Date().toISOString().split("T")[0],
      proyectoBloqueado: false,
      modalNuevoProyecto: false,
      modalAtributos: false,
      nuevoProyecto: { nombre: "" },
      mostrarDetalle: false,
      idPamBloqueado: true,
      mostrarModalCoberturas: false,
      atributoPamIdSeleccionado: null,
      tiposCobertura: [
        { value: 'NINGUNA', label: 'Ninguna', img: require('@/assets/ninguna.png') },
        { value: 'I', label: 'Cobertura I', img: require('@/assets/tipo1.jpg') },
        { value: 'II', label: 'Cobertura II', img: require('@/assets/tipo2.jpg') },
        { value: 'III', label: 'Cobertura III', img: require('@/assets/tipo3.jpg') },
        { value: 'IV', label: 'Cobertura IV', img: require('@/assets/tipo4.jpg') }
      ],
      estimacion: {
        usuarioId: localStorage.getItem('idUser'),
        proyectoId: null,
        codPam: null,
        tipoPamId: null,
        valores: {}
      },
      atributoDescripciones: {
        Volumen: "Volumen (m³)",
        Área: "Área (m²)",
        GeneracionDAR: "¿Es Generador de Drenaje Ácido de Roca (DAR)?",
        TipoCierre: "Tipo de Cierre",
        TipoCobertura: "Tipo de Cobertura",
        Cobertura: '¿Requiere Cobertura?',
        DistanciaTraslado: "Distancia de Traslado (km)",
      },
      detalleEstimacion: {},
      estimacionGuardada: false, 
      atributosIngresados: false, 
    };
  },
  async created() {
    await this.cargarProyectos();
  },
  methods: {
    async goBack() {
      this.$router.go(-1);
    },
    async cargarProyectos() {
      this.proyectos = await bdService.getProyectos();
    },
    formatNumero(valor) {
      if (valor == null || isNaN(valor)) return 'S/ 0';
      const roundedValue = Math.round(valor);
      return `S/ ${Number(roundedValue).toLocaleString('es-PE')}`;
    },
    async cargarTiposPAM() {
      this.tiposPAM = await bdService.getTiposPAM();
    },
    toggleIdPam() {
      if (this.idPamBloqueado) {
        this.estimacion.codPam = null;
      }
    },
    async cargarCostosByProyectoId() {
      if (!this.estimacion.proyectoId) {
        console.warn("⚠️ No hay proyectoId definido");
        return;
      }
      try {
        const response = await bdService.getCostoTotalByProyectoId(this.estimacion.proyectoId);
        console.log("🔹 Respuesta de la API para totalCost:", response);
        if (response && response.totalCost !== undefined) {
          this.totalProyecto = response.totalCost;
        } else {
          console.warn("⚠️ La respuesta de la API no contiene totalCost");
          this.totalProyecto = 0;
        }
      } catch (error) {
        console.error("❌ Error al obtener el costo total del proyecto:", error);
        this.totalProyecto = 0;
      }
    },
    async guardarNuevoProyecto() {
      if (!this.nuevoProyecto.nombre.trim()) {
        alert("El nombre del proyecto no puede estar vacío.");
        return;
      }

      try {
        const nuevoProyecto = await bdService.createProyecto(this.nuevoProyecto.nombre);

        await this.cargarProyectos();

        this.estimacion.proyectoId = nuevoProyecto.proyectoId;

        this.cargarTiposPAM();
        this.cerrarModalNuevoProyecto();
      } catch (error) {
        alert("Error al crear el proyecto.");
      }
    },
    async cargarAtributos() {
  if (!this.estimacion.tipoPamId) return;
  try {
    const response = await bdService.getAtributosByTipoPamId(this.estimacion.tipoPamId);
    this.atributos = response || [];
    this.valoresAtributos = {};

    // Inicializa valores por defecto
    this.atributos.forEach(atributo => {
      if (atributo.tipoDato === "decimal") {
        this.valoresAtributos[atributo.atributoPamId] = 0.0;
      } else if (atributo.tipoDato === "bool") {
        this.valoresAtributos[atributo.atributoPamId] = false;
      } else {
        this.valoresAtributos[atributo.atributoPamId] = "";
      }
    });

    console.log("Atributos cargados:", this.atributos);
    console.log("Valores asignados:", this.valoresAtributos);

    // Abrir automáticamente el modal de atributos solo si no ha ingresado valores antes
    if (this.atributos.length > 0 && !this.atributosIngresados) {
      this.modalAtributos = true;
    }
  } catch (error) {
    console.error("Error al cargar los atributos:", error);
  }
},
    abrirModalAtributos() {
      this.modalAtributos = true;
    },
    cerrarModalAtributos() {
      this.modalAtributos = false;
    },
    guardarAtributos() {
      this.atributosIngresados = true;
      this.cerrarModalAtributos();
    },
    mostrarModalNuevoProyecto() {
      this.modalNuevoProyecto = true;
    },
    cerrarModalNuevoProyecto() {
      this.modalNuevoProyecto = false;
      this.nuevoProyecto.nombre = ""; 
    },
    bloquearProyecto() {
      this.proyectoBloqueado = !this.proyectoBloqueado;
    },
    limpiarFormulario() {
      this.estimacion.codPam = null;
      this.estimacion.tipoPamId = null;
      this.atributos = [];
      this.valoresAtributos = {};
      this.costoEstimado = null;
      this.estimacionGuardada = false; 
    },
    toggleDetalle() {
      this.mostrarDetalle = !this.mostrarDetalle;
    },
    async guardarEstimacion() {
      const storedUserId = localStorage.getItem("idUser");
      if (!storedUserId) {
        this.error = "No se encontró un usuario autenticado.";
        this.clearErrorAfterTimeout();
        return;
      }
      this.estimacion.usuarioId = storedUserId;
      this.estimacion.codPam = this.estimacion.codPam ? this.estimacion.codPam.toString() : "0";
      this.estimacion.tipoPam = this.detalleEstimacion;
      this.estimacion.valores = {};
      let valid = true;
      this.atributos.forEach(atributo => {
        const valor = this.valoresAtributos[atributo.atributoPamId];
        if (atributo.tipoDato !== 'bool' && !valor) {
          valid = false;
        }
        this.estimacion.valores[parseInt(atributo.atributoPamId)] = String(valor);
      });
      if (!valid) {
        this.error = "Todos los atributos son obligatorios.";
        this.clearErrorAfterTimeout();
        return;
      }
      console.log("Datos enviados al backend:", JSON.stringify(this.estimacion, null, 2));
      try {
        const response = await bdService.createEstimacion(this.estimacion);
        if (response && response.costoEstimado) {
          this.costoEstimado = response.costoEstimado;
          console.log("Total Estimado:", this.costoEstimado.totalEstimado);
          this.mostrarDetalle = true; 
          this.estimacionGuardada = true; 
          console.warn("No se recibieron costos estimados en la respuesta.");
        }
        await this.cargarCostosByProyectoId();
      } catch (error) {
        console.error("Error al crear la estimación:", error.response ? error.response.data : error);
        this.error = "Error al crear estimación";
        this.clearErrorAfterTimeout();
      }
    },
    clearErrorAfterTimeout() {
      setTimeout(() => {
        this.error = null;
      }, 4000); // 4 segundos
    },
    obtenerDescripcionAtributo(nombre) {
      return this.atributoDescripciones[nombre] || nombre;
    },
    mostrarModalCobertura(atributoPamId) {
      this.cerrarModalAtributos(); 
    this.atributoPamIdSeleccionado = atributoPamId;
    this.mostrarModalCoberturas = true;
    },
    cerrarModalCobertura() {
      this.mostrarModalCoberturas = false;
      this.abrirModalAtributos(); 

    },
    seleccionarCobertura(value) {
      this.valoresAtributos[this.atributoPamIdSeleccionado] = value;
      this.cerrarModalCobertura();
    }
  }
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap');

*, html, body {
  margin: 0;
  padding: 0;


}



.alert-container {
  position: absolute;
  top: 20px;
  right: 20px;
  background: rgba(255, 0, 0, 0.8);
  padding: 10px 20px;
  border-radius: 5px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  z-index: 1000;
  transition: opacity 0.5s ease-in-out;
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter, .fade-leave-to  {
  opacity: 0;
}

.error-message {
  color: white;
  font-size: 0.9rem;
}
body {
  font-family: 'Poppins', sans-serif;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(to bottom right, #53704b, #7ba58d, #4fd87d);
}

.texto{
  font-size: 20px;
  margin-bottom: 8px;
}


.container {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
}

.fondo {
  background: url("@/assets/senora-carrusel-ai-brush-removebg-vm9tw8af.png") no-repeat center center fixed;
  background-size: 100%; /* Reduce el tamaño al 80% del contenedor */
  background-position: bottom center; /* Ajusta la posición para que no se pierda la alineación */
  background-attachment: fixed;
}

.blur-background {
  filter: blur(5px);
}

.btn-floating .icon-eye {
  margin-left: 5px;
  font-size: 1.2rem;
}

.design {
  position: absolute;
  width: 100%;
  height: 100%;
  overflow: hidden;
  z-index: 0; 
}

.btn-primary {
  padding: 14px 40px;
  color: white;
  font-size: 1.4rem;
  background: #74c905;
  border: none;
  border-radius: 30px;
  cursor: pointer;
  transition: transform 0.3s;
  margin-left: 10px;
}

.btn-secondary {
  padding: 10px 20px;
  color: white;
  font-size: 1rem;
  background: #13863a;
  border: none;
  border-radius: 30px;
  cursor: pointer;
  transition: transform 0.3s;
  margin-top: 10px;
}
.btn-nuevoproy {
  width: 40px;
  height: 40px;
  background: url('@/assets/addproy.png') no-repeat center center;
  background-size: contain;
  border: none;
  cursor: pointer;
  flex-shrink: 0;
  position: relative;
}
.btn-nuevoproy::after {
  content: "Crear nuevo proyecto";
  visibility: hidden;
  opacity: 0;
  background: rgba(0, 0, 0, 0.8);
  color: white;
  padding: 6px 10px;
  border-radius: 5px;
  position: absolute;
  bottom: -35px; 
  left: 50%;
  transform: translateX(-50%);
  white-space: nowrap;
  font-size: 0.8rem;
  transition: opacity 0.3s ease-in-out, visibility 0.3s ease-in-out;
  z-index: 1000; 
}


.btn-nuevoproy:hover::after {
  visibility: visible;
  opacity: 1;
}

.Fijar-desfijar {
  width: 70px;
  height: 40px;
  padding: 6px 12px;
  color: white;
  font-size: 0.9rem;
  background: #13863a;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  transition: transform 0.3s;
  flex-shrink: 0;
}
.botones-proyecto {
  display: flex;
  align-items: center;
  gap: 10px; /* Espacio entre los botones */
}

.btn-secondary:hover {
  transform: scale(1.1);
}

.btn-floating {
  padding: 10px 20px;
  color: white;
  font-size: 1rem;
  background: #13863a;
  border: none;
  border-radius: 30px;
  cursor: pointer;
  transition: transform 0.3s;
  margin-top: 10px;
}

.X {
    position: relative;
    float: right;
    background: none;
    border: none;
    cursor: pointer;
    color: #13863a;
    font-size: 1.5rem;
    transition: transform 0.3s;
}

.btn-floating:hover {
  transform: scale(1.1);
}

.btn-back {
  background: none;
  border: none;
  cursor: pointer;
  color: #13863a;
  font-size: 1.5rem;
  transition: transform 0.3s;
  position: absolute;

}



.btn-back:hover {
  transform: scale(1.1);
}

.btn-close {
  position: absolute;
  top: 10px;
  right: 10px;
  background: none;
  border: none;
  cursor: pointer;
  color: #13863a;
  font-size: 1.5rem;
  transition: transform 0.3s;
}

.btn-close:hover {
  transform: scale(1.1);
}

.estimacion-box {
  background: rgba(255, 255, 255, 0.9);
  padding: 50px;
  border-radius: 15px;
  box-shadow: 0 0 25px 12px rgb(0 0 0 / 30%);
  z-index: 1;
  width: 80%;
  position: relative;
  max-width: 1200px;
}

.estimacion {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.estimacion h1.title {
  margin: 20px 0;
  font-size: 2rem;
}

.mb-4 {
  width: 100%;
  margin-bottom: 1rem;
}

.block {
  display: block;
}

.text-sm {
  
  font-size: 0.875rem;
}

.font-medium {
  font-weight: 500;
}

.w-full {
  width: 100%;
}

.p-2 {
  padding: 0.5rem;
}

.border {
  border: 1px solid #ccc;
}

.rounded {
  border-radius: 0.375rem;
}

.bg-gray-200 {
  background-color: #edf2f7;
}

.bg-blue-500 {
  background-color: #4299e1;
}

.text-white {
  color: #fff;
}

.px-6 {
  padding-left: 1.5rem;
  padding-right: 1.5rem;
}

.py-3 {
  padding-top: 0.75rem;
  padding-bottom: 0.75rem;
}

.rounded-lg {
  border-radius: 0.5rem;
}

.shadow-md {
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.hover\:bg-blue-600:hover {
  background-color: #3182ce;
}

.transition {
  transition: all 0.2s;
}

.mt-8 {
  margin-top: 2rem;
}

.p-4 {
  padding: 1rem;
}

.bg-white {
  background-color: #fff;
}

.grid {
  display: grid;
}

.grid-cols-2 {
  grid-template-columns: repeat(2, minmax(0, 1fr));
}

.gap-4 {
  gap: 1rem;
}

.text-lg {
  font-size: 1.125rem;
}

.font-semibold {
  font-weight: 600;
}

.mb-4 {
  margin-bottom: 1rem;
}

.mt-4 {
  margin-top: 1rem;
}
.costo-estimado-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);  
  gap: 16px;
  padding: 16px;
}

.costo-item {
  font-size: 14px;
  line-height: 1.6;
  margin-bottom: 8px;
  color: #333;
}

.total-estimado-container {
  margin-top: 20px;
  padding: 16px;
  background-color: #f5f5f5; 
  border-radius: 8px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.total-estimado {
  font-weight: bold;
  font-size: 16px;
  color: #006400; 
}

.costo-item strong {
  font-weight: bold;
  color: #555;
}

.costo-item {
  border-bottom: 1px solid #ccc;
  padding-bottom: 8px;
}

@media (max-width: 600px) {
  .costo-estimado-grid {
    grid-template-columns: 1fr; 
  }
}
.text-blue-600 {
  color: #3182ce;
}

.detalle-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: rgba(0, 0, 0, 0.5);
  z-index: 1000;
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s, visibility 0.3s;
}

.detalle-overlay.show {
  opacity: 1;
  visibility: visible;
}

.detalle-box {
  background: white;
  padding: 25px;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.25); 
  z-index: 1001;
  max-width: 380px; 
  width: 90%;
  text-align: center;
  position: relative;
 
}
.input-standard {
  height: 40px;
  box-sizing: border-box;
}
.btn-close {
  position: absolute;
  top: 10px;
  right: 15px;
  background: none;
  border: none;
  cursor: pointer;
  color: #13863a;
  font-size: 1.5rem;
  transition: transform 0.3s;
}

.btn-close:hover {
  transform: scale(1.2);
}

.detalle-box {
  background: white;
  padding: 25px;
  border-radius: 10px;
  box-shadow: 0 4px 25px rgba(0, 0, 0, 0.3);
  z-index: 1001;
  max-width: 500px; 
  width: 95%;
  text-align: center;
  position: relative;
}

.detalle-overlay {
  position: fixed;  
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  display: flex; 
  justify-content: center;
  align-items: center;
  background: rgba(0, 0, 0, 0.5); 
  z-index: 1000; 
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.3s ease-in-out, visibility 0.3s ease-in-out;
}

.detalle-overlay.show {
  opacity: 1;
  visibility: visible;
}
.modal-content {
  margin-top: 10px;
  text-align: left;
}

.ml-2s {
  font-size: 10rem; 
}

.modal-content label {
  display: block;
  font-size: 0.9rem;
  font-weight: 600;
  margin-bottom: 5px;
  color: #333;
}

.modal-input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 0.9rem;
  outline: none;
}
.modal-footer {
  display: flex;
  justify-content: center; 
  gap: 15px; 
  margin-top: 15px;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: rgba(0, 0, 0, 0.5);
  z-index: 1000;
}

.modal-box {
  background: white;
  padding: 25px;
  border-radius: 10px;
  box-shadow: 0 4px 25px rgba(0, 0, 0, 0.3);
  z-index: 1001;
  max-width: 500px;
  width: 95%;
  text-align: center;
  position: relative;
}

.btn-close {
  position: absolute;
  top: 10px;
  right: 15px;
  background: none;
  border: none;
  cursor: pointer;
  color: #13863a;
  font-size: 1.5rem;
  transition: transform 0.3s;
}
.mb-2 {
  margin-bottom: 0.5rem;
}
.checkbox-text {
  margin-left: 3px;
  font-size: 0.8rem; 
}
.btn-close:hover {
  transform: scale(1.2);
}

.cobertura-option {
  cursor: pointer;
  text-align: center;
}

.cobertura-img {
  width: 100%;
  height: auto;
  border: 1px solid #ccc;
  border-radius: 5px;
  transition: transform 0.3s;
}

.cobertura-img-small {
  width: 80%; 
  height: 80%;
  border: 1px solid #ccc;
  border-radius: 5px;
  transition: transform 0.3s;
}
.cobertura-img:hover {
  transform: scale(1.05);
}


.proyecto-fecha-container {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 20px;
  width: 100%;
  border-bottom: 2px solid rgb(118, 177, 126); /* Línea negra de separación */
  padding-bottom: 10px; /* Espaciado para evitar que quede pegado */
  margin-bottom: 15px; /* Espaciado con el siguiente bloque */
}

.contenedor-proyecto {
  display: flex;
  flex-direction: column;
  width: 50%;
  font-family: sans-serif;
}
.fecha-container {
  display: flex;
  flex-direction: column;
  width: 30%;
  text-align: center;
}
.proyecto-input {
  display: flex;
  align-items: center;
  gap: 10px;
}

.input-standard {
  width: 100%;
}

.input-fecha {
  width: 100%;
  padding: 8px;
  border: 1px solid #ffffff;
  border-radius: 5px;
  background-color: #ffffff;
  text-align: center;
  
}
.input-standard{
  font-size: 16px;
  font-family: poppins, sans-serif;

}

</style>