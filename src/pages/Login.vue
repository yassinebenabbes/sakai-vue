<template>
  <div
    class="
      surface-0
      flex
      align-items-center
      justify-content-center
      min-h-screen min-w-screen
      overflow-hidden
    "
  >
    <div class="grid justify-content-center p-2 lg:p-0" style="min-width: 80%">
      <div class="col-12 mt-5 xl:mt-0 text-center">
        <!-- <img
          :src="'images/logo.jpeg'"
          class="mb-5"
          style="width: 81px; height: 60px"
        /> -->
      </div>
      <div
        class="col-12 xl:col-6"
        style="
          border-radius: 56px;
          padding: 0.3rem;
          background: linear-gradient(
            180deg,
            var(--primary-color),
            rgba(33, 150, 243, 0) 30%
          );
        "
      >
        <div
          class="h-full w-full m-0 py-7 px-4"
          style="
            border-radius: 53px;
            background: linear-gradient(
              180deg,
              var(--surface-50) 38.9%,
              var(--surface-0)
            );
          "
        >
          <div class="text-center mb-5">
            <div class="text-900 text-3xl font-medium mb-3">
              Connectez-vous à Weego
            </div>
          </div>
          <form>
            <div class="w-full md:w-10 mx-auto">
              <label
                for="email1"
                class="block text-900 text-xl font-medium mb-2"
                >Email</label
              ><label class="error" v-if="this.errors.email != ''">{{
                errors.email
              }}</label>
              <InputText
                id="email1"
                v-model="login.email"
                type="text"
                class="w-full mb-3"
                placeholder="Email"
                style="padding: 1rem"
                :class="{ 'is-invalid': errors.email }"
              />

              <label
                for="password1"
                class="block text-900 font-medium text-xl mb-2"
                >Password </label
              ><label class="error" v-if="this.errors.password != ''">{{
                errors.password
              }}</label>
              <InputText
                id="password1"
                v-model="login.password"
                placeholder="Password"
                :toggleMask="true"
                class="w-full mb-3"
                inputClass="w-full"
                style="padding: 1rem"
                :class="{ 'is-invalid': errors.password }"
                type="Password"
              ></InputText>

              <Button
                @click="hundellerrors"
                label="Sign In"
                class="w-full p-3 text-xl"
              ></Button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Swal from "sweetalert2";

export default {
  data() {
    return {
      login: {
        email: "",
        password: "",
      },
      errors: { email: "", password: "" },
    };
  },
  methods: {
    hundellerrors() {
      if (this.login.email == "") {
        this.errors.email = "Email est obligatoire";
      }
      if (this.login.password == "") {
        this.errors.password = "Le mot de passe est obligatoire";
      } else if (this.login.password.length < 6) {
        this.errors.password =
          "mot de passe doit contenir au moins 6 caractaire";
      }
      if (this.login.password != "" && this.login.email != "") {
        this.conect();
      }
    },
    conect() {
      axios
        .post("login", this.login)
        .then((response) => {
       this.token = response.data.token.original.access_token;
       axios.defaults.headers.common["Authorization"] ="Bearer" + this.token;
          this.$router.push({ path: "/" });
        })
        .catch((err) => {
          if (err.response.data.message) {
           Swal.fire("Erreur d'authentification", err.response.data.message);
          } else {
           Swal.fire("Erreur d'authentification", err.message);
          }
        });
    },
 
  },
  computed: {
    logoColor() {
      if (this.$appState.darkTheme) return "white";
      return "dark";
    },
  },
};
</script>

<style scoped>
.p-password-meter {
  display: none;
}
.pi-eye {
  transform: scale(1.6);
  margin-right: 1rem;
}

.pi-eye-slash {
  transform: scale(1.6);
  margin-right: 1rem;
}
.error {
  color: #d8000c;
}
.is-invalid {
  border-color: #d8000c;
}
</style>