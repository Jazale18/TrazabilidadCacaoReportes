<template>
  <v-dialog v-model="dialogNuevoLimpiezaVehiculo" scrollable max-width="500px" transition="dialog-transition" :fullscreen="$vuetify.breakpoint.xs ? true : false">
    <v-card tile>
      <v-card-title class="justify-center white primary--text">
        <v-spacer></v-spacer>
        <h5>Nuevo Limpieza Vehiculo</h5>
        <v-spacer></v-spacer>
        <v-btn icon><v-icon class="primary--text" @click="cerrarDialogo()">mdi-close</v-icon></v-btn>
      </v-card-title>
      
      <v-card-text>
         <v-row>
          <v-col cols="12">
          <v-select
            v-model="fincaid"
            @input="cargarListaVehiculo"
            placeholder="Finca"
            class="style-chooser"
            label="findescripcionfinca"
            :reduce="(listaFinca) => listaFinca.fincaid"
            :options="listaFinca"
          >
            <template v-slot:no-options="{ search, searching }">
              <template v-if="searching">
                No hay resultados para <em>{{ search }}</em
                >.
              </template>
              <em style="opacity: 0.5" v-else>Empiece a escribir una Placa de Vehiculo</em>
            </template>
          </v-select>
            <v-text-field class="custom px-2" v-model="limpieza_vehiculo.limvehproductoutilizado" filled dense label="Producto utilizado"></v-text-field>
            <v-checkbox
              v-model="limpieza_vehiculo.limvehescobillon"
              label="Escobillon"
              color="success"
              value="true"
              hide-details
            ></v-checkbox>
            <v-checkbox
              v-model="limpieza_vehiculo.limvehescoba"
              label="Escoba"
              color="success"
              value="true"
              hide-details
            ></v-checkbox>
            <v-checkbox
              v-model="limpieza_vehiculo.limvehagua"
              label="Agua"
              color="success"
              value="true"
              hide-details
            ></v-checkbox>
            <v-checkbox
              v-model="limpieza_vehiculo.limvehaspiradora"
              label="Aspiradora"
              color="success"
              value="true"
              hide-details
            ></v-checkbox>
            <br>
            <v-select
            v-model="limpieza_vehiculo.vehiculoid"
            placeholder="Vehiculo"
            class="style-chooser"
            label="vehplaca"
            :reduce="(listaVehiculos) => listaVehiculos.vehiculoid"
            :options="listaVehiculos"
          >
            <template v-slot:no-options="{ search, searching }">
              <template v-if="searching">
                No hay resultados para <em>{{ search }}</em
                >.
              </template>
              <em style="opacity: 0.5" v-else>Empiece a escribir una Placa de Vehiculo</em>
            </template>
          </v-select>
            <v-menu v-model="menuMostrarCalendario" transition="scale-transition" offset-y max-width="290px" min-width="290px">
              <template v-slot:activator="{ on, attrs }">
                <v-text-field class="custom px-2" filled dense
                  label="Fecha" 
                  v-model="limpieza_vehiculo.limvehfecha" 
                  :rules="[reglas.campoVacio(limpieza_vehiculo.limvehfecha)]"
                  readonly
                  v-bind="attrs"
                  v-on="on">
                </v-text-field>
              </template>
              <v-date-picker v-model="limpieza_vehiculo.limvehfecha" @input="menuMostrarCalendario = false" :show-current="fechaActual" locale="es-419" ></v-date-picker>
            </v-menu>
          </v-col>
        </v-row>
      </v-card-text>
      <v-card-actions>
        <v-col>
        <!--<v-btn color="error" block @click="dialogNuevoLimpiezaVehiculo = !dialogNuevoLimpiezaVehiculo" >Cancelar</v-btn>-->
        <v-btn color="primary" large elevation="0" @click="guardarLimpiezaVehiculo()" width="300px" block class="mt-2">Guardar</v-btn>
        </v-col>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import { mapMutations, mapState } from "vuex";
import vSelect from "vue-select";
import "vue-select/dist/vue-select.css";
import ServicioFinca from '../services/ServicioFinca';
import ServicioVehiculo from '../services/ServicioVehiculo';
import ServicioLimpiezaVehiculo from '../services/ServicioLimpiezaVehiculo';


export default {
  name: "DialogNuevoLimpiezaVehiculo",
  components: {
    vSelect,
  },

  props: {},

  data() {
    return {
      fincaid:0,
      listaFinca:[],
      listaVehiculos:[],

      menuMostrarCalendario: "",
      fechaActual: new Date().toISOString().substr(0, 10), // Fecha actual
      itemsGenero: ['Masculino', 'Femenino'],
      itemsCiudades: ['Machala', 'Pasaje', 'Santa Rosa'],
    };
  },

  computed: {
    // Según el valor de la variable dialogNuevoLimpiezaVehiculo muestra u oculta el dialogo
    dialogNuevoLimpiezaVehiculo: {
      get() {
        return this.$store.getters["gestionDialogos/dialogNuevoLimpiezaVehiculo"];
      },
      set(v) {
        return this.$store.commit("gestionDialogos/toggleDialogNuevoLimpiezaVehiculo", v);
      },
    },

    ...mapState("moduloLimpiezaVehiculo", ["limpieza_vehiculo"]),   // Modulo LimpiezaVehiculo

    ...mapState("validacionForm", ["reglas"]),                  // Reglas de validacion
    
  },
  mounted() {
    
    this.obtenerTodosFincas();
  },

  methods: {
    cerrarDialogo(){
      this.dialogNuevoLimpiezaVehiculo = !this.dialogNuevoLimpiezaVehiculo;
      this.vaciarLimpiezaVehiculo;
    },
    async guardarLimpiezaVehiculo()
    {
            //console.log(this.limpieza_vehiculo)
      try {
        let respuesta = await ServicioLimpiezaVehiculo.agregarLimpiezaVehiculo(this.limpieza_vehiculo);  
        this.$toast.success(respuesta.data.message);
        this.cerrarDialogo();
        this.cargarListaLimpiezaVehiculo();
      } catch (error) {
        this.$toast.error(error.response.data.message);
      }
    },
    async cargarListaVehiculo(){
      let respuesta = await ServicioVehiculo.obtenerVehiculoFinca(this.fincaid);  // Obtener respuesta de backend
      this.listaVehiculos = await respuesta.data;     
    },
    async obtenerTodosFincas() {
      try {
      let respuesta=null;
      if(localStorage.getItem('productor')!==null){
        let usuariosesion=JSON.parse(localStorage.getItem('productor'));
        respuesta = await ServicioFinca.obtenerFincaPropietario(usuariosesion.productorid);


      }else{
        respuesta = await ServicioFinca.obtenerTodosFincas();
      }
        this.listaFinca = respuesta.data;
     
        
      } catch (error) {
        
        
      }
    },
    // #  MANIPULACIÓN DE DATOS  #
    async cargarListaLimpiezaVehiculo () {
      try {
        let respuesta=null;
      if(localStorage.getItem('productor')!==null){
        let usuariosesion=JSON.parse(localStorage.getItem('productor'));
        respuesta = await ServicioLimpiezaVehiculo.obtenerProductorLimpiezaVehiculo(usuariosesion.productorid);  // Obtener respuesta de backend


      }else{
        respuesta = await ServicioLimpiezaVehiculo.obtenerTodosLimpiezaVehiculo();  // Obtener respuesta de backend

      }
        let datosLimpiezaVehiculo = await respuesta.data;
        this.$store.commit("moduloLimpiezaVehiculo/vaciarLista",null);                                    // Rescatar datos de la respuesta
        datosLimpiezaVehiculo.forEach((LimpiezaVehiculo) => {                                  // Guardar cada registro en la 'lista de datos' 
        this.$store.commit("moduloLimpiezaVehiculo/addListaLimpiezaVe",LimpiezaVehiculo);
      });
        
      } catch (error) {
        
      } 
      
      
    },   
  }
};
</script>
