<template>
  <div class="puzzle">
    <form @submit.prevent="setVariables">
      <section class="form">
        <div class="form_ranges">
          <div class="form_ranges_block">
            <input
              type="range"
              v-model="rowRange"
              step="1"
              min="2"
              max="10"
              id="row"
            />
            <label for="row">The number of rows: {{ rowRange }}</label>
          </div>
          <div class="form_ranges_block">
            <input
              type="range"
              v-model="colRange"
              step="1"
              min="2"
              max="10"
              id="col"
            />

            <label for="col">The number of columns: {{ colRange }}</label>
          </div>
          <div class="form_ranges_block">
            <input
              type="range"
              v-model="tileSizeRange"
              step="1"
              min="40"
              max="150"
              id="tileSize"
            />
            <label for="tileSize"
              >The size of tiles: {{ tileSizeRange }}px</label
            >
          </div>
        </div>

        <div>
          <button type="submit" class="form_button">Set Sizes</button>
        </div>
      </section>
    </form>
    <div
      class="square"
      :style="`width:${tileSize * col + 10}px;height:${tileSize * row + 10}px;`"
    >
      <template v-for="x in row">
        <template v-for="y in col">
          <article
            :key="x + 'x' + y + 'y'"
            v-if="x !== row || col !== y"
            class="tile"
            @click="status((x - 1) * col + y)"
            :style="`width:${tileSize}px;height:${tileSize}px;left:${
              (y - 1) * tileSize
            }px; top: ${(x - 1) * tileSize}px;`"
          >
            {{ (x - 1) * col + y }}
          </article>
        </template>
      </template>
    </div>
  </div>
</template>

<script>
export default {
  name: "PuzzleWorld",
  data() {
    return {
      emptyTile: {
        index: 0,
        top: 0,
        bottom: 0,
        left: 0,
        right: 0,
      },
      clickedTile: {
        top: 0,
        left: 0,
      },
      tilesNumbers: [],
      correctTitlesNumbers: [],
      checkPuzzleCorrection: false,
      rowRange: 3,
      colRange: 3,
      tileSizeRange: 100,
      row: 3,
      col: 3,
      tileSize: 100,
    };
  },
  beforeMount() {
    if (this.$route.query) {
      Object.entries(this.$route.query).forEach(([key, value]) => {
        this[key] = +value;
        this[`${key}Range`] = +value;
      });
    }
  },
  mounted() {
    this.generateEmptyTile();
    this.checkEmptyTilePosition();
    setTimeout(() => {
      this.prePlay();
    }, 10);
    setTimeout(() => {
      this.checkPuzzleCorrection = true;
    }, 1000);
  },
  computed: {},
  methods: {
    generateEmptyTile() {
      for (let i = 1; i < this.row * this.col; i++) {
        this.tilesNumbers.push(i);
      }
      this.tilesNumbers.push("x");
      this.correctTitlesNumbers = [...this.tilesNumbers];
    },
    checkEmptyTilePosition() {
      this.emptyTile.top = (this.row - 1) * this.tileSize + "px";
      this.emptyTile.bottom = this.row * this.tileSize + "px";
      this.emptyTile.left = (this.col - 1) * this.tileSize + "px";
      this.emptyTile.right = this.col * this.tileSize + "px";
      this.emptyTile.index = this.row * this.col;
    },
    setVariables() {
      this.$router.replace(
        {
          query: {
            row: this.rowRange,
            col: this.colRange,
            tileSize: this.tileSizeRange,
          },
        },
        () => {}
      );
      location.reload();
    },
    prePlay() {
      for (let i = 1; i < 2000; i++) {
        this.emptyTile.index = this.tilesNumbers.indexOf("x");
        let selectableNumbers, chosenIndex;
        switch (this.emptyTile.index + 1) {
          case 1:
            selectableNumbers = [];
            selectableNumbers.push(
              this.tilesNumbers[this.emptyTile.index + 1],
              this.tilesNumbers[this.emptyTile.index + this.col]
            );
            chosenIndex = Math.floor(Math.random() * 2);
            this.status(selectableNumbers[chosenIndex]);
            break;
          case this.col:
            selectableNumbers = [];
            selectableNumbers.push(
              this.tilesNumbers[this.emptyTile.index - 1],
              this.tilesNumbers[this.emptyTile.index + this.col]
            );
            chosenIndex = Math.floor(Math.random() * 2);
            this.status(selectableNumbers[chosenIndex]);
            break;
          case this.col * this.row:
            selectableNumbers = [];
            selectableNumbers.push(
              this.tilesNumbers[this.emptyTile.index - 1],
              this.tilesNumbers[this.emptyTile.index - this.col]
            );
            chosenIndex = Math.floor(Math.random() * 2);
            this.status(selectableNumbers[chosenIndex]);
            break;
          case this.col * this.row - this.col:
            selectableNumbers = [];
            selectableNumbers.push(
              this.tilesNumbers[this.emptyTile.index + 1],
              this.tilesNumbers[this.emptyTile.index - this.col]
            );
            chosenIndex = Math.floor(Math.random() * 2);
            this.status(selectableNumbers[chosenIndex]);
            break;
          default:
            if (this.emptyTile.index + 1 < this.col) {
              selectableNumbers = [];
              selectableNumbers.push(
                this.tilesNumbers[this.emptyTile.index - 1],
                this.tilesNumbers[this.emptyTile.index + this.col],
                this.tilesNumbers[this.emptyTile.index + 1]
              );
              chosenIndex = Math.floor(Math.random() * 3);
              this.status(selectableNumbers[chosenIndex]);
            } else if (
              this.emptyTile.index + 1 >
              this.row * this.col - this.col
            ) {
              selectableNumbers = [];
              selectableNumbers.push(
                this.tilesNumbers[this.emptyTile.index - 1],
                this.tilesNumbers[this.emptyTile.index - this.col],
                this.tilesNumbers[this.emptyTile.index + 1]
              );
              chosenIndex = Math.floor(Math.random() * 3);
              this.status(selectableNumbers[chosenIndex]);
            } else if ((this.emptyTile.index + 1) % this.col === 0) {
              selectableNumbers = [];
              selectableNumbers.push(
                this.tilesNumbers[this.emptyTile.index - 1],
                this.tilesNumbers[this.emptyTile.index + this.col],
                this.tilesNumbers[this.emptyTile.index - this.col]
              );
              chosenIndex = Math.floor(Math.random() * 3);
              this.status(selectableNumbers[chosenIndex]);
            } else if (this.emptyTile.index % this.col === 0) {
              selectableNumbers = [];
              selectableNumbers.push(
                this.tilesNumbers[this.emptyTile.index + 1],
                this.tilesNumbers[this.emptyTile.index + this.col],
                this.tilesNumbers[this.emptyTile.index - this.col]
              );
              chosenIndex = Math.floor(Math.random() * 3);
              this.status(selectableNumbers[chosenIndex]);
            } else {
              selectableNumbers = [];
              selectableNumbers.push(
                this.tilesNumbers[this.emptyTile.index + 1],
                this.tilesNumbers[this.emptyTile.index - 1],
                this.tilesNumbers[this.emptyTile.index + this.col],
                this.tilesNumbers[this.emptyTile.index - this.col]
              );
              chosenIndex = Math.floor(Math.random() * 4);
              this.status(selectableNumbers[chosenIndex]);
            }
        }
      }
    },
    status(i) {
      const clickedTile = i - 1;
      this.clickedTile.top =
        document.getElementsByClassName("tile")[clickedTile].style.top;
      this.clickedTile.left =
        document.getElementsByClassName("tile")[clickedTile].style.left;
      if (this.clickedTile.left === this.emptyTile.left) {
        if (this.clickedTile.top === this.emptyTile.bottom) {
          // click from down to up
          this.emptyTile.index = this.tilesNumbers.indexOf("x");
          this.tilesNumbers.splice(
            this.emptyTile.index,
            1,
            this.tilesNumbers[this.emptyTile.index + this.col]
          );
          this.tilesNumbers.splice(this.emptyTile.index + this.col, 1, "x");
          this.emptyTile.index = this.emptyTile.index + this.col;
          document.getElementsByClassName("tile")[clickedTile].style.top =
            this.emptyTile.top;
          this.emptyTile.top = this.emptyTile.bottom;
          this.emptyTile.bottom =
            parseInt(this.emptyTile.bottom) + this.tileSize + "px";
        } else if (
          parseInt(this.clickedTile.top) + this.tileSize + "px" ===
          this.emptyTile.top
        ) {
          // click from up to down
          this.emptyTile.index = this.tilesNumbers.indexOf("x");
          this.tilesNumbers.splice(
            this.emptyTile.index,
            1,
            this.tilesNumbers[this.emptyTile.index - this.col]
          );
          this.tilesNumbers.splice(this.emptyTile.index - this.col, 1, "x");
          this.emptyTile.index = this.emptyTile.index - this.col;
          document.getElementsByClassName("tile")[clickedTile].style.top =
            this.emptyTile.top;
          this.emptyTile.bottom = this.emptyTile.top;
          this.emptyTile.top =
            parseInt(this.emptyTile.top) - this.tileSize + "px";
        }
      } else if (this.clickedTile.top === this.emptyTile.top) {
        if (this.clickedTile.left === this.emptyTile.right) {
          // click from right to left
          this.emptyTile.index = this.tilesNumbers.indexOf("x");
          this.tilesNumbers.splice(this.emptyTile.index, 1);
          this.tilesNumbers.splice(this.emptyTile.index + 1, 0, "x");
          this.emptyTile.index++;
          document.getElementsByClassName("tile")[clickedTile].style.left =
            this.emptyTile.left;
          this.emptyTile.left = this.emptyTile.right;
          this.emptyTile.right =
            parseInt(this.emptyTile.right) + this.tileSize + "px";
        } else if (
          parseInt(this.clickedTile.left) + this.tileSize + "px" ===
          this.emptyTile.left
        ) {
          // click from left to right
          this.emptyTile.index = this.tilesNumbers.indexOf("x");
          this.tilesNumbers.splice(this.emptyTile.index, 1);
          this.tilesNumbers.splice(this.emptyTile.index - 1, 0, "x");
          this.emptyTile.index--;
          document.getElementsByClassName("tile")[clickedTile].style.left =
            this.emptyTile.left;
          this.emptyTile.right = this.emptyTile.left;
          this.emptyTile.left =
            parseInt(this.emptyTile.left) - this.tileSize + "px";
        }
      }
      if (this.checkPuzzleCorrection) {
        if (
          this.correctTitlesNumbers.toString() === this.tilesNumbers.toString()
        ) {
          let check;
          clearTimeout(check);
          check = setTimeout(() => {
            if (!alert("Congratssss!")) {
              location.reload();
            }
          }, 300);
        }
      }
    },
  },
};
</script>

<style scoped lang="scss">
.puzzle {
  width: 100%;
  min-height: 50vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  .form {
    margin-bottom: 50px;
    display: flex;
    align-items: center;
    justify-content: center;
    &_ranges {
      &_block {
        border: 1px solid gray;
        border-radius: 5px;
        padding: 10px;
        margin: 10px;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
      }
    }
    &_button {
      padding: 8px 15px;
      font-size: 18px;
      margin-left: 50px;
      color: dodgerblue;
    }
  }
  input {
    display: block;
  }
  .square {
    display: flex;
    flex-wrap: wrap;
    width: auto;
    height: auto;
    background-color: dodgerblue;
    position: relative;
    .tile {
      display: flex;
      align-items: center;
      justify-content: center;
      position: absolute;
      border: 1px solid darkgrey;
      background-color: lightgrey;
      color: darkblue;
      margin: 5px;
      cursor: default;
      transition: 0.2s linear;
      box-sizing: border-box;
      -moz-box-sizing: border-box;
      -webkit-box-sizing: border-box;
    }
  }
}
</style>
