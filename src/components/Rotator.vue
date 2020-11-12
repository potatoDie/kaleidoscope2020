<template>
    <div id="rotator" ref="rotator">
        <img src="../assets/rotator.svg" alt="" />
    </div>
</template>

<script>
import gsap from "gsap";
import { Draggable } from "gsap/Draggable.js";
gsap.registerPlugin(Draggable);

export default {
    props: ["rotation"],
    mounted: function () {
        this.createDraggable();
    },
    methods: {
        createDraggable: function () {
            let draggables = Draggable.create(this.$refs.rotator, {
                type: "rotation",
                callbackScope: this,
                onDrag: function () {
                    // emit rotation to parent
                    // or use global store
                    // anyway beware of single source of truth, in case rotation might change
                    // by means of other controls or events
                    this.$emit("update", draggable.rotation);
                },
            });
            let draggable = draggables[0];

            gsap.set(draggable.target, { rotation: this.rotation });
        },
    },
};
</script>

<style scoped>
#rotator {
    position: absolute;
    left: 0;
    top: 0;
    height: 100%;
    width: 100%;
    background: transparent;
    opacity: 1;
    border-radius: 50%;
    border: 0px yellow dashed;
    cursor: grab;
}

img {
    position: absolute;
    left: 0;
    top: 0;
    height: 100%;
    width: 100%;
}
</style>