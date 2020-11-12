<template>
    <div id="dashboard">
        <div id="dashboard-source"></div>
        <div id="dashboard-destination"></div>
        <span class="credits">
            Created by <a href="https://potatodie.nl">potatoDie</a> 2020
        </span>
    </div>
</template>

<script>
export default {
    data() {
        return {};
    },
};
</script>

<style scoped>
#dashboard {
    order: 3;
    font-size: 12px;
    padding: 1em;
    background: orange;
    position: relative;
    width: 240px;
}
.credits {
    position: absolute;
    bottom: 12px;
    right: 1em;
}

a {
    color: rgb(218, 58, 0);
    text-decoration: none;
    font-weight: 600;
}

@media (max-width: 1024px) {
    #dashboard {
        width: 100%;
        display: flex;
    }

    #dashboard > div {
        width: 50%;
    }
}
</style>