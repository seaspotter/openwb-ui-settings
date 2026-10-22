<template>
  <div class="web-theme-quartz">
    <openwb-base-alert subtype="info"> Für Quartz gibt es aktuell keine Einstellungen. </openwb-base-alert>
  </div>
</template>

<script>
import WebThemeConfigMixin from "../WebThemeConfigMixin.vue";

export default {
  name: "WebThemeQuartz",
  mixins: [WebThemeConfigMixin],
};
</script>
