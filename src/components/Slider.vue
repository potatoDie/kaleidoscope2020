<template>
    <!-- Build a casing around vue-slider including a label/title and value next to the slider.
    You can't use v-model insode custom components. Define v-bind and v-on change instead -->
    <div class="form-control">
        {{ label }}
        <div class="row">
            <vue-slider
                :modelValue="modelValue"
                @change="$emit('update:modelValue', $event)"
                :min="min"
                :max="max"
                :tooltip="'none'"
                :processStyle="{
                    backgroundColor: 'rgb(218, 58, 0)',
                }"
            ></vue-slider>
            <span>{{ modelValue }}</span>
        </div>
    </div>
</template>

<script>
import VueSlider from "vue-slider-component";
import "vue-slider-component/theme/default.css";

export default {
    props: ["label", "modelValue", "min", "max"],
    components: {
        VueSlider,
    },
};
</script>

<style scoped>
.form-control {
    margin-bottom: 18px;
}
.vue-slider {
    margin-right: 20px;
    width: calc(100% - 50px) !important;
    max-width: 200px;
}

.vue-slider + span {
    font-size: 18px;
    vertical-align: middle;
}
.row {
    display: flex;
}
</style>