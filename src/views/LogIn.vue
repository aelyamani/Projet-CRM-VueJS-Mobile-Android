<template>
  <div class="container">
    <div class="columns">
      <div class="column is-4 is-offset-4">
        <h1 class="title">Se connecter</h1>

        <form @submit.prevent="submitForm">
          <div class="field">
            <label>Nom d'utilisateur</label>
            <div class="control">
              <input type="text" name="username" class="input" v-model="username" />
            </div>
          </div>

          <div class="field">
            <label>Mot de passe</label>
            <div class="control">
              <input type="password" name="password" class="input" v-model="password" />
            </div>
          </div>

          <div class="notification is-danger" v-if="errors.length">
            <p v-for="error in errors" v-bind:key="error">{{ error }}</p>
          </div>

          <div class="field">
            <div class="control">
              <button class="button is-success">Connexion</button>
            </div>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "LogIn",
  data() {
    return {
      username: "",
      password: "",
      errors: [],
    };
  },
  methods: {
    async submitForm() {
      this.$store.commit("setIsLoading", true);

      axios.defaults.headers.common["Authorization"] = "";
      localStorage.removeItem("token");

      const formData = {
        username: this.username,
        password: this.password,
      };

      await axios
        .post("/api/v1/token/login/", formData)
        .then((response) => {
          console.log(response);
          const token = response.data.auth_token;

          this.$store.commit("setToken", token);

          axios.defaults.headers.common["Authorization"] = "Token " + token;

          localStorage.setItem("token", token);
        })
        .catch((error) => {
          console.log(error);

          if (error.response) {
            for (const property in error.response.data) {
              this.errors.push(`${property}: ${error.response.data[property]}`);
            }
          } else if (error.message) {
            this.errors.push("Quelque chose ne va pas. Veuillez réessayer svp!");
          }
        });

      await axios
        .get("/api/v1/users/me")
        .then((response) => {
          this.$store.commit("setUser", {
            id: response.data.id,
            username: response.data.username,
          });
          localStorage.setItem("userid", response.data.id);
          localStorage.setItem("username", response.data.username);
          this.$router.push("/dashboard/");
        })
        .catch((error) => {
          console.log(error);
        });

      await axios
        .get("/api/v1/teams/get_my_team/")
        .then((response) => {
          this.$store.commit("setTeam", {
            id: response.data.id,
            name: response.data.name,
          });
          // console.log(response);
          this.$router.push("/dashboard/");
        })
        .catch((error) => {
          console.log(error);
        });
      this.$store.commit("setIsLoading", false);
    },
  },
};
</script>
