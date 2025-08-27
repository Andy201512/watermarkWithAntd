<template>
    <a-config-provider :theme="{
        token: {
            colorPrimary: '#f1c58f',
        },
    }">
        <div class="mainView">
            <div class="previewPart">
                <div id="previewContainer">
                    <div id="backgroundContainer" ref="backgroundContainer"
                        @dragover="(event) => { event.preventDefault() }">
                        <div id="watermarkContainer" ref="watermarkContainer" draggable="true" @dragstart="RecordPos"
                            @dragend="SetDraggingTranslateSyle" tabindex="0" @keydown="handleKeydown">
                        </div>
                    </div>
                </div>
            </div>
            <a-float-button @click="handleClick" :style="{
                top: '45%',
                right: '39%',
            }">
                <template #icon>
                    <DownloadOutlined />
                </template>
            </a-float-button>
            <div class="operatePart">
                <div>
                    <a-slider v-model:value="rusultData.watermarkData.offsetX" :marks="xMarks" :max="xMax" />
                </div>
                <div>
                    <div id="verticalSlider">
                        <a-slider vertical reverse v-model:value="rusultData.watermarkData.offsetY" :marks="yMarks"
                            :max="yMax" />
                    </div>
                    <div>
                        <a-row>
                            <a-col :span="12" class="labelContainer">
                                <span>上传背景图片：</span>
                            </a-col>
                            <a-col :span="12">
                                <a-upload v-model:file-list="fileList" name="file"
                                    action="https://www.mocky.io/v2/5cc8019d300000980a055e76" :headers="headers"
                                    @change="handleChange">
                                    <a-button>
                                        <upload-outlined></upload-outlined>
                                        Click to Upload
                                    </a-button>
                                </a-upload>
                            </a-col>
                        </a-row>
                        <a-row>
                            <a-col :span="12" class="labelContainer">
                                <span>上传水印图片：</span>
                            </a-col>
                            <a-col :span="12">
                                <a-upload v-model:file-list="fileList" name="file"
                                    action="https://www.mocky.io/v2/5cc8019d300000980a055e76" :headers="headers"
                                    @change="handleChange">
                                    <a-button>
                                        <upload-outlined></upload-outlined>
                                        Click to Upload
                                    </a-button>
                                </a-upload>
                            </a-col>
                        </a-row>
                        <a-row>
                            <a-col :span="12" class="labelContainer">
                                <span>水印横向偏移：</span>
                            </a-col>
                            <a-col :span="12">
                                <a-input-number v-model:value="rusultData.watermarkData.offsetX" addonAfter="px"
                                    :min="0" :max="xMax" @pressEnter="(event) => { event.target.blur(); }" />
                            </a-col>
                        </a-row>
                        <a-row>
                            <a-col :span="12" class="labelContainer">
                                <span>水印纵向偏移：</span>
                            </a-col>
                            <a-col :span="12">
                                <a-input-number v-model:value="rusultData.watermarkData.offsetY" addonAfter="px"
                                    :min="0" :max="yMax" @pressEnter="(event) => { event.target.blur(); }" />
                            </a-col>
                        </a-row>
                        <a-row>
                            <a-col :span="12" class="labelContainer">
                                <span>背景图片缩放比：</span>
                            </a-col>
                            <a-col :span="12">
                                <a-input-number v-model:value="rusultData.backgroundData.scale" addonAfter="%"
                                    disabled />
                            </a-col>
                        </a-row>
                        <a-row>
                            <a-col :span="12" class="labelContainer">
                                <span>水印图片缩放比：</span>
                            </a-col>
                            <a-col :span="12">
                                <a-input-number v-model:value="rusultData.watermarkData.scale" addonAfter="%" />
                            </a-col>
                        </a-row>
                    </div>
                </div>
            </div>
        </div>
    </a-config-provider>
</template>

<script setup>
import { ref, onMounted, useTemplateRef, watch } from 'vue'
import { DownloadOutlined } from '@ant-design/icons-vue';


// 图片打水印的关键数据
const rusultData = ref({
    backgroundData: {
        scale: 100,
    },
    watermarkData: {
        scale: 100,
        offsetX: 0,
        offsetY: 0,
    },
});

watch(rusultData, (newData) => {
    watermarkContainerRef.value.style.transform = `translate(${newData.watermarkData.offsetX}px, ${newData.watermarkData.offsetY}px)`;
}, { deep: true });

const backgroundContainerRef = useTemplateRef("backgroundContainer");
const watermarkContainerRef = useTemplateRef("watermarkContainer");

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

    lastCursorPosX = e.clientX;
    lastCursorPosY = e.clientY;


    let lastWatermarkXY = e.target.style.transform.match(/-?\d+/g) || [0, 0];
    watermarkTranslateX = parseInt(lastWatermarkXY[0]) || 0;
    watermarkTranslateY = parseInt(lastWatermarkXY[1]) || 0;

};

// 拖拽水印偏移
function SetDraggingTranslateSyle(e) {

    let offsetX = watermarkTranslateX + (e.clientX - lastCursorPosX);
    let offsetY = watermarkTranslateY + (e.clientY - lastCursorPosY);

    if (
        offsetX >= 0 &&
        offsetX <= xMax.value &&
        offsetY >= 0 &&
        offsetY <= yMax.value
    ) {
        rusultData.value.watermarkData.offsetX = offsetX
        rusultData.value.watermarkData.offsetY = offsetY
    }

};

// 键盘方向键水印偏移
function handleKeydown(event) {

    switch (event.code) {
        case "ArrowDown":
            // Handle "down"
            if (rusultData.value.watermarkData.offsetY > -1 &&
                rusultData.value.watermarkData.offsetY < yMax.value
            ) { rusultData.value.watermarkData.offsetY++ };
            break;
        case "ArrowUp":
            // Handle "up"
            if (rusultData.value.watermarkData.offsetY > 0 &&
                rusultData.value.watermarkData.offsetY < yMax.value + 1
            ) { rusultData.value.watermarkData.offsetY-- };
            break;
        case "ArrowLeft":
            // Handle "turn left"
            if (rusultData.value.watermarkData.offsetX > 0 &&
                rusultData.value.watermarkData.offsetX < xMax.value + 1
            ) { rusultData.value.watermarkData.offsetX-- };
            break;
        case "ArrowRight":
            // Handle "turn right"
            if (rusultData.value.watermarkData.offsetX > -1 &&
                rusultData.value.watermarkData.offsetX < xMax.value
            ) { rusultData.value.watermarkData.offsetX++ };
            break;
    }
}

function updateFundermantialData() {
    rusultData.value.watermarkData.offsetX = 0;
    rusultData.value.watermarkData.offsetY = 0;

    // 设置滑动条的水印偏移值范围
    let height = backgroundContainerRef.value.clientHeight - watermarkContainerRef.value.clientHeight;
    let width = backgroundContainerRef.value.clientWidth - watermarkContainerRef.value.clientWidth;
    delete yMarks.value[yMax.value];
    delete xMarks.value[xMax.value];
    yMarks.value[height] = `${height}px`;
    xMarks.value[width] = `${width}px`;
    yMax.value = height;
    xMax.value = width;
}

onMounted(() => {
    updateFundermantialData()
    window.addEventListener('resize', updateFundermantialData);
});

</script>

<style scoped>
@import "./assets/moreThan900px.css" screen and (min-width: 900px);
@import "./assets/lessThan900px.css" screen and (max-width: 900px);

#previewContainer {
    height: 100%;
    width: 100%;
    padding: 5% 10%;
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
    background-image: url("./assets/mark.svg");
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

.operatePart {
    display: flex;
    flex-direction: column;
    margin: 5%;
    padding: 5%;
    border: dashed gray;
}

.operatePart>div:nth-child(1) {
    flex-grow: 1;
    width: 90%;
    align-self: center;
}

.operatePart>div:nth-child(2) {
    flex-grow: 9;
    display: flex;
    flex-direction: row;
}

#verticalSlider {
    flex-grow: 5;
    height: 90%;
}

#verticalSlider+div {
    flex-grow: 5;
    display: flex;
    flex-direction: column;
    align-content: space-around;
}

#verticalSlider+div>div {
    flex-grow: 5;
    align-content: center;
}

.labelContainer {
    align-content: center;
    text-align: center;
}
</style>
