<template>
  <div class="vehicle-soc-saic-ismart">
    <openwb-base-alert subtype="warning">
      Zu häufiges Abfragen der Fahrzeugdaten kann das Fahrzeug aus dem Ruhezustand wecken
      und die 12V-Batterie belasten. Es wird empfohlen, "Ohne laufende Ladung" auf einen
      ausreichend hohen Wert zu setzen und "Nur aktualisieren wenn angesteckt" zu aktivieren.
    </openwb-base-alert>
    <openwb-base-text-input
      title="Benutzername"
      required
      subtype="user"
      :model-value="vehicle.configuration.saic_user"
      @update:model-value="updateConfiguration($event, 'configuration.saic_user')"
    >
      <template #help> Der Benutzername (E-Mail-Adresse) für die Anmeldung im MG-iSMART-Account. </template>
    </openwb-base-text-input>
    <openwb-base-text-input
      title="Kennwort"
      required
      subtype="password"
      :model-value="vehicle.configuration.saic_password"
      @update:model-value="updateConfiguration($event, 'configuration.saic_password')"
    >
      <template #help> Das Passwort für die Anmeldung im MG-iSMART-Account. </template>
    </openwb-base-text-input>
    <openwb-base-select-input
      title="Region"
      required
      not-selected="Bitte auswählen"
      :options="[
        { value: 'eu', text: 'Europa' },
        { value: 'au', text: 'Australien/Neuseeland' },
        { value: 'tr', text: 'Türkiye' },
      ]"
      :model-value="vehicle.configuration.region"
      @update:model-value="updateConfiguration($event, 'configuration.region')"
    >
      <template #help> Die Region, in der das Fahrzeug betrieben wird. </template>
    </openwb-base-select-input>
    <openwb-base-text-input
      title="VIN"
      :model-value="vehicle.configuration.vin"
      @update:model-value="updateConfiguration($event, 'configuration.vin')"
    >
      <template #help>
        Die Fahrgestellnummer des Fahrzeugs. Kann leer gelassen werden, falls nur ein
        Fahrzeug im Account vorhanden ist - dieses wird dann automatisch verwendet.
      </template>
    </openwb-base-text-input>
    <openwb-base-button-group-input
      title="SoC während der Ladung berechnen"
      :buttons="[
        {
          buttonValue: false,
          text: 'Nein',
          class: 'btn-outline-danger',
        },
        {
          buttonValue: true,
          text: 'Ja',
          class: 'btn-outline-success',
        },
      ]"
      :model-value="vehicle.configuration.calculate_soc"
      @update:model-value="updateConfiguration($event, 'configuration.calculate_soc')"
    >
      <template #help>
        Berechnet den Ladestand (SoC) während der Ladung über Ladeleistung und Ladedauer,
        statt bei jedem Zyklus die MG-Cloud abzufragen.
      </template>
    </openwb-base-button-group-input>
  </div>
</template>

<script>
import VehicleConfigMixin from "../VehicleConfigMixin.vue";

export default {
  name: "VehicleSocSaicIsmart",
  mixins: [VehicleConfigMixin],
};
</script>
