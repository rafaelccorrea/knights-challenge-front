<template>
  <v-container :class="{ loading: loading }">
    <v-row class="mb-1" no-gutters>
      <v-col cols="4">
        <v-sheet class="pa-2 ma-2">
          <v-switch
            v-model="model"
            hide-details
            true-value="Hall Heróis"
            false-value="Todos Cavaleiros"
            :label="`${model}`"
            :class="{ 'red--text': model === 'Hall Heróis' }"
          ></v-switch>
        </v-sheet>
      </v-col>
      <v-col cols="4" offset="4">
        <v-sheet class="pa-2 ma-2">
          <v-btn @click="dialog = true" color="error">Criar cavaleiro</v-btn>
        </v-sheet>
      </v-col>
    </v-row>
    <v-dialog v-model="dialog" max-width="500px">
      <v-card>
        <v-card-title>Cadastrar Cavaleiro</v-card-title>
        <v-card-text>
          <v-text-field
            v-model="name"
            label="Nome"
            required
            :error="!name"
          ></v-text-field>
          <v-text-field
            v-model="nickname"
            label="Apelido"
            required
            :error="!nickname"
          ></v-text-field>
          <v-text-field
            v-model="birthday"
            label="Data de nascimento"
            type="date"
            required
            @blur="validateDateOfBirth"
            :error="!birthday || isNaN(Date.parse(birthday))"
          ></v-text-field>
          <v-container v-for="(weapon, index) in weapons" :key="index">
            <v-text-field
              v-model="weapon.name"
              :label="`Nome da arma ${index + 1}`"
              :error="!weapon.name"
            ></v-text-field>
            <v-text-field
              v-model="weapon.mod"
              disabled
              :label="`Modificador da arma ${index + 1}`"
              :error="!weapon.mod"
            ></v-text-field>
            <v-select
              v-model="weapon.attr"
              :items="attributes"
              :label="`Atributo da arma ${index + 1}`"
              required
              :error="!weapon.attr"
            ></v-select>
            <v-checkbox
              v-model="weapon.equipped"
              :label="`Equipada ${index + 1}`"
            >
            </v-checkbox>

            <v-btn @click="removeWeapon(index)" color="error">Remover</v-btn>
          </v-container>

          <v-btn @click="addWeapon" color="primary">+ Adicionar arma</v-btn>
        </v-card-text>
        <v-card-actions>
          <v-btn
            @click="
              resetForm();
              closeDialog();
            "
            >Cancelar</v-btn
          >
          <v-btn @click="createKnight" :class="{ loading: loading }"
            >Salvar</v-btn
          >
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>
  
    
<style>
.red--text {
  color: #cc1515;
}
</style>
    
    <script>
import axios from "axios";

export default {
  data: () => ({
    model: "Todos Cavaleiros",
    dialog: false,
    name: "",
    nickname: "",
    birthday: "",
    weapons: [],
    attributes: [
      "Força",
      "Destreza",
      "Constituição",
      "Inteligência",
      "Sabedoria",
      "Carisma",
    ],
  }),

  watch: {
    weapons: {
      handler: "updateWeaponEquippedStatus",
      deep: true,
    },
  },
  methods: {
    addWeapon() {
      this.updateWeaponEquippedStatus();
      this.weapons.push({
        name: "",
        mod: Math.floor(Math.random() * 6) - 2, // Generate random number between -2 and 3
        attr: "",
        equipped: true,
      });
      if (this.weapons[this.weapons.length - 1].mod < -2) {
        this.weapons[this.weapons.length - 1].mod = -2; // Don't allow modifier to be less than -2
      }
    },

    updateWeaponEquippedStatus() {
      if (this.weapons.length === 1) {
        this.weapons[0].equipped = true;
        return;
      }

      let equippedFound = false;
      this.weapons.forEach((weapon, index) => {
        if (weapon.equipped) {
          if (!equippedFound) {
            equippedFound = true;
          } else {
            weapon.equipped = false;
          }
        }
        weapon.equippedNumber = index + 1;
      });
    },

    depara(weapons) {
      const weaponsFormat = {
        Força: "strength",
        Destreza: "dexterity",
        Constituição: "constitution",
        Inteligência: "intelligence",
        Sabedoria: "wisdom",
        Carisma: "charisma",
      };

      return weaponsFormat[weapons];
    },

    async createKnight() {
      try {
        // Validate fields
        if (!this.name || !this.birthday || isNaN(Date.parse(this.birthday))) {
          throw new Error(
            "Preencha todos os campos obrigatórios corretamente."
          );
        }

        // Check for empty inputs
        const emptyInputs = document.querySelectorAll(
          "input:required[value='']"
        );
        emptyInputs.forEach((input) => {
          input.classList.add("error--text");
          input.setAttribute("data-vv-validate-on", "submit");
          input.setAttribute("data-vv-as", input.getAttribute("label"));
        });
        if (emptyInputs.length > 0) {
          throw new Error("Preencha todos os campos obrigatórios.");
        }

        // Set equippedRequired for required weapons
        this.weapons.forEach((weapon) => {
          weapon.equippedRequired =
            weapon.attr && weapon.attr !== "" && !weapon.equipped;

          // Call the depara function to format the weapon attribute
          weapon.attr = this.depara(weapon.attr);
        });

        // Loop through the weapons and set keyAttribute if equipped is true
        let keyAttribute = "";
        for (let i = 0; i < this.weapons.length; i++) {
          const weapon = this.weapons[i];
          if (weapon.equipped) {
            keyAttribute = weapon.attr;
            break;
          }
        }

        this.loading = true;
        await axios.post("http://localhost:3030/api/v1/knights", {
          name: this.name,
          nickname: this.nickname,
          birthday: this.birthday,
          keyAttribute,
          weapons: this.weapons,
        });
        this.dialog = false;
        this.loading = false;
      } catch (error) {
        console.error(error);
        this.snackbarText = "Erro ao salvar cavaleiro: " + error.message;
        this.snackbarColor = "red";
        this.snackbarVisible = true;
        this.loading = false;
      }
    },

    removeWeapon(index) {
      this.weapons.splice(index, 1);
    },

    closeDialog() {
      this.dialog = false;
    },

    resetForm() {
      this.name = "";
      this.nickname = "";
      this.birthday = "";
      this.weapons = [];
      this.weapons.push({
        name: "",
        mod: "",
        attr: "",
        equipped: false,
      });
    },
  },
};
</script>
    