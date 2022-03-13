<script setup lang="ts">import { get } from "@vueuse/core"

const onClick = (item: BlockState) => {
  if(!mineGenerated) {
    generateMines(item)
    console.log(state)
    updateNumbers()
    mineGenerated = true
  }
 item.revealed = true
 expandZero(item)
 if(item.mine) {
   alert('Boom!!!')
 }
}

let mineGenerated = false
let dev = false

interface BlockState {
  x: number
  y: number
  revealed?: boolean
  mine?: boolean
  flagged?: boolean
  adjacentMines: number
}
const directions = [
  [1,1],
  [1,0],
  [1,-1],
  [0,1],
  [0,-1],
  [-1,1],
  [-1,0],
  [-1,-1]
]
const WIDTH = 10
const HEIGHT = 10
const state = reactive(Array.from({length: HEIGHT}, (_, y) => Array.from({length: WIDTH}, (_, x) =>( <BlockState>{x, y, adjacentMines: 0, revealed: false}))))

const numberColor = [
  'text-transparent',
  'text-blue-500',
  'text-green-500',
  'text-yellow-500',
  'text-orange-500',
  'text-purple-500',
  'text-pink-500',
  'text-teal-500',
  'text-transparent'
]

function generateMines(initial: BlockState) {
  for(const row of state) {
    for (const block of row) {
      if(Math.abs(initial.x - block.x) <= 1 && Math.abs(initial.y - block.y) <= 1) continue
      block.mine = Math.random() < 0.3
    }
  }
}
function updateNumbers() {
  state.forEach(row => {
    row.forEach(block => {
      console.log(getSiblings(block))
      if(block.mine) return;
      getSiblings(block).forEach(b => {
        if(b.mine === true) {
          block.adjacentMines++
        }
      })
    })
  })
}
function getSiblings(block: BlockState) {
  return directions.map(([dx, dy])=> {
        const x2 = dx + block.x
        const y2 = dy + block.y
        if(x2<0 || x2 >= WIDTH || y2 < 0 || y2 >= HEIGHT) return undefined;
        return state[y2][x2] 
      }).filter(Boolean) as BlockState[]
}

function getBlockClass(block: BlockState) {
  if(!block.revealed) return 'bg-gray-500/10';
  return block.mine ? 'bg-red-500/50' : numberColor[block.adjacentMines]
}

function expandZero(block: BlockState) {
  if(block.adjacentMines) return
  getSiblings(block).forEach(b => {
    if(!b.revealed) {
      b.revealed = true
      expandZero(b)
    }
  })
}

</script>

<template>
  <div>
    Minesweeper
    <div p5>
      <div v-for="(row, y) in state" :key='y' flex='~' padding items-center justify-center >
        <button hover="bg-gray/10" @click="onClick(item)" flex='~' items-center m="0.5" justify-center :class="getBlockClass(item)" w-10 h-10 border='1 gray-400/10' v-for="(item, x) in row" :key="x">
          <template v-if="item.revealed || dev">
            <div v-if="item.mine" i-mdi-mine>
          </div>
          <div v-else>
            {{item.adjacentMines}}
          </div>
          </template>
        </button>
      </div>
    </div>
  </div>
</template>

