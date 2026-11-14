<template>
  <div class="tariff-fallback">
    <openwb-base-alert
      v-if="Object.keys(tariff.configuration).length == 0"
      subtype="info"
    >
      Der Anbieter "{{ tariff.name }}" bietet keine Einstellungen.
    </openwb-base-alert>
    <div v-else>
      <openwb-base-alert subtype="warning">
        Es wurde keine Konfigurationsseite für den Anbieter "{{ tariff.name }}" gefunden. Die Einstellungen können als
        JSON direkt bearbeitet werden.
      </openwb-base-alert>
      <openwb-base-textarea
        title="Konfiguration"
        subtype="json"
        :model-value="tariff.configuration"
        @update:model-value="updateConfiguration($event, 'configuration')"
      >
        <template #help> Bitte prüfe, ob die Eingaben richtig interpretiert werden. </template>
      </openwb-base-textarea>
      <openwb-base-alert subtype="info">
        <pre>{{ JSON.stringify(tariff.configuration, undefined, 2) }}</pre>
      </openwb-base-alert>
    </div>
  </div>
</template>

<script>
import TariffConfigMixin from "./TariffConfigMixin.vue";

export default {
  name: "TariffConfigFallback",
  mixins: [TariffConfigMixin],
};
</script>
