<script setup>

import VModal from './VModal.vue'
import RangeBox from './RangeBox.vue'
import TextBox from './TextBox.vue'
import IpBox from './IpBox.vue'
import MemberBox from './MemberBox.vue'
import SelectBox from './SelectBox.vue'

import * as M from '../model.js'
import { ref } from 'vue'

const emit = defineEmits(['add', 'edit'])

const props = defineProps({
  vm: Object,
  isAdd: Boolean, // otherwise, editing existing instead of adding
})

let modalRef = ref(null);

function setVm() {
  const vm = props.vm;
  const form = document.getElementById("editOrAddVm")
  const valueFor = (name) => {
    const input = form.querySelector(`input[name=${name}]`)
    if (!input) console.log("ERROR: no input for name", name, "in", form)
    return input.value
  }

  console.log("saving VM...", vm, form)

  // valida IP de forma manual
  for (let i = 0; i < 4; i++) {
    let input = form.querySelector(`input[name=e-ip-${i}]`)
    let o = input.value
    input.setCustomValidity(o >= 0 && o < 255 ? "" : "Expected an integer between 0 and 255")
  }
  const ip = [0, 1, 2, 3].map(n => valueFor(`e-ip-${n}`)).join(".")

  // comprueba validez de todos los campos, y sobreescribe resultado
  if ( ! form.checkValidity()) {
    // fuerza a que se muestren los errores simulando un envío
    // (pero como hay errores, no se va a enviar nada :-)
    form.querySelector("button[type=submit]").click()
    return; 
  }    

  // todo válido: lanza evento a padre, y cierra modal
  emit(props.isAdd ? 'add' : 'edit', new M.Vm(vm.id,
    valueFor("e-name"),
    valueFor("e-ram"), valueFor("e-hdd"),
    valueFor("e-cpu"), valueFor("e-cores"),
    ip,
    valueFor("e-up"), valueFor("e-down"),
    +valueFor("e-iso"), M.VmState.STOPPED, vm.disk, vm.memory, 
    JSON.parse(valueFor("e-groups"))
  ))
  modalRef.value.hide()    
}

// para que el padre pueda llamar a show (hide no debería hacer falta)
function show() {
  modalRef.value.show();
}
defineExpose({ show });
</script>

<template>
  <VModal ref="modalRef" id="vmAddOrEditModal"
    :title="isAdd ? 'Añadiendo VM' : `Editando VM: ${vm.name}`" >
    <template #body>
      <form id="editOrAddVm" 
        @submit.prevent="e => setVm()">
        <div class="container">
          <TextBox :start="vm.name" id="e-name" label="nombre" />
          <MemberBox :start="vm.groups" :all="M.getGroups()" id="e-groups" label="grupos" />
          <br>
         
        <div id="e-ram">
          <div class="range-box-container">
            <div class="range-box">
              <RangeBox :start="vm.ram" id="e-ram" label="ram" :min="0" :max="128" />
            </div>
          </div>
            <span class="units">Gb</span>
        </div>

        <div id="e-hdd">
          <div class="range-box-container">
            <div class="range-box">
              <RangeBox :start="vm.ram" id="e-hdd" label="hdd" :min="0" :max="4 * 1024" :step="256" />
            </div>
          </div>
            <span class="units">Gb</span>
        </div>

        <div id="e-cpu">
          <div class="range-box-container">
            <div class="range-box">
              <RangeBox :start="vm.ram" id="e-cpu" label="cpu" :min="0" :max="100"  />
            </div>
          </div>
            <span class="units">%   </span>
        </div>

        <div id="e-cores">
          <div class="range-box-container">
            <div class="range-box">
              <RangeBox :start="vm.ram" id="e-cores" label="cores" :min="1" :max="8"  />
            </div>
          </div>
            <span class="units">#</span>
        </div>

         
  
          
        <br>
        <div id="e-ip">
        <IpBox :start="vm.ip" id="e-ip"  />

        <span class="units">ip</span>
        </div>
          

          <div id="e-up">
          <div class="range-box-container">
            <div class="range-box">
              <RangeBox :start="vm.ram" id="e-up" label="up" :min="0" :max="1024 * 20"  />
            </div>
          </div>
            <span class="units">Kbps</span>
        </div>
        <div id="e-down">
          <div class="range-box-container">
            <div class="range-box">
              <RangeBox :start="vm.ram" id="e-down" label="down" :min="0" :max="1024 * 20"  />
            </div>
          </div>
            <span class="units">Kbps</span>
        </div>
         
          <br>
          <SelectBox :start="vm.iso" :all="M.getFiles()" id="e-iso" label="Fichero ISO" />
        </div>
        <button type="submit" class="invisible">Submit</button>
      </form>
    </template>
    <template #footer>
      <button @click.prevent="() => setVm()" class="btn btn-primary">
        {{ isAdd ? 'Añadir esta VM' : `Confirmar cambios a ${vm.name}` }}
      </button>
    </template>
  </VModal>
</template>

<style scoped>


#e-ip {
    display: flex;
    align-items: center;
    justify-content: center;
  }

#e-ram {
    display: flex;
    align-items: center;
    justify-content: center;
  }

  #e-ram .range-box-container {
    margin-right: 10px; /* Ajusta según sea necesario */
    align-items: center;
  }

  #e-ram .label {
    margin-right: 5px;
  }

  #e-hdd {
    display: flex;
    align-items: center;
    justify-content: center;
  }
  #e-hdd .range-box-container {
    margin-right: 10px; /* Ajusta según sea necesario */
    align-items: center;
  }
  #e-hdd .label {
    margin-right: 5px;
  }

  #e-cpu {
    display: flex;
    align-items: center;
    justify-content: center;
  }
  #e-cpu .range-box-container {
    margin-right: 10px; /* Ajusta según sea necesario */
    align-items: center;
  }
  #e-cpu .label {
    margin-right: 5px;
  }
  #e-cores {
    display: flex;
    align-items: center;
    justify-content: center;
  }
  #e-cores .range-box-container {
    margin-right: 10px; /* Ajusta según sea necesario */
    align-items: center;
  }
  #e-cores .label {
    margin-right: 5px;
  }

  #e-up {
    display: flex;
    align-items: center;
    justify-content: center;
  }
  #e-up .range-box-container {
    margin-right: 10px; /* Ajusta según sea necesario */
    align-items: center;
  }
  #e-up .label {
    margin-right: 5px;
  }

  #e-down {
    display: flex;
    align-items: center;
    justify-content: center;
  }
  #e-down .range-box-container {
    margin-right: 10px; /* Ajusta según sea necesario */
    align-items: center;
  }
  #e-down .label {
    margin-right: 5px;
  }
  .range-box {
    width: 300px; /* Ajusta el ancho según sea necesario */
    background-color: #e0e0e0;
    border: 1px solid #ccc;
    padding: 10px;
    border-radius: 5px;
  }
  
  #editOrAddVm{
     /* Cambia el color del texto en modo oscuro por defecto */
    background-color: #141c24; /* Cambia el fondo a un tono oscuro por defecto */
  }

  
</style>
