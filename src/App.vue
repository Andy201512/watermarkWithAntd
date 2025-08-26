<template>
    <div class="mainView">
        <div class="previewPart">
            <div id="previewContainer">
                <div id="backgroundContainer" ref="backgroundContainer">
                    <div id="watermarkContainer" ref="watermarkContainer"></div>
                </div>
            </div>
        </div>
        <div class="operatePart">
            <a-slider />
        </div>
        <div class="paramPart">
        </div>
    </div>
</template>

<script setup>
import { useTemplateRef, onMounted } from 'vue'
import { debounce } from './assets/utils';

const rusultData = {
    backgroundData: {
        scale: 0,
    },
    watermarkData: {
        scale: 0,
        offsetx: 0,
        offsety: 0,
    },
}

var watermarkContainer = useTemplateRef("watermarkContainer");
var dragging = false;
var lastCursorPosX = 0;
var lastCursorPosY = 0;
var lastWatermarkX = 0;
var lastWatermarkY = 0; 

onMounted(() => {

    watermarkContainer.value.addEventListener('mousedown', function (e) {
        dragging = true;
        lastCursorPosX = e.clientX;
        lastCursorPosY = e.clientY;
        if(watermarkContainer.value.style.transform.match(/translate/g)){
            let lastWatermarkXY = watermarkContainer.value.style.transform.match(/-?\d+/g);
            lastWatermarkX = parseInt(lastWatermarkXY[0]);
            lastWatermarkY = parseInt(lastWatermarkXY[1]);
        }
    });

    watermarkContainer.value.addEventListener('mouseup', function (e) {
        dragging = false;
    });

    watermarkContainer.value.addEventListener('mouseleave', function (e) {
        dragging = false;
    });

    watermarkContainer.value.addEventListener('mousemove', function (e) {
        if (dragging) {
            let translateX = lastWatermarkX + (e.clientX - lastCursorPosX);
            let translateY = lastWatermarkY + (e.clientY - lastCursorPosY);

            watermarkContainer.value.style.transform = `translate(${translateX}px, ${translateY}px)`;
        }
    });
})

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
    background-color: hsl(300, 100%, 50%);
}

#watermarkContainer:hover {
    cursor: move;
    filter: grayscale(20%);
}

#watermarkContainer:active {
    cursor: grab;
    filter: none;
}
</style>
