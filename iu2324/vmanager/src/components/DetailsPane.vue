<script setup>
import { resolve, VmState, turnOnAll, turnOffAll } from '../model.js'

defineEmits(['editVm', 'filterVm', 'rmVm', 'editGroup', 'filterGroup', 'rmGroup', 'setState'])

const props = defineProps({
  element: Object
})

function list(state) {
  return props.element.members
    .map(v=>resolve(v))
    .filter(vm=>state ? vm.state == state : true)
    .map(o=>o.name).join(' ');
}
</script>

<template>
  <div v-if="element == null || element.id == -1">
    (selecciona una Vm o un grupo para ver sus detalles)
  </div>
  <div v-else-if="Array.isArray(element.groups)">
    <h4>máquina virtual <span class="name">{{element.name}}</span></h4>

    <table>
      <tr>
        <th>Estado</th>
        <td>{{ element.state }} </td>
      </tr>
      <tr>
        <th>Memoria</th>
        <td>{{ element.ram }} Gb</td>
      </tr>
      <tr>
        <th>Disco</th>
        <td>{{ element.hd }} Gb</td>
      </tr>
      <tr>
        <th>Máximo uso de CPU</th>
        <td>{{ element.cpu }} %</td>
      </tr>
      <tr>
        <th>Núcleos de CPU</th>
        <td>{{ element.cores }}</td>
      </tr>
      <tr>
        <th>Dirección IPv4</th>
        <td>{{ element.ip }}</td>
      </tr>
      <tr>
        <th>Ancho de banda máximo</th>
        <td>{{ element.up }} Kbps de subida<br>{{ element.down }} Kbps de bajada</td>
      </tr>
      <tr>
        <th>Disco externo virtual</th>
        <td v-if="element.iso != -1">{{ resolve(element.iso).name }}</td>
        <td v-else> (ninguno) </td>
      </tr>
      <tr>
        <th>Grupos a los que pertenece</th>
        <td v-if="element.groups.length">
          {{ element.groups.map(g => resolve(g).name).join(' ') }}
        </td>
        <td v-else> (ninguno) </td>
      </tr>
    </table>
  
    <h5>Acciones</h5>
<div class="btn-group">
  <button @click="$emit('editVm')" class="btn btn-outline-success" title="Editar Máquina Virtual" :disabled="element.state === VmState.RUNNING  || element.state === VmState.SUSPENDED">✏️</button>
  <button v-if="element.groups.length" class="btn btn-outline-warning" @click="$emit('filterVm')" title="Filtrar Máquina Virtual">🔬</button>
  <button v-if="element.state !== VmState.RUNNING" class="btn btn-outline-success" @click="$emit('setState', VmState.RUNNING)" title="Iniciar Máquina Virtual">▶</button>
  <button v-if="element.state !== VmState.SUSPENDED" class="btn btn-outline-secondary" @click="$emit('setState', VmState.SUSPENDED)" title="Suspender Máquina Virtual">💤</button>
  <button v-if="element.state !== VmState.STOPPED" class="btn btn-outline-secondary" @click="$emit('setState', VmState.STOPPED)" title="Detener Máquina Virtual">🛑</button>
  <button @click="confirmDeleteVm" class="btn btn-outline-danger" title="Eliminar Máquina Virtual">🗑️</button>
</div>



    </div>

  <div v-else>
    <h4>grupo <span class="name">{{element.name}}</span></h4>

    <b>{{ element.members.length }} integrantes</b>
    <table>
      <tr>
        <th>{{ element.members.length }} integrantes</th>
        <td v-if="element.members.length">{{ list(false) }}
        </td>
        <td v-else> (no hay) </td>
      </tr>
      <tr>
        <th>Encendidas</th>
        <td v-if="list(VmState.RUNNING).length" style="color: green;">{{ list(VmState.RUNNING) }}</td>
        <td v-else> (no hay) </td>
      </tr>
      <tr>
        <th>Suspendidas</th>
        <td v-if="list(VmState.SUSPENDED).length" style="color: orange;">{{ list(VmState.SUSPENDED) }}</td>
        <td v-else> (no hay) </td>
      </tr>
      <tr>
        <th>Apagadas</th>
        <td v-if="list(VmState.STOPPED).length" style="color: red;">{{ list(VmState.STOPPED) }}</td>
        <td v-else> (no hay) </td>
      </tr>
    </table>

    <h5>Acciones</h5>
    <div class="btn-group">
      <button @click="$emit('editGroup')" class="btn btn-outline-success" title="Editar Grupo">✏️</button>
      <button @click="$emit('filterGroup')" class="btn btn-outline-warning" title="Filtrar Grupo">🔬</button>
      <button @click="confirmDeleteG" class="btn btn-outline-danger" title="Eliminar Grupo">🗑️</button>
      <button @click="turnOnAllMachines" class="btn btn-outline-success" title="Encender todas las maquinas del grupo">▶</button>
      <button @click="turnOffAllMachines" class="btn btn-outline-danger" title="Apagar todas las maquinas del grupo">🛑</button>
      
    </div>
  </div>
</template>

<script>
export default {
  methods: {
    confirmDeleteVm() {
      const confirmed = window.confirm('¿Estás seguro de que deseas eliminar esta máquina?');

      if (confirmed) {
        // Emitir el evento para eliminar el grupo
        this.$emit('rmVM');
      }
    },
    confirmDeleteG() {
      const confirmed = window.confirm('¿Estás seguro de que deseas eliminar este Grupo?');

      if (confirmed) {
        // Emitir el evento para eliminar el grupo
        this.$emit('rmGroup');
      }
    },
    turnOnAllMachines(){

      turnOnAll(this.element.id);
      this.$forceUpdate(); //Asi no se hace, pero funciona

    },
  
    turnOffAllMachines(){

      turnOffAll(this.element.id);
      this.$forceUpdate(); //Asi no se hace, pero funciona

    }
  }
  
};
</script>



<style scoped>
  tr>th {
    width: 10em;
    text-align: right;
  }
  .name {
    font-weight: 1000;
  }
  td, th {
    padding: 4px;
  }
  h5 {
    margin-top: 1em;
  }

</style>
