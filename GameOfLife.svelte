<script lang="ts">
  import { onDestroy } from 'svelte';

  console.log('GameOfLife component mounted');

  // Helpers
  function createGrid(rows: number, cols: number) {
    return Array.from({ length: rows }, () => Array.from({ length: cols }, () => 0));
  }

  function nextGeneration(grid: number[][]) {
    const rows = grid.length;
    const cols = rows ? grid[0].length : 0;
    const out = createGrid(rows, cols);
    for (let r = 0; r < rows; r++) {
      for (let c = 0; c < cols; c++) {
        let n = 0;
        for (let dr = -1; dr <= 1; dr++) {
          for (let dc = -1; dc <= 1; dc++) {
            if (dr === 0 && dc === 0) continue;
            const rr = r + dr;
            const cc = c + dc;
            if (rr >= 0 && rr < rows && cc >= 0 && cc < cols) n += grid[rr][cc];
          }
        }
        const alive = grid[r][c] === 1;
        out[r][c] = (alive && (n === 2 || n === 3)) || (!alive && n === 3) ? 1 : 0;
      }
    }
    return out;
  }

  function toggleCell(grid: number[][], r: number, c: number) {
    const copy = grid.map(row => row.slice());
    copy[r][c] = copy[r][c] ? 0 : 1;
    return copy;
  }

  // UI state
  let rows = 40;
  let cols = 40;
  let grid = createGrid(rows, cols);
  let gridStyle = `grid-template-columns: repeat(${cols}, 30px);`;

  let running = false;
  let timer: ReturnType<typeof setInterval> | null = null;

  function start() {
    if (running) return;
    running = true;
    timer = setInterval(() => {
      grid = nextGeneration(grid);
    }, 250);
  }

  function stop() {
    running = false;
    if (timer) {
      clearInterval(timer);
      timer = null;
    }
  }

  function toggle(r: number, c: number) {
    grid = toggleCell(grid, r, c);
  }

  onDestroy(() => {
    if (timer) clearInterval(timer);
  });

  // Initialize grid
  grid = createGrid(rows, cols);
</script>

<style>
  :global(body) {
    background: #000;
  }

  .game {
    width: 100%;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 12px;
    padding: 12px;
    background: #2f03f3;
  }

  .controls {
    display: flex;
    gap: 16px;
    flex-wrap: wrap;
    align-items: center;
    padding: 6px;
  }

  .controls button {
    background: #f3f4f6;
    border: 1px solid #d1d5db;
    color: #0f172a;
    padding: 6px 8px;
    border-radius: 6px;
    cursor: pointer;
  }

  .board {
    padding: 16px;
    overflow: auto;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .grid {
    display: inline-grid;
    user-select: none;
    gap: 20;
    padding: 0;
    background: #000;
    border-radius: 16px;
  }

  .cell {
    box-sizing: border-box;
    background: #ffffff;
    border: 0.5px solid #000;
    width: 30px;
    height: 30px;
  }

  .cell.alive { background: #16a34a }
</style>

<div class="game">
  <div class="controls">
    <button on:click={running ? stop : start}>{running ? 'Stop' : 'Start'}</button>
  </div>

  <div class="board">
    <div
      class="grid"
      style={gridStyle}
    >
      {#each grid as row, r}
        {#each row as cell, c}
          <div
            class="cell {cell ? 'alive' : ''}"
            role="button"
            tabindex="0"
            on:click={() => toggle(r, c)}
            on:keydown={(e) => { if (e.key === 'Enter' || e.key === ' ') { toggle(r, c); e.preventDefault(); } }}
          ></div>
        {/each}
      {/each}
    </div>
  </div>
</div>
