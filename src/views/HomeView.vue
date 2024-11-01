<template>
  <div class="container-fluid mt-4 text-center p-3">
    <div class="row justify-content-center mt-4">
      <div class="col-auto my-2">
        <input
          type="text"
          class="form-control"
          v-model="despesaForm.descricao"
          placeholder="Descrição do item"
        />
      </div>
      <div class="col-auto my-2">
        <input
          type="number"
          class="form-control"
          v-model="despesaForm.quantidade"
          placeholder="Quantidade/Unidades"
        />
      </div>
      <div class="col-auto my-2">
        <input
          type="number"
          class="form-control"
          v-model="despesaForm.valor"
          placeholder="Valor"
        />
      </div>
    </div>
    <div class="row justify-content-center mt-2 mb-4">
      <div class="col-12 col-md-3 col-xl-2 my-2">
        <div class="form-floating">
          <select
            id="origemSelect"
            class="form-select"
            aria-label="BRL"
            v-model="despesaForm.moedaOrigem"
          >
            <option value="BRL">BRL</option>
            <option value="USD">USD</option>
            <option value="EUR">EUR</option>
          </select>
          <label for="origemSelect">Moeda de Origem</label>
        </div>
      </div>
      <div class="col-1 align-center">
        <arows />
      </div>
      <div class="col-12 col-md-3 col-xl-2 my-2">
        <div class="form-floating">
          <select
            id="destinoSelect"
            class="form-select"
            aria-label="BRL"
            v-model="despesaForm.moedaDestino"
          >
            <option value="BRL">BRL</option>
            <option value="USD">USD</option>
            <option value="EUR">EUR</option>
          </select>
          <label for="destinoSelect">Moeda de Destino</label>
        </div>
      </div>
    </div>
    <div class="row justify-content-center">
      <div class="col-auto">
        <button
          type="buttom"
          @click="isEditing ? atualizaDespesa() : adicionarDespesa()"
          class="btn btn-primary m-4"
        >
          {{ isEditing ? 'Atualizar' : 'Adicionar' }}
        </button>
      </div>
    </div>

    <div
      class="row py-2 justify-content-center border border-light-subtle"
      v-for="(despesa, index) in despesas"
      :key="despesa.id"
    >
      <div class="col-10 col-md-8 col-lg-6">
        {{
          `${despesa.descricao} (${despesa.quantidade}): ${despesa.valor} ${despesa.moedaOrigem} => ${despesa.valorConvertido} ${despesa.moedaDestino}`
        }}
      </div>
      <div class="col-1">
        <button
          @click="executarAcao('edit', despesa, index)"
          type="button"
          class="btn btn-outline-info btn-sm"
        >
          <edit />
        </button>
      </div>
      <div class="col-1">
        <button
          @click="executarAcao('del', despesa, index)"
          type="button"
          class="btn btn-outline-danger btn-sm"
        >
          <trash />
        </button>
      </div>
    </div>

    <div class="row justify-content-center mt-4">
      <h5>Total (Moeda de Origem): {{ totalOrigem.toFixed(2) }}</h5>
      <h5>Total (Moeda de Destino): {{ totalDestino.toFixed(2) }}</h5>
    </div>
  </div>
</template>

<script setup>
import { computed, ref } from 'vue'
import arows from '../components/icons/arows.vue'
import edit from '../components/icons/edit.vue'
import trash from '../components/icons/trash.vue'

const isEditing = ref(false)
const despesas = ref([])
const despesaForm = ref({
  id: '',
  descricao: '',
  quantidade: null,
  valor: null,
  valorConvertido: null,
  moedaOrigem: '',
  moedaDestino: '',
})

const totalOrigem = computed(() => {
  let tot = 0
  despesas.value.forEach(element => {
    tot += element.quantidade * element.valor
  })
  return tot
})
const totalDestino = computed(() => {
  let tot = 0
  despesas.value.forEach(element => {
    tot += parseFloat(element.valorConvertido)
  })
  return tot
})

async function atualizaDespesa() {
  if (validaDadosForm()) {
    let conv = await realizarConversao(
      despesaForm.value.moedaOrigem,
      despesaForm.value.moedaDestino
    )
    despesaForm.value.valorConvertido = (
      despesaForm.value.quantidade *
      despesaForm.value.valor *
      conv
    ).toFixed(2)

    let index = despesas.value.findIndex(
      item => item.id === despesaForm.value.id
    )
    despesas.value[index] = despesaForm.value
    isEditing.value = false
    despesaForm.value = {}
  } else {
    alert('Por favor, preencha todos os campos!')
  }
}

async function adicionarDespesa() {
  if (validaDadosForm()) {
    let conv = await realizarConversao(
      despesaForm.value.moedaOrigem,
      despesaForm.value.moedaDestino
    )
    despesaForm.value.valorConvertido = (
      despesaForm.value.quantidade *
      despesaForm.value.valor *
      conv
    ).toFixed(2)

    despesaForm.value.id = Date.now()
    despesas.value.push(despesaForm.value)
    despesaForm.value = {}
  } else {
    alert('Por favor, preencha todos os campos!')
  }
}

function executarAcao(tipo, data, index) {
  if (tipo == 'edit') {
    despesaForm.value = {
      id: data.id,
      descricao: data.descricao,
      quantidade: data.quantidade,
      valor: data.valor,
      valorConvertido: data.valorConvertido,
      moedaOrigem: data.moedaOrigem,
      moedaDestino: data.moedaDestino,
    }
    isEditing.value = true
  } else if (tipo == 'del') {
    if (confirm('Deseja remover a despesa?')) {
      despesas.value.splice(index, 1)
    }
  }
}

async function realizarConversao(origem, destino) {
  const response = await fetch(
    'https://api.exchangerate-api.com/v4/latest/' + origem
  )
  let ret = await response.json()
  return ret.rates[destino]
}

function validaDadosForm() {
  let valid = true
  for (let item in despesaForm.value) {
    if (
      item != 'id' &&
      item != 'valorConvertido' &&
      (!despesaForm.value[item] || despesaForm.value[item] == '')
    ) {
      valid = false
    }
  }
  return valid
}
</script>

<style scoped>
</style>
