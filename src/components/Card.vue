<template>
  <v-container>
    <v-row justify="space-around">
      <v-card
        v-for="knight in knights"
        :key="knight.id"
        width="300"
        class="card-bottom-margin"
        style="padding-bottom: 20px"
      >
        <v-img height="200" src="../assets/banner.jpg" cover class="text-white">
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
                <strong class="color-descriptions">{{ knight.weapons }}</strong>
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
                <strong class="color-descriptions">{{ knight.attack }}</strong>
              </v-sheet>
            </v-col>
          </v-row>
          <v-row no-gutters>
            <v-col>
              <v-sheet class="pa-2 ma-2"> Exp:</v-sheet>
            </v-col>
          </v-row>
          <v-progress-linear v-model="progress" color="red" height="25">
            <template v-slot:default="{ value }">
              <strong>{{ Math.ceil(value) }}%</strong>
            </template>
          </v-progress-linear>
        </v-card-text>
      </v-card>
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
        <v-btn color="blue darken-1" text @click="confirmDeleteKnightId = null">
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
        <v-btn color="blue darken-1" text @click="saveNickname"> Salvar </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import axios from "axios";

export default {
  data: () => ({
    knights: [],
    editedNickname: "",
    modalOpen: false,
    confirmDeleteKnightId: null,
    confirmEditModal: false,
    attributeMap: {
      strength: "Força",
      dexterity: "Destreza",
      constitution: "Constituição",
      intelligence: "Inteligência",
      wisdom: "Sabedoria",
      charisma: "Carisma",
    },
  }),

  computed: {
    progress() {
      const baseExp = 10000;
      const totalExp = this.knights.reduce(
        (sum, knight) => sum + knight.exp,
        0
      );
      return (totalExp / baseExp) * 100;
    },
  },

  methods: {
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
</style>