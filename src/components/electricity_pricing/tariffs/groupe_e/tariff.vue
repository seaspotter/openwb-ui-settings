<template>
  <div class="tariff-groupe-e">
    <openwb-base-alert subtype="info">
      Tarif Vario der Groupe E (Westschweiz) - www.groupe-e.ch/de/strom/vario
      <br />
      Der Tarif passt sich alle 15 Minuten an, abhängig von der aktuellen Netzauslastung und dem Marktpreis.
      <br />
      Die Preise für den nächsten Tag werden jeweils am Vortag um 18 Uhr publiziert.
    </openwb-base-alert>
  </div>
</template>

<script>
import TariffConfigMixin from "../TariffConfigMixin.vue";

export default {
  name: "TariffGroupeE",
  mixins: [TariffConfigMixin],
};
</script>
