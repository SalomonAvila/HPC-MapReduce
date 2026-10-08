# Plan de implementación: simulador 3D de MapReduce

**Autor:** Salomon Alfredo Avila Larrotta
**Entregable:** un solo archivo `index.html` estático, sin backend ni build, en la rama `main`.

---

## 1. Tecnología

| Aspecto | Decisión |
|---|---|
| Archivo | `index.html` con todo el CSS y el JS dentro |
| 3D | **Three.js** con versión fija (`three@0.160.0`), cargado por CDN mediante `importmap` |
| Cámara | `OrbitControls` para rotar, hacer zoom y desplazar |
| Etiquetas | `CSS2DRenderer` para que los pares `<Tokyo, 38>` se lean con nitidez sobre los objetos 3D |
| Animación | `requestAnimationFrame` con *tweening* propio (sin librerías extra) |
| Interfaz | Panel lateral HTML/CSS con sliders, botones y métricas |

## 2. Modelo de datos (ejemplo de IBM)

- **Input por defecto:** los 15 registros `<Ciudad, Temperatura>` de la imagen (Tokyo, Toronto, New York, London).
- **Función Map:** emite `(ciudad, temp)`.
- **Partición:** `hash(ciudad) % R`, donde R es el número de reducers.
- **Función Reduce:** máximo por ciudad (la de IBM). Como extra se podrá elegir mínimo, promedio o conteo.
- **Resultado esperado:** `<Tokyo,38> <London,27> <New York,33> <Toronto,32>`.
- Toda la simulación se calcula primero en JS puro (`computeMapReduce(config)`). La parte 3D solo reproduce ese resultado, así que los datos mostrados siempre son correctos.

## 3. Escena 3D: las seis etapas como columnas

Las etapas se ubican de izquierda a derecha sobre el eje X, igual que en el diagrama de IBM:

1. **Input:** un bloque grande (el archivo) con los 15 registros.
2. **Splitting:** el bloque se divide en N fragmentos que se mueven hacia sus mappers.
3. **Mapping:** N servidores 3D (cajas tipo *rack* con un LED de estado), cada uno con su lista de pares.
4. **Shuffling:** partículas, una por registro, viajan por curvas Bézier desde cada mapper hasta el reducer de su clave. Así se ve el cruce "todos con todos" del diagrama.
5. **Reducing:** R servidores, cada uno con el valor agregado de su clave.
6. **Result:** el bloque final con la salida combinada.

Detalles visuales:
- Cada ciudad tiene su color (por ejemplo Tokyo naranja, London verde, New York morado, Toronto azul), que se mantiene en todas las etapas.
- Sobre cada columna aparece el nombre de la etapa, y la etapa activa se ilumina.
- Hay una rejilla en el piso, iluminación suave y modo claro/oscuro.

## 4. Interactividad: "diferentes recursos"

Panel de configuración:
- **Número de mappers / splits:** de 1 a 6. Se redistribuyen los registros y se ve el efecto en la carga.
- **Número de reducers:** de 1 a 4. Con menos reducers que claves, un reducer procesa varias ciudades, lo que muestra el particionamiento.
- **Velocidad por nodo:** nodos homogéneos o heterogéneos, para provocar un *straggler* (nodo lento) visible.
- **Ancho de banda de red:** cambia la duración del shuffle.
- **Combiner on/off:** agrega antes del shuffle y reduce las partículas transferidas.
- **Simular fallo de un nodo:** un mapper se pone en rojo y su tarea se reejecuta en otro nodo.
- **Dataset:** el de IBM, uno aleatorio de tamaño configurable o uno editado a mano en un textarea.
- **Función reduce:** máx, mín, promedio o conteo.

Controles de reproducción:
- ▶ Play, ⏸ Pausa, ⏭ Siguiente etapa, ⟲ Reiniciar y un slider de velocidad (0.25× a 4×).
- Un clic en un nodo muestra su detalle (registros, tiempo, carga).
- Botones de cámara: vista general, enfocar la etapa actual y vista superior 2D, equivalente al diagrama de IBM.

## 5. Métricas en vivo

- Tiempo simulado por etapa y tiempo total.
- Registros transferidos en el shuffle, con y sin combiner.
- Carga por mapper y por reducer en barras pequeñas, con un indicador de desbalance.
- Una explicación de texto breve de lo que ocurre en cada etapa, que sirve como apoyo didáctico.

## 6. Autoría

- `<meta name="author" content="Salomon Alfredo Avila Larrotta">` y comentario de cabecera en el código.
- Título visible en el encabezado: *"Simulador MapReduce 3D — Salomon Alfredo Avila Larrotta"*.
- Pie de página fijo con el nombre, el curso (HPC) y la nota "Desarrollado con apoyo de IA".

## 7. Estructura interna del código

```
index.html
 ├─ <style>            layout, panel, tema
 ├─ <script importmap> three, addons
 └─ <script module>
     ├─ DATA           dataset IBM + generador
     ├─ ENGINE         split → map → (combine) → partition → reduce, con tiempos
     ├─ SCENE          setup Three.js, cámara, luces, controles
     ├─ BUILDERS       crearNodo(), crearBloque(), crearEtiqueta()
     ├─ ANIMATOR       línea de tiempo por etapas, tweens, partículas
     ├─ UI             panel, eventos, métricas
     └─ MAIN           rebuild(config) al cambiar parámetros
```

## 8. Hitos

1. Esqueleto HTML, escena Three.js y las 6 columnas con su rótulo.
2. Motor MapReduce en JS con pruebas en consola contra el resultado de IBM.
3. Construcción estática de nodos y etiquetas a partir del resultado.
4. Animación por etapas: split, map, shuffle, reduce y result.
5. Panel de recursos que reconstruye la escena al cambiar parámetros.
6. Métricas, extras (combiner, fallo, straggler) y autoría.
7. Prueba en navegador (captura de pantalla y revisión de la consola) en escritorio y móvil.
8. Commit y push a `main`.

## 9. Criterios de aceptación

- Abre con doble clic o en GitHub Pages, sin servidor.
- Con la configuración por defecto (3 mappers y 4 reducers) reproduce exactamente el diagrama de IBM.
- Cada etapa se distingue con claridad y se puede recorrer paso a paso.
- Cambiar los recursos altera la visualización y las métricas de forma coherente.
- El nombre del autor aparece en la página y en los metadatos.
