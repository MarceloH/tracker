<template>
  <div class="box formulario">
    <div class="columns">
      <div
        class="column is-5"
        role="form"
        aria-label="Formulario para criação de uma nova tarefa"
      >
        <input
          type="text"
          class="input"
          placeholder="Qual tarefa você deseja iniciar?"
          v-model="descricao"
        />
      </div>
      <div class="column is-3">
        <div class="select">
          <select v-model="idProjeto">
            <option value="">Selecione o projeto</option>
            <option
              :value="projeto.id"
              v-for="projeto in projetos"
              :key="projeto.id"
            >
              {{ projeto.nome }}
            </option>
          </select>
        </div>
      </div>
      <div class="column">
        <Temporizador @aoTemporizadorFinalizado="finalizarTarefa" />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, ref } from "vue";
import Temporizador from "./Temporizador.vue";
import { useStore } from "vuex";
import { key } from "@/store";

export default defineComponent({
  name: "FormularioCriacao",
  emits: ["aoSalvarTarefa"],
  components: {
    Temporizador,
  },
  setup(props, { emit }) {
    const store = useStore(key)
    
    const descricao = ref("")
    const idProjeto = ref("")

    const projetos = computed(() => store.state.projeto.projetos)

    const finalizarTarefa = (tempoEmSegundos: number): void => {
      emit("aoSalvarTarefa", {
        duracaoEmSegundos: tempoEmSegundos,
        descricao: descricao.value,
        projeto: projetos.value.find(projeto => projeto.id === idProjeto.value)
      })
      descricao.value = "";
    }

    return {
      descricao,
      idProjeto,
      projetos,
      finalizarTarefa,
    };
  },
});
</script>

<style>
.formulario {
  color: var(--texto-primario);
  background: var(--bg-primario);
}
</style>
