<template>
  <div class="game-container">
    <Win v-if="hasWon" @playAgain="setup"/>
    <div v-else v-for="y in size" :key="y" class="game-row">
      <Switch v-for="(toggled, x) in grid[y-1]" :key="x"
              :isOn="toggled === true" @click="toggle(x,y-1)"/>
    </div>
  </div>
</template>

<script>
import Switch from '@/components/Switch.vue';
import Win from '@/components/Win.vue';

export default {
  name: 'Game',
  components: {
    Win,
    Switch,
  },
  props: {
    size: {
      type: Number,
    },
  },
  data() {
    return {
      grid: [],
      hasWon: false,
    };
  },
  methods: {
    setup() {
      // Generate a row in a grid, randomizing the toggled state based on rate.
      function generateGridRow(n, rate) {
        const row = [];

        for (let i = 0; i < n; i++) {
          row.push(Math.random() < rate);
        }

        return row;
      }

      // Checks if any cell of the grid is toggled on
      function hasAnyOn(grid) {
        for (let y = 0; y < grid.length; y++) {
          for (let x = 0; x < grid.length; x++) {
            if (grid[y][x] === true) {
              return true;
            }
          }
        }

        return false;
      }

      const rate = .05; // 5% chance to be switched on

      this.hasWon = false; // reset the game state to 'playing'

      // generate a grid, check if at least one of the grid cells is on.
      // if all are off, generate another one.
      do {
        this.grid = [];

        for (let n = 0; n < this.size; n++) {
          // for each row in n, generate a row
          this.grid.push(generateGridRow(this.size, rate));
        }
      } while (!hasAnyOn(this.grid));
    },
    toggle(x, y) {
      // toggle switch of clicked element and all adjacent ones
      this.toggleAt(x, y);
      this.toggleAt(x - 1, y);
      this.toggleAt(x + 1, y);
      this.toggleAt(x, y - 1);
      this.toggleAt(x, y + 1);

      this.checkForWin();
    },
    toggleAt(x, y) {
      // toggle a specific cell
      if (y >= 0 && x >= 0 && y < this.size && x < this.size) {
        this.grid[y][x] = !this.grid[y][x];
      }
    },
    checkForWin() {
      // flattens the grid and checks if all the switches are off
      const onSwitches = this.grid.flat()
        .filter((toggled) => toggled === true);

      this.hasWon = onSwitches.length === 0;
    },
  },
  watch: {
    size() {
      // when n changes, re-generate the grid
      this.setup();
    },
  },

  mounted() {
    this.setup();
  },
};
</script>

<style lang="scss" scoped>
.game-container {
  margin-top: 50px;
  display: flex;
  flex-direction: column;
  align-items: center;
  flex-wrap: wrap;

  .game-row {
    display: flex;
    flex-direction: row;
  }
}
</style>
