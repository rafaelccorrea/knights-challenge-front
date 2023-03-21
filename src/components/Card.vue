<template>
  <v-container>
    <v-row justify="space-around">
      <v-card width="300">
        <v-img height="200" src="../assets/banner.jpg" cover class="text-white">
          <v-toolbar color="rgba(0, 0, 0, 0)" theme="dark">
            <v-toolbar-title class="text-h6 color-name">
              {{ name }}
            </v-toolbar-title>

            <template v-slot:append>
              <v-btn @click="editNickname()" color="error" icon>
                <v-icon>mdi-pencil</v-icon>
              </v-btn>
              <v-btn @click="deleteKnight(id)" color="error" icon>
                <v-icon>mdi-delete</v-icon>
              </v-btn>
            </template>
          </v-toolbar>
        </v-img>

        <v-card-text>
          <v-row no-gutters>
            <v-col>
              <v-sheet class="pa-2 ma-2"
                >Idade: <strong class="color-descriptions">{{ idade }}</strong>
              </v-sheet>
            </v-col>
            <v-col>
              <v-sheet class="pa-2 ma-2">
                Qtd.Armas:
                <strong class="color-descriptions">{{ qtd_armas }}</strong>
              </v-sheet>
            </v-col>
          </v-row>
          <v-row no-gutters>
            <v-col>
              <v-sheet class="pa-2 ma-2">
                Atributo:
                <strong class="color-descriptions">{{ atributo }}</strong>
              </v-sheet>
            </v-col>
            <v-col>
              <v-sheet class="pa-2 ma-2">
                Ataque: <strong class="color-descriptions">{{ ataque }}</strong>
              </v-sheet>
            </v-col>
          </v-row>
          <v-row no-gutters>
            <v-col>
              <v-sheet class="pa-2 ma-2"> Exp: </v-sheet>
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
  </v-container>
  <v-dialog v-model="modalOpen" max-width="500">
    <v-card>
      <v-card-title class="headline">Editar Nickname</v-card-title>
      <v-card-text>
        <v-form>
          <v-text-field
            v-model="editedNickname"
            label="Nickname"
            maxlength="8"
          ></v-text-field>
        </v-form>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="blue darken-1" text @click="modalOpen = false">
          Cancelar
        </v-btn>
        <v-btn color="blue darken-1" text @click="saveNickname()">
          Salvar
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import axios from "axios";
export default {
  data: () => ({
    name: "Rafael",
    ataque: 50,
    atributo: "Força",
    qtd_armas: 4,
    idade: 25,
    exp: 1800,
    modalOpen: false,
    editedNickname: "",
  }),

  computed: {
    progress() {
      const baseExp = 3547; // set a base value for exp
      const progressMax = 100; // set the maximum value for the progress bar
      const expRatio = this.exp / baseExp; // calculate the ratio of exp to the base value
      const progressValue = Math.pow(expRatio, 1.5) * progressMax; // calculate the progress value using the ratio and an exponential function
      return progressValue;
    },
  },

  methods: {
    deleteKnight(id) {
      console.log(`deletou o ${id} `);
    },
    editNickname() {
      this.modalOpen = true;
      this.editedNickname = this.name;
    },
    saveNickname() {
      if (this.editedNickname.trim() !== "") {
        this.name = this.editedNickname;
        this.modalOpen = false;
      }
    },

    async getKnights() {
      const res = await axios.get();
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

.color-descriptions {
  color: #cc1515;
}
</style>