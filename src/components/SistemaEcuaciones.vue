<template>
  <div class="container">
    <!-- Panel izquierdo -->
    <aside class="panel">
      <h2>Opciones</h2>

      <label>Tamaño del sistema</label>
      <input
        type="number"
        v-model.number="tamano"
        min="1"
        max="120"
        @input="generarMatriz"
        placeholder="Ej: 3"
      />

      <label>Metodo numerico</label>
      <select v-model="metodo">
        <option>Eliminacion de Gauss</option>
        <option>Gauss-Jordan</option>
        <option>Gauss-Seidel</option>
      </select>

      <button @click="limpiar">Limpiar</button>
    </aside>

    <!-- Contenido principal -->
    <main class="principal">
      <h1>Resolver Sistemas de Ecuaciones Lineales</h1>

      <section class="entrada">
        <h3>Entrada</h3>

        <div v-for="(fila, i) in matriz" :key="i" class="fila">
          <template v-for="(valor, j) in fila" :key="j">
            <input
              type="number"
              v-model.number="matriz[i][j]"
              step="any"
            />
            <span v-if="j < tamano - 1">x{{ j + 1 }} +</span>
            <span v-else>x{{ j + 1 }} =</span>
          </template>
          <input
            type="number"
            v-model.number="resultados[i]"
            step="any"
            placeholder="b{{ i + 1 }}"
          />
        </div>

        <button class="resolver" @click="resolver">Resolver</button>
      </section>

      <section class="resultado">
        <h3>Resultado</h3>
        <div v-if="soluciones.length === 0 && errorMsg === ''">Sin resultados aun.</div>
        <div v-if="errorMsg" class="error">{{ errorMsg }}</div>
        <ul v-else>
          <li v-for="(sol, i) in soluciones" :key="i">x{{ i + 1 }} = {{ sol.toFixed(6) }}</li>
        </ul>
      </section>
    </main>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// Variables principales
const tamano = ref(3)
const metodo = ref('Eliminación de Gauss')
const matriz = ref([])
const resultados = ref([])
const soluciones = ref([])
const errorMsg = ref('')

// Genera matriz vacía
function generarMatriz() {
  if (tamano.value > 120) tamano.value = 120
  matriz.value = Array.from({ length: tamano.value }, () =>
    Array(tamano.value).fill(0)
  )
  resultados.value = Array(tamano.value).fill(0)
  soluciones.value = []
  errorMsg.value = ''
}

// Limpia todo
function limpiar() {
  generarMatriz()
}

// Método principal
function resolver() {
  errorMsg.value = ''
  soluciones.value = []

  try {
    if (metodo.value === 'Eliminación de Gauss') {
      soluciones.value = gauss(matriz.value, resultados.value)
    } else if (metodo.value === 'Gauss-Jordan') {
      soluciones.value = gaussJordan(matriz.value, resultados.value)
    } else if (metodo.value === 'Gauss-Seidel') {
      soluciones.value = gaussSeidel(matriz.value, resultados.value)
    }
  } catch (e) {
    errorMsg.value = e.message || 'Error al resolver el sistema.'
  }
}

// --------------------- MÉTODOS NUMÉRICOS ---------------------

// Eliminación de Gauss
function gauss(A, b) {
  const n = A.length
  const M = A.map((fila, i) => [...fila, b[i]])

  // Eliminación hacia adelante
  for (let i = 0; i < n; i++) {
    let maxRow = i
    for (let k = i + 1; k < n; k++) {
      if (Math.abs(M[k][i]) > Math.abs(M[maxRow][i])) {
        maxRow = k
      }
    }
    if (M[maxRow][i] === 0) throw new Error('El sistema no tiene solución única.')

    // Intercambiar filas
    ;[M[i], M[maxRow]] = [M[maxRow], M[i]]

    for (let k = i + 1; k < n; k++) {
      const factor = M[k][i] / M[i][i]
      for (let j = i; j <= n; j++) {
        M[k][j] -= factor * M[i][j]
      }
    }
  }

  // Sustitución hacia atrás
  const x = Array(n).fill(0)
  for (let i = n - 1; i >= 0; i--) {
    let sum = 0
    for (let j = i + 1; j < n; j++) sum += M[i][j] * x[j]
    x[i] = (M[i][n] - sum) / M[i][i]
  }
  return x
}

// Gauss-Jordan
function gaussJordan(A, b) {
  const n = A.length
  const M = A.map((fila, i) => [...fila, b[i]])

  for (let i = 0; i < n; i++) {
    let diag = M[i][i]
    if (diag === 0) throw new Error('División entre cero en pivote.')

    for (let j = 0; j <= n; j++) M[i][j] /= diag

    for (let k = 0; k < n; k++) {
      if (k !== i) {
        const factor = M[k][i]
        for (let j = 0; j <= n; j++) {
          M[k][j] -= factor * M[i][j]
        }
      }
    }
  }

  return M.map((fila) => fila[n])
}

// Gauss-Seidel
function gaussSeidel(A, b, tol = 1e-6, maxIter = 1000) {
  const n = A.length
  const x = Array(n).fill(0)
  let iter = 0
  while (iter < maxIter) {
    const prev = [...x]
    for (let i = 0; i < n; i++) {
      let sum = 0
      for (let j = 0; j < n; j++) {
        if (j !== i) sum += A[i][j] * x[j]
      }
      x[i] = (b[i] - sum) / A[i][i]
    }

    const err = Math.sqrt(x.reduce((acc, xi, i) => acc + (xi - prev[i]) ** 2, 0))
    if (err < tol) return x
    iter++
  }
  throw new Error('Gauss-Seidel no convergió.')
}

// -------------------------------------------------------------

// Inicializar al cargar
generarMatriz()
</script>

<style scoped>
.container {
  display: flex;
  background-color: #0d1117;
  color: #c9d1d9;
  min-height: 100vh;
  padding: 20px;
  gap: 40px;
  font-family: "Segoe UI", sans-serif;
}

/* Panel izquierdo */
.panel {
  background: #161b22;
  padding: 20px;
  border-radius: 12px;
  width: 240px;
  height: fit-content;
  box-shadow: 0 0 10px #0006;
}
.panel h2 {
  color: #58a6ff;
  margin-bottom: 10px;
}
.panel label {
  display: block;
  margin-top: 10px;
  font-weight: bold;
}
.panel select,
.panel input,
.panel button {
  width: 100%;
  padding: 6px;
  margin-top: 5px;
  border-radius: 6px;
  border: none;
  outline: none;
}
.panel input {
  background: #21262d;
  color: #fff;
  border: 1px solid #30363d;
}
.panel button {
  background: #238636;
  color: white;
  cursor: pointer;
  transition: 0.3s;
}
.panel button:hover {
  background: #2ea043;
}

/* Contenido principal */
.principal {
  flex: 1;
}
h1 {
  color: #79c0ff;
  text-align: center;
  margin-bottom: 20px;
}
.entrada h3,
.resultado h3 {
  color: #9be9a8;
}
.fila {
  display: flex;
  align-items: center;
  gap: 5px;
  margin-bottom: 8px;
}
input[type="number"] {
  width: 70px;
  background: #21262d;
  border: 1px solid #30363d;
  border-radius: 6px;
  color: white;
  padding: 4px;
  text-align: center;
}
.resolver {
  background: #1f6feb;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 8px 20px;
  margin-top: 15px;
  cursor: pointer;
  transition: 0.3s;
}
.resolver:hover {
  background: #388bfd;
}
.resultado {
  margin-top: 20px;
  background: #161b22;
  padding: 15px;
  border-radius: 12px;
}
.error {
  color: #f85149;
  background: #1c0f0f;
  border: 1px solid #f85149;
  padding: 8px;
  border-radius: 6px;
}
</style>
