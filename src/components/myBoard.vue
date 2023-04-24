<template>
    <el-card>
        <!-- <el-card>
            <div style="display: flex; justify-content: center;">  Board </div>
        </el-card> -->

        <!-- 20个按钮 -->
        <div class="btn-group">
            <!-- 此处的i与floors中的楼层以及下标都对应！！！ -->
            <div v-for="(buttons, i) in buttons_group" :key="i" class="btn-floor-span">
                <el-card class="up-down-card">
                    <div class="up-down-div">
                        <el-button class="btn-floor" :disabled="buttons_group[i].up" @click="buttonClick(i + 1, 1)">
                            <up-circle-two-tone />
                        </el-button>
                    </div>
                    <div class="up-down-div">{{ i + 1 }}</div>
                    <div class="up-down-div">
                        <el-button class="btn-floor" :disabled="buttons_group[i].down" @click="buttonClick(i + 1, -1)">
                            <down-circle-two-tone />
                        </el-button>
                    </div>
                </el-card>


            </div>
        </div>

    </el-card>
</template>

<script setup>

import { ref, onMounted, defineExpose, defineEmits } from 'vue';
import { UpCircleTwoTone, DownCircleTwoTone } from '@ant-design/icons-vue';

let buttons_group = ref([]);

/// 生命周期钩子
onMounted(() => {
    for (let i = 0; i < 20; i++) {
        buttons_group.value.push({
            up: false,
            down: false
        });
    }
});

defineExpose({
    buttons_group
});

const emit = defineEmits(["upRequest", "downRequest"]);

const buttonClick = (floor, direction) => {
    if (direction === 1 && buttons_group.value[floor - 1].up === false) {
        buttons_group.value[floor - 1].up = true;
        emit("upRequest", {
            floor_calling: floor,
            direction_calling: direction
        });
    }
    else if (direction === -1 && buttons_group.value[floor - 1].down === false) {
        buttons_group.value[floor - 1].down = true;
        emit("downRequest", {
            floor_calling: floor,
            direction_calling: direction
        });
    }
}



</script>
 

<style scoped>
.btn-group {
    display: flex;
    flex-wrap: wrap;
    /* padding-top: 10px; */
    /* padding-bottom: 10px; */

}

.up-down-card {
    display: flex;
    justify-content: center;
}

.up-down-div {
    display: flex;
    justify-content: center;
    width: 100%;
}

.button-box {
    display: flex;
    flex-wrap: wrap;
}

.btn-floor-span {
    display: flex;
    flex-wrap: wrap;
    /* margin-top: 10px; */
    width: 10%;
    justify-content: center;
}

.btn-floor {
    /* margin-left: 33%; */
    width: 90%;
    height: 10px;
    justify-content: center;

}

.mt-btn-danger {
    margin-top: 10px;
    width: 30%;
    margin-left: 35%;

    margin-right: 35%;
}
</style>