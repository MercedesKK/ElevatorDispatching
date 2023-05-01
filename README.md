# 电梯调度

同济大学软件学院OS第一次作业

## 在线运行

https://mercedeskk.github.io/ElevatorDispatching/

## 本地运行
```
git clone git@github.com:MercedesKK/ElevatorDispatching.git
cd ElevatorDispatching
npm install
npm run serve

打开 http://localhost:8080
```

## 开发环境

开发框架：vue3、Element-plus、Ant Design Vue

开发语言：JavaScript、html、css

### 简要说明

​		虽然JavaScript为单线程语言，但事件、DOM操作、ajax、定时器（setTimeOut、setImmediate）、Promise、async/await等为异步任务。当同步任务执行结束后（组价挂载等），任务队列中的异步任务开始执行，故可以异步完成电梯调度。

### 项目结构

```
.
├─ public
│    ├─ favicon.ico
│    └─ index.html
└─ src
    ├─ App.vue
    ├─ assets
    │    ├─ logo.png
    │    └─ myPhoto.png
    ├─ components
    │    ├─ myBoard.vue
    │    ├─ myElevator.vue
    │    └─ myNavBar.vue
    ├─ main.js
    ├─ router
    │    └─ index.js
    ├─ store
    │    └─ index.js
    └─ views
    └─ HomeView.vue
```



## 算法介绍

### 数据结构

```js
/// 数据结构
let floors = ref([]);   			///< 所有楼层
let cur_floor = ref(5);				///< 当前层
let cur_floor_moving = ref(false);	///< 当前电梯是否在上下移动
let cur_door_open = ref(false);		///< 当前电梯门是否开
let cur_direction = ref(0);			///< 当前电梯移动方向 1向上 -1向下
let task_queue = ref([]);			///< 电梯任务队列
```

### 关键算法

#### 电梯内部

​		代码设计高内聚低耦合。

​		电梯**内部**采用LOOK算法：算法核心是**根据当前电梯所在的楼层和前进方向，来对总的乘客请求列表进行分析，判断电梯按当前方向运行是否还会遇见乘客**(乘客前进方向是否和电梯当前运行方向相同无需考虑)，如果仍有乘客，或者电梯内部仍有乘客的话(即为满足运行条件)，电梯就会继续沿当前方向运行，遇见乘客后，如果乘客前进方向和电梯相同，且电梯可以继续搭载乘客，则该乘客进入电梯；如果在该方向上没有乘客，且当前电梯内部为空，则改变电梯方向再次进行判断，如果仍然不满足运行条件，则让该电梯线程wait。

​		部分关键算法如下：

```js
/// 电梯方向选择
const autoSetNextDirection = () => {
    if (task_queue.value.length === 0) {
        cur_direction.value = 0;
        return;
    }

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

/// 每1.5秒运行检查电梯状态
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
```



#### 电梯外部

​		外部算法为：

1. 若有电梯处于静止状态，或有同方向的正在运行的电梯，则距离当前最近的电梯来接。
2. 若当前五部电梯都处于运行状态，但没有同方向正在运行的电梯，则选择所有电梯中task_queue最任务最少的。

上行算法如下：（下行同理）

```js
const upHandle = (argv) => {
  // console.log(argv);

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
```



#### 作者

**联系方式** 2455650395@qq.com
