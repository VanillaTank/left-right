<script setup>
import {onMounted, ref, watch} from "vue";
import draggable from 'vuedraggable'

const orderModel = ref([
  {
    color: '#ec9a69',
    name: 'персиковый',
    id: 1,
  },
  {
    color: '#ffe158',
    name: 'желтый',
    id: 2,
  },
  {
    color: '#25a7c9',
    name: 'голубой',
    id: 3,
  },
  {
    color: '#9b5ec7',
    name: 'лиловый',
    id: 4,
  },
  {
    color: '#4bbe76',
    name: 'зеленый',
    id: 5,
  },
  {
    color: '#ee92c9',
    name: 'розовый',
    id: 6,
  },
  {
    color: '#c72929',
    name: 'красный',
    id: 7,
  },
])
const task = ref({})
const result = ref('')


const getRandomNumber = ({min, max, notAllowedArr = []}) => {
  const random = Math.floor(min + Math.random() * (max + 1 - min))
  if (notAllowedArr.length && notAllowedArr.includes(random)) {
    return getRandomNumber({min, max, notAllowedArr})
  }
  return random
}

const getNewTask = () => {
  const lastItemIndex = orderModel.value.length - 1
  const driftingItemIndex = getRandomNumber({min: 0, max: lastItemIndex})
  const newPlace = getRandomNumber({min: 0, max: lastItemIndex, notAllowedArr: [driftingItemIndex]})

  const randomOrder = JSON.parse(JSON.stringify(orderModel.value))
  randomOrder.splice(driftingItemIndex, 1)
  const driftingItem = orderModel.value[driftingItemIndex]
  randomOrder.splice(newPlace, 0, driftingItem)

  let randomLeft = 0
  let left = ''
  if (newPlace > 0) {
    randomLeft = getRandomNumber({min: 0, max: newPlace - 1})
    if (randomLeft > driftingItemIndex) {
      return getNewTask()
    }
    left = `правее ${randomOrder[randomLeft].name.replace(/..$/, 'ого')}`
  }

  let right = ''
  let randomRight = getRandomNumber({min: newPlace + 1, max: lastItemIndex - 1})
  if (randomRight > driftingItemIndex) {
    return getNewTask()
  }
  right = `левее ${randomOrder[randomRight].name.replace(/..$/, 'ого')}`

  return {
    leftBorder: randomLeft,
    rightBorder: randomRight,
    driftingItemId: driftingItem.id,
    text: getText(driftingItem.name, left, right),
  }
}

const getText = (driftingItemName, left, right) => {
  const rand = getRandomNumber({min: 0, max: 1})
  if (rand) {
    return `Перемести ${driftingItemName} блок ${left} ${(left && right) ? 'и ' + right : right}`
  } else {
    return `Перемести ${driftingItemName} блок ${right} ${(left && right) ? 'и ' + left : left}`
  }
}
onMounted(() => {
  task.value = getNewTask()
})

const check = () => {
  const modelDriftingItemIndex = orderModel.value.findIndex(({id}) => id === task.value.driftingItemId)
  if (
      task.value.leftBorder < modelDriftingItemIndex
      && modelDriftingItemIndex < task.value.rightBorder
  ) {
    result.value = true
    task.value = getNewTask()
  } else {
    result.value = false
  }
}

watch(orderModel, () => {
  check()
})
</script>
}

<template>
  <div class="container">
    <div
        v-if="task.text"
        class="task"
    >
      {{ task.text }}
    </div>
    <div class="drag-wrap">
      <draggable
          v-model="orderModel"
          class="drag"
          item-key="id">
        <template #item="{element}">
          <div
              class="drag__item"
              :style="{'background': element.color}"
          ></div>
        </template>
      </draggable>
      <div class="you">YOU</div>
    </div>
    <div v-if="result !== ''">{{ result ? 'OK' : 'NOT OK' }}</div>
  </div>
</template>

<style scoped>
.container {
  margin: 0 auto;
  max-width: 1200px;
  display: flex;
  align-items: center;
  flex-direction: column;
}

.task {
  margin: 10px;
}

.drag-wrap {
  display: flex;
  align-items: center;
  flex-direction: column;
}

.drag {
  display: flex;
  height: 70px;
  flex-direction: row;
  gap: 10px;
  border: 2px solid #e0d22d;
  padding: 10px;
}

.drag__item {
  border: 1px solid darkgrey;
  width: 40px;
  cursor: pointer;
}

.you {
  margin: 15px auto 0;
  width: 100px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  border: 1px solid darkgrey;
}
</style>
