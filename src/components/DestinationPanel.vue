<template>
    <div class="panel">
        <div class="lasagne-container">
            <div class="lasagne">
                <canvas id="canvas_destination"></canvas>
                <Rotator :rotation="rotation" @update="updateRotator" />
            </div>
        </div>

        <teleport to="#dashboard-destination">
            <form>
                <Slider
                    label="Pattern scale (%)"
                    v-model="patternScale"
                    :min="25"
                    :max="250"
                    :interval="1"
                />
            </form>
            <div class="tile-display">
                <label>Source tile</label>
                <div class="canvas_thumbnail">
                    <canvas id="canvas_fragment"></canvas>
                </div>
            </div>
            <div class="tile-display">
                <label>Pattern tile</label>
                <div class="canvas_thumbnail">
                    <canvas id="canvas_tile"></canvas>
                </div>
            </div>
        </teleport>
    </div>
</template>

<script>
import store from "../store.js";
import Slider from "./Slider.vue";
import Rotator from "./Rotator.vue";

export default {
    data() {
        return {
            store: store,
            ctx: null,
            tileCanvas: null,
            tileCtx: null,
            fragmentCanvas: null,
            fragmentCtx: null,
            // Slider doesn't work yet with fraction intervals
            // So lets use percentages (1 = 100%)
            patternScale: 100, // The destination tile may be scaled
            rotation: 20,
        };
    },
    components: {
        Slider,
        Rotator,
    },
    mounted() {
        this.ctx = document
            .getElementById("canvas_destination")
            .getContext("2d");

        // Create a canvas to construct tile
        this.tileCanvas = document.getElementById("canvas_tile");
        this.tileCtx = this.tileCanvas.getContext("2d");

        // We'll also need a canvas to put the source fragment on.
        // I used tileCanvas for this first, but that canvas may be too small if destination dimensions are small
        this.fragmentCanvas = document.getElementById("canvas_fragment");
        this.fragmentCtx = this.fragmentCanvas.getContext("2d");
    },
    watch: {
        "store.diameter"(value) {
            this.ctx.canvas.width = value;
            this.ctx.canvas.height = value;
        },
        "store.sourceTile"(value) {
            this.draw(value);
        },
        patternScale() {
            this.draw(this.store.sourceTile);
        },
        rotation() {
            this.draw(this.store.sourceTile);
        },
    },
    methods: {
        draw(imageData) {
            const w = imageData.width;
            const h = imageData.height;

            /**
             * putImageData doesn't take the canvas transformation matrix into account.
             * But drawImage does. So we create a tile by first copying the imageData on fragmentCanvas and then use that to
             * create the pattern tile
             */

            this.fragmentCtx.canvas.width = w;
            this.fragmentCtx.canvas.height = h;
            this.fragmentCtx.putImageData(imageData, 0, 0);

            const dw = (w * 2 * this.patternScale) / 100;
            const dh = (h * 2 * this.patternScale) / 100;

            this.tileCtx.canvas.width = dw;
            this.tileCtx.canvas.height = dh;

            // Scale the imagedata from fragmentCanvas, to tileCanvas
            // This creates the left top quadrant
            this.tileCtx.drawImage(
                this.fragmentCanvas,
                0,
                0,
                w,
                h,
                0,
                0,
                dw / 2,
                dh / 2
            );

            // You could do the same for the other 3 quadrants. We can also draw the canvas on it self
            // Saving one drawImage call
            this.tileCtx.save();
            this.tileCtx.scale(1, -1);
            this.tileCtx.drawImage(
                this.tileCanvas,
                0,
                0,
                dw / 2,
                dh / 2,
                0,
                -dh,
                dw / 2,
                dh / 2
            );

            this.tileCtx.scale(-1, 1);
            this.tileCtx.drawImage(
                this.tileCanvas,
                0,
                0,
                dw / 2,
                dh,
                -dw,
                -dh,
                dw / 2,
                dh
            );
            this.tileCtx.restore();

            /**
             * Use the finished tile as pattern and fill the output canvas
             * Move the context origin to the center, so scaling the pattern tile scales from the center
             */

            this.ctx.clearRect(
                0,
                0,
                this.ctx.canvas.width,
                this.ctx.canvas.height
            );

            let tilePattern = this.ctx.createPattern(this.tileCanvas, "repeat");
            this.ctx.fillStyle = tilePattern;
            let r = this.ctx.canvas.width / 2;

            this.ctx.save();
            this.ctx.translate(r, r);
            this.ctx.rotate((this.rotation * Math.PI) / 180);
            this.ctx.beginPath();
            this.ctx.arc(0, 0, r, 0, 2 * Math.PI);
            this.ctx.fill();
            this.ctx.restore();
        },
        updateRotator(value) {
            this.rotation = value;
        },
    },
};
</script>

<style scoped>
.panel {
    background: rgb(247, 247, 240);
    width: calc(50% - 120px);
    display: flex;
    align-items: center;
    justify-content: center;
}
@media only screen and (max-width: 1024px) {
    .panel {
        width: 50%;
    }
}

.lasagne-container {
    /* background: white; */
    padding: 5%;
    background: rgb(226, 206, 139);
}

.lasagne {
    position: relative;
    overflow: hidden; /* Cuts off the rotator bounding box */
}

canvas {
    max-width: 100%;
}

#canvas_destination {
    padding: 5%;
}

.canvas_thumbnail {
    width: 102px;
    height: 102px;
    display: inline-flex;
    justify-content: center;
    background: rgb(243, 232, 232);
}
@media only screen and (max-width: 1024px) {
    .canvas_thumbnail {
        width: 80px;
    }
}
.canvas_thumbnail canvas {
    object-fit: contain;
    border: 1px solid black;
}

.tile-display {
    margin-top: 40px;
    display: inline-block;
}
.tile-display:not(:first-of-type) {
    margin-left: 12px;
}
.tile-display label {
    display: block;
    font-variant: small-caps;
}
</style>