<template>
    <!-- Dashboard comes first, because other components teleport to it -->
    <Dashboard />
    <SourcePanel />
    <DestinationPanel />
</template>

<script>
import SourcePanel from "./components/SourcePanel.vue";
import DestinationPanel from "./components/DestinationPanel.vue";
import Dashboard from "./components/Dashboard.vue";

export default {
    name: "App",
    components: {
        SourcePanel,
        DestinationPanel,
        Dashboard,
    },
};
</script>
