<template>
    <el-card class="elevator-card">
        <el-card>
            <div style="display: flex; justify-content: center;"> 当前{{ cur_floor }}楼</div>
            <div style="display: flex; justify-content: center;"> {{ upDownIdle }}</div>
            <div style="display: flex; justify-content: center;"> {{ doorOpen }}</div>
        </el-card>

        <!-- 20个按钮 -->
        <div class="btn-group">
            <!-- 此处的i与floors中的楼层以及下标都对应！！！ -->
            <div v-for="(floor, i) in floors" :key="i" class="btn-floor-span">
                <el-button round class="btn-floor" @click="floorBtnClick(i + 1)" :disabled="floors[i]">{{ i + 1
                }}</el-button>
            </div>
        </div>


        <div class="button-box">
            <el-button type="danger" class="mt-btn-danger" round @click="showModal">报警</el-button>
            <a-modal v-model:visible="visible" title="Basic Modal" @ok="handleOk">
                <p>别慌，速速联系我email，我来救你！！！</p>
            </a-modal>
        </div>

        <div class="btn-group">
            <div class="btn-floor-span"> <el-button type="info" @click="openDoor(3000)">开门</el-button></div>

            <div class="btn-floor-span"><el-button type="info">关门</el-button></div>
        </div>

    </el-card>
</template>

<script setup>
import { ref, onBeforeMount, onUnmounted, defineEmits, defineExpose } from 'vue';
import { computed } from 'vue';

/// UI响应函数
const visible = ref(false);
const showModal = () => {
    visible.value = true;
};
const handleOk = e => {
    console.log(e);
    visible.value = false;
};

/// 数据结构
let floors = ref([]);   ///< 所有楼层
let cur_floor = ref(5);
let cur_floor_moving = ref(false);
let cur_door_open = ref(false);
let cur_direction = ref(0);
let task_queue = ref([]);


/// 组件传值
defineExpose({
    floors,
    cur_floor,
    cur_floor_moving,
    cur_door_open,
    cur_direction,
    task_queue,
    openDoor,
});
const emit = defineEmits(['upArrive', 'downArrive']);

/// 钩子函数
onBeforeMount(() => {
    for (let i = 1; i <= 20; i++) {
        floors.value.push(false)
    }
});

onUnmounted(() => {
    clearInterval(my_clock);
});

/// 计算属性
const upDownIdle = computed(() => {
    if (cur_direction.value === 1) {
        return "上楼";
    }
    else if (cur_direction.value === 0) {
        return "静止";
    }
    else if (cur_direction.value === -1) {
        return "下楼";
    }
    else {
        return "";
    }
})

const doorOpen = computed(() => {
    if (cur_door_open.value) {
        return "门开了";
    }
    else {
        return "门关着";
    }
})

/// 电梯内部函数
// 开门
function openDoor(time) {
    console.log("open");
    if (cur_floor_moving.value || cur_door_open.value) {
        return;
    }
    cur_door_open.value = true;
    cur_floor_moving.value = false;

    setTimeout(() => {
        closeDoor();
    }, time);
}

// 关门
function closeDoor() {
    if (cur_floor_moving.value || !cur_door_open.value) {
        return;
    }

    cur_door_open.value = false;
    console.log("close");
    autoSetNextDirection();

    if (task_queue.value.length === 0) {
        cur_floor_moving.value = false;
    }
    else {
        cur_floor_moving.value = true;
    }
}

// 关门后自动设置电梯运行方向
const autoSetNextDirection = () => {
    if (task_queue.value.length === 0) {
        cur_direction.value = 0;
        return;
    }

    // if (cur_door_open.value) {
    //     cur_direction.value = 0; // 门开了要下来
    //     return;
    // }

    let max_floor = Math.max.apply(Math, task_queue.value);
    let min_floor = Math.min.apply(Math, task_queue.value);



    if (cur_direction.value === 1) {
        if (max_floor > cur_floor.value) {
            return;
        }
        else {
            cur_direction.value = -1;
        }
    }
    else if (cur_direction.value === -1) {
        if (min_floor < cur_floor.value) {
            return;
        }
        else {
            cur_direction.value = 1;
        }
    }
    else if (cur_direction.value === 0) {
        if (task_queue.value.length === 0) {
            return;
        }
        else {
            // 选队列首（可能上可能下）
            let aim_floor = task_queue.value[0];
            if (aim_floor > cur_floor.value) {
                cur_direction.value = 1;
            }
            else if (aim_floor < cur_floor.value) {
                cur_direction.value = -1;
            }
            else if (aim_floor === cur_floor.value) {
                cur_direction.value = 0;
            }
        }
    }
}

const floorBtnClick = i => {
    console.log(i + "was clicked");

    //todo 应该是开门的时候再点击一次当前层，3秒应该重新计时
    if (i === cur_floor.value && cur_floor_moving.value === false) {
        console.log("在当前层停止");
        openDoor(3000);
    }

    if (task_queue.value.indexOf(i) !== -1) {
        console.log("已经在队列中了");
        return;
    }

    task_queue.value.push(i);
    floors.value[i - 1] = true;
}

const updateStatus = () => {
    if (cur_door_open.value) {
        return;
    }

    autoSetNextDirection();

    if (task_queue.value.length === 0) {
        cur_floor_moving.value = false;
        return;
    }
    else {
        cur_floor_moving.value = true;
    }


    // 改变当前层数
    cur_floor.value = cur_floor.value + cur_direction.value;

    let cur_floor_idx_in_queue = task_queue.value.indexOf(cur_floor.value);

    if (cur_floor_idx_in_queue === -1) { // 当前楼层没有被按下
        return;
    }
    else {
        cur_floor_moving.value = false;
        task_queue.value.splice(cur_floor_idx_in_queue, 1); // 从队列中移除当前层
        floors.value[cur_floor.value - 1] = false;
        openDoor(3000);
    }

}

// 与外部电梯交互
const updateExternalUpAndDown = () => {
    if (cur_direction.value === 0) {
        emit("upArrive", cur_floor.value);
        emit("downArrive", cur_floor.value);
    }
    else if (cur_direction.value === 1) {
        emit("upArrive", cur_floor.value);
    }
    else if (cur_direction.value === -1) {
        emit("downArrive", cur_floor.value);
    }
}

let my_clock = setInterval(() => {
    updateStatus();
    updateExternalUpAndDown();
}, 1500);


</script>
 

<style scoped>
.btn-group {
    display: flex;
    flex-wrap: wrap;
    padding-top: 20px;
    padding-bottom: 20px;

}

.button-box {
    display: flex;
    flex-wrap: wrap;
}

.btn-floor-span {
    display: flex;
    margin-top: 10px;
    width: 50%;
    justify-content: center;
}

.btn-floor {
    /* margin-left: 33%;
    width: 20px; */
    justify-content: center;

}

.mt-btn-danger {
    margin-top: 10px;
    width: 30%;
    margin-left: 35%;

    margin-right: 35%;
}

.elevator-card {
    margin: 20px;
}
</style>