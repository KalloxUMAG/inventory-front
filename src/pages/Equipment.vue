<template>
  <q-page padding>
    <div v-if="equipment != null" class="row justify-center bg-secondary">
      <q-card class="my-card" flat bordered>
        <q-item class="row justify-center">
          <div class="text-h5">
            {{ equipment.name }} - {{ equipment.serial_number }}
          </div>
        </q-item>
        <q-separator />
        <q-card-section horizontal>
          <!--Datos carrucel-->
          <q-card-section class="col-4">
            <Carousel v-if="equipment != null" :api_endpoint="img_api" />
          </q-card-section>
          <q-separator vertical />
          <!--Datos producto-->
          <q-card-section class="col">
            <div class="text-h5 q-mb-md text-center">Datos producto</div>
            <div class="row q-mb-xs">
              <div class="col-5 field-label text-right q-mr-md">Nombre:</div>
              <div class="col field-content q-ml-xs">{{ equipment.name }}</div>
            </div>
            <div class="row q-mb-xs">
              <div class="col-5 field-label text-right q-mr-md">
                Codigo serial:
              </div>
              <div class="col field-content q-ml-xs">
                {{ equipment.serial_number }}
              </div>
            </div>
            <div class="row q-mb-xs">
              <div class="col-5 field-label text-right q-mr-md">
                Inventario UMAG:
              </div>
              <div class="col field-content q-ml-xs">
                {{ equipment.umag_inventory_code }}
              </div>
            </div>
            <div class="row q-mb-xs">
              <div class="col-5 field-label text-right q-mr-md">Marca:</div>
              <div class="col field-content q-ml-xs">
                {{ equipment.brand_name }}
              </div>
            </div>
            <div class="row q-mb-xs">
              <div class="col-5 field-label text-right q-mr-md">Modelo:</div>
              <div class="col field-content q-ml-xs">
                {{ equipment.model_name }}
              </div>
            </div>
            <div class="row q-mb-xs">
              <div class="col-5 field-label text-right q-mr-md">
                Número de modelo:
              </div>
              <div class="col field-content q-ml-xs">
                {{ equipment.model_number }}
              </div>
            </div>
            <div class="row q-mb-xs">
              <div class="col-5 field-label text-right q-mr-md">
                Periodo de mantención:
              </div>
              <div
                v-if="equipment.maintenance_period == null"
                class="col field-content q-ml-xs"
              >
                No aplica
              </div>
              <div v-else class="col field-content q-ml-xs">
                Cada {{ equipment.maintenance_period }} meses
              </div>
            </div>
            <div class="row q-mb-xs">
              <div class="col-5 field-label text-right q-mr-md">
                Observación:
              </div>
              <div class="col field-content q-ml-xs">
                {{ equipment.observation }}
              </div>
            </div>
          </q-card-section>
        </q-card-section>
        <q-separator horizontal />
        <q-card-section horizontal class="row">
          <q-card-section class="col-4">
            <div class="text-h5 q-mb-md text-center">Datos Ubicación</div>
            <div class="row q-mb-xs">
              <div class="col-5 field-label text-right q-mr-md">Sala:</div>
              <div class="col field-content q-ml-xs">
                {{ equipment.room_name }}
              </div>
            </div>
            <div class="row q-mb-xs">
              <div class="col-5 field-label text-right q-mr-md">Unidad:</div>
              <div class="col field-content q-ml-xs">
                {{ equipment.unit_name }}
              </div>
            </div>
            <div class="row q-mb-xs">
              <div class="col-5 field-label text-right q-mr-md">Edificio:</div>
              <div class="col field-content q-ml-xs">
                {{ equipment.building_name }}
              </div>
            </div>
          </q-card-section>
          <q-separator />
          <!--Datos compra-->
          <q-card-section class="col">
            <div class="text-h5 q-mb-md text-center">Datos compra</div>
            <div class="row q-mb-xs">
              <div class="col-5 field-label text-right q-mr-md">Proveedor:</div>
              <div class="col field-content q-ml-xs">
                {{ equipment.supplier_name }}
              </div>
            </div>
            <div class="row q-mb-xs">
              <div class="col-5 field-label text-right q-mr-md">
                Fecha de recepción:
              </div>
              <div class="col field-content q-ml-xs">
                {{ equipment.reception_date }}
              </div>
            </div>
            <div class="row q-mb-xs">
              <div class="col-5 field-label text-right q-mr-md">
                Número factura:
              </div>
              <div class="col field-content q-ml-xs">
                {{ equipment.invoice_number }}
              </div>
            </div>
            <div class="row q-mb-xs">
              <div class="col-5 field-label text-right q-mr-md">Proyecto:</div>
              <div class="col field-content q-ml-xs" v-if="project != null">
                {{ project.project_name }}
              </div>
            </div>
            <div class="row q-mb-xs">
              <div class="col-5 field-label text-right q-mr-md">Etapa:</div>
              <div class="col field-content q-ml-xs" v-if="project != null">
                {{ project.stage_name }}
              </div>
            </div>
            <div class="row q-mb-xs">
              <div class="col-5 field-label text-right q-mr-md">
                Dueño proyecto:
              </div>
              <div
                class="col field-content q-ml-xs"
                v-if="project != null && project.project_owner != null"
              >
                {{ project.project_owner }}
              </div>
              <div class="col field-content q-ml-xs" v-else>Ninguno</div>
            </div>
          </q-card-section>
        </q-card-section>
        <q-card-section v-if="equipment.maintenance_period != null">
          <NoRedirectTable
            title="Mantenimientos"
            :columns="columns_maintenances"
            :rows="maintenances"
            :addFunction="addFunction"
          />
        </q-card-section>
      </q-card>
    </div>
  </q-page>
</template>

<script setup>
import { useRoute } from "vue-router";
import { computed, onMounted, ref } from "vue";
import axios from "axios";
import Carousel from "src/components/Carousel.vue";
import NoRedirectTable from "src/components/NoRedirectTable.vue";
import FormModal from "src/components/FormModal.vue";
import { useQuasar } from "quasar";

const $q = useQuasar();

const columns_maintenances = [
  {
    name: "date",
    align: "left",
    label: "Fecha",
    field: "date",
    sortable: true,
  },
  {
    name: "maintenance_type",
    align: "left",
    label: "Tipo mantenimiento",
    field: "maintenance_type",
    sortable: true,
  },
  {
    name: "observations",
    align: "left",
    label: "Observaciones",
    field: "observations",
    sortable: false,
  },
];

const content_loaded = ref(false);

const img_api = ref(null);
const equipment = ref(null);
const maintenances = ref([]);
const project = ref(null);

const route = useRoute();
const id = computed(() => route.params.id);
const query_equipment =
  "https://inventory-back-production.up.railway.app/api/equipments/" + id.value;
const query_maintenances =
  "https://inventory-back-production.up.railway.app/api/maintenances/" +
  id.value;
const query_projects =
  "https://inventory-back-production.up.railway.app/api/equipment_projects/" +
  id.value;

function getEquipment() {
  axios.get(query_equipment).then((response) => {
    equipment.value = response.data;
    img_api.value =
      "https://inventory-back-production.up.railway.app/api/equipments/image/" +
      equipment.value.id;
  });
}
function getMaintenances() {
  axios.get(query_maintenances).then((response) => {
    maintenances.value = response.data;
  });
}
function getProjects() {
  axios.get(query_projects).then((response) => (project.value = response.data));
}

function addFunction() {
  $q.dialog({
    component: FormModal,
    componentProps: {
      title: "Agregar mantenimiento",
      fields: [
        {
          label: "Fecha",
          type: "date",
          defaultvalue: null,
          rules: [(val) => (val && val != null) || "Este campo es obligatorio"],
        },
        {
          label: "Tipo de mantenimiento",
          type: "select",
          defaultvalue: null,
          options: [
            { id: "Programada", name: "Programada" },
            { id: "Correctiva", name: "Correctiva" },
          ],
          option_value: "id",
          option_label: "name",
          not_found_label: "No hay tipos de mantenimiento",
          rules: [(val) => (val && val != null) || "Este campo es obligatorio"],
        },
        {
          label: "Observaciones",
          type: "text",
          defaultvalue: null,
          autogrow: true,
          rules: [],
        },
      ],
    },
  })
    .onOk((data) => {
      const maintenance_data = {
        date: data[0],
        observations: data[2],
        maintenance_type: data[1],
        equiptment_id: equipment.value.id,
      };

      axios
        .post(
          "https://inventory-back-production.up.railway.app/api/maintenances",
          maintenance_data,
          {
            headers: {
              "Content-Type": "application/json",
            },
          }
        )
        .then((response) => getMaintenances());
    })
    .onCancel(() => {});
}

onMounted(() => {
  getEquipment();
  getMaintenances();
  getProjects();
  content_loaded.value = true;
});
</script>

<style scoped>
.my-card {
  width: 100%;
}
.field-label {
  font-size: 16px;
  font-weight: bold;
}

.field-content {
  font-size: 16px;
  font-weight: 500;
}

.q-page {
  color: #262626;
}
</style>
