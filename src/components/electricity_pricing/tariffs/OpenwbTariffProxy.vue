<template>
  <openwb-base-alert
    v-if="tariff.official"
    subtype="success"
  >
    <font-awesome-icon :icon="['fas', 'certificate']" />
    Das ausgewählte Modul "{{ tariff.name }}" wird von openWB gepflegt.
  </openwb-base-alert>
  <openwb-base-alert
    v-else
    subtype="info"
  >
    <font-awesome-icon :icon="['fas', 'people-group']" />
    Das ausgewählte Modul "{{ tariff.name }}" wird in unserer Community gepflegt. Rückfragen oder Probleme bitte im
    Forum diskutieren.
  </openwb-base-alert>
  <openwb-base-heading> Einstellungen für Modul "{{ tariff.name }}" </openwb-base-heading>
  <component
    :is="getTariffComponent()"
    :tariff="tariff"
    @update:configuration="updateConfiguration($event)"
  />
</template>

<script>
import { library } from "@fortawesome/fontawesome-svg-core";
import { faPeopleGroup as fasPeopleGroup, faCertificate as fasCertificate } from "@fortawesome/free-solid-svg-icons";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";

library.add(fasPeopleGroup, fasCertificate);

import { defineAsyncComponent } from "vue";
import OpenwbTariffFallback from "./OpenwbTariffConfigFallback.vue";

export default {
  name: "OpenwbTariffProxy",
  components: {
    FontAwesomeIcon,
  },
  props: {
    tariff: { type: Object, required: true },
  },
  emits: ["update:configuration"],
  methods: {
    getTariffComponent() {
      console.debug(`loading tariff: ${this.tariff.type}`);
      return defineAsyncComponent({
        loader: () => import(`./${this.tariff.type}/tariff.vue`),
        errorComponent: OpenwbTariffFallback,
      });
    },
    updateConfiguration(event) {
      this.$emit("update:configuration", event);
    },
  },
};
</script>
