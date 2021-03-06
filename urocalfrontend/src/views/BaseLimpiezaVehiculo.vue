<template>
  <v-container fluid>
    <!-- Dialog para registrar nuevo LimpiezaVehiculo -->
    <DialogNuevoLimpiezaVehiculo
      ref="DialogNuevoLimpiezaVehiculo"
    ></DialogNuevoLimpiezaVehiculo>

    <!-- Tarjeta que contiene la caja de búsqueda, tabla y botón de agregar -->
    <v-card elevation="0" class="mt-5">
      <v-card-title class="py-2">
        <v-row no-gutters justify-md="space-between">
          <v-col cols="12" md="6">
            <div
              :class="[`text-h4`, `mb-4`]"
              class="transition-swing primary--text"
              v-text="nombre"
            ></div>
          </v-col>
          <v-col cols="12" md="6">
            <v-text-field
              v-model="buscarLimpiezaVehiculo"
              append-icon="mdi-magnify"
              label="Buscar"
              class="custom"
              filled
              dense
            ></v-text-field>
          </v-col>
        </v-row>
      </v-card-title>

      <v-card-text>
        <v-data-table
          :height="tablaResponsiva()"
          :headers="cabeceraTablaLimpiezaVehiculos"
          sort-by="id_lote"
          :items="listaLimpiezaVehiculos"
          :search="buscarLimpiezaVehiculo"
          class="elevation-1"
        >
          <template v-slot:top>
            <DialogEditarLimpiezaVehiculo
              ref="DialogEditarLimpiezaVehiculo"
            ></DialogEditarLimpiezaVehiculo>
          </template>

          <template v-slot:item.actions="{ item }">
            <v-icon color="primary" @click="cargarDialogEditarLimpiezaVehiculo(item)"
              >mdi-eye</v-icon
            >
          </template>
        </v-data-table>
      </v-card-text>

      <v-card-actions class="justify-center">
        <v-btn
          :block="$vuetify.breakpoint.xs ? true : false"
          width="300px"
          large elevation="0"
          color="primary"
          @click="cargarDialogNuevoLimpiezaVehiculo()"
          >Nuevo LimpiezaVehiculo</v-btn
        >
      </v-card-actions>
    </v-card>
  </v-container>
</template>

<script>
import { mapMutations, mapState } from "vuex";

import DialogNuevoLimpiezaVehiculo from "../components/DialogNuevoLimpiezaVehiculo";
import DialogEditarLimpiezaVehiculo from "../components/DialogEditarLimpiezaVehiculo";
import { autenticacionMixin, myMixin } from "@/mixins/MyMixin"; // Instancia al mixin de autenticacion
import ServicioLimpiezaVehiculo from '../services/ServicioLimpiezaVehiculo';
import ServicioVehiculo from '../services/ServicioVehiculo';

export default {
  name: "BaseLimpiezaVehiculo",

  components: {
    DialogNuevoLimpiezaVehiculo,
    DialogEditarLimpiezaVehiculo,
  },

  data() {

    return {
      nombre: "Limpieza de Vehículo",
      buscarLimpiezaVehiculo: "", // Guarda el texto de búsqueda
      cabeceraTablaLimpiezaVehiculos: [
        // Detalla las cabeceras de la tabla (Los campos más relevantes)
        {
          text: "Fecha",
          value: "limvehfecha",
          align: "center",
          sortable: false,
          class: "grey lighten-3",
        },
        {
          text: "Producto utilizado",
          value: "limvehproductoutilizado",
          sortable: false,
          align: "center",
          class: "grey lighten-3",
        },
        {
          text: "Escobillon",
          value: "limvehescobillon",
          sortable: false,
          align: "center",
          class: "grey lighten-3",
        },
        {
          text: "Escoba",
          value: "limvehescoba",
          sortable: false,
          align: "center",
          class: "grey lighten-3",
        },
        {
          text: "Agua",
          value: "limvehagua",
          sortable: false,
          align: "center",
          class: "grey lighten-3",
        },
        {
          text: "Aspiradora",
          value: "limvehaspiradora",
          sortable: false,
          align: "center",
          class: "grey lighten-3",
        },
                {
          text: "Finca",
          value: "finnombrefinca",
          align: "center",
          sortable: false,
          class: "grey lighten-3",
        },
        {
          text: "Vehiculo",
          value: "vehplaca",
          sortable: false,
          align: "center",
          class: "grey lighten-3",
        },
        {
          text: "Detalles",
          value: "actions",
          sortable: false,
          align: "center",
          class: "grey lighten-3",
        },
      ],
    listaLimpiezaVehiculos: this.$store.getters["moduloLimpiezaVehiculo/listaLimpiezaVehiculoStore"],
    };
  },

  computed: {

//...mapState("moduloLimpiezaVehiculo", ["limpieza_vehiculo"]),   // Modulo LimpiezaVehiculo
    /* Obtiene y establece el estado de la variable dialogNuevoLimpiezaVehiculo
    que muestra u oculta el dialogo*/
    dialogNuevoLimpiezaVehiculo: {
      get() {
        return this.$store.getters["gestionDialogos/dialogNuevoLimpiezaVehiculo"];
      },
      set(v) {
        return this.$store.commit("gestionDialogos/toggleDialogNuevoLimpiezaVehiculo", v);
      },
    },
    listaVehiculoStore: {
      get() {
        return this.$store.getters["moduloLimpiezaVehiculo/listavehiculoStore"];
      },
      set(v) {
        return this.$store.commit("moduloLimpiezaVehiculo/nuevoListaVehiculoStore", v);
      },
    },


    /* Obtiene y modifica el estado de la variable dialogTabMostrarLote
    que muestra u oculta el dialogo*/
    dialogEditarLimpiezaVehiculo: {
      get() {
        return this.$store.getters["gestionDialogos/dialogEditarLimpiezaVehiculo"];
      },
      set(v) {
        return this.$store.commit(
          "gestionDialogos/toggleDialogEditarLimpiezaVehiculo",
          v
        );
      },
    },
  },

     mounted()
  {
    this.cargarListaVehiculo();

  },

  methods: {
    // Vacia el modelo LimpiezaVehiculo
    ...mapMutations("moduloLimpiezaVehiculo", ["vaciarLimpiezaVehiculo"]),

    // Dialogo Nuevo LimpiezaVehiculo
    cargarDialogNuevoLimpiezaVehiculo() {
      this.dialogNuevoLimpiezaVehiculo = !this.dialogNuevoLimpiezaVehiculo; // Abre el DialogNuevoLimpiezaVehiculo
      this.vaciarLimpiezaVehiculo();
    },
    async cargarListaVehiculoPlaca(){
      let limpiezavehiculo=this.$store.getters["moduloLimpiezaVehiculo/limpieza_vehiculo"];
            
      let respuesta = await ServicioVehiculo.obtenerVehiculoFinca(limpiezavehiculo.fincaid);  // Obtener respuesta de backend
      console.log(respuesta);
      this.listaVehiculoStore = await respuesta.data;     
    },

    conversion(params){
      let resultado=false;
      if(params=="Si"){
        resultado=true;
      }else if(params=="No"){
        resultado=false;
      }
      return resultado;
    },
    async cargarListaVehiculo()
    {
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

    // Dialogo Editar LimpiezaVehiculo
        // Dialogo Editar LimpiezaVehiculo
    cargarDialogEditarLimpiezaVehiculo(item) {
      this.dialogEditarLimpiezaVehiculo = !this.dialogEditarLimpiezaVehiculo;
      this.vaciarLimpiezaVehiculo();
      //this.limpiezaVehiculo=item;
      this.$store.commit("moduloLimpiezaVehiculo/nuevoLimpiezaVehiculo", {limpiezavehiculoid:item.limpiezavehiculoid,
    limvehfecha:item.limvehfecha,
    limvehproductoutilizado:item.limvehproductoutilizado, 
    limvehescobillon:this.conversion(item.limvehescobillon),
    limvehescoba:this.conversion(item.limvehescoba),
    limvehagua:this.conversion(item.limvehagua),
    limvehaspiradora:this.conversion(item.limvehaspiradora),
    vehiculoid:item.vehiculoid,
    vehplaca:item.vehplaca,
    fincaid: item.fincaid,
    finnombrefinca: item.finnombrefinca,});
    this.cargarListaVehiculoPlaca();
    },
  },

  mixins: [autenticacionMixin, myMixin],

  created() {
    let usuario = JSON.parse(localStorage.getItem("usuario"));
    if (usuario.rol === "Administrador")
      this.$store.commit("colocarLayout", "LayoutAdministrador");
    if (usuario.rol === "Productor")
      this.$store.commit("colocarLayout", "LayoutProductor");
  },
};
</script>
