<script setup lang="ts">
import SimpleGrid from "~/components/game/SimpleGrid.vue";
import {type Cell, type Cord, FieldType, GameError, type HitResponse} from "#shared/gameTypes";
import {useSocket} from "~/utils/useSocketIO";
import {GameJoinError} from "#shared/types";

const socket = useSocket();

const route = useRoute()
console.log(route.params.id)

const gridStore = useMyGridStore();

const gridSize = 10;

const myGrid: Ref<Cell[][]> = ref(gridStore.grid);
const opponentsGrid: Ref<Cell[][]> = ref(initGrid());

function initGrid() {
  const grid: Cell[][] = [];

  for (let x = 0; x < gridSize; x++) {
    grid[x] = [];

    for (let y = 0; y < gridSize; y++) {
      grid[x][y] = {
        type: {
          fieldType: FieldType.WATER,
          isHit: false
        },
        id: undefined,
        color: "white",
        x: x,
        y: y,
        originX: x,
        originY: y,
      }
    }
  }

  return grid;
}

function click(cord: Cord) {
  socket.emit("click", cord, hitResponse);
}

function hitResponse(hitResponse: HitResponse | GameError) {
  switch (hitResponse) {
    case GameError.WRONG_PLAYER: {
      console.error("Dein Gegner ist an der Reihe!");
      break;
    }
    case GameError.INVALID_CORD: {
      console.error("Ungültige Coordinaten");
      break;
    }
    case GameError.INVALID_ID: {
      console.error("ID ist falsch");
      break;
    }
    default: {
      console.log(hitResponse);
      break;
    }
  }
}


</script>

<template>
  <div>
    <h1>game {{ route.params.id }}</h1>

    <div id="fields">
      <div class="grid-container">
        <div class="player-grid">
          <h3>Player1</h3>

          <SimpleGrid :grid="myGrid" :has-listener="false"/>
        </div>

        <div class="player-grid">
          <h3>Player2</h3>

          <SimpleGrid :grid="opponentsGrid" :has-listener="true" @clicked="args => click(args)"/>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>

</style>