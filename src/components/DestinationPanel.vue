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
                    <canvas
                        v-show="store.mode == 'rectangle'"
                        id="canvas_fragment"
                    ></canvas>
                    <canvas
                        v-show="store.mode == 'triangle'"
                        id="canvas_clip"
                    ></canvas>
                </div>
            </div>
            <div class="tile-display">
                <label>Pattern tile</label>
                <div class="canvas_thumbnail">
                    <canvas id="canvas_pattern"></canvas>
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
            destinationCtx: null,
            patternCtx: null,
            fragmentCtx: null,
            clipCtx: null,
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
        this.destinationCtx = document
            .getElementById("canvas_destination")
            .getContext("2d");

        // Create a canvas to construct pattern
        this.patternCtx = document
            .getElementById("canvas_pattern")
            .getContext("2d");

        // We'll also need a canvas to put the source fragment on.
        // I used patternCtx.canvas for this first, but that canvas may be too small if destination dimensions are small
        this.fragmentCtx = document
            .getElementById("canvas_fragment")
            .getContext("2d");

        this.clipCtx = document.getElementById("canvas_clip").getContext("2d");
    },
    watch: {
        "store.diameter"(value) {
            this.destinationCtx.canvas.width = value;
            this.destinationCtx.canvas.height = value;
        },
        "store.sourceTile"(value) {
            this.update(value);
        },
        "store.mode"(value) {
            this.update(this.store.sourceTile);
        },
        patternScale() {
            this.update(this.store.sourceTile);
        },
        rotation() {
            this.update(this.store.sourceTile);
        },
    },
    methods: {
        update(imageData) {
            /**
             * Create a rectangular tile that is used as pattern.
             * Then redraw / fill the output (destination) canvas
             */

            let fCanvas, pCanvas;

            if (this.store.mode == "rectangle") {
                // Draw fragment on canvas
                // This step is needed because putImageData doesn't take the canvas transformation matrix and clipping into account.
                fCanvas = this.drawFragmentCanvasRectangle(imageData);

                // Draw pattern rectangle to be used for fill pattern without seams
                pCanvas = this.drawPatternCanvasRectangle(fCanvas);
            } else {
                // Draw clipped fragment on canvas
                fCanvas = this.drawFragmentCanvasTriangle(imageData);

                // Draw pattern rectangle to be used for fill pattern without seams
                pCanvas = this.drawPatternCanvasTriangle(fCanvas);
            }

            let ctx = this.destinationCtx;

            // Use the finished tile as pattern
            ctx.fillStyle = ctx.createPattern(pCanvas, "repeat");

            // Fill the output canvas
            this.draw(ctx);
        },
        drawFragmentCanvasRectangle(imageData) {
            this.fragmentCtx.canvas.width = imageData.width;
            this.fragmentCtx.canvas.height = imageData.height;
            this.fragmentCtx.putImageData(imageData, 0, 0);

            return this.fragmentCtx.canvas;
        },
        drawFragmentCanvasTriangle(imageData) {
            // So mode is 'triangle'. Let's assume that the size of the triangle
            // is represented by width

            this.fragmentCtx.canvas.width = imageData.width;
            this.fragmentCtx.canvas.height = imageData.height;
            this.fragmentCtx.putImageData(imageData, 0, 0);

            const clipCtx = this.clipCtx;
            clipCtx.canvas.width = imageData.width;
            clipCtx.canvas.height = imageData.height;

            clipCtx.beginPath();
            clipCtx.moveTo(0, Math.ceil(imageData.height));
            clipCtx.lineTo(imageData.width, Math.ceil(imageData.height));
            clipCtx.lineTo(imageData.width / 2, 0);
            clipCtx.clip();

            clipCtx.drawImage(this.fragmentCtx.canvas, 0, 0);

            return clipCtx.canvas;
        },
        drawPatternCanvasRectangle(fCanvas) {
            const w = fCanvas.width;
            const h = fCanvas.height;

            const dw = (w * 2 * this.patternScale) / 100;
            const dh = (h * 2 * this.patternScale) / 100;

            const pCtx = this.patternCtx;
            const pCanvas = pCtx.canvas;

            pCanvas.width = dw;
            pCanvas.height = dh;

            // Scale the imagedata from fragmentCtx.canvas, to pCanvas
            // This creates the left top quadrant
            pCtx.drawImage(fCanvas, 0, 0, w, h, 0, 0, dw / 2, dh / 2);

            // You could do the same for the other 3 quadrants. We can also draw the canvas on it self
            // Saving one drawImage call
            pCtx.save();
            pCtx.scale(1, -1);
            pCtx.drawImage(
                pCanvas,
                0,
                0,
                dw / 2,
                dh / 2,
                0,
                -dh,
                dw / 2,
                dh / 2
            );

            pCtx.scale(-1, 1);
            pCtx.drawImage(pCanvas, 0, 0, dw / 2, dh, -dw, -dh, dw / 2, dh);
            pCtx.restore();

            return pCanvas;
        },
        drawPatternCanvasTriangle(fCanvas) {
            const d = fCanvas.width;
            const h = fCanvas.height;

            // Nou misschien is het wel makkelijker om eerst een pattern-tegel te maken met
            // ongeschaalde fragmenten. En pas als het patroon klaar is er een geschaalde versie van te maken
            // Simpeler, en je laat drawImage ook niet steeds dezelfde herschaling uitvoeren.
            // Kwaliteit tenslotte kan ook beter uitpakken
            // ??? De laatste schalingsslag kan binnen pCanvas plaatsvinden: drawImage op pCanvas zelf, vervolgens
            // dimensies aanpassen.

            const dw = d * 3;
            const dh = h * 2;

            const pCtx = this.patternCtx;
            const pCanvas = pCtx.canvas;

            pCanvas.width = dw;
            pCanvas.height = dh;

            this.createTriangleSymmetryRow(pCtx, fCanvas);

            // Mirror what we have so far
            pCtx.save();
            pCtx.scale(1, -1);
            pCtx.drawImage(pCanvas, 0, -2 * h);

            pCtx.restore();

            // Herschaal using this.patternScale
            const sw = (dw * this.patternScale) / 100;
            const sh = (dh * this.patternScale) / 100;

            // Create new canvas with these dimensions
            let sCanvas = document.createElement("canvas");
            let sCtx = sCanvas.getContext("2d");

            sCanvas.width = sw;
            sCanvas.height = sh;

            // Then copy pCanvas onto it
            sCtx.drawImage(pCanvas, 0, 0, dw, dh, 0, 0, sw, sh);

            return sCanvas;
        },
        createTriangleSymmetryRow(ctx, triangleCanvas) {
            const d = triangleCanvas.width;
            const h = triangleCanvas.height;

            // The center of the triangle is 2/3 of the height from the top
            const b = Math.round(h / 3),
                c = Math.round((2 * h) / 3);

            // This works as well, let me explain
            // translate to the point where you want the (rotational) center of the triangle to be
            // If you rotate an odd number times pi/3 the triangle will point downwards and the centre
            // should lie h/3 from the top of the canvas
            // If you rotate an even number times pi/3 the triangle will point upwards (like the original) and the
            // centre should lie 2*h/3 from the top of the canvas
            for (let i = 0; i < 7; i++) {
                ctx.save();
                if (i % 2) {
                    ctx.translate((i * d) / 2, b);
                    ctx.scale(-1, 1);
                } else {
                    ctx.translate((i * d) / 2, c);
                }
                ctx.rotate((i * Math.PI) / 3);
                ctx.scale(1.01, 1.01); // Avoids tiny slits
                ctx.drawImage(triangleCanvas, -d / 2, -c);
                ctx.restore();
            }
        },
        draw(ctx) {
            ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);

            let r = ctx.canvas.width / 2;

            ctx.save();

            // Move the context origin to the center, so scaling the pattern tile scales from the center
            ctx.translate(r, r);

            ctx.rotate((this.rotation * Math.PI) / 180);
            ctx.beginPath();
            ctx.arc(0, 0, r, 0, 2 * Math.PI);
            ctx.fill();
            ctx.restore();
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
    height: calc(102px * 0.866); /* Fit triangle in thumbnail */
    display: inline-flex;
    justify-content: center;
    background-color: #eee;
    background-image: linear-gradient(
            45deg,
            #aaa 25%,
            transparent 25%,
            transparent 75%,
            #aaa 75%,
            #aaa
        ),
        linear-gradient(
            45deg,
            #aaa 25%,
            transparent 25%,
            transparent 75%,
            #aaa 75%,
            #aaa
        );
    background-size: 12px 12px;
    background-position: 0 0, 6px 6px;
    border: 1px solid black;
}
@media only screen and (max-width: 1024px) {
    .canvas_thumbnail {
        width: 80px;
    }
}
.canvas_thumbnail canvas {
    object-fit: contain;
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