<template>
  <div class="tariff-ostrom">
    <openwb-base-alert subtype="info">
      Deine persönlichen Zugangsdaten (Client-ID und Client-Secret) erhältst Du über das
      <a
        href="https://developer.ostrom-api.io/"
        target="_blank"
        rel="noopener noreferrer"
      >
        Ostrom Developer Portal
      </a>
      . Behandle diese Daten wie ein Passwort, da sich darüber auch persönliche Daten aus Deinem ostrom-Account abfragen
      lassen!
    </openwb-base-alert>
    <openwb-base-text-input
      title="Client-ID"
      subtype="user"
      required
      :model-value="tariff.configuration.client_id"
      @update:model-value="updateConfiguration($event, 'configuration.client_id')"
    />
    <openwb-base-text-input
      title="Client-Secret"
      subtype="password"
      required
      :model-value="tariff.configuration.client_secret"
      @update:model-value="updateConfiguration($event, 'configuration.client_secret')"
    />
    <openwb-base-text-input
      title="Postleitzahl"
      :model-value="tariff.configuration.zip"
      @update:model-value="updateConfiguration($event, 'configuration.zip')"
    >
      <template #help>
        Die Postleitzahl wird zur Ermittlung der Abgaben und Umlagen benötigt. Ohne Angabe der Postleitzahl wird durch
        die ostrom API lediglich der Börsenstrompreis einschließlich Mehrwertsteuer geliefert.
      </template>
    </openwb-base-text-input>
  </div>
</template>

<script>
import TariffConfigMixin from "../TariffConfigMixin.vue";

export default {
  name: "TariffOstrom",
  mixins: [TariffConfigMixin],
};
</script>
