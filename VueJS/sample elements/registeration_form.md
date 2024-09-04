<template>
    <v-sheet class="mx-auto" max-width="300">
      <v-row>
        <v-col>
            <p 
            class="text-capitalize mx-auto text-center font-weight-bold text-h4"
            >List Title</p>
            <v-spacer style="height: 20px;"></v-spacer>
            <v-form
              ref="registerForm"
              @submit.prevent="onSubmit"
              validate-on="blur"
              v-model="validForm"
              >
              <v-text-field
                v-model="input.username"
                label="Username"
                :rules="userNameRules"
                clearable
                class="font-weight-bold"
                color="orange-darken-4"
                variant="underlined"
                required
              ></v-text-field>
              <v-text-field
                v-model="input.email"
                label="E-Mail"
                :rules="emailRules"
                clearable
                class="font-weight-bold"
                color="orange-darken-4"
                variant="underlined"
                required
              ></v-text-field>
              <v-text-field
                v-model="input.password"
                :rules="passwordRules"
                :type="visible ? 'text' : 'password'"
                label="Password"
                class="font-weight-bold"
                clearable
                color="orange-darken-4"
                variant="underlined"
                :append-inner-icon="visible ? 'mdi-eye-outline':'mdi-eye-off-outline'"
                @click:append-inner="toggleVisibility"
                required
              ></v-text-field>
              <v-text-field
                v-model="input.passwordCheck"
                type="password"
                label="Passwort bestätigen"
                :rules="confirmPasswordRules"
                class="font-weight-bold"
                clearable
                color="orange-darken-4"
                variant="underlined"
                required
              ></v-text-field>
              <v-container class="px-0">
                <v-btn
                  :disabled="!validForm"
                  @click="!spinnerstate"
                  text="Registrieren"
                  size="large"
                  type="submit"
                  block
                  class="mx-auto text-capitalize font-weight-black bg-grey-lighten-2 text-grey-darken-1 py-7 mt-5"
                  rounded="lg"
                  variant="tonal"
                  :loading="spinnerstate"
                ></v-btn>
                <ErrorMessage :errortext="errortext"/>
              </v-container>
            </v-form>
        </v-col>
      </v-row>
    </v-sheet>
</template>

<script>

import { ref } from 'vue'
import ErrorMessage from '@/components/ErrorMessage'
import { myXMLHTTPRequests } from "@/services/Requests";
import { register } from 'register-service-worker';

export default {
  name: 'Register',
  components:{
    ErrorMessage
  },
  data(){
    return{
      validForm: false,
      visible: false,
      input:{
          username: "",
          email: "",
          password: "",
          passwordCheck: "",
      },
      userNameRules: [
        value => !!value || 'A user name must be entered.',
      ],
      emailRules: [
        value => !!value || 'An email address must be entered.,
        value => /^(([^<>()[\]\\.,;:\s@']+(\.[^<>()\\[\]\\.,;:\s@']+)*)|('.+'))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/.test(value) || 'The email must be valid.',
      ],
      passwordRules: [
        value => !!value || 'Please choose your password.',
        value => /(?=.*\d)(?=.*[a-z])(?=.*[A-Z]).{6,}/.test(value) || 'The password must contain at least one lowercase letter, number, special character and uppercase letter.,
        value => value.length >= 8 || 'Password must be at least 8 characters'
      ],
      spinnerstate:false,
      errortext: '',
    };
  },
  computed: {
    confirmPasswordRules(){
      return[
        value => !!value || 'Please confirm your password again.',
        value => value === this.input.password || 'Passwords are not match.',
      ];
    },
  },
  methods:{
    toggleVisibility() {
      this.visible = !this.visible;
    },
    async onSubmit() {
      await this.register(); 
    },
    async register (){
      //the reference name of the form declared in the html element matters here.
      const { valid } = await this.$refs.registerForm.validate()
      if (valid) {
        this.errortext = 'connecting...';

            const user = this.input.username;
            const passw = this.input.password;
            const email = this.input.email;

            let aberrortext = async function (user, passw, email) {
                return await myXMLHTTPRequests.callRegister(user, passw, email);
            };

            aberrortext(user, passw, email).then(
                (fulfilledResult) => {
                    //console.log(fulfilledResult);
                    this.errortext = 'success';
                },
                (rejectedResult) => {
                    //console.log(rejectedResult)
                    this.errortext = rejectedResult.error;
                }
            );
    }
    
  },
},
  setup(){
    const spinnerstate = ref(false);

    return {
      spinnerstate,
    };
  }
}
</script>