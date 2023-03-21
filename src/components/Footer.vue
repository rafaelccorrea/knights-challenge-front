<template>
  <v-footer class="d-flex flex-column fixed-footer w-100" app>
    <div class="bg-footerSociais d-flex w-100 align-center px-4 text">
      <strong>Siga-me em minhas redes socias...</strong>

      <v-spacer></v-spacer>

      <v-btn
        v-for="icon in icons"
        :key="icon"
        class="mx-4"
        :icon="icon"
        variant="plain"
        size="small"
        @click="redirectTo(icon)"
      ></v-btn>
    </div>

    <div class="px-4 py-2 bg-footer text-center w-100 text">
      {{ new Date().getFullYear() }} — <strong>Rafael Correa</strong>
    </div>
  </v-footer>
</template>

<script>
export default {
  data: () => ({
    icons: ["mdi-github", "mdi-linkedin", "mdi-instagram"],
    socialUrls: {
      "mdi-github": "https://github.com/rafaelccorrea",
      "mdi-linkedin": "https://www.linkedin.com/in/rafael-correa-925067146/",
      "mdi-instagram": "https://www.instagram.com/rafaelccorrea/",
    },
  }),
  methods: {
    redirectTo(icon) {
      let url = this.socialUrls[icon];
      if (url) {
        window.open(url, "_blank");
      }
    },
  },
};
</script>

<style>
.fixed-footer {
  position: fixed;
  bottom: 0;
}

.text {
  color: floralwhite;
}

.bg-footerSociais {
  background: #616161;
}

.bg-footer {
  background: #454645;
}
</style>
