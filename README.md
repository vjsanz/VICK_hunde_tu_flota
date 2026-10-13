# Hundir la Flota

Versión en Python del clásico juego de barcos para jugar contra la máquina desde un notebook de Jupyter. El tablero es de 10 × 10 y la lógica se apoya en NumPy. Las posiciones válidas para colocar cada barco se calculan con **multiplicación de matrices**.

## Requisitos

- Python 3
- NumPy
- Jupyter (Notebook o JupyterLab). El juego usa `IPython.display.clear_output` para refrescar la pantalla.

```bash
pip install numpy jupyter
```

## Cómo ejecutarlo

1. Abre `juego.ipynb`.
2. Ejecuta la **celda 1**: importa las librerías y define los tableros, los barcos y todas las funciones.
3. Ejecuta la **celda 2**: arranca el juego.

Para empezar una partida nueva, vuelve a ejecutar las celdas 1 y 2 en ese orden. Así los tableros y los registros de barcos empiezan vacíos.

## Cómo se juega

### 1. Menú

Escribe `1` para jugar o `2` para salir.

### 2. Colocar tu flota

Por cada barco:

1. **Elige el barco** por su número. Los que ya has colocado aparecen tachados.
2. **Elige la orientación**: `H` (horizontal) o `V` (vertical).
3. **Elige la casilla de inicio.** El juego muestra solo las coordenadas donde el barco cabe. El resto aparece como `X`. En horizontal el barco se extiende hacia la derecha; en vertical, hacia abajo.

| Nº | Barco        | Casillas |
|----|--------------|----------|
| 5  | Portaaviones | 5        |
| 4  | Acorazado    | 4        |
| 3  | Destructor   | 3        |
| 2  | Submarino    | 3        |
| 1  | Patrullero   | 2        |

### 3. La partida

- La máquina coloca sus barcos al azar.
- En tu turno escribes una coordenada (`A1` … `J10`).
- **Si aciertas, vuelves a disparar.** Lo mismo vale para la máquina.
- No puedes disparar dos veces a la misma casilla. Si escribes una coordenada repetida o no válida, el juego vuelve a preguntar.
- Gana quien hunda primero toda la flota contraria.

### Símbolos

| Símbolo | Significado |
|---------|-------------|
| `~`     | Agua |
| `O`     | Barco |
| `X`     | Tocado |
| `-`     | Disparo al agua |

En pantalla se ven tres cosas:

- **Tu tablero:** tus barcos y los disparos de la máquina.
- **Tablero de ataque:** las coordenadas, que se van cambiando por `X` o `-` según disparas.
- **Mensajes del último turno:** agua, tocado o tocado y hundido, con el nombre del barco.

## Cómo funciona por dentro

### Representación

- `tablero` y `tablero_rival` son matrices 10 × 10 de enteros: `0` = agua, `1` = barco.
- `tablero_posiciones` contiene las etiquetas `"A1"` … `"J10"`. Se genera de una vez con *broadcasting*:
  ```python
  np.char.add(letras[:, None], numeros[None, :])
  ```
- Al empezar la partida, cada tablero se convierte en símbolos con `np.where(tablero == 1, "O", "~")`. Sobre esa copia se marcan los disparos.

### Posiciones válidas con multiplicación de matrices

Para saber dónde puede empezar un barco de tamaño `t` sin salirse del tablero ni pisar otro barco, no se recorre casilla a casilla.

**1. `matriz_posibles(tamano, orientacion, tablero)`**

Construye una matriz `M` de 10 × 10:

- **En horizontal**, la columna `i` tiene unos en las filas `i … i + t − 1`. Representa un barco que empieza en la columna `i`.
- **En vertical** es al revés: la fila `i` tiene unos en las columnas `i … i + t − 1`.

**2. `combinacion_posibles(tablero, M, orientacion, tamano)`**

- **Horizontal:** `tablero @ M`. El elemento `[fila, i]` es la suma de las casillas `i … i + t − 1` de esa fila, es decir, cuántas casillas ocupadas pisaría el barco.
- **Vertical:** `M @ tablero` hace lo mismo por columnas.
- `== 0` marca como válidos los inicios que no pisan ningún barco.
- Las últimas `t − 1` columnas (o filas, en vertical) se fuerzan a `False`, porque ahí el barco se saldría del tablero.

Ejemplo con una fila del tablero y un barco de 3 en horizontal:

```
fila del tablero   0 0 0 1 0 0 0 0 0 0
fila @ M           0 1 1 1 0 0 0 0 0 0
== 0               T F F F T T T T T T
límite aplicado    T F F F T T T T F F
```

**3. Uso del resultado**

El resultado es una máscara booleana 10 × 10 alineada con `tablero_posiciones`, que se aprovecha en tres sitios:

- `np.where(j, tablero_posiciones, "X")` muestra al jugador solo las casillas válidas.
- `set(tablero_posiciones[j])` valida la coordenada que escribe el jugador.
- `np.random.choice(tablero_posiciones[j])` lo usa la máquina para colocar al azar.

### Registro de barcos, tocado y hundido

- `coloca_barco` pinta el barco en el tablero y **devuelve la lista de casillas** que ocupa. Esa lista se guarda con el nombre del barco en `barcos_jugador` o en `barcos_rival`.
- `disparar` cambia `O` por `X` (o agua por `-`). Lo marca en el tablero atacado y en el tablero de ataque de quien dispara.
- `barco_tocado` busca a qué barco pertenece la casilla.
- `barco_hundido` comprueba si todas las casillas de ese barco son `X`.
- `quedan_barcos` usa `np.any(tablero == "O")`. Cuando devuelve `False`, la partida termina.

## Funciones principales

| Función | Qué hace |
|---------|----------|
| `lista_barcos` | Muestra los barcos por colocar y tacha los ya colocados |
| `matriz_posibles` | Construye la matriz `M` para un tamaño y una orientación |
| `combinacion_posibles` | Multiplica tablero y `M` y devuelve la máscara de inicios válidos |
| `coloca_barco` | Pinta el barco y devuelve sus casillas |
| `imprime_tablero_barcos` | Convierte el tablero 0/1 en símbolos `O` / `~` |
| `coloca_barcos_aleatorio` | Coloca la flota de la máquina al azar |
| `coordenada_a_indices` | Convierte `"B7"` en `(1, 6)` |
| `disparar` | Resuelve un disparo: agua, tocado o hundido |
| `barco_tocado` / `barco_hundido` | Identifican el barco alcanzado y si se ha hundido |
| `quedan_barcos` | Indica si a un jugador le queda algún barco a flote |
| `mensaje_disparo` | Genera el texto del resultado de cada disparo |

## Modo test

En la celda 1:

```python
MODO_TEST = True
```

Con `True` se muestra el tablero rival durante la partida, útil para comprobar tocado, hundido y el final. Para jugar normal, ponlo a `False`.

## Limitaciones y posibles mejoras

- Los barcos pueden quedar pegados entre sí. Se podría ampliar la matriz de posibles para exigir una casilla de agua alrededor.
- La máquina dispara siempre al azar. Podría perseguir un barco después de tocarlo.
- Para jugar otra vez hay que volver a ejecutar las celdas.
- Los tableros se imprimen con el formato de NumPy.
