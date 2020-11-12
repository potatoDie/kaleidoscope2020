<template>
    <div>
        <input
            type="file"
            id="file"
            v-on:change="readData($event.target.files[0])"
        />
        <label for="file">Choose image</label>
    </div>
</template>

<script>
export default {
    data: function () {
        return {
            fileName: "",
        };
    },
    props: ["imageSrc"],
    methods: {
        readData: function (file) {
            const reader = new FileReader();
            this.fileName = file.name;

            reader.onload = () => {
                this.$emit("input", reader.result, file.name);
            };

            reader.readAsDataURL(file);
        },
    },
};
</script>

<style>
[type="file"] {
    border: 0;
    clip: rect(0, 0, 0, 0);
    height: 1px;
    overflow: hidden;
    padding: 0;
    position: absolute !important;
    white-space: nowrap;
    width: 1px;
}

[type="file"] + label {
    background-color: rgb(218, 58, 0);
    border-radius: 2px;
    color: #fff;
    cursor: pointer;
    display: inline-block;
    padding: 10px;
    letter-spacing: 0.5px;
    text-transform: uppercase;
}

/* [type="file"]:focus + label, */
[type="file"] + label:hover {
    background-color: #ba3500;
}

/* [type="file"]:focus + label {
    outline: 1px dotted #000;
} */
</style>