export type Tile = {
  id: string;
  value: number;
  row: number;
  col: number;
  isNew?: boolean;
  mergedFrom?: string[];
};

export type Grid = (Tile | null)[][];

export type Direction = 'up' | 'down' | 'left' | 'right';

function createEmptyGrid(): Grid {
  return Array(4).fill(null).map(() => Array(4).fill(null));
}

export function initializeGame(): { grid: Grid; score: number } {
  const grid = createEmptyGrid();
  addRandomTile(grid);
  addRandomTile(grid);
  return { grid, score: 0 };
}

function addRandomTile(grid: Grid): void {
  const emptyCells: { row: number; col: number }[] = [];
  
  for (let row = 0; row < 4; row++) {
    for (let col = 0; col < 4; col++) {
      if (!grid[row][col]) {
        emptyCells.push({ row, col });
      }
    }
  }
  
  if (emptyCells.length === 0) return;
  
  const { row, col } = emptyCells[Math.floor(Math.random() * emptyCells.length)];
  const value = Math.random() < 0.9 ? 2 : 4;
  
  grid[row][col] = {
    id: `${Date.now()}-${Math.random()}`,
    value,
    row,
    col,
    isNew: true,
  };
}

function rotateCW(grid: Grid): Grid {
  const n = 4;
  const rotated = createEmptyGrid();
  for (let row = 0; row < n; row++) {
    for (let col = 0; col < n; col++) {
      rotated[col][n - 1 - row] = grid[row][col];
      if (rotated[col][n - 1 - row]) {
        rotated[col][n - 1 - row] = {
          ...rotated[col][n - 1 - row]!,
          row: col,
          col: n - 1 - row,
        };
      }
    }
  }
  return rotated;
}

function rotateCCW(grid: Grid): Grid {
  const n = 4;
  const rotated = createEmptyGrid();
  for (let row = 0; row < n; row++) {
    for (let col = 0; col < n; col++) {
      rotated[n - 1 - col][row] = grid[row][col];
      if (rotated[n - 1 - col][row]) {
        rotated[n - 1 - col][row] = {
          ...rotated[n - 1 - col][row]!,
          row: n - 1 - col,
          col: row,
        };
      }
    }
  }
  return rotated;
}

function moveLeft(grid: Grid): { grid: Grid; score: number; moved: boolean } {
  let totalScore = 0;
  let moved = false;
  const newGrid = createEmptyGrid();

  for (let row = 0; row < 4; row++) {
    const tiles = grid[row].filter(tile => tile !== null) as Tile[];
    let col = 0;

    for (let i = 0; i < tiles.length; i++) {
      const tile = tiles[i];
      
      if (i < tiles.length - 1 && tile.value === tiles[i + 1].value) {
        const mergedValue = tile.value * 2;
        newGrid[row][col] = {
          id: `${Date.now()}-${Math.random()}`,
          value: mergedValue,
          row,
          col,
          mergedFrom: [tile.id, tiles[i + 1].id],
        };
        totalScore += mergedValue;
        i++;
        moved = true;
      } else {
        newGrid[row][col] = {
          ...tile,
          row,
          col,
        };
        if (tile.col !== col) moved = true;
      }
      col++;
    }
  }

  return { grid: newGrid, score: totalScore, moved };
}

export function move(grid: Grid, direction: Direction): { grid: Grid; score: number; moved: boolean } {
  let workingGrid = grid.map(row => [...row]);
  
  if (direction === 'up') {
    workingGrid = rotateCCW(workingGrid);
    const result = moveLeft(workingGrid);
    result.grid = rotateCW(result.grid);
    return result;
  } else if (direction === 'down') {
    workingGrid = rotateCW(workingGrid);
    const result = moveLeft(workingGrid);
    result.grid = rotateCCW(result.grid);
    return result;
  } else if (direction === 'right') {
    workingGrid = rotateCW(rotateCW(workingGrid));
    const result = moveLeft(workingGrid);
    result.grid = rotateCCW(rotateCCW(result.grid));
    return result;
  } else {
    return moveLeft(workingGrid);
  }
}

export function canMove(grid: Grid): boolean {
  for (let row = 0; row < 4; row++) {
    for (let col = 0; col < 4; col++) {
      if (!grid[row][col]) return true;
      
      const current = grid[row][col]!;
      
      if (col < 3 && grid[row][col + 1]?.value === current.value) return true;
      if (row < 3 && grid[row + 1][col]?.value === current.value) return true;
    }
  }
  return false;
}

export function makeMove(grid: Grid, direction: Direction, currentScore: number): { grid: Grid; score: number; gameOver: boolean } {
  const result = move(grid, direction);
  
  if (result.moved) {
    addRandomTile(result.grid);
    const newScore = currentScore + result.score;
    const gameOver = !canMove(result.grid);
    return { grid: result.grid, score: newScore, gameOver };
  }
  
  return { grid, score: currentScore, gameOver: !canMove(grid) };
}

