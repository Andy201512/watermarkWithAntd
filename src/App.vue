<template>
    <div class="mainView">
        <div class="previewPart">
            <div id="previewContainer">
                <div id="backgroundContainer" ref="backgroundContainer">
                    <div id="watermarkContainer" ref="watermarkContainer" @mousedown="RecordPos"
                        @mouseup="{ dragging = false; }" @mousemove="SetTranslateSyle">
                    </div>
                </div>
            </div>
        </div>
        <div class="operatePart">
            <a-slider :value="rusultData.watermarkData.offsetX"/>
            <a-slider :value="rusultData.watermarkData.offsetY"/>
        </div>
        <div class="paramPart">
            {{ rusultData.backgroundData.scale }}
            {{ rusultData.watermarkData.scale }}
            {{ rusultData.watermarkData.offsetX }}
            {{ rusultData.watermarkData.offsetY }}
        </div>
    </div>
</template>

<script setup>
import { reactive } from 'vue'

const rusultData = reactive({
    backgroundData: {
        scale: 0,
    },
    watermarkData: {
        scale: 0,
        offsetX: 0,
        offsetY: 0,
    },
});

var dragging = false;

var lastCursorPosX = 0;
var lastCursorPosY = 0;

var watermarkTranslateX = 0;
var watermarkTranslateY = 0;

function RecordPos(e) {
    dragging = true;

    lastCursorPosX = e.clientX;
    lastCursorPosY = e.clientY;

    if (e.target.style.transform.match(/translate/g)) {
        let lastWatermarkXY = e.target.style.transform.match(/-?\d+/g);
        watermarkTranslateX = parseInt(lastWatermarkXY[0]);
        watermarkTranslateY = parseInt(lastWatermarkXY[1]);
    }
};

function SetTranslateSyle(e) {
    if (dragging) {
        rusultData.watermarkData.offsetX = watermarkTranslateX + (e.clientX - lastCursorPosX);
        rusultData.watermarkData.offsetY = watermarkTranslateY + (e.clientY - lastCursorPosY);

        e.target.style.transform = `translate(${rusultData.watermarkData.offsetX}px, ${rusultData.watermarkData.offsetY}px)`;
    }
};

</script>

<style scoped>
@import "./assets/moreThan900px.css" screen and (min-width: 900px);
@import "./assets/lessThan900px.css" screen and (max-width: 900px);

#previewContainer {
    height: 100%;
    width: 100%;
    padding: 19.1%;
    display: flex;
    justify-content: center;
    align-items: center;
}

#backgroundContainer {
    height: 100%;
    width: 100%;
    position: relative;
    border: dashed red;
    background-image: url("./assets/background.jpg");
    background-color: whitesmoke;
    background-size: contain;
    background-position: center;
    background-repeat: no-repeat;
}

#watermarkContainer {
    position: absolute;
    top: 0;
    left: 0;
    height: 50px;
    width: 50px;
    background-image: url("./assets/watermark.png");
    background-size: contain;
    background-position: center;
    background-repeat: no-repeat;
}

#watermarkContainer:hover {
    cursor: move;
    filter: grayscale(20%);
}

#watermarkContainer:active {
    cursor: grab;
    filter: none;
}

.paramPart{
}
</style>
