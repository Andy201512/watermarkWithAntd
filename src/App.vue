<template>
    <a-config-provider :theme="{
        token: {
            colorPrimary: '#f1c58f',
        },
    }">
        <div class="mainView">
            <div class="previewPart">
                <div id="previewContainer">
                    <div id="backgroundContainer" ref="backgroundContainer">
                        <div id="watermarkContainer" ref="watermarkContainer" @mousedown="RecordPos"
                            @mouseup="{ dragging = false; }" @mousemove="SetDraggingTranslateSyle">
                        </div>
                    </div>
                </div>
            </div>
            <div class="operatePart">
                <a-row>
                    <a-col :span="8" class="labelContainer">
                        <span>水印横向偏移：</span>
                    </a-col>
                    <a-col :span="16">
                        <a-slider v-model:value="rusultData.watermarkData.offsetX" :marks="xMarks" :max="xMax" @change="(value)=>SetSliderTranslateSyle(value,'x')"/>
                    </a-col>
                </a-row>
                <a-row>
                    <a-col :span="8" class="labelContainer">
                        <span>水印纵向偏移：</span>
                    </a-col>
                    <a-col :span="16">
                        <a-slider v-model:value="rusultData.watermarkData.offsetY" :marks="yMarks" :max="yMax"  @change="(value)=>SetSliderTranslateSyle(value, 'y')"/>
                    </a-col>
                </a-row>
            </div>
            <div class="paramPart">
                {{ rusultData.backgroundData.scale }}
                {{ rusultData.watermarkData.scale }}
                {{ rusultData.watermarkData.offsetX }}
                {{ rusultData.watermarkData.offsetY }}
            </div>
        </div>
    </a-config-provider>
</template>

<script setup>
import { reactive, ref, onMounted, useTemplateRef } from 'vue'

// 图片打水印的关键数据
const rusultData = ref({
    backgroundData: {
        scale: 0,
    },
    watermarkData: {
        scale: 0,
        offsetX: 0,
        offsetY: 0,
    },
});

const backgroundContainerRef = useTemplateRef("backgroundContainer");
const watermarkContainerRef = useTemplateRef("watermarkContainer");

// 水印进入滑动过程的标志
var dragging = false;

// 水印开始滑动时的鼠标位置
var lastCursorPosX = 0;
var lastCursorPosY = 0;

// 从css里读取的水印偏移量
var watermarkTranslateX = 0;
var watermarkTranslateY = 0;

// 滑动条取值范围
const xMarks = ref({
    0: '0px',
    100: '100px'
});
const yMarks = ref({
    0: '0px',
    100: '100px'
});
const xMax = ref(100)
const yMax = ref(100)

// 记录拖动开始
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

// 设置水印偏移
function SetDraggingTranslateSyle(e) {
    if (dragging) {
        rusultData.value.watermarkData.offsetX = watermarkTranslateX + (e.clientX - lastCursorPosX);
        rusultData.value.watermarkData.offsetY = watermarkTranslateY + (e.clientY - lastCursorPosY);

        if (
            rusultData.value.watermarkData.offsetX >= 0 &&
            rusultData.value.watermarkData.offsetX <= xMax.value &&
            rusultData.value.watermarkData.offsetY >= 0 &&
            rusultData.value.watermarkData.offsetY <= yMax.value
        ) {
            e.target.style.transform = `translate(${rusultData.value.watermarkData.offsetX}px, ${rusultData.value.watermarkData.offsetY}px)`;

        }
    }
};

function SetSliderTranslateSyle(value, flage) {
    var lastWatermarkXY = watermarkContainerRef.value.style?.transform.match(/-?\d+/g) || [0,0];
    var offsetX = parseInt(lastWatermarkXY[0]) || 0;
    var offsetY = parseInt(lastWatermarkXY[1]) || 0;
    if(flage === 'x'){
        offsetX = value
    }
    if(flage === 'y'){
        offsetY = value
    }
    watermarkContainerRef.value.style.transform = `translate(${offsetX}px, ${offsetY}px)`
}

onMounted(() => {
    // 设置滑动条的水印偏移值范围
    let height = backgroundContainerRef.value.clientHeight - watermarkContainerRef.value.clientHeight;
    let width = backgroundContainerRef.value.clientWidth - watermarkContainerRef.value.clientWidth;
    delete yMarks.value[100];
    delete xMarks.value[100];
    yMarks.value[height] = `${height}px`;
    xMarks.value[width] = `${width}px`;
    yMax.value = height;
    xMax.value = width;
});

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
    transform: translate(0px,0px);
}

#watermarkContainer:hover {
    cursor: move;
    filter: grayscale(20%);
}

#watermarkContainer:active {
    cursor: grab;
    filter: none;
}

.operatePart {
    padding: 20px;
}

.labelContainer {
    align-content: center;
    text-align: center;
}

.paramPart {}
</style>
