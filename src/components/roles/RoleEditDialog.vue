<template>
  <v-dialog :value="value" width="600px" @input="$emit('input', false)">
    <v-card class="pa-4">
      <h2>{{ formTitle }}</h2>
      <validation-observer ref="form" v-slot="{ invalid, handleSubmit }">
        <form @submit.prevent="handleSubmit(onSubmit)" @keypress.enter.prevent>
          <validation-provider
            v-slot="{ errors }"
            rules="required|alpha_spaces|max:30"
            name="title"
          >
            <v-text-field
              v-model="role.title"
              label="Title"
              :error-messages="errors"
            />
          </validation-provider>
          <validation-provider
            v-slot="{ errors }"
            rules="requiredSelect"
            name="local group"
          >
            <v-select
              v-model="role.localGroupId"
              :items="localGroups"
              item-value="id"
              item-text="title"
              label="Local group"
              :error-messages="errors"
            />
          </validation-provider>
          <validation-provider
            v-slot="{ errors }"
            rules="requiredSelect"
            name="working circle"
          >
            <v-select
              v-model="role.workingCircleId"
              :items="workingCircles"
              item-value="id"
              item-text="title"
              label="Working circle"
              :error-messages="errors"
            />
          </validation-provider>
          <p style="color: gray" class="caption">
            Responsibilities
          </p>
          <validation-provider
            v-slot="{ errors }"
            :rules="`${role.responsibilities.length < 1 ? 'requiredList' : ''}`"
            mode="eager"
            name="responsibility"
          >
            <v-text-field
              v-model="newResponsibility"
              label="Add new responsibility"
              solo
              :error-messages="errorResponsibility || errors"
              @keypress.enter="addResponsibility"
            >
              <template v-slot:append>
                <v-btn
                  text
                  icon
                  small
                  color="primary"
                  :disabled="!validResponsibility"
                  @click="addResponsibility"
                >
                  <v-icon>mdi-plus</v-icon>
                </v-btn>
              </template>
            </v-text-field>
          </validation-provider>
          <v-card v-if="role.responsibilities.length > 0" class="mb-4">
            <template v-for="(responsibility, i) in role.responsibilities">
              <v-divider v-if="i !== 0" :key="`${i}-divider`" />
              <v-list-item
                :key="`${i}-${responsibility}`"
                class="d-flex justify-space-between"
              >
                <span>{{ responsibility }}</span>
                <v-btn
                  text
                  icon
                  color="gray"
                  @click="role.responsibilities.splice(i, 1)"
                >
                  <v-icon>mdi-close</v-icon>
                </v-btn>
              </v-list-item>
            </template>
          </v-card>
          <validation-provider
            v-slot="{ errors }"
            rules="max:1000"
            name="description"
          >
            <v-textarea
              v-model="role.description"
              label="Description (optional)"
              placeholder="Any additional information not specified in the set of responsibilities.

This can include information about the circle or the specific project that the role is needed for."
              :error-messages="errors"
            />
          </validation-provider>
          <validation-provider
            v-slot="{ errors }"
            rules="max:1000"
            name="requirements"
          >
            <v-textarea
              v-model="role.requirements"
              label="Requirements (optional)"
              placeholder="Skills, experience, equipment"
              :error-messages="errors"
            />
          </validation-provider>

          <validation-provider
            v-slot="{ errors }"
            rules="requiredSelect"
            name="time commitment"
          >
            <v-select
              v-model="role.timeCommitmentMin"
              :items="timeCommitments"
              item-value="min"
              return-object
              label="Time commitment"
              :error-messages="errors"
              @change="onTimeCommitmentChange"
            >
              <template v-slot:item="{ item }">
                {{ item.min }} - {{ item.max }} hours/week
              </template>
              <template v-slot:selection="{ item }">
                {{ item.min }} - {{ item.max }} hours/week
              </template>
            </v-select>
          </validation-provider>
          <p class="caption mb-0" style="color: gray">
            Contact details
          </p>
          <validation-provider
            v-slot="{ errors }"
            name="email address"
            mode="eager"
            rules="required|email|max:50"
          >
            <v-text-field
              v-model="role.email"
              label="Email"
              :error-messages="errors"
            />
          </validation-provider>
          <validation-provider
            v-slot="{ errors }"
            name="Mattermost Id"
            mode="eager"
            rules="mattermost|max:50"
          >
            <v-text-field
              v-model="role.mattermostId"
              label="Mattermost id (optional)"
              :error-messages="errors"
            />
          </validation-provider>
          <validation-provider
            v-slot="{ errors }"
            rules="phone|max:20"
            mode="eager"
            name="phone number"
          >
            <v-text-field
              v-model="role.phone"
              label="Phone number (optional)"
              :error-messages="errors"
            />
          </validation-provider>
          <v-card-actions class="d-flex justify-end">
            <v-btn color="primary" text @click="$emit('input', false)">
              Cancel
            </v-btn>
            <v-tooltip top>
              <template v-slot:activator="{ on }">
                <div v-on="invalid ? on : undefined">
                  <v-btn color="primary" :disabled="invalid" type="submit">
                    Submit
                  </v-btn>
                </div>
              </template>
              <span>
                Form is incomplete
              </span>
            </v-tooltip>
          </v-card-actions>
        </form>
      </validation-observer>
    </v-card>
  </v-dialog>
</template>

<script>
import { mapState, mapActions } from "vuex";
import { extend, ValidationProvider, ValidationObserver } from "vee-validate";
import { required, alpha_spaces, max, email } from "vee-validate/dist/rules";

extend("required", {
  ...required,
  message: fieldName => {
    let article;
    if (fieldName.split(" ")[0].endsWith("s")) {
      article = "";
    } else if (/^([aeiou])/i.test(fieldName)) {
      article = "an";
    } else {
      article = "a";
    }

    return `You must specify ${article} ${fieldName}.`;
  },
});
extend("requiredSelect", {
  ...required,
  message: "You must select a {_field_}.",
});
extend("requiredList", {
  validate: () => false,
  message: "You must add at least one {_field_}.",
});
extend("email", { ...email, message: "You must enter a valid email address." });
extend("alpha_spaces", {
  ...alpha_spaces,
  message: "The {_field_} can only contain letters and spaces.",
});
extend("max", {
  ...max,
  params: ["length"],
  message: "The {_field_} must be under {length} characters.",
});
extend("phone", {
  validate: value => {
    const phoneRegex = RegExp(/^[+]*[(]{0,1}[0-9]{1,4}[)]{0,1}[-\s./0-9]*$/);
    return phoneRegex.test(value);
  },
  message: "You must enter a valid phone number",
});
extend("mattermost", {
  validate: value => {
    const mattermostRegex = RegExp(/^@\S+$/);
    return mattermostRegex.test(value);
  },
  message: "You must enter a valid Mattermost Id.",
});

let initialState = () => ({
  role: {
    title: undefined,
    responsibilities: [],
    description: undefined,
    requirements: undefined,
    localGroupId: undefined,
    workingCircleId: undefined,
    timeCommitmentMin: undefined,
    timeCommitmentMax: undefined,
    email: undefined,
    mattermostId: undefined,
    phone: undefined,
  },
  newResponsibility: undefined,
});

export default {
  name: "RoleEditDialog",
  components: {
    ValidationProvider,
    ValidationObserver,
  },
  props: {
    value: {
      required: true,
      type: Boolean,
    },
    editRole: {
      default: null,
      type: Object,
    },
  },
  data: () => initialState(),
  computed: {
    ...mapState("defaults", ["timeCommitments"]),
    ...mapState("groups", ["localGroups"]),
    ...mapState("groups", ["workingCircles"]),
    errorResponsibility: function() {
      const maxCharsResponsibility = 200;
      if (this.newResponsibility) {
        if (this.role.responsibilities.length == 10) {
          return "You can enter a maximum of 10 responsibilities";
        }
        if (this.newResponsibility.length > maxCharsResponsibility) {
          return `The responsibility must be under ${maxCharsResponsibility} characters.`;
        }
      }
      return null;
    },
    validResponsibility: function() {
      return !this.isEmpty(this.newResponsibility) && !this.errorResponsibility;
    },
    formTitle: function() {
      return this.editRole ? "Edit Role" : "New Role";
    },
  },
  watch: {
    editRole: {
      handler: function(editRole) {
        if (editRole) {
          for (var key in this.role) {
            if (key in editRole) {
              this.role[key] = editRole[key];
            }
          }
          this.role.workingCircleId = this.editRole.workingCircle.id;
          this.role.localGroupId = this.editRole.localGroup.id;
        }
      },
      immediate: true,
    },
  },
  created: function() {
    //   This feels a little dangerous, find a cleaner way
    const keys = Object.keys(this.role);
    keys.forEach(value => (this[value] = this.role[value]));
    this.workingCircleId = this.role.workingCircle.id;
    this.localGroupId = this.role.localGroup.id;
  },
  methods: {
    ...mapActions("roles", ["updateRole", "createRole"]),
    ...mapActions("alerts", ["displaySuccess"]),
    addResponsibility: function() {
      if (this.validResponsibility) {
        this.role.responsibilities.push(this.newResponsibility);
        this.newResponsibility = undefined;
      }
    },
    onTimeCommitmentChange: function(timeCommitment) {
      this.role.timeCommitmentMin = timeCommitment.min;
      this.role.timeCommitmentMax = timeCommitment.max;
    },
    resetState: function() {
      Object.assign(this.$data, initialState());
    },
    onSubmit: function() {
      if (this.editRole) {
        this.updateRole({ id: this.editRole.id, ...this.role });
      } else {
        this.createRole(this.role);
      }
      this.$emit("input", false);
      this.resetState();
      this.$nextTick(() => {
        this.$refs.form.reset();
      });

      this.displaySuccess(this.editRole ? "Role edited" : "Role created");
    },
    isEmpty: text => !text || text.length == 0 || !text.trim(),
  },
};
</script>

<style lang="scss" scoped></style>
