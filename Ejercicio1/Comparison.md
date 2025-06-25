# Comparación de Resultados: SEQUENTIAL_THRESHOLD y Speedup

| SEQUENTIAL_THRESHOLD | testParSimpleTwoMillion | testParManyTaskTwoMillion | testParManyTaskTwoHundredMillion | Fallos Totales | Máx. Speedup Observado | Notas sobre el rendimiento                |
|---------------------:|:----------------------:|:------------------------:|:-------------------------------:|:--------------:|:---------------------:|:------------------------------------------|
| 500                  | 1.0x (fallo)           | 2.0x (fallo)             | 3.76x (fallo)                   | 3              | ~3.76x                | Muchas tareas pequeñas, alto overhead      |
| 1,000                | 1.0x (fallo)           | 2.0x (fallo)             | 3.76x (fallo)                   | 3              | ~3.76x                | Similar a 500, overhead alto              |
| 5,000                | 1.0x (fallo)           | 2.0x (fallo)             | 3.03x (fallo)                   | 3              | ~3.03x                | Menos tareas, menor overhead              |
| 10,000               | 1.0x (ok)              | 2.0x (fallo)             | 3.11x (fallo)                   | 2              | ~3.11x                | Mejor balance, overhead aceptable         |
| 20,000               | 1.0x (ok)              | 2.0x (fallo)             | 3.48x (fallo)                   | 2              | ~3.48x                | Mejor resultado hasta ese punto           |
| 50,000               | 1.0x (ok)              | 2.0x (fallo)             | 3.51x (fallo)                   | 2              | ~3.51x                | Similar a 20,000, overhead muy bajo       |
| 100,000              | 1.0x (ok)              | 2.0x (fallo)             | 4.98x (fallo)                   | 2              | ~4.98x                | Mejor speedup tras reinicio, muy eficiente |
| 150,000              | 1.0x (ok)              | 1.0x (fallo)             | 4.27x (fallo)                   | 2              | ~4.27x                | Speedup alto, pero test de 2M tasks bajo  |
| 300,000              | 1.0x (ok)              | 2.0x (fallo)             | 3.80x (fallo)                   | 2              | ~3.80x                | Muy bajo overhead, pero menos paralelismo |

---

## Explicación

- **SEQUENTIAL_THRESHOLD** define el tamaño mínimo de trabajo que una tarea realiza de forma secuencial antes de dividirse (split) en subtareas usando fork/join.
- **Umbral bajo (por ejemplo, 500):**
  - Se crean muchas tareas pequeñas.
  - El sistema gasta mucho tiempo y recursos en gestionar tareas (overhead de fork/join).
  - El rendimiento paralelo real disminuye porque el costo de coordinación supera el beneficio de paralelizar.
- **Umbral alto (por ejemplo, 100,000 o más):**
  - Se crean menos tareas, cada una con más trabajo.
  - Menor overhead, pero si el umbral es demasiado alto, se pierde paralelismo porque hay menos tareas que hilos disponibles.
  - En tu caso, valores altos como 10,000, 20,000, 50,000 o 100,000 logran el mejor balance entre overhead y paralelismo.

**En resumen:**  
El umbral debe ser lo suficientemente bajo para aprovechar todos los núcleos, pero lo suficientemente alto para que el overhead de crear y gestionar tareas no supere el beneficio de paralelizar. El valor óptimo depende del hardware, el tamaño del arreglo y la implementación de la JVM.

---

**Conclusión:**  
A medida que aumentas el threshold, reduces el overhead y mejoras el speedup, pero llega un punto donde ya no hay mejora significativa. Por eso, con 10,000 en adelante, solo fallan los tests de "many tasks", y el speedup real se estabiliza. Esto es lo esperado en la práctica y tu código es