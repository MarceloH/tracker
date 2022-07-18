<template>
  <Formulario @aoSalvarTarefa="adicionarTarefa" />
  <div class="lista">
    <Box v-if="listaEstaVazia">
      Você não está muito produtivo hoje
      <span class="has-text-wight-bold">:(</span>
    </Box>
    <div class="field">
      <p class="control has-icons-left has-icons-right">
        <input
          class="input"
          type="email"
          placeholder="Digite para filtrar"
          v-model="filtro"
        />
        <span class="icon is-small is-left">
          <i class="fas fa-search"></i>
        </span>
      </p>
    </div>
    <Tarefa
      v-for="(tarefa, index) in tarefas"
      :key="index"
      :tarefa="tarefa"
      @aoTarefaClicada="selecionarTarefa"
    />
    <Modal :mostrar="tarefaSelecionada != null">
      <template v-slot:cabecalho>
        <p class="modal-card-title">Modal title</p>
        <button @click="fecharModal" class="delete" aria-label="close"></button>
      </template>
      <template v-slot:corpo>
        <label for="descricaoDaTarefa" class="label"> Descrição </label>
        <input
          type="text"
          class="input"
          v-model="tarefaSelecionada.descricao"
          id="descricaoDaTarefa"
          name="descricaoDaTarefa"
        />
      </template>
      <template v-slot:rodape>
        <button @click="alterarTarefa" class="button is-success">
          Salvar alterações
        </button>
        <button @click="fecharModal" class="button">Cancelar</button>
      </template>
    </Modal>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, ref, watchEffect } from "vue";
import Formulario from "../components/Formulario.vue";
import Tarefa from "../components/Tarefa.vue";
import Box from "../components/Box.vue";
import Modal from "../components/Modal.vue";
import { useStore } from "@/store";
import {
  OBTER_PROJETOS,
  OBTER_TAREFAS,
  CADASTRAR_TAREFA,
  ALTERAR_TAREFA,
} from "@/store/tipo-acoes";
import { NOTIFICAR } from "@/store/tipo-mutacoes";
import { TipoNotificacao } from "@/interfaces/INotificacao";
import ITarefa from "@/interfaces/ITarefa";


export default defineComponent({
  name: "App",
  components: {
    Formulario,
    Tarefa,
    Box,
    Modal,
  },
  data() {
    return {
      tarefaSelecionada: null as ITarefa | null,
    };
  },
  computed: {
    listaEstaVazia(): boolean {
      return this.tarefas.length === 0;
    },
  },
  methods: {
    adicionarTarefa(tarefa: ITarefa): void {
      if (!tarefa.projeto) {
        this.store.commit(NOTIFICAR, {
          titulo: "Ops!",
          texto: "Selecione um projeto antes de finalizar a tarefa!",
          tipo: TipoNotificacao.ERRO,
        });
        return;
      }
      this.store.dispatch(CADASTRAR_TAREFA, tarefa);
    },
    selecionarTarefa(tarefa: ITarefa): void {
      this.tarefaSelecionada = tarefa;
    },
    fecharModal(): void {
      this.tarefaSelecionada = null;
    },
    alterarTarefa() {
      this.store
        .dispatch(ALTERAR_TAREFA, this.tarefaSelecionada)
        .then(() => this.fecharModal());
    },
  },
  setup() {
    const store = useStore();
    store.dispatch(OBTER_TAREFAS);
    store.dispatch(OBTER_PROJETOS);

    const filtro = ref("");
    // const tarefas = computed(() =>
    //   store.state.tarefa.tarefas.filter(
    //     (t) => !filtro.value || t.descricao.includes(filtro.value)
    //   )
    // );

    watchEffect(() => {
      store.dispatch(OBTER_TAREFAS, filtro.value);
    });

    return {
      tarefas: computed(() => store.state.tarefa.tarefas),
      store,
      filtro,
    };
  },
});
</script>
