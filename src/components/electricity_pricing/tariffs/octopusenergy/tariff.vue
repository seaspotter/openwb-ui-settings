<template>
  <div class="tariff-octopusenergy">
    <openwb-base-text-input
      title="Benutzername"
      subtype="user"
      required
      :model-value="tariff.configuration.email"
      @update:model-value="updateConfiguration($event, 'configuration.email')"
    />
    <openwb-base-text-input
      title="Passwort"
      subtype="password"
      required
      :model-value="tariff.configuration.password"
      @update:model-value="updateConfiguration($event, 'configuration.password')"
    />
    <openwb-base-text-input
      title="Vertragsnummer (beginnt mit A-)"
      required
      :model-value="tariff.configuration.accountId"
      @update:model-value="updateConfiguration($event, 'configuration.accountId')"
    />
  </div>
</template>

<script>
import TariffConfigMixin from "../TariffConfigMixin.vue";

export default {
  name: "TariffOctopusEnergy",
  mixins: [TariffConfigMixin],
};
</script>
