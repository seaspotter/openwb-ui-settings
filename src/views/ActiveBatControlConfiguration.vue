<template>
  <div class="batteryConfig">
    <form name="batteryConfigForm">
      <openwb-base-card title="Passive Speicherbeachtung (PV)">
        <div v-if="$store.state.mqtt['openWB/general/extern'] === true">
          <openwb-base-alert subtype="info">
            Diese Einstellungen sind nicht verfügbar, solange sich diese openWB im Steuerungsmodus "secondary" befindet.
          </openwb-base-alert>
        </div>
        <div v-else>
          <openwb-base-alert subtype="info">
            Die Regelmodi der Speicherbeachtung erfolgen "passiv" durch Anpassung der Fahrzeug-Ladeleistung.
            PV-Überschuss wird, je nach Konfiguration, entweder dem Fahrzeug zugeteilt oder dem Speicher überlassen.
            Netz- und Speicherbezug wird, sofern nicht anders konfiguriert, vermieden.
          </openwb-base-alert>
          <openwb-base-button-group-input
            v-model="batMode"
            title="Ladepriorität"
            :buttons="[{ buttonValue: 'ev_mode' }, { buttonValue: 'bat_mode' }, { buttonValue: 'min_soc_bat_mode' }]"
          >
            <template #label-ev_mode>
              <font-awesome-icon
                fixed-width
                :icon="['fas', 'car-side']"
              />
              Fahrzeuge
            </template>
            <template #label-bat_mode>
              <font-awesome-icon
                fixed-width
                :icon="['fas', 'fa-car-battery']"
              />
              Speicher
            </template>
            <template #label-min_soc_bat_mode>
              <font-awesome-icon
                fixed-width
                :icon="['fas', 'fa-battery-half']"
              />
              Nach SoC des Speichers
            </template>
            <template #help>
              <div v-if="batMode === 'ev_mode'">
                Der gesamte Überschuss wird in das EV geladen. Wird mehr Überschuss erzeugt als die Fahrzeuge abnehmen,
                findet auch eine Speicherladung statt.
              </div>
              <div v-if="batMode === 'bat_mode'">
                Der gesamte Überschuss wird in den Speicher geladen. Ist die maximale Ladeleistung des Speichers
                erreicht und es wird eingespeist, wird dieser Überschuss unter Beachtung der Einschaltschwelle in die
                Fahrzeuge geladen.
              </div>
              <div v-if="batMode === 'min_soc_bat_mode'">
                Verhält sich bis zum Erreichen des Mindest-SoC wie "Ladepriorität Speicher" und oberhalb des Mindest-SoC
                wie "Ladepriorität Fahrzeuge". Die maximale Leistung der Speicherbe- und entladung lässt sich hier
                festlegen.
              </div>
            </template>
          </openwb-base-button-group-input>
          <div v-if="batMode === 'min_soc_bat_mode'">
            <openwb-base-range-input
              title="Mindest-SoC des Speichers"
              :min="0"
              :max="100"
              :step="1"
              unit="%"
              required
              :model-value="$store.state.mqtt['openWB/general/chargemode_config/bat/min_soc']"
              @update:model-value="
                (updateState('openWB/general/chargemode_config/bat/min_soc', $event),
                updateState(
                  'openWB/general/chargemode_config/bat/max_soc',
                  $store.state.mqtt['openWB/general/chargemode_config/bat/min_soc'] <
                    $store.state.mqtt['openWB/general/chargemode_config/bat/max_soc']
                    ? $store.state.mqtt['openWB/general/chargemode_config/bat/max_soc']
                    : $event,
                ))
              "
            >
              <template #help>
                Unterhalb des Mindest-SoC wird vorhandener PV-Überschuss bevorzugt in den Speicher geladen. Oberhalb des
                Mindest-SoC hat die Fahrzeugladung Priorität.
              </template>
            </openwb-base-range-input>
            <openwb-base-range-input
              title="Maximal-SoC des Speichers"
              :min="0"
              :max="100"
              :step="1"
              unit="%"
              required
              :model-value="$store.state.mqtt['openWB/general/chargemode_config/bat/max_soc']"
              @update:model-value="
                (updateState('openWB/general/chargemode_config/bat/max_soc', $event),
                updateState(
                  'openWB/general/chargemode_config/bat/min_soc',
                  $store.state.mqtt['openWB/general/chargemode_config/bat/max_soc'] >
                    $store.state.mqtt['openWB/general/chargemode_config/bat/min_soc']
                    ? $store.state.mqtt['openWB/general/chargemode_config/bat/min_soc']
                    : $event,
                ))
              "
            >
              <template #help>
                Wird der Maximal-SoC überschritten, darf der Speicher bis zum Erreichen des Mindest-SoC zur
                Fahrzeugladung mitbenutzt werden.
              </template>
            </openwb-base-range-input>
            <openwb-base-alert
              v-if="
                $store.state.mqtt['openWB/general/chargemode_config/bat/min_soc'] ==
                $store.state.mqtt['openWB/general/chargemode_config/bat/max_soc']
              "
              subtype="info"
            >
              Bei identischen SoC Angaben findet keine Speicherhysterese statt.
            </openwb-base-alert>
            <openwb-base-heading> Speicher-Ladeleistung unterhalb Mindest-SoC </openwb-base-heading>
            <openwb-base-button-group-input
              title="Nur eine bestimmte Ladeleistung reservieren"
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
              :model-value="$store.state.mqtt['openWB/general/chargemode_config/bat/power_reserve_active']"
              @update:model-value="updateState('openWB/general/chargemode_config/bat/power_reserve_active', $event)"
            >
              <template
                v-if="$store.state.mqtt['openWB/general/chargemode_config/bat/power_reserve_active']"
                #help
              >
                ACHTUNG: Der hier eingestellte Wert darf die maximale Ladeleistung des Speichers nicht überschreiten!<br />
                Befindet sich der Speicher unterhalb des Mindest-SoC, wird er mit der hier eingestellten
                Speicherladeleistung geladen. Verbleibender Überschuss wird in die Fahrzeuge geladen.
              </template>
              <template
                v-else
                #help
              >
                Befindet sich der Speicher unterhalb des Mindest-SoC, wird er priorisiert geladen.
              </template>
            </openwb-base-button-group-input>
            <openwb-base-number-input
              v-if="$store.state.mqtt['openWB/general/chargemode_config/bat/power_reserve_active']"
              title="Reservierte Ladeleistung"
              :min="0.1"
              :step="0.1"
              unit="kW"
              required
              :model-value="$store.state.mqtt['openWB/general/chargemode_config/bat/power_reserve'] / 1000"
              @update:model-value="updateState('openWB/general/chargemode_config/bat/power_reserve', $event * 1000)"
            />
            <openwb-base-heading> Speicher-SoC oberhalb Maximal-SoC </openwb-base-heading>
            <openwb-base-button-group-input
              title="Entladung des Speichers erlauben"
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
              :model-value="$store.state.mqtt['openWB/general/chargemode_config/bat/power_discharge_active']"
              @update:model-value="updateState('openWB/general/chargemode_config/bat/power_discharge_active', $event)"
            >
              <template
                v-if="$store.state.mqtt['openWB/general/chargemode_config/bat/power_discharge_active']"
                #help
              >
                ACHTUNG: Der hier eingestellte Wert darf die maximale Entladeleistung des Speichers nicht überschreiten!
                Wird der Maximal-SoC überschritten, wird die PV-Ladung mit der hier eingestellten
                Speicherentladeleistung unterstützt. Der Speicher darf bis zum Mindest-SoC entladen werden. Die erlaubte
                Entladeleistung des Speichers wird dem Überschuss zum Erreichen der Einschaltschwelle hinzugerechnet.
              </template>
              <template
                v-else
                #help
              >
                Oberhalb des Maximal-SoC wird der Speicher nicht für die Fahrzeugladung mitgenutzt.
              </template>
            </openwb-base-button-group-input>
            <openwb-base-number-input
              v-if="$store.state.mqtt['openWB/general/chargemode_config/bat/power_discharge_active']"
              title="Erlaubte Entladeleistung"
              :min="0.1"
              :step="0.1"
              unit="kW"
              required
              :model-value="$store.state.mqtt['openWB/general/chargemode_config/bat/power_discharge'] / 1000"
              @update:model-value="updateState('openWB/general/chargemode_config/bat/power_discharge', $event * 1000)"
            />
          </div>
        </div>
      </openwb-base-card>
      <openwb-base-card title="Aktive Speichersteuerung">
        <div v-if="$store.state.mqtt['openWB/general/extern'] === true">
          <openwb-base-alert subtype="info">
            Diese Einstellungen sind nicht verfügbar, solange sich diese openWB im Steuerungsmodus "secondary" befindet.
          </openwb-base-alert>
        </div>
        <openwb-base-alert
          v-else
          subtype="warning"
          class="mb-3"
        >
          <p>
            Die aktive Speichersteuerung durch openWB basiert auf öffentlich zugänglichen Informationen zu den
            verschiedenen Speichersystemen. Diese können auch nicht vom Hersteller freigegebene Informationen
            beinhalten.<br />
            Fragen bezüglich der Gewährleistung und Hardwarekompatibilität sind vor der Nutzung mit dem Hersteller zu
            klären. openWB übernimmt keine Haftung für Schäden, welche aus der Nutzung der "aktiven Speichersteuerung"
            entstehen. Mit der Aktivierung der aktiven Speichersteuerung (Schalter unten) bestätigst du, dass du die
            Hinweise gelesen und verstanden hast und die Verantwortung für die Nutzung der aktiven Speichersteuerung
            übernimmst.
          </p>
        </openwb-base-alert>
        <openwb-base-button-group-input
          title="Speicher aktiv Steuern"
          :buttons="[
            { buttonValue: false, text: 'Nein', class: 'btn-outline-danger' },
            { buttonValue: true, text: 'Ja', class: 'btn-outline-success' },
          ]"
          :model-value="$store.state.mqtt['openWB/bat/config/bat_control_activated']"
          @update:model-value="updateState('openWB/bat/config/bat_control_activated', $event)"
        >
          <template
            v-if="$store.state.mqtt['openWB/bat/config/bat_control_activated']"
            #help
          >
            Speicher wird aktiv gesteuert. Grundlage ist die nachfolgende Konfiguration.
          </template>
          <template
            v-else
            #help
          >
            Speicher wird nicht aktiv gesteuert, sondern regelt eigenständig.<br />
            Es greifen die Regelparameter der Speicherbeachtung.
          </template>
        </openwb-base-button-group-input>
        <div
          v-if="$store.state.mqtt['openWB/bat/config/bat_control_activated'] === true"
          class="mb-3"
        >
          <openwb-base-heading class="mt-0"> Regelmodi der aktiven Speichersteuerung </openwb-base-heading>
          <openwb-base-alert subtype="info">
            Die aktive Speichersteuerung kann Speicherentladung begrenzen, den Speicher zur Ladung zwingen oder die
            Ladeleistung begrenzen. Die erlaubte Entladeleistung des Speichers (Speicherbeachtung PV) wird bei aktiver
            Speichersteuerung überschrieben, da Speicherentladung unter Umständen aktiv begrenzt wird.
          </openwb-base-alert>
        </div>
        <div v-if="$store.state.mqtt['openWB/bat/config/bat_control_activated']">
          <openwb-base-card title="Aktiv steuerbare Speicher">
            <openwb-base-alert
              v-if="hasNonControllableBatteries"
              subtype="danger"
            >
              Es sind nicht steuerbare Speicher im System vorhanden. Solche Speicher führen gewöhnlich eigenständig eine
              Nullpunktausregelung durch, versuchen also Überschuss zu speichern (Einspeisung zu verhindern) und
              Netzbezug durch eigene Entladung zu vermeiden.<br />
              Ein solcher Speicher versucht ebenso aktiv gesteuerte Speicher auszugleichen.
            </openwb-base-alert>
            <div
              v-for="(batteryConfig, key) in controllableBatteryConfigs"
              :key="key"
              class="mb-3"
            >
              <openwb-base-card
                :title="batteryConfig.name + ' (ID: ' + batteryConfig.id + ')'"
                :collapsible="true"
                :collapsed="true"
                subtype="warning"
              >
                <template #header>
                  <font-awesome-icon
                    :icon="['fas', 'fa-car-battery']"
                    class="fa-border"
                    :style="{
                      backgroundColor: batteryConfig.color,
                      color: getContrastColor(batteryConfig.color),
                      '--fa-border-color': getContrastColor(batteryConfig.color),
                    }"
                  />
                  {{ batteryConfig.name }} (ID: {{ batteryConfig.id }})
                </template>
                <openwb-base-number-input
                  title="Maximale Entladeleistung"
                  :min="0.01"
                  :step="0.01"
                  unit="kW"
                  required
                  :model-value="
                    $store.state.mqtt['openWB/bat/' + batteryConfig.id + '/get/max_discharge_power'] / -1000
                  "
                  @update:model-value="
                    updateState('openWB/bat/' + batteryConfig.id + '/get/max_discharge_power', $event * -1000)
                  "
                />
                <openwb-base-number-input
                  title="Maximale Ladeleistung"
                  :min="0.01"
                  :step="0.01"
                  unit="kW"
                  required
                  :model-value="$store.state.mqtt['openWB/bat/' + batteryConfig.id + '/get/max_charge_power'] / 1000"
                  @update:model-value="
                    updateState('openWB/bat/' + batteryConfig.id + '/get/max_charge_power', $event * 1000)
                  "
                />
              </openwb-base-card>
            </div>
          </openwb-base-card>
          <div v-if="hasControllableBatteries || hasChargePowerLimitControllableBatteries">
            <openwb-base-select-input
              title="Regelmodus"
              :options="controlModeOptions"
              :model-value="selectedControlMode"
              @update:model-value="updateState('openWB/bat/config/control_mode', $event)"
            />
            <openwb-base-alert
              subtype="info"
              class="mb-3"
            >
              Aktuell aktiv: {{ effectiveModeLabel }}
            </openwb-base-alert>
            <openwb-base-alert
              v-if="selectedControlMode === 'peak_shaving'"
              subtype="warning"
            >
              PeakShaving ist noch nicht implementiert. Diese Auswahl hat aktuell keine Wirkung, der Speicher regelt
              eigenständig.
            </openwb-base-alert>
            <openwb-base-alert
              v-if="selectedControlMode === 'limit_charge_power' && !hasChargePowerLimitControllableBatteries"
              subtype="danger"
            >
              Kein im System vorhandener Speicher unterstützt die Ladeleistungsbegrenzung.
            </openwb-base-alert>
            <openwb-base-alert
              v-if="
                selectedControlMode !== 'self_regulation' &&
                selectedControlMode !== 'limit_charge_power' &&
                selectedControlMode !== 'peak_shaving' &&
                selectedControlMode !== 'scheduled' &&
                !hasControllableBatteries
              "
              subtype="danger"
            >
              Die Speicher-Entladung ins Fahrzeug kann nicht gesteuert werden, da die Entladeleistung nicht an den/die
              konfigurierten Speicher übergeben werden kann.
            </openwb-base-alert>

            <div v-if="socSlidersVisible">
              <openwb-base-range-input
                title="Untere Entladeschranke"
                :min="5"
                :max="100"
                :step="1"
                unit="%"
                required
                :model-value="$store.state.mqtt['openWB/bat/config/bat_control_min_soc']"
                @update:model-value="
                  (updateState('openWB/bat/config/bat_control_min_soc', $event),
                  updateState(
                    'openWB/bat/config/bat_control_max_soc',
                    $store.state.mqtt['openWB/bat/config/bat_control_min_soc'] <
                      $store.state.mqtt['openWB/bat/config/bat_control_max_soc']
                      ? $store.state.mqtt['openWB/bat/config/bat_control_max_soc']
                      : $event,
                  ))
                "
              >
                <template #help>
                  Speicher, welche durch die aktive Steuerung entladen werden, schalten unterhalb des eingestellten SoC
                  auf "Eigenregelung", um mögliche Tiefentladung zu verhindern. Die aktive Ladung ist weiterhin möglich.
                </template>
              </openwb-base-range-input>
              <openwb-base-range-input
                title="Obere Ladeschranke"
                :min="5"
                :max="100"
                :step="1"
                unit="%"
                required
                :model-value="$store.state.mqtt['openWB/bat/config/bat_control_max_soc']"
                @update:model-value="
                  (updateState('openWB/bat/config/bat_control_max_soc', $event),
                  updateState(
                    'openWB/bat/config/bat_control_min_soc',
                    $store.state.mqtt['openWB/bat/config/bat_control_max_soc'] >
                      $store.state.mqtt['openWB/bat/config/bat_control_min_soc']
                      ? $store.state.mqtt['openWB/bat/config/bat_control_min_soc']
                      : $event,
                  ))
                "
              >
                <template #help>
                  Speicher, welche aktiv geladen werden, sperren oberhalb des eingestellten SoC die Entladung oder
                  schalten auf Eigenregelung des Speichers.
                </template>
              </openwb-base-range-input>
            </div>

            <div v-if="selectedControlMode === 'force_charge_below_price' || selectedControlMode === 'scheduled'">
              <openwb-base-number-input
                title="Speicher aktiv laden für Strompreise unter"
                :step="0.001"
                :precision="3"
                unit="ct/kWh"
                required
                :model-value="$store.state.mqtt['openWB/bat/config/charge_limit'] * 100"
                @update:model-value="updateState('openWB/bat/config/charge_limit', $event / 100)"
              />
            </div>
            <div v-if="selectedControlMode === 'block_discharge_above_price' || selectedControlMode === 'scheduled'">
              <openwb-base-number-input
                title="Entladesperre für Strompreise über"
                :step="0.001"
                :precision="3"
                unit="ct/kWh"
                required
                :model-value="$store.state.mqtt['openWB/bat/config/price_limit'] * 100"
                @update:model-value="updateState('openWB/bat/config/price_limit', $event / 100)"
              />
            </div>
            <openwb-base-alert
              v-if="
                (selectedControlMode === 'force_charge_below_price' ||
                  selectedControlMode === 'block_discharge_above_price' ||
                  selectedControlMode === 'scheduled') &&
                !$store.state.mqtt['openWB/optional/ep/configured']
              "
              subtype="warning"
            >
              Bitte in den übergreifenden Ladeeinstellungen einen Strompreis-Anbieter konfigurieren. Ohne
              Strompreis-Anbieter schaltet der Speicher auf Eigenregelung.
            </openwb-base-alert>

            <div v-if="selectedControlMode === 'manual'">
              <openwb-base-heading class="mt-0"> Manuelle Vorgabe </openwb-base-heading>
              <openwb-base-button-group-input
                title="Speicher"
                :buttons="[
                  { buttonValue: 'charge', text: 'Laden', class: 'btn-outline-success' },
                  { buttonValue: 'stop', text: 'Stop', class: 'btn-outline-secondary' },
                  { buttonValue: 'discharge', text: 'Entladen', class: 'btn-outline-danger', disabled: true },
                ]"
                :model-value="$store.state.mqtt['openWB/bat/config/manual_control']"
                @update:model-value="updateState('openWB/bat/config/manual_control', $event)"
              >
                <template #help>
                  Aktives Entladen des Speichers unabhängig vom Hausverbrauch (Netzeinspeisung aus dem Speicher
                  erzwingen) ist in Deutschland nicht erlaubt und daher nicht wählbar.
                </template>
              </openwb-base-button-group-input>
              <openwb-base-number-input
                v-if="$store.state.mqtt['openWB/bat/config/manual_control'] === 'charge'"
                title="Ladeleistung"
                :min="0.1"
                :step="0.1"
                unit="kW"
                :model-value="wattsToKw($store.state.mqtt['openWB/bat/config/manual_power'])"
                @update:model-value="updateState('openWB/bat/config/manual_power', kwToWatts($event))"
              >
                <template #help> Leer lassen für maximale Ladeleistung. </template>
              </openwb-base-number-input>
            </div>

            <div v-if="selectedControlMode === 'limit_charge_power'">
              <openwb-base-heading class="mt-0"> Ladeleistung begrenzen </openwb-base-heading>
              <openwb-base-number-input
                title="Ladeleistung begrenzen auf"
                :min="0.1"
                :step="0.1"
                unit="kW"
                :model-value="wattsToKw($store.state.mqtt['openWB/bat/config/charge_power_limit'])"
                @update:model-value="updateState('openWB/bat/config/charge_power_limit', kwToWatts($event))"
              >
                <template #help>
                  Der Speicher lädt mit maximal der hier eingestellten Leistung, ansonsten in Eigenregelung (Entladung,
                  Timing etc. werden nicht vorgegeben). Leer lassen für die volle konfigurierte maximale Ladeleistung.
                </template>
              </openwb-base-number-input>
            </div>

            <div v-if="selectedControlMode === 'scheduled'">
              <openwb-base-heading class="mt-0">
                Zeitpläne für die Speichersteuerung
                <template #actions>
                  <openwb-base-avatar
                    class="bg-success clickable"
                    title="Neuen Zeitplan anlegen"
                    @click.stop="addBatModePlan()"
                  >
                    <font-awesome-icon :icon="['fas', 'plus']" />
                  </openwb-base-avatar>
                </template>
              </openwb-base-heading>
              <bat-mode-plan
                v-for="(plan, planKey) in $store.state.mqtt['openWB/bat/config/mode_plans']"
                :key="planKey"
                :model-value="plan"
                :control-mode-options="controlModeOptionsForPlan"
                @update:model-value="updateState('openWB/bat/config/mode_plans', $event, `${planKey}`)"
                @send-command="$emit('sendCommand', $event)"
              />
            </div>
          </div>
          <div v-else>
            <openwb-base-alert subtype="info">
              Die Speicher-Entladung bzw. -Ladung kann nicht gesteuert werden, da keiner der konfigurierten Speicher
              eine Leistungsvorgabe oder -begrenzung unterstützt.
            </openwb-base-alert>
          </div>
        </div>
      </openwb-base-card>
      <openwb-base-submit-buttons
        form-name="batteryConfigForm"
        @save="$emit('save', mqttTopicsToPublish)"
        @reset="$emit('reset')"
        @defaults="$emit('defaults')"
      />
    </form>
  </div>
</template>

<script>
import ComponentState from "../components/mixins/ComponentState.vue";
import BatModePlan from "../components/bat/BatModePlan.vue";

import { library } from "@fortawesome/fontawesome-svg-core";
import {
  faCarBattery as fasCarBattery,
  faCarSide as fasCarSide,
  faBatteryHalf as fasBatteryHalf,
  faPlus as fasPlus,
} from "@fortawesome/free-solid-svg-icons";
import { FontAwesomeIcon } from "@fortawesome/vue-fontawesome";

library.add(fasCarBattery, fasCarSide, fasBatteryHalf, fasPlus);

export default {
  name: "OpenwbActiveBatControlConfigurationView",
  components: {
    FontAwesomeIcon,
    BatModePlan,
  },
  mixins: [ComponentState],
  emits: ["save", "reset", "defaults", "sendCommand"],
  data() {
    return {
      controlModeOptions: [
        { value: "self_regulation", text: "Eigenregelung (Standard)" },
        {
          value: "home_consumption_while_charging",
          text: "Speicher für Hausverbrauch reservieren, solange ein Fahrzeug lädt",
        },
        { value: "block_discharge", text: "Entladung sperren (immer)" },
        { value: "pv_yield_while_charging", text: "PV-Ertrag im Speicher laden, solange ein Fahrzeug lädt" },
        { value: "force_charge_below_price", text: "Laden erzwingen, solange Strompreis unter Grenze" },
        { value: "block_discharge_above_price", text: "Entladung sperren, solange Strompreis über Grenze" },
        { value: "manual", text: "Manuelle Vorgabe" },
        { value: "limit_charge_power", text: "Ladeleistung begrenzen" },
        { value: "peak_shaving", text: "PeakShaving, prognosebasiert (in Vorbereitung)" },
        { value: "scheduled", text: "Zeitgesteuert" },
      ],
      mqttTopics: [
        { topic: "openWB/bat/+/get/max_charge_power", writeable: true },
        { topic: "openWB/bat/+/get/max_discharge_power", writeable: true },
        { topic: "openWB/bat/+/get/power_limit_controllable", writeable: false },
        { topic: "openWB/bat/+/get/charge_power_limit_controllable", writeable: false },
        { topic: "openWB/bat/get/effective_control_mode", writeable: false },
        { topic: "openWB/bat/config/bat_control_activated", writeable: true },
        { topic: "openWB/bat/config/control_mode", writeable: true },
        { topic: "openWB/bat/config/manual_control", writeable: true },
        { topic: "openWB/bat/config/manual_power", writeable: true },
        { topic: "openWB/bat/config/charge_power_limit", writeable: true },
        { topic: "openWB/bat/config/mode_plans", writeable: true },
        { topic: "openWB/bat/config/bat_control_max_soc", writeable: true },
        { topic: "openWB/bat/config/bat_control_min_soc", writeable: true },
        { topic: "openWB/bat/config/charge_limit", writeable: true },
        { topic: "openWB/bat/config/price_limit", writeable: true },
        { topic: "openWB/general/chargemode_config/bat/mode", writeable: true },
        { topic: "openWB/general/chargemode_config/bat/power_discharge", writeable: true },
        { topic: "openWB/general/chargemode_config/bat/power_discharge_active", writeable: true },
        { topic: "openWB/general/chargemode_config/bat/power_reserve", writeable: true },
        { topic: "openWB/general/chargemode_config/bat/power_reserve_active", writeable: true },
        { topic: "openWB/general/chargemode_config/bat/max_soc", writeable: true },
        { topic: "openWB/general/chargemode_config/bat/min_soc", writeable: true },
        { topic: "openWB/general/extern", writeable: false },
        { topic: "openWB/system/device/+/component/+/config", writeable: false },
        { topic: "openWB/optional/ep/configured", writeable: false },
      ],
    };
  },
  computed: {
    batMode: {
      get() {
        return this.$store.state.mqtt["openWB/general/chargemode_config/bat/mode"];
      },
      set(newMode) {
        this.updateState("openWB/general/chargemode_config/bat/mode", newMode);
      },
    },
    numBatteriesInstalled() {
      return Object.keys(this.batteryConfigs).length;
    },
    batteryConfigs() {
      if (this.$store.state.mqtt["openWB/general/extern"] === true) {
        return {};
      }
      return this.filterComponentsByType(this.getWildcardTopics("openWB/system/device/+/component/+/config"), "bat");
    },
    controllableBatteryConfigs() {
      // Speicher, die mind. eine der beiden Steuerungs-Primitiven unterstuetzen - fuer beide wird
      // die maximale Lade-/Entladeleistung benoetigt, daher ein Feld fuer die Karte darunter.
      if (this.$store.state.mqtt["openWB/general/extern"] === true) {
        return {};
      }
      return Object.keys(this.batteryConfigs)
        .filter((key) => {
          const id = this.batteryConfigs[key].id;
          return (
            this.$store.state.mqtt[`openWB/bat/${id}/get/power_limit_controllable`] === true ||
            this.$store.state.mqtt[`openWB/bat/${id}/get/charge_power_limit_controllable`] === true
          );
        })
        .reduce((obj, key) => {
          return {
            ...obj,
            [key]: this.batteryConfigs[key],
          };
        }, {});
    },
    hasNonControllableBatteries() {
      if (this.$store.state.mqtt["openWB/general/extern"] === true) {
        return false;
      }
      return Object.keys(this.batteryConfigs).some((key) => {
        const id = this.batteryConfigs[key].id;
        return (
          this.$store.state.mqtt[`openWB/bat/${id}/get/power_limit_controllable`] === false &&
          this.$store.state.mqtt[`openWB/bat/${id}/get/charge_power_limit_controllable`] === false
        );
      });
    },
    hasControllableBatteries() {
      if (this.$store.state.mqtt["openWB/general/extern"] === true) {
        return false;
      }
      return Object.keys(this.batteryConfigs).some((key) => {
        const id = this.batteryConfigs[key].id;
        return this.$store.state.mqtt[`openWB/bat/${id}/get/power_limit_controllable`] === true;
      });
    },
    hasChargePowerLimitControllableBatteries() {
      if (this.$store.state.mqtt["openWB/general/extern"] === true) {
        return false;
      }
      return Object.keys(this.batteryConfigs).some((key) => {
        const id = this.batteryConfigs[key].id;
        return this.$store.state.mqtt[`openWB/bat/${id}/get/charge_power_limit_controllable`] === true;
      });
    },
    selectedControlMode() {
      return this.$store.state.mqtt["openWB/bat/config/control_mode"];
    },
    effectiveModeLabel() {
      const value = this.$store.state.mqtt["openWB/bat/get/effective_control_mode"];
      const option = this.controlModeOptions.find((entry) => entry.value === value);
      return option ? option.text : value;
    },
    controlModeOptionsForPlan() {
      return this.controlModeOptions.filter((option) => option.value !== "scheduled");
    },
    socSlidersVisible() {
      return [
        "home_consumption_while_charging",
        "block_discharge",
        "pv_yield_while_charging",
        "force_charge_below_price",
        "block_discharge_above_price",
        "manual",
        "scheduled",
      ].includes(this.selectedControlMode);
    },
  },
  methods: {
    filterComponentsByType(components, type) {
      return Object.keys(components)
        .filter((key) => {
          return components[key].type.includes(type);
        })
        .reduce((obj, key) => {
          return {
            ...obj,
            [key]: components[key],
          };
        }, {});
    },
    wattsToKw(value) {
      return value === null || value === undefined ? null : value / 1000;
    },
    kwToWatts(value) {
      return value === null || value === undefined ? null : value * 1000;
    },
    addBatModePlan() {
      this.$emit("sendCommand", { command: "addBatModePlan", data: {} });
    },
  },
};
</script>
