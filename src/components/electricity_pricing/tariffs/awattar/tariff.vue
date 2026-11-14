<template>
  <div class="tariff-awattar">
    <openwb-base-select-input
      title="Land"
      not-selected="Bitte auswählen"
      :options="[
        { value: 'de', text: 'Deutschland' },
        { value: 'at', text: 'Österreich' },
      ]"
      :model-value="tariff.configuration.country"
      @update:model-value="updateConfiguration($event, 'configuration.country')"
    >
      <template #help>
        Es werden die abgefragten Börsenpreise verwendet, die aWATTar bereitstellt. aWATTar-Gebühren oder Steuern werden
        nicht berücksichtigt.
      </template>
    </openwb-base-select-input>

    <openwb-base-button-group-input
      title="Preisart"
      :buttons="[
        { buttonValue: true, text: 'Netto' },
        { buttonValue: false, text: 'Brutto' },
      ]"
      :model-value="tariff.configuration.net ?? true"
      @update:model-value="updateConfiguration($event, 'configuration.net')"
    >
      <template #help>
        Netto: Reine Börsenpreise ohne Aufschläge<br />
        Brutto: Börsenpreise mit konfigurierbaren Aufschlägen und MwSt.
      </template>
    </openwb-base-button-group-input>

    <div
      v-if="!tariff.configuration.net"
      class="mt-3"
    >
      <h5 class="mb-3">Preisberechnung</h5>
      <div class="alert alert-info mb-3">
        <strong>Formel:</strong><br />
        Stündliche Preise EPEX Spot® {{ tariff.configuration.country?.toUpperCase() || "AT" }}<br />
        + | Stündliche Preise EPEX Spot® {{ tariff.configuration.country?.toUpperCase() || "AT" }} | ×
        {{ (tariff.configuration.proportional / 100 || 0).toFixed(2) }} ({{
          (tariff.configuration.proportional || 0).toFixed(0)
        }}%)<br />
        + {{ (tariff.configuration.fix * 100000 || 0).toFixed(3) }} Cent/kWh<br />
        + {{ (tariff.configuration.tax || 0).toFixed(0) }}% MwSt.
      </div>

      <div class="row">
        <div class="col-md-4">
          <openwb-base-number-input
            title="Proportionaler Aufschlag"
            unit="%"
            :min="0"
            :max="100"
            :step="0.01"
            :model-value="tariff.configuration.proportional || 0"
            @update:model-value="updateConfiguration($event, 'configuration.proportional')"
          >
            <template #help> Prozentuale Gebühr auf den Börsenpreis (z.B. 3% = 0.03) </template>
          </openwb-base-number-input>
        </div>

        <div class="col-md-4">
          <openwb-base-number-input
            title="Fester Aufschlag"
            unit="Cent/kWh"
            :min="0"
            :step="0.001"
            :model-value="tariff.configuration.fix * 100000 || 0"
            @update:model-value="updateConfiguration($event / 100000, 'configuration.fix')"
          >
            <template #help> Feste Gebühr pro kWh (z.B. 1,500 Cent/kWh) </template>
          </openwb-base-number-input>
        </div>

        <div class="col-md-4">
          <openwb-base-number-input
            title="Mehrwertsteuer"
            unit="%"
            :min="0"
            :max="50"
            :step="0.01"
            :model-value="tariff.configuration.tax || 0"
            @update:model-value="updateConfiguration($event, 'configuration.tax')"
          >
            <template #help> Mehrwertsteuer in Prozent (z.B. 20%) </template>
          </openwb-base-number-input>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import TariffConfigMixin from "../TariffConfigMixin.vue";

export default {
  name: "TariffAwattar",
  mixins: [TariffConfigMixin],
};
</script>
