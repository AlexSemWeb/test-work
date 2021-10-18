<template>
  <v-container>
    <v-row class="justify-center mt-4">
      <v-col align="center" md="4">
        <validation-observer ref="observer" v-slot="{ invalid }">
          <v-form @submit.prevent="submit" :disabled="isSending">
            <validation-provider
              v-slot="{ errors, valid }"
              name="Имя пользователя"
              rules="required"
            >
              <v-text-field
                v-model="name"
                :error-messages="errors"
                label="Имя пользователя*"
                required
                :success="valid"
              ></v-text-field>
            </validation-provider>
            <validation-provider
              v-slot="{ errors, valid }"
              name="E-mail"
              rules="required|email"
            >
              <v-text-field
                v-model="email"
                :error-messages="errors"
                label="E-mail*"
                required
                :success="valid"
              ></v-text-field>
            </validation-provider>
            <v-menu
              ref="menu"
              v-model="menu"
              :close-on-content-click="false"
              :return-value.sync="birthdate"
              transition="scale-transition"
              offset-y
              min-width="auto"
            >
              <template v-slot:activator="{ on, attrs }">
                <v-text-field
                  v-model="birthdate"
                  label="Дата рождения"
                  prepend-icon="mdi-calendar"
                  readonly
                  v-bind="attrs"
                  v-on="on"
                ></v-text-field>
              </template>
              <v-date-picker v-model="birthdate" no-title scrollable>
                <v-spacer></v-spacer>
                <v-btn text color="primary" @click="menu = false">
                  Отмена
                </v-btn>
                <v-btn text color="primary" @click="$refs.menu.save(birthdate)">
                  OK
                </v-btn>
              </v-date-picker>
            </v-menu>
            <validation-provider v-slot="{ errors }" name="Должность">
              <v-select
                v-model="position"
                :items="items"
                item-text="name"
                item-value="id"
                :error-messages="errors"
                label="Должность"
                data-vv-name="select"
              ></v-select>
            </validation-provider>
            <validation-provider
              v-slot="{ errors }"
              name="Права администратора"
            >
              <v-checkbox
                v-model="isAdmin"
                :error-messages="errors"
                label="Права администратора"
                type="checkbox"
              ></v-checkbox>
            </validation-provider>
            <v-row v-if="isAdmin">
              <v-col align="center" md="2"
                ><v-checkbox
                  v-model="selectedRights"
                  disabled
                  label="Чтение"
                  value="1"
                ></v-checkbox
              ></v-col>
              <v-col align="center" md="2">
                <v-checkbox
                  v-model="selectedRights"
                  label="Удаление"
                  value="2"
                ></v-checkbox
              ></v-col>
            </v-row>

            <validation-provider
              v-slot="{ errors, valid }"
              name="Номер телефона"
              :rules="{
                digits: 11,
                regex: '^(71|72|74|76|81|82|84|85|86|87|88|89)\\d{9}$',
              }"
            >
              <v-text-field
                v-model="phoneNumber"
                :counter="11"
                :error-messages="errors"
                label="Номер телефона"
                :success="valid && phoneNumber.length === 11"
              ></v-text-field>
            </validation-provider>

            <validation-provider
              name="password"
              rules="required"
              v-slot="{ errors, valid }"
            >
              <v-text-field
                v-model="password"
                :append-icon="show1 ? 'mdi-eye' : 'mdi-eye-off'"
                :type="show1 ? 'text' : 'password'"
                label="Пароль*"
                hint="Длина пароля должна быть не меньше 8"
                :error-messages="errors"
                counter
                @click:append="show1 = !show1"
                required
                :success="valid"
              ></v-text-field>
            </validation-provider>
            <validation-provider
              name="confirm"
              rules="required|password:@password"
              v-slot="{ errors, valid }"
            >
              <v-text-field
                v-model="confirmation"
                type="password"
                label="Повторите пароль*"
                hint="Длина пароля должна быть не меньше 8"
                :error-messages="errors"
                counter
                required
                :success="valid"
              ></v-text-field>
            </validation-provider>

            <v-btn class="mr-4" @click="generatePassword" :disabled="isSending">
              Генерация пароля
            </v-btn>

            <v-btn
              class="mr-4"
              type="submit"
              :disabled="invalid || isSending"
              color="success"
            >
              Создать
            </v-btn>
            <v-btn @click="clear" :disabled="isSending" color="error">
              Отчистеть
            </v-btn>
          </v-form>
        </validation-observer>
      </v-col>
    </v-row>
    <v-snackbar
      v-model="snackbar"
      :multi-line="true"
      color="success"
      :timeout="2000"
      top
      right
    >
      Форма успешно отправилась
      <template v-slot:action="{ attrs }">
        <v-btn color="white" text v-bind="attrs" @click="snackbar = false">
          Закрыть
        </v-btn>
      </template>
    </v-snackbar>
  </v-container>
</template>
<script>
import { required, digits, email, max, regex } from "vee-validate/dist/rules";
import {
  extend,
  ValidationObserver,
  ValidationProvider,
  setInteractionMode,
} from "vee-validate";
import { nanoid } from "nanoid";

setInteractionMode("eager");

extend("digits", {
  ...digits,
  message: "{_field_} должен состоять {length} цифр. ({_value_})",
});

extend("required", {
  ...required,
  message: "{_field_} не может быть пустым",
});

extend("max", {
  ...max,
  message: "{_field_} не должен превышать длину в {length}",
});

extend("regex", {
  ...regex,
  message:
    "{_field_} {_value_} должен начинаться с 71,72,74,76,81,82,84,85,86,87,88,89",
});

extend("email", {
  ...email,
  message: "Введите корректный e-mail",
});

extend("password", {
  params: ["target"],
  validate(value, { target }) {
    return value === target;
  },
  message: "Пароль не совпадает",
});

export default {
  components: {
    ValidationProvider,
    ValidationObserver,
  },
  data: () => ({
    name: "",
    phoneNumber: "",
    email: "",
    position: null,
    items: [
      { id: 1, name: "Administrator" },
      { id: 2, name: "Software Engineer" },
      { id: 3, name: "Content manager" },
    ],
    isAdmin: false,
    birthdate: "",
    menu: false,
    password: "",
    confirmation: "",
    show1: false,
    isSending: false,
    selectedRights: ["1"],
    snackbar: false,
  }),

  methods: {
    generatePassword() {
      let password = nanoid();
      this.password = password;
      this.confirmation = password;
    },
    formatPhoneNumber(phoneNumberString) {
      var cleaned = ("" + phoneNumberString).replace(/\D/g, "");
      var match = cleaned.match(/(\d{1})(\d{3})(\d{3})(\d{2})(\d{2})$/);
      if (match) {
        return [
          "+",
          match[1],
          "(",
          match[2],
          ")",
          match[3],
          "-",
          match[4],
          "-",
          match[5],
        ].join("");
      }
      return "";
    },
    timeout() {
      return new Promise((resolve) => setTimeout(resolve, 1000));
    },
    submit() {
      let data = {
        nickname: this.name,
        email: this.email,
        birth_date: this.birthdate,
        position_id: this.position,
        is_admin: this.isAdmin,
        user_rights: this.selectedRights,
        password: this.password,
        phone: this.formatPhoneNumber(this.phoneNumber),
      };
      this.isSending = true;
      this.timeout().then(() => {
        this.isSending = false;
        this.snackbar = true;
        console.log(JSON.stringify(data));
      });
    },
    clear() {
      this.name = "";
      this.phoneNumber = "";
      this.email = "";
      this.position = null;
      this.isAdmin = null;
      this.birthdate = "";
      this.password = "";
      this.confirmation = "";
      this.show1 = false;
      this.isDelete = false;
      this.$refs.observer.reset();
    },
  },
};
</script>
