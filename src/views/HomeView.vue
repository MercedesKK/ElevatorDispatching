<template>
  <div class="home container">
    <myNavBar />
    <el-row>
      <div class="elevators">
        <el-col :span="4" v-for="i in elevator_number" :key="i">
          <myElevator ref="elevator_group" @upArrive="upArriveInteraction" @downArrive="downArriveInteraction" />
        </el-col>
      </div>
    </el-row>
    <el-row>
      <div class="elevators">
        <el-col :span="22">
          <myBoard ref="floor_board" @upRequest="upHandle" @downRequest="downHandle" />
        </el-col>
      </div>
    </el-row>
  </div>
</template>

<script setup>
// @ is an alias to /src
import myElevator from "../components/myElevator.vue"
import myNavBar from "../components/myNavBar.vue"
import myBoard from "../components/myBoard.vue"
import { getCurrentInstance } from "vue";

const elevator_number = 5;
// const currentInstance = getCurrentInstance()

const { proxy } = getCurrentInstance();




// 返回所有电梯中task_queue中任务最少的
const caculateMinTasks = elevators => {
  let min_tasks = 50; // 电梯超过50层后要改的
  let min_elevator_idx = 0;

  for (let i = 0; i < elevator_number; i++) {
    if (elevators[i].task_queue.length < min_tasks) {
      min_tasks = elevators[i].task_queue.length;
      min_elevator_idx = i;
    }
  }

  return min_elevator_idx;
}


// 返回最佳电梯下标 0~4（队列已经处理好了的）
const caculateMinDistance = (candidate_queue, floor_calling, elevators) => {
  let min_dis = 50; // 电梯超过50层后要改的
  let min_elevator_idx = 0;

  for (let i of candidate_queue) {
    if (Math.abs(elevators[i].cur_floor - floor_calling) < min_dis) {
      min_dis = Math.abs(elevators[i].cur_floor - floor_calling);
      min_elevator_idx = i;
    }
  }

  return min_elevator_idx;
};

/// 我自己不用管的，子组件电梯当开门的时候会自动传入这里
const upArriveInteraction = (floor) => {
  proxy.$refs.floor_board.buttons_group[floor - 1].up = false;
};
const downArriveInteraction = (floor) => {
  proxy.$refs.floor_board.buttons_group[floor - 1].down = false;
}

//todo 按了之后过去之后要复原的
const upHandle = (argv) => {
  console.log(argv);

  let elevators = proxy.$refs.elevator_group; // 电梯组件（列表）
  let { direction_calling, floor_calling } = argv; // 本次从子组件上传的呼叫方向和电梯数
  let candidate_queue = []; // 存放已经候选的电梯下标

  for (let i = 0; i < elevator_number; i++) {
    if (elevators[i].cur_direction === 0) {
      if (elevators[i].cur_floor === floor_calling) {
        elevators[i].openDoor(3000);
        // console.log(elevators[i]);
        return;
      }
      candidate_queue.push(i); // 静止但不在此层加入候选队列
    }

    if (elevators[i].cur_direction === direction_calling && floor_calling - elevators[i].cur_floor > 0) { // 电梯同向且还没有加入此层
      candidate_queue.push(i);
    }
  }

  // 如果候选队列有电梯可以来接
  if (candidate_queue.length !== 0) {
    let elevator_idx = caculateMinDistance(candidate_queue, floor_calling, elevators);

    if (elevators[elevator_idx].task_queue.indexOf(floor_calling) === -1) {
      elevators[elevator_idx].task_queue.push(floor_calling);
      // console.log("ok");
      // console.log(elevator_idx);
    }
  }
  else { // 选任务数量最少的电梯加入
    let elevator_idx = caculateMinTasks(elevators);
    if (elevators[elevator_idx].task_queue.indexOf(floor_calling) === -1) {
      elevators[elevator_idx].task_queue.push(floor_calling);
    }

  }


}

const downHandle = (argv) => {
  console.log(argv);

  let elevators = proxy.$refs.elevator_group; // 电梯组件（列表）
  let { direction_calling, floor_calling } = argv; // 本次从子组件上传的呼叫方向和电梯数
  let candidate_queue = []; // 存放已经候选的电梯下标

  for (let i = 0; i < elevator_number; i++) {
    if (elevators[i].cur_direction === 0) {
      if (elevators[i].cur_floor === floor_calling) {
        downArriveInteraction(floor_calling);
        elevators[i].openDoor(3000);
        // console.log(elevators[i]);
        return;
      }
      candidate_queue.push(i); // 静止但不在此层加入候选队列
    }

    if (elevators[i].cur_direction === direction_calling && floor_calling - elevators[i].cur_floor < 0) { // 电梯同向且还没有加入此层
      candidate_queue.push(i);
    }
  }

  // 如果候选队列有电梯可以来接
  if (candidate_queue.length !== 0) {
    let elevator_idx = caculateMinDistance(candidate_queue, floor_calling, elevators);

    if (elevators[elevator_idx].task_queue.indexOf(floor_calling) === -1) {
      elevators[elevator_idx].task_queue.push(floor_calling);
      // console.log("ok");
      // console.log(elevator_idx);
    }
  }
  else { // 选任务数量最少的电梯加入
    let elevator_idx = caculateMinTasks(elevators);
    if (elevators[elevator_idx].task_queue.indexOf(floor_calling) === -1) {
      elevators[elevator_idx].task_queue.push(floor_calling);
    }

  }
}



</script>

<style scoped>
.elevators {
  display: flex;
  justify-content: center;
}
</style>