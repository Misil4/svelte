<script>
  import { onMount } from "svelte";

  const ROWS = 20;
  const COLS = 10;

  class Piece {
    constructor(shape, x, y) {
      this.shape = shape;
      this.x = x;
      this.y = y;
    }
	// Obtener las coordenadas x e y de cada celda ocupada por la pieza
    getCoordinates() {
      const coordinates = [];
      for (let y = 0; y < this.shape.length; y++) {
        for (let x = 0; x < this.shape[y].length; x++) {
          if (this.shape[y][x] === 1) {
            coordinates.push({ x: this.x + x, y: this.y + y });
          }
        }
      }
      return coordinates;
    }

    // Verificar si la pieza colisiona con otra pieza o con el borde del tablero
    collidesWith(board, deltaX, deltaY) {
      const coordinates = this.getCoordinates();
      for (let i = 0; i < coordinates.length; i++) {
        const x = coordinates[i].x + deltaX;
        const y = coordinates[i].y + deltaY;
        if (x < 0 || x >= COLS || y < 0 || y >= ROWS || board[y][x] === 1) {
          return true;
        }
      }
      return false;
    }
  }

  const SHAPES = {
    L: [
      [0, 1, 0],
      [0, 1, 0],
      [0, 1, 1],
    ],
  };

  let board = Array.from({ length: ROWS }, () => Array(COLS).fill(0));
  let piece = new Piece(SHAPES.L, 3, 0);
  const FALL_SPEED = 500; // Tiempo en milisegundos entre cada caída de la pieza

  function generateRandomPiece() {
    // Seleccionar una pieza aleatoria de la lista de formas
    const shapes = Object.values(SHAPES);
    const shape = shapes[Math.floor(Math.random() * shapes.length)];

    // Calcular la posición inicial de la pieza
    const x = Math.floor(Math.random() * (COLS - shape[0].length + 1));
    const y = -shape.length;

    // Crear la nueva pieza
    return new Piece(shape, x, y);
  }
  let intervalId = setInterval(() => {
    // Verificar si la pieza ha llegado al fondo del tablero o se ha chocado con otra pieza
    if (piece.collidesWith(board, 0, 1)) {
      // Actualizar la matriz `board` con la posición final de la pieza
      const coordinates = piece.getCoordinates();
      for (let i = 0; i < coordinates.length; i++) {
        const x = coordinates[i].x;
        const y = coordinates[i].y;
        board[y][x] = 1;
      }

      // Generar una nueva pieza aleatoria
      piece = generateRandomPiece();
    } else {
      piece.y++;
    }
  }, FALL_SPEED);
  function movePiece(deltaX, deltaY) {
  if (!piece.collidesWith(board, deltaX, deltaY)) {
    piece.x += deltaX;
    piece.y += deltaY;
  }
}
</script>
<svelte:window on:keydown={event => {
	if (event.key === 'ArrowLeft') {
	  movePiece(-1, 0);
	}
	if (event.key === 'ArrowRight') {
	  movePiece(+1, 0);
	}
  }} />
<svelte:head>
  <title>Prueba</title>
  <meta name="description" content="Prueba Svelte" />
</svelte:head>
<header>
  <h1>Svelte Tetris</h1>
</header>
<section>
  <table>
    {#each board as row, y}
      <tr>
        {#each row as cell, x}
          <td
            class:filled={cell === 1 ||
              (piece.shape[y - piece.y] &&
                piece.shape[y - piece.y][x - piece.x] === 1)}
            class:piece={piece.shape[y - piece.y] &&
              piece.shape[y - piece.y][x - piece.x] === 1}
          />
        {/each}
      </tr>
    {/each}
  </table>
</section>

<style>
  table {
    border-collapse: collapse;
    margin: 0 auto;
  }

  td {
    width: 40px;
    height: 35px;
    border: 1px solid #ccc;
  }

  .filled {
    background-color: #333;
  }
</style>
