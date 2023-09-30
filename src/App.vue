<template>
  <div class="game" :style="{ height: `${gameHeight}px` }">
    <div class="title">密码推断</div>
    <div class="stage">
      <div class="keyboard">
        <div class="item code" :class="{ disabled: item.disabled }" v-for="item in keyCodesForDisplay" :key="item.code"
          :draggable="!item.disabled" @dragend="keyCodeDragEnd(item.code)">
          {{ item.code }}
        </div>
      </div>
      <div class="lock">
        <div class="item code" v-for="(item, index) in lockCodes" :key="item" :draggable="true"
          @dragenter="lockCodeDragEnter(index)" @dragend="lockCodeDragEnd(index)">
          {{ item }}
        </div>
        <button class="item btn" :disabled="!canTry" @click="tryAnswer">
          <template v-if="locked">
            <svg xmlns="http://www.w3.org/2000/svg" width="1em" height="1em" fill="currentColor" class="bi bi-lock-fill"
              viewBox="0 0 16 16">
              <path
                d="M8 1a2 2 0 0 1 2 2v4H6V3a2 2 0 0 1 2-2zm3 6V3a3 3 0 0 0-6 0v4a2 2 0 0 0-2 2v5a2 2 0 0 0 2 2h6a2 2 0 0 0 2-2V9a2 2 0 0 0-2-2z" />
            </svg>
          </template>
          <template v-else>
            <svg xmlns="http://www.w3.org/2000/svg" width="1em" height="1em" fill="currentColor" class="bi bi-unlock-fill"
              viewBox="0 0 16 16">
              <path
                d="M11 1a2 2 0 0 0-2 2v4a2 2 0 0 1 2 2v5a2 2 0 0 1-2 2H3a2 2 0 0 1-2-2V9a2 2 0 0 1 2-2h5V3a3 3 0 0 1 6 0v4a.5.5 0 0 1-1 0V3a2 2 0 0 0-2-2z" />
            </svg>
          </template>
        </button>
      </div>
    </div>

    <div class="triedAnswers">
      <div class="item" v-for="(item, index ) in triedAnswers" :key="item.answer">
        <div>{{ triedAnswers.length - index }}/{{ TOTAL_CHANCES }}</div>
        <div>{{ item.answer }}</div>
        <div>{{ item.inPlace }}★ {{ item.notInPlace }}☆</div>
      </div>
    </div>

    <div class="actions">

      <div class="lockNumbersControl">
        <button class="button" @click="TOTAL_CHANCES = Math.max(1, TOTAL_CHANCES - 1)">-</button>
        <div>{{ TOTAL_CHANCES }}步</div>
        <button class="button" @click="TOTAL_CHANCES = TOTAL_CHANCES + 1">+</button>
      </div>
      <div class="lockNumbersControl">
        <button class="button" @click="LOCK_NUMBERS = Math.max(3, LOCK_NUMBERS - 1)">-</button>
        <div>{{ LOCK_NUMBERS }}位</div>
        <button class="button" @click="LOCK_NUMBERS = Math.min(6, LOCK_NUMBERS + 1)">+</button>
      </div>
      <button class="button" @click="newGame()">新游戏</button>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { computed, onMounted, ref, watch } from 'vue';

const gameHeight = window.innerHeight

const TOTAL_CHANCES = ref(10) // 可尝试的总次数
const LOCK_NUMBERS = ref(5) // 锁上的数字个数

const locked = ref(true)

const baseQuestion = ref('')

// 锁上面的code
const lockCodes = ref<string[]>([])


// 数字键
const keyCodes = ['1', '2', '3', '4', '5', '6', '7', '8', '9', '0']
// 已尝试的组合
const triedAnswers = ref<{
  answer: string,
  inPlace: number;
  notInPlace: number;
}[]>([])

const question = computed(() => {
  return baseQuestion.value.slice(0, LOCK_NUMBERS.value)
})
// 剩余次数
const remnantChances = computed(() => {
  return TOTAL_CHANCES.value - triedAnswers.value.length
})


// 拖动时被覆盖的密码位
let overLockCodeIndex = -1

// 
const canTry = computed(() => {
  const answer = lockCodes.value.join('')
  return remnantChances.value > 0 &&
    locked.value &&
    answer.length === LOCK_NUMBERS.value &&
    triedAnswers.value.every(item => item.answer !== answer)
})

const keyCodesForDisplay = computed(() => {
  return keyCodes.map(item => {
    return {
      code: item,
      disabled: lockCodes.value.includes(item)
    }
  })
})


const lockCodeDragEnter = (index: number) => {
  overLockCodeIndex = index
}


const keyCodeDragEnd = (code: string) => {
  if (overLockCodeIndex > -1 && code !== lockCodes.value[overLockCodeIndex]) {
    lockCodes.value[overLockCodeIndex] = code
  }
  overLockCodeIndex = -1
}

const lockCodeDragEnd = (index: number) => {
  if (overLockCodeIndex > -1) {
    const code1 = lockCodes.value[overLockCodeIndex]
    const code2 = lockCodes.value[index]
    lockCodes.value[overLockCodeIndex] = code2
    lockCodes.value[index] = code1
  }
  overLockCodeIndex = -1

}

/**
 * 生成题目
 */
const newGame = (keepBaseQuestion = false) => {
  if (!keepBaseQuestion) {
    baseQuestion.value = [...keyCodes].sort(() => 0.5 - Math.random()).join('')
  }
  locked.value = true
  lockCodes.value = Array.from({ length: LOCK_NUMBERS.value }, () => '')
  triedAnswers.value = []
  console.log(baseQuestion.value)
}


const tryAnswer = () => {
  const answer = lockCodes.value.join('')
  let notInPlace = 0, inPlace = 0;
  const len = lockCodes.value.length
  for (let i = 0; i < len; i++) {
    for (let j = 0; j < len; j++) {
      if (answer[i] === question.value[j]) {
        if (i === j) {
          inPlace++;
        } else {
          notInPlace++;
        }
        break;
      }
    }
  }

  triedAnswers.value.unshift({
    answer,
    inPlace,
    notInPlace,
  })
  if (answer === question.value) {
    locked.value = false
  }
}

watch(() => LOCK_NUMBERS.value, () => {
  newGame(true)
})

onMounted(() => {
  newGame()
})
</script>

<style lang="scss">
.game {
  width: 100%;
  max-width: 600px;
  padding: 16px;
  background: #f5f9f8;
  margin: 0 auto;
  display: flex;
  flex-direction: column;

  button {
    border: none;
    height: 30px;
    border-radius: 4px;
    color: #fff;
    background: #fbb911;
    padding: 0 12px;

    &:disabled {
      background: #ddd;
    }

    &:active {
      opacity: 0.8;
    }
  }

  .title {
    text-align: center;
    padding: 24px;
    font-size: 20px;
    font-weight: bold;
  }

  .stage {
    background: #fff;
    padding: 16px;
    border-radius: 12px;

    .keyboard {
      display: flex;
      gap: 2px;

      .item {
        flex: 1;
        flex-shrink: 0;
        text-align: center;
        height: 30px;
        display: flex;
        flex-direction: column;
        place-content: center;
        place-items: center;
        border-radius: 4px;
      }

      .code {
        background: #efefef;
        color: #000;
        font-size: 18px;

        &.disabled {
          color: #ccc;
        }
      }

      .chances {
        background: transparent;
        background: #01bd0b;
        color: #fff;
        text-wrap: nowrap;
        flex: 0;
        padding: 0 11px;
        flex-shrink: 0;
      }
    }

    .lock {

      margin-top: 10px;
      display: flex;
      gap: 1px;
      background: #656565;
      padding: 8px;
      border-radius: 4px;

      .item {
        flex: 1;
        flex-shrink: 0;
        text-align: center;
        height: 60px;
        display: flex;
        flex-direction: column;
        place-content: center;
        place-items: center;
        border-radius: 4px;
      }

      .code {
        color: #313131;
        font-size: 24px;
        background: linear-gradient(0, #fff, #d3d3d3);
      }

      button {

        margin-left: 6px;
        font-size: 24px;
      }


    }

  }


  .triedAnswers {
    margin: 20px 0;
    background: #fff;
    padding: 16px;
    border-radius: 12px;
    flex: 1;
    overflow-y: auto;

    .item {
      display: flex;
      padding: 6px 0;
      border-bottom: 1px solid #f5f9f8;

      div {
        flex: 1;
        text-align: center;

        &:first-child {
          text-align: left;
        }

        &:last-child {
          text-align: right;
        }
      }

    }
  }

  .actions {
    display: flex;
    justify-content: space-between;
    background: #fff;
    padding: 16px;
    border-radius: 12px;

    .lockNumbersControl {
      display: flex;
      align-items: center;
      gap: 6px;
    }
  }
}</style>