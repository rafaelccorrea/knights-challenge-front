<template>
  <div v-if="knights.length === 0" class="text-center">
    <strong class="color-descriptions">Nenhum cavaleiro encontrado!</strong>
  </div>
  <div v-else>
    <v-container fluid>
      <v-row justify="space-around">
        <v-col
          v-for="knight in knights"
          :key="knight.id"
          cols="12"
          md="6"
          lg="4"
          xl="3"
        >
          <v-card class="card-bottom-margin" style="padding-bottom: 20px">
            <v-img
              height="200"
              src="../assets/banner.jpg"
              cover
              class="text-white"
            >
              <v-toolbar color="rgba(0, 0, 0, 0)" theme="dark">
                <v-toolbar-title class="text-h6 color-name">
                  {{ knight.nickname }}
                </v-toolbar-title>

                <template v-slot:append>
                  <v-btn @click="editNickname(knight.id)" color="error" icon>
                    <v-icon>mdi-pencil</v-icon>
                  </v-btn>
                  <v-btn @click="deleteKnight(knight.id)" color="error" icon>
                    <v-icon>mdi-delete</v-icon>
                  </v-btn>
                </template>
              </v-toolbar>
            </v-img>
            <v-card-text>
              <v-row no-gutters>
                <v-col>
                  <v-sheet class="pa-2 ma-2"
                    >Idade:
                    <strong class="color-descriptions">{{ knight.age }}</strong>
                  </v-sheet>
                </v-col>
                <v-col>
                  <v-sheet class="pa-2 ma-2">
                    Qtd.Armas:
                    <strong class="color-descriptions">{{
                      knight.weapons
                    }}</strong>
                  </v-sheet>
                </v-col>
              </v-row>
              <v-row no-gutters>
                <v-col>
                  <v-sheet class="pa-3 ma-2">
                    Atributo:
                    <strong class="color-descriptions">{{
                      attributeMap[knight.attribute]
                    }}</strong>
                  </v-sheet>
                </v-col>
                <v-col>
                  <v-sheet class="pa-2 ma-2">
                    Ataque:
                    <strong class="color-descriptions">{{
                      knight.attack
                    }}</strong>
                  </v-sheet>
                </v-col>
              </v-row>
              <v-row no-gutters>
                <v-col>
                  <v-sheet class="pa-2 ma-2"> Exp:</v-sheet>
                </v-col>
              </v-row>
              <v-progress-linear
                max="8500"
                v-model="knight.exp"
                color="red"
                height="25"
              >
                <template v-slot:default="{ value }">
                  <strong>{{ Math.ceil(value) }}%</strong>
                </template>
              </v-progress-linear>
            </v-card-text>
            <v-card-actions>
              <v-btn text @click="showDetailsModal(knight)"> Detalhes </v-btn>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
      <v-dialog v-model="modalOpen" max-width="500">
        <v-card>
          <v-card-title class="headline">Editar Nickname</v-card-title>
          <v-card-text>
            <v-form>
              <v-text-field
                v-model="tempNickname"
                label="Nickname"
                maxlength="8"
              ></v-text-field>
            </v-form>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" text @click="modalOpen = false"
              >Cancelar</v-btn
            >
            <v-btn color="blue darken-1" text @click="saveNickname()"
              >Salvar</v-btn
            >
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-container>

    <v-dialog v-model="confirmDeleteKnightId" max-width="500">
      <v-card>
        <v-card-title class="headline">Confirmar exclusão</v-card-title>
        <v-card-text>
          Tem certeza de que deseja excluir este cavaleiro?
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            color="blue darken-1"
            text
            @click="confirmDeleteKnightId = null"
          >
            Cancelar
          </v-btn>
          <v-btn color="red darken-1" text @click="deleteConfirmed()">
            Excluir
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="confirmModal" max-width="500">
      <v-card>
        <v-card-title class="headline">Confirmação de edição</v-card-title>
        <v-card-text>
          Tem certeza que deseja editar o apelido do cavaleiro?
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="confirmModal = false">
            Cancelar
          </v-btn>
          <v-btn color="blue darken-1" text @click="saveNickname">
            Salvar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog
      v-model="detailsModalOpen"
      max-width="500"
      transition="scale-transition"
    >
      <v-card class="custom-card">
        <v-card-title class="headline">Detalhes do Cavaleiro</v-card-title>
        <v-card-text>
          <v-row>
            <v-col cols="12" md="6">
              <p><strong>Apelido:</strong> {{ selectedKnight.nickname }}</p>
              <p><strong>Idade:</strong> {{ selectedKnight.age }}</p>
              <p><strong>Armas:</strong> {{ selectedKnight.weapons }}</p>
            </v-col>
            <v-col cols="12" md="6">
              <p>
                <strong>Atributo:</strong>
                {{ attributeMap[selectedKnight.attribute] }}
              </p>
              <p><strong>Ataque:</strong> {{ selectedKnight.attack }}</p>
              <p><strong>Exp:</strong> {{ selectedKnight.exp }}</p>
            </v-col>
          </v-row>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            class="primary"
            color="black"
            text
            outlined
            @click="detailsModalOpen = false"
          >
            Fechar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import axios from "axios";

export default {
  props: {
    heroes: {
      type: Array,
      required: true,
    },
  },

  watch: {
    heroes: {
      handler(newHeroes) {
        this.knights = newHeroes;
      },
      deep: true,
    },
  },

  data: () => ({
    knights: [],
    editedNickname: "",
    modalOpen: false,
    confirmDeleteKnightId: null,
    confirmEditModal: false,
    detailsModalOpen: false,
    selectedKnight: null,
    attributeMap: {
      strength: "Força",
      dexterity: "Destreza",
      constitution: "Constituição",
      intelligence: "Inteligência",
      wisdom: "Sabedoria",
      charisma: "Carisma",
    },
  }),

  methods: {
    showDetailsModal(knight) {
      this.selectedKnight = knight;
      this.detailsModalOpen = true;
    },

    async getKnights() {
      await axios.get("http://localhost:3030/api/v1/knights").then((res) => {
        this.knights = res.data.data;
      });
    },
    async deleteKnight(id) {
      this.confirmDeleteKnightId = id;
    },

    async deleteConfirmed() {
      try {
        await axios.delete(
          `http://localhost:3030/api/v1/knights/${this.confirmDeleteKnightId}`
        );
        this.knights = this.knights.filter(
          (knight) => knight.id !== this.confirmDeleteKnightId
        );

        this.confirmDeleteKnightId = null;
      } catch (error) {
        console.error(
          `Erro ao deletar o cavaleiro ${this.confirmDeleteKnightId}: ${error.message}`
        );
      }
    },

    editNickname(id) {
      this.modalOpen = true;
      this.editedKnightId = id;
      this.confirmModal = true;
    },

    async saveNickname() {
      try {
        await axios.patch(
          `http://localhost:3030/api/v1/knights/${this.editedKnightId}`,
          {
            nickname: this.tempNickname,
          }
        );
        const knightIndex = this.knights.findIndex(
          (knight) => knight.id === this.editedKnightId
        );
        this.knights[knightIndex].nickname = this.tempNickname;
        this.modalOpen = false;
        this.confirmModal = false; // fechar o modal de confirmação
      } catch (error) {
        console.error(
          `Erro ao salvar o apelido do cavaleiro ${this.editedKnightId}: ${error.message}`
        );
      }
    },
  },

  mounted() {
    this.getKnights();
  },
};
</script>


<style>
.color-name {
  color: #cc1515;
}

.card-bottom-margin {
  margin-bottom: 3%;
}

.color-descriptions {
  color: #cc1515;
}

.custom-card {
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.v-btn.primary {
  color: #fff;
}

.v-btn.primary:hover {
  background-color: #2196f3;
}

.v-btn.primary:active {
  background-color: #1976d2;
}

.v-btn.primary:focus {
  box-shadow: 0 0 0 2px #2196f3;
}

.v-dialog {
  transition: scale 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
}

.scale-transition-enter {
  transform: scale(0);
}

.scale-transition-enter-active {
  transform: scale(1);
  transition: transform 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
}

.scale-transition-leave-active {
  transform: scale(0);
  transition: transform 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
}
</style>