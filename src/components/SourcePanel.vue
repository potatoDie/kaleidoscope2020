<template>
    <div class="panel">
        <div class="lasagne-container">
            <div class="lasagne">
                <canvas id="canvas_source"></canvas>
                <Rotator :rotation="rotation" @update="updateRotator" />
                <CrossHairs
                    :position="position"
                    :size="visorSize"
                    :container="container"
                    @update="updateCrossHairs"
                />
            </div>
        </div>
        <teleport to="#dashboard-source">
            <p>
                File: <b>{{ fileName }}</b> <br />
                <img :src="imageSrc" class="thumbnail" alt="" /><br />
                Original size: {{ store.width }} x {{ store.height }} px
            </p>
            <form>
                <FileInput
                    v-on:input="updateImageSrc"
                    v-bind:imageSrc="imageSrc"
                />
            </form>
            <form>
                Mode: <br />
                <input
                    type="radio"
                    id="one"
                    value="rectangle"
                    v-model="store.mode"
                />
                <label for="one">Rectangle</label>
                <input
                    type="radio"
                    id="two"
                    value="triangle"
                    v-model="store.mode"
                />
                <label for="two">Triangle</label>
            </form>
            <form>
                <Slider
                    :label="sourceTileLabel"
                    v-model="visorSize.width"
                    :min="1"
                    :max="1000"
                />
                <Slider
                    v-show="store.mode == 'rectangle'"
                    label="Source tile height"
                    v-model="visorSize.height"
                    :min="1"
                    :max="1000"
                />
            </form>
        </teleport>
    </div>
</template>

<script>
import Rotator from "./Rotator.vue";
import CrossHairs from "./CrossHairs.vue";
import Slider from "./Slider.vue";
import FileInput from "./FileInput.vue";
import imageSrc from "../assets/p059.jpg";
import store from "../store.js";
// import VueSlider from "vue-slider-component";
// import "vue-slider-component/theme/default.css";

const sqrt3 = Math.sqrt(3);

export default {
    data() {
        return {
            m: 33,
            ctx: null,
            rotation: 70,
            image: null, // image object whose source could be...
            imageSrc: imageSrc, // which can be changed by file input element
            position: {
                // Where is the crosshairs (center of visor)?
                x: 0,
                y: 0,
            },
            visorSize: {
                width: 600,
                height: 480, // Chosen so an equilateral triangle fits in the visor
            },
            container: {
                // Determines viewBox dimensions for the crossHairs component
                // Equals dimensions of canvas and image (intrinsic)
                width: 0,
                height: 0,
            },
            fileName: "p059.jpg",

            store: store, // global data, shared with other components
        };
    },
    computed: {
        sourceTileLabel() {
            return this.store.mode == "triangle"
                ? "Source tile size"
                : "Source tile width";
        },
    },
    components: {
        Rotator,
        CrossHairs,
        // VueSlider,
        Slider,
        FileInput,
    },
    mounted() {
        this.ctx = document.getElementById("canvas_source").getContext("2d");

        if (this.store.mode == "triangle") {
            this.adjustVisorHeight();
        }
        this.handleImage();
    },
    watch: {
        rotation() {
            this.updateCanvas();
            this.cutOutTile();
        },
        visorSize: {
            // Perhaps better not to watch but use @change instead of v-model
            handler() {
                this.adjustVisorHeight();

                this.cutOutTile();
            },
            deep: true,
        },
        "store.mode"() {
            this.adjustVisorHeight();
        },
    },
    methods: {
        handleImage() {
            // Create image object. Load image data into it (as source). Wait till it's loaded
            this.image = new Image();

            this.image.addEventListener("load", () => {
                this.initCanvasses(this.image);
                // this.positionCrossHairs();
                this.cutOutTile();
            });
            this.image.src = this.imageSrc;
        },
        initCanvasses(image) {
            let w = image.naturalWidth,
                h = image.naturalHeight,
                d = Math.sqrt(w * w + h * h);

            // Perhaps you can keep width and height private?
            this.store.width = w;
            this.store.height = h;
            this.store.diameter = Math.ceil(d);

            this.ctx.canvas.width = d;
            this.ctx.canvas.height = d;

            this.container.width = d;
            this.container.height = d;

            // Start position in the middle
            this.position.x = this.position.y = d / 2;

            this.ctx.translate(
                this.store.diameter / 2,
                this.store.diameter / 2
            );

            this.updateCanvas();
        },
        updateCanvas: function () {
            let w = this.store.width;
            let h = this.store.height;
            let d = this.store.diameter;

            let ctx = this.ctx;

            // Clear image
            ctx.clearRect(-d / 2, -d / 2, d, d);

            ctx.save();
            ctx.rotate((this.rotation * Math.PI) / 180);
            ctx.drawImage(this.image, -w / 2, -h / 2);
            ctx.restore();
        },
        cutOutTile() {
            this.store.sourceTile = this.ctx.getImageData(
                this.position.x - this.visorSize.width / 2,
                this.position.y - this.visorSize.height / 2,
                this.visorSize.width,
                this.visorSize.height
            );
        },
        updateRotator(value) {
            this.rotation = value;
        },
        updateCrossHairs(delta) {
            this.position.x += delta.x;
            this.position.y += delta.y;

            this.cutOutTile(); // Perhaps watch position?
        },
        updateImageSrc: function (imgData, fileName) {
            this.fileName = fileName;
            this.imageSrc = imgData;
            this.handleImage();
        },
        adjustVisorHeight() {
            if (this.store.mode == "triangle") {
                // Ignore height slider value and calculate height from width
                // TODO: adjust slider view depending on mode
                this.visorSize.height = Math.round(
                    (this.visorSize.width * sqrt3) / 2
                );
            }
        },
    },
};
</script>

<style scoped>
.panel {
    background: rgb(243, 232, 232);
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
    background: white;
    padding: 5%;
}

.lasagne {
    position: relative;
    overflow: hidden; /* Cuts off the rotator bounding box */
}
#canvas_source {
    max-width: 100%;
}

form {
    margin-bottom: 32px;
}

img.thumbnail {
    width: 102px;
    margin: 6px 12px 0px 0;
}
</style>