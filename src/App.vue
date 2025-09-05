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
            <a-float-button @click="handleClick" :style="isMoreThan900px ? {
                top: '45%',
                right: '39%',
            } : {
                top: '90%',
                left: '13%',
            }">
                <template #icon>
                    <DownloadOutlined />
                </template>
            </a-float-button>
            <a-float-button v-if="!isMoreThan900px" @click="handleBgUploadClick" :style="{
                top: '90%',
                left: '46%',
            }">
                <template #icon>
                    <FileImageOutlined />
                    <input type="file" ref="bgInput" @change="handleBgInputChange" style="display:none" />
                </template>
            </a-float-button>
            <a-float-button v-if="!isMoreThan900px" @click="handleWmUploadClick" :style="{
                top: '90%',
                left: '80%',
            }">
                <template #icon>
                    <HeartOutlined />
                    <input type="file" ref="wmInput" @change="handleWmInputChange" style="display:none" />
                </template>
            </a-float-button>
            <div class="operatePart" v-if="isMoreThan900px">
                <div>
                    <a-slider v-model:value="keyData.watermarkData.offsetX" :marks="xMarks" :max="xMax" />
                </div>
                <div>
                    <div id="verticalSlider">
                        <a-slider vertical reverse v-model:value="keyData.watermarkData.offsetY" :marks="yMarks"
                            :max="yMax" />
                    </div>
                    <div>
                        <a-row>
                            <a-col :span="12" class="labelContainer">
                                <span>上传背景图片：</span>
                            </a-col>
                            <a-col :span="12">
                                <a-upload v-model:file-list="bFileList" name="backgroundFile"
                                    :beforeUpload="beforeUploadBG" @change="handleChange" :maxCount="1">
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
                                <a-upload v-model:file-list="wFileList" name="watermarkFile"
                                    :beforeUpload="beforeUploadWM" @change="handleChange" :maxCount="1">
                                    <a-button>
                                        <upload-outlined></upload-outlined>
                                        Click to Upload
                                    </a-button>
                                </a-upload>
                            </a-col>
                        </a-row>
                        <a-row>
                            <a-col :span="12" class="labelContainer">
                                <span>背景图片缩放比：</span>
                            </a-col>
                            <a-col :span="12">
                                <a-input-number v-model:value="keyData.backgroundData.scale" disabled />
                            </a-col>
                        </a-row>
                        <a-row>
                            <a-col :span="12" class="labelContainer">
                                <span>水印图片缩放比：</span>
                            </a-col>
                            <a-col :span="12">
                                <a-input-number v-model:value="keyData.watermarkData.scale" disabled />
                            </a-col>
                        </a-row>
                        <a-row>
                            <a-col :span="12" class="labelContainer">
                                <span>水印横向偏移：</span>
                            </a-col>
                            <a-col :span="12">
                                <a-input-number v-model:value="keyData.watermarkData.offsetX" addonAfter="px" :min="0"
                                    :max="xMax" @pressEnter="(event) => { event.target.blur(); }" />
                            </a-col>
                        </a-row>
                        <a-row>
                            <a-col :span="12" class="labelContainer">
                                <span>水印纵向偏移：</span>
                            </a-col>
                            <a-col :span="12">
                                <a-input-number v-model:value="keyData.watermarkData.offsetY" addonAfter="px" :min="0"
                                    :max="yMax" @pressEnter="(event) => { event.target.blur(); }" />
                            </a-col>
                        </a-row>
                    </div>
                </div>
            </div>
        </div>
    </a-config-provider>
</template>

<script setup>
import { ref, onMounted, useTemplateRef, watch, computed } from 'vue'
import { DownloadOutlined, FileImageOutlined, HeartOutlined } from '@ant-design/icons-vue';
import { UploadOutlined } from '@ant-design/icons-vue';
import { loadImage } from './assets/utils'

const bFileList = ref([{
    uid: '-1',
    name: 'background.jpg',
    status: 'done',
    url: './assets/background.jpg',
},]);
const wFileList = ref([{
    uid: '-1',
    name: 'mark.svg',
    status: 'done',
    url: './assets/mark.svg',
},]);


// 图片打水印的关键数据
const keyData = ref({
    backgroundData: {
        height: 640,
        width: 427,
        scale: 1,
    },
    watermarkData: {
        height: 31,
        width: 30,
        scale: 1,
        offsetX: 0,
        offsetY: 0,
    },
});

// 浏览器宽高
const windowData = ref({
    height: 0,
    width: 0,
})

watch(keyData, (newData) => {
    watermarkContainerRef.value.style.transform = `translate(${newData.watermarkData.offsetX}px, ${newData.watermarkData.offsetY}px)`;
}, { deep: true });

const isMoreThan900px = computed(() => { return windowData.value.width > 900 })

const backgroundContainerRef = useTemplateRef("backgroundContainer");
const watermarkContainerRef = useTemplateRef("watermarkContainer");
const bgInputRef = useTemplateRef("bgInput");
const wmInputRef = useTemplateRef("wmInput");

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
        keyData.value.watermarkData.offsetX = offsetX
        keyData.value.watermarkData.offsetY = offsetY
    }

};

// 键盘方向键水印偏移
function handleKeydown(event) {

    switch (event.code) {
        case "ArrowDown":
            // Handle "down"
            if (keyData.value.watermarkData.offsetY > -1 &&
                keyData.value.watermarkData.offsetY < yMax.value
            ) { keyData.value.watermarkData.offsetY++ };
            break;
        case "ArrowUp":
            // Handle "up"
            if (keyData.value.watermarkData.offsetY > 0 &&
                keyData.value.watermarkData.offsetY < yMax.value + 1
            ) { keyData.value.watermarkData.offsetY-- };
            break;
        case "ArrowLeft":
            // Handle "turn left"
            if (keyData.value.watermarkData.offsetX > 0 &&
                keyData.value.watermarkData.offsetX < xMax.value + 1
            ) { keyData.value.watermarkData.offsetX-- };
            break;
        case "ArrowRight":
            // Handle "turn right"
            if (keyData.value.watermarkData.offsetX > -1 &&
                keyData.value.watermarkData.offsetX < xMax.value
            ) { keyData.value.watermarkData.offsetX++ };
            break;
    }
}

// 上传事件
const handleChange = info => {
    let resFileList = [...info.fileList];

    if (info.file.status = 'uploading' && resFileList.length > 0) {
        resFileList[0].status = 'done'
    }
};

const beforeUploadBG = file => {
    handleImage(file, 'backgroundData');

    // 阻止真实上传
    return false;
};

const beforeUploadWM = file => {
    handleImage(file, 'watermarkData');

    // 阻止真实上传
    return false;
};

const handleBgUploadClick = () => {
    const event = new MouseEvent('click');
    bgInputRef.value.dispatchEvent(event);
};

const handleBgInputChange = (event) => {
    var file = event.target.files[0]; // 获取选中的第一个文件
    handleImage(file, 'backgroundData');
};

const handleWmUploadClick = () => {
    const event = new MouseEvent('click');
    wmInputRef.value.dispatchEvent(event);
};

const handleWmInputChange = (event) => {
    var file = event.target.files[0]; // 获取选中的第一个文件
    handleImage(file, 'watermarkData');
};

// 读取文件数据，不发起真实上传
const handleImage = (file, type) => {

    const reader = new FileReader();
    reader.readAsDataURL(file);
    reader.onload = () => {
        const img = document.createElement('img');
        img.src = reader.result;
        img.onload = () => {
            keyData.value[type].height = img.height;
            keyData.value[type].width = img.width;

            if (type === "backgroundData") {
                backgroundContainerRef.value.style.backgroundImage = `url(${img.src})`;
            };

            if (type === "watermarkData") {
                let scale = keyData.value.backgroundData.scale;
                watermarkContainerRef.value.style.backgroundImage = `url(${img.src})`;
                watermarkContainerRef.value.style.height = parseInt(img.height * scale) + "px";
                watermarkContainerRef.value.style.width = parseInt(img.width * scale) + "px";
            };

            updateFundermantialData()
        };
    };
};

// 下载事件
async function handleClick() {

    let bgImg, wmImg;

    let bgSrc = window.getComputedStyle(backgroundContainerRef.value, null).backgroundImage.slice(5, -2);
    let wmSrc = window.getComputedStyle(watermarkContainerRef.value, null).backgroundImage.slice(5, -2);

    bgImg = await loadImage(bgSrc);
    wmImg = await loadImage(wmSrc);

    const canvas = document.createElement('canvas');
    canvas.width = keyData.value.backgroundData.width;
    canvas.height = keyData.value.backgroundData.height;

    let scale = keyData.value.backgroundData.scale;
    let { offsetX, offsetY } = keyData.value.watermarkData;

    const ctx = canvas.getContext('2d');
    ctx.drawImage(bgImg, 0, 0);
    ctx.drawImage(wmImg, offsetX / scale, offsetY / scale);

    const el = document.createElement('a');
    el.href = canvas.toDataURL("image/jpeg", 0.8);
    el.download = '带水印图片' + new Date().getTime();

    const event = new MouseEvent('click');
    el.dispatchEvent(event);

};

function updateFundermantialData() {
    keyData.value.watermarkData.offsetX = 0;
    keyData.value.watermarkData.offsetY = 0;

    // 计算背景、水印图片缩放值
    keyData.value.backgroundData.scale = Math.min(backgroundContainerRef.value.clientHeight / keyData.value.backgroundData.height,
        backgroundContainerRef.value.clientWidth / keyData.value.backgroundData.width);
    keyData.value.watermarkData.scale = Math.min(watermarkContainerRef.value.clientHeight / keyData.value.watermarkData.height,
        watermarkContainerRef.value.clientWidth / keyData.value.watermarkData.width);

    // 设置水印图片top、left，确保它在背景图片内
    watermarkContainerRef.value.style.top = parseInt((backgroundContainerRef.value.clientHeight - keyData.value.backgroundData.height * keyData.value.backgroundData.scale) / 2) + "px";
    watermarkContainerRef.value.style.left = parseInt((backgroundContainerRef.value.clientWidth - keyData.value.backgroundData.width * keyData.value.backgroundData.scale) / 2) + "px";

    // 设置滑动条的水印偏移值范围
    let diffHeight = parseInt(keyData.value.backgroundData.height * keyData.value.backgroundData.scale - keyData.value.watermarkData.height * keyData.value.watermarkData.scale);
    let diffWidth = parseInt(keyData.value.backgroundData.width * keyData.value.backgroundData.scale - keyData.value.watermarkData.width * keyData.value.watermarkData.scale);

    delete yMarks.value[yMax.value];
    delete xMarks.value[xMax.value];
    yMarks.value[diffHeight] = `${diffHeight}px`;
    xMarks.value[diffWidth] = `${diffWidth}px`;
    yMax.value = diffHeight;
    xMax.value = diffWidth;

    windowData.value.width = document.innerWidth||document.documentElement.clientWidth||document.body.clientWidth;
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
    height: 31px;
    width: 30px;
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
