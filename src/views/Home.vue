<template>
  <div class="all Sudoku">
    <h1 class="title">Simple Sudoku</h1>
    <div style="margin-bottom: 20px">
      Passed Time: <b>{{ time }}</b>
    </div>
    <div style="margin-bottom: 64px">
      <button :title="msgPause" v-on:click="pause">Pause</button>
      <button :title="msgRestart" v-on:click="restart">Restart</button>
      <button :title="msgReload" v-on:click="reload">Reload</button>
    </div>
    <div v-if="viewUpdated">
      <div class="row" v-for="posX in numArr" :key="posX">
        <div class="square" v-for="posY in numArr" :key="posY">
          {{ data[parseInt(posX + "" + posY)] }}
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import Component from "vue-class-component";

// Function used to shuffle array given.
function shuffle(numArr: number[]): number[] {
  let randomArr: number[] = [];

  randomArr.push(...numArr);

  for (let i = randomArr.length - 1; i > 0; --i) {
    let j = Math.floor(Math.random() * (i + 1));
    [randomArr[i], randomArr[j]] = [randomArr[j], randomArr[i]];
  }

  return randomArr;
}

// Function used to get the union of two maps.
function unionMap(
  mapA: Map<number, boolean>,
  mapB: Map<number, boolean>
): Map<number, boolean>;

function unionMap(
  mapA: Map<number, boolean>,
  mapB: Map<number, boolean>,
  mapC: Map<number, boolean>
): Map<number, boolean>;

function unionMap(
  mapA: Map<number, boolean>,
  mapB: Map<number, boolean>,
  mapC?: Map<number, boolean>
): Map<number, boolean> {
  if (!mapC) {
    let uMap: Map<number, boolean> = new Map();
    for (let entry of mapB.entries()) {
      uMap.set(entry[0], entry[1]);
    }
    for (let entry of mapA.entries()) {
      if (uMap.has(entry[0]) == false) {
        uMap.set(entry[0], entry[1]);
      }
    }
    return uMap;
  } else {
    return unionMap(unionMap(mapA, mapB), mapC);
  }
}

// Function used to get the difference (subtraction) of the map given and Universe.
function getComplement(
  U: number[],
  map: Map<number, boolean>
): Map<number, boolean> {
  let cMap = new Map();
  for (let i in U) {
    if (map.has(U[i]) == false) {
      cMap.set(U[i], true);
    }
  }
  return cMap;
}

@Component
export default class Sudoku extends Vue {
  time: Date = new Date();
  // This store the number from 1 to 9.
  numArr: number[] = [1, 2, 3, 4, 5, 6, 7, 8, 9];

  // Starting from [row 1, col 1], ending at [row 9, col 9];
  // 11 means [row 1, col 1], 99 means [row 9, col 9].
  // data[35] = 7 means the number at [row 3, col 5] is 7.
  data: number[] = [];
  dataBackup: number[] = [];

  // This stores the center pos of each block.
  // e.g. 22 means [row 2, col 2] is the center pos of first block.
  // cenPosArr: number[] = [22, 25, 28, 52, 55, 58, 82, 85, 88];

  // This stores the blanks to be filled.
  // blank[i] = _X_Y means [row _X, col _Y] is to be filled.
  blank: number[] = [];

  viewUpdated = false;

  // Prompting messages.
  msgPause = "Press this to pause the Timer.";
  msgRestart = "Press this to restart this game; layout will not change.";
  msgReload = "Press this to reload a new game; layout will change.";
  /*
  ^ ^ ^ | + + + | + + +
  ^ ^ ^ | + + + | + + +
  ^ ^ ^ | + + + | + + +
  ------|-------|------
  + + + | ^ ^ ^ | + + +
  + + + | ^ ^ ^ | + + +
  + + + | ^ ^ ^ | + + +
  ------|-------|------
  + + + | + + + | ^ ^ ^
  + + + | + + + | ^ ^ ^
  + + + | + + + | ^ ^ ^
*/

  private getBlockPos(cenPos: number): number[] {
    let blockPos: number[] = [
      cenPos - 11,
      cenPos - 10,
      cenPos - 9,
      cenPos - 1,
      cenPos,
      cenPos + 1,
      cenPos + 9,
      cenPos + 10,
      cenPos + 11,
    ];
    return blockPos;
  }

  private getRowMap(posX: number): Map<number, boolean> {
    let rowMap: Map<number, boolean> = new Map();
    let tmp: number;
    for (let i = 1; i < 10; ++i) {
      if ((tmp = this.data[parseInt(posX + "" + i)])) {
        rowMap.set(tmp, false);
      }
    }
    return rowMap;
  }

  private getColMap(posY: number): Map<number, boolean> {
    let colMap: Map<number, boolean> = new Map();
    let tmp: number;
    for (let i = 1; i < 10; ++i) {
      if ((tmp = this.data[parseInt(i + "" + posY)])) {
        colMap.set(tmp, false);
      }
    }
    return colMap;
  }

  private getBlockMap(posX: number, posY: number): Map<number, boolean> {
    let blockMap: Map<number, boolean> = new Map();
    let cenPosX: number = Math.floor((posX - 1) / 3) * 3 + 2;
    let cenPosY: number = Math.floor((posY - 1) / 3) * 3 + 2;
    let cenPos = parseInt(cenPosX + "" + cenPosY);
    let blockPos: number[] = this.getBlockPos(cenPos);
    let tmp: number;
    for (let i = 0; i < 9; ++i) {
      if ((tmp = this.data[blockPos[i]])) {
        blockMap.set(tmp, false);
      }
    }
    return blockMap;
  }

  private setDiagnoal(cenPos: number): void {
    let randomArr: number[] = shuffle(this.numArr);
    let blockPos: number[] = this.getBlockPos(cenPos);
    for (let i = 0; i < 9; ++i) {
      this.data[blockPos[i]] = randomArr[i];
    }
  }

  private setCell(posX: number, posY: number): boolean {
    let pos: number = parseInt(posX + "" + posY);
    let rowMap: Map<number, boolean> = this.getRowMap(posX);
    let colMap: Map<number, boolean> = this.getColMap(posY);
    let blockMap: Map<number, boolean> = this.getBlockMap(posX, posY);
    let invalid: Map<number, boolean> = unionMap(rowMap, colMap, blockMap);
    let valid: Map<number, boolean> = getComplement(this.numArr, invalid);

    if (valid.size == 0) {
      return false;
    } else {
      let i: number = Math.floor(Math.random() * (valid.size - 1));
      let j = 0;
      for (let entry of valid.entries()) {
        if (j < i) {
          ++j;
        } else {
          j = entry[0];
          break;
        }
      }
      this.data[pos] = j;
      return true;
    }
  }

  private init(): void {
    this.viewUpdated = false;
    this.data.splice(1, this.data.length);
    try {
      // First initialize the diagonal.
      // In fact, you can choose any three cenPos as long as they have different X and Y.
      this.setDiagnoal(22);
      this.setDiagnoal(55);
      this.setDiagnoal(88);

      // Set remaining cells.
      for (let i = 1; i < 10; ++i) {
        for (let j = 1; j < 10; ++j) {
          if (this.data[parseInt(i + "" + j)]) {
            continue;
          } else {
            let isSet: boolean;
            isSet = this.setCell(i, j);
            if (isSet == false) {
              this.init();
              return;
            }
          }
        }
      }
    } catch (exception) {
      console.log(exception);
      this.init();
    }
    this.dataBackup = this.data.slice();
    this.viewUpdated = true;
    this.$forceUpdate();
  }

  public pause(): void {
    console.log("pause button clicked!");
    this.time = new Date();
  }

  public restart(): void {
    console.log("restart button clicked!");
    this.init();
  }

  public reload(): void {
    console.log("reload button clicked!");
  }

  mounted(): void {
    this.init();
  }
}
</script>

<style type="text/css">
.all {
  margin: 0 auto;
  text-align: center;
}
.title {
  font-family: "Courier New", Courier, monospace;
  font-size: 48px;
  letter-spacing: 2px;
}

button {
  width: 120px;
  font-family: "Gill Sans", "Gill Sans MT", Calibri, "Trebuchet MS", sans-serif;
  font-size: 20px;
  margin: 5px 25px 5px 25px;
  border-radius: 20%;
}

.row {
  margin: 0 auto;
  display: flex;
  line-height: 40px;
  width: 400px;
}

.square {
  font-family: "Courier New", Courier, monospace;
  width: 40px;
  background-color: aqua;
  margin: 1px 1px 1px 1px;
  border: 1px solid black;
}
</style>
