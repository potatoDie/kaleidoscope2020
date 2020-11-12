<!--
Create a draggable element that indicates the portion of the source image

The data is stored in the parent of the crosshairs. The data of the draggable itself is not used.
gsap sets a transform (if using type "x,y"), but we overwrite it each time.
We just notify the parent of the delta that the draggable has moved in the ondrag handler. 
To render the crosshairs the transform is calculated when the parent data is changed
To keep having a single source of truth, we don't use the data managed by the draggable. We
just use deltaX and deltaY to track the drag
-->
<template>
    <svg class="crosshairs" :viewBox="viewBox">
        <g ref="visor" :transform="transform">
            <rect
                :width="size.width"
                :height="size.height"
                :x="-size.width / 2"
                :y="-size.height / 2"
            />
        </g>
    </svg>
</template>

<script>
import gsap from "gsap";
import { Draggable } from "gsap/Draggable.js";
gsap.registerPlugin(Draggable);

export default {
    props: ["position", "size", "container"],
    computed: {
        viewBox() {
            return `0 0 ${this.container.width} ${this.container.height}`;
        },
        transform() {
            //  This transform overwrites the transform that gsap sets
            return `translate(${this.position.x}, ${this.position.y})`;
        },
    },
    mounted() {
        this.createDraggable();
    },
    methods: {
        createDraggable: function () {
            const element = this.$refs.visor;
            const self = this;
            Draggable.create(element, {
                type: "x, y",
                onDrag() {
                    self.$emit("update", {
                        x: this.deltaX,
                        y: this.deltaY,
                    });
                },
            });
        },
    },
};
</script>

<style scoped>
.crosshairs {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    background: transparent;
    fill: orange;
    stroke: white;
    stroke-width: 6px;
    fill-opacity: 0.15;
    stroke-opacity: 1;
    pointer-events: none; /* There are GUI elements below the SVG, so let events go through */
    filter: drop-shadow(3px 3px 2px rgba(0, 0, 0, 0.7));
}
.crosshairs > * {
    pointer-events: painted; /* These can be moved around */
}
</style>