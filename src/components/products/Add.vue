<template>
  <div v-if="usuario.rol.grado <=1">
    
    <div class="container-fluid mt-2">
      <!-- Page Heading -->
      <div class="d-sm-flex align-items-center justify-content-between mb-4">
        <h1 class="h3 mb-0 text-gray-800">Panel de Administración</h1>
        <router-link to="/productos" class="btn btn-primary"
          >Regresar</router-link
        >
      </div>

      <!-- Content Row -->
      <div class="row">
        <div class="col-lg-6 m-auto">
          <form autocomplete="off">
            <div class="form-group form-floating mb-3 ">
              <label>ubicacion</label>
              <select
               
                id="ubicacion"
                v-model="producto.ubicacion"
                name="proveedor"
                class="form-control"
              >
                <option
                  v-for="ubicaciones in ubicacion"
                  :key="ubicaciones"
                 
                  :value="ubicaciones"
                >
                  {{ ubicaciones }}
                </option>
              </select>
            </div>
            <div class="form-group form-floating mb-3 d-none">
              <label>Proveedor</label>
              <select
                :class="{ 'is-invalid': producto.proveedor_id === '' }"
                id="proveedor"
                v-model="producto.proveedor_id"
                name="proveedor"
                class="form-control"
              >
                <option
                  v-for="proveedor in proveedores"
                  :key="proveedor.id"
                  v-show="proveedor.status === true"
                  :value="proveedor._id"
                >
                  {{ proveedor.nombre }}
                </option>
              </select>
            </div>
            <div class="mb-2" v-for="(item, index) of form" :key="index">
              <label :for="item.valor">{{ item.valor }}</label>
              <input
                :class="{ 'is-invalid': producto[item.valor] === '' }"
                v-if="!item.number"
                v-model="producto[item.valor]"
                type="text"
                :placeholder="item.valor"
                :id="item.valor"
                class="form-control"
              />
              <input
                v-if="item.number"
                :class="{ 'is-invalid': producto[item.valor] === '' }"
                v-model.number="producto[item.valor]"
                type="Number"
                :placeholder="item.valor"
                :id="item.valor"
                class="form-control"
              />
            </div>
             
            <div  class="form-check mb-4 d-none"> 
<input  class="form-check-input " v-model="producto.iva" type="checkbox" id="iva">
              <label for="iva" class="form-check-label">iva 16%</label>
            </div>
            <button
              @click.prevent="sendProduct()"
              value=""
              class="btn btn-primary"
            >
              Guardar Producto
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
  <NoAccess v-else />
</template>

<script>
import { ref } from "@vue/reactivity";
import { useStore } from "vuex";
import { createToast } from "mosha-vue-toastify";
import { useRouter } from "vue-router";
import { computed } from "@vue/runtime-core";
import axios from "axios";
import NoAccess from '../403.vue'
export default {
  components:{NoAccess},
  setup() {
    const toask = computed(() => store.state.toask);
    const store = useStore();
    const router = useRouter();
    
    store.dispatch("getProveedores");
    const proveedores = computed(() => store.state.proveedores);
    let ubicacion = ["barquisimeto","valencia", "el vigia"]
    let token = computed(()=> store.state.token)
    const form = ref([
      { valor: "nombre" },
      //{ valor: "marca" },
   //   { valor: "modelo" },
      { valor: "descripcion" },
      { valor: "cantidad", number: true },
      //{ valor: "precio", number: true },
      { valor: "codigo" },
     
    ])
    let id = "";
    let producto = ref({
      proveedor_id: null,
      nombre: null,
    });
    const api = computed(() => store.state.api);
    const sendProduct = async function () {
      try {
        // if (!producto.value.proveedor_id) {
        //   producto.value.proveedor_id = "";
        //   return;
        // }
        // if (!producto.value.nombre) {
        //   producto.value.nombre = "";
        //   return;
        // }
        // if (!producto.value.marca) {
        //   producto.value.marca = "";
        //   return;
        // }
        // if (!producto.value.modelo) {
        //   producto.value.modelo = "";
        //   return;
        // }
        // if (!producto.value.descripcion) {
        //   producto.value.descripcion = "";
        //   return;
        // }
        // if (!producto.value.cantidad) {
        //   producto.value.cantidad = "";
        //   return;
        // }
        // if (!producto.value.precio) {
        //   producto.value.precio = "";
        //   return;
        // }
        // if (!producto.value.codigo) {
        //   producto.value.codigo = "";
        //   return;
        // }
        // if (!producto.value.iva) {
        //   producto.value.iva = 0
          
        // }
        // if (producto.value.iva) {
        //   producto.value.iva = 16
          
        // }
        const { data } = await axios.post(
          `${api.value}/productos/${id}`,
          producto.value, {headers:{xtoken:token.value}}
        );

        if (data.status === true) {
         // producto.value = { proveedor_id: null };
          router.push("/productos");
          id = "";
          createToast(data.value, toask.value.success);
          return;
        } else {
          createToast(data.value, toask.value.danger);
        }
      } catch (error) {
          createToast('no se pudo conectar al servidor');

      }
    };
    function buscarProduct() {
      try {
        let uri = window.location.href.split("?");
        if (uri.length === 2) {
          const { value } = computed(() => store.state.productos);

          if (value.length === 0) {
            router.push("/productos/add");
            return;
          }
          id = uri[1];
          const res = value.filter((item) => (item._id === id ? item : false));

          !res ? (producto.value = {}) : (producto.value = res[0], !res[0].iva?res[0].iva =false : res[0].iva = true  ,delete form.value.splice(4, 2) );
        }
      } catch (error) {
        0;
      }
    }
    buscarProduct();
    let usuario = computed(()=>store.state.usuario)
    return { producto, form, sendProduct, proveedores, token , usuario, ubicacion};
  },
};
</script>

