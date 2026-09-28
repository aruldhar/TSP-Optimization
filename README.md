# Traveling Salesman Problem: a 13,509-City Heuristic Pipeline

A heuristic pipeline for the 13,509-city `usa13509` TSP instance (every U.S. city with population of at least 500), built as the final project for ISEN 320 (Operations Research I) at Texas A&M, December 2025.

**Team:** Arul Dhar and Zakaria Majidi

## Results

| Stage | Tour length | Notes |
| --- | --- | --- |
| Nearest neighbor, 104 diverse starts | 24,985,586 (best) to 25,366,391 (worst), mean 25,209,350 | About 15 s per start; about 25 min for all 104 |
| Best-improvement 2-opt on the 10 shortest NN tours (5 passes each) | **24,573,793** | 1.8% shorter than its starting tour |
| Published optimal tour ([Waterloo TSP](https://www.math.uwaterloo.ca/tsp/usa13509/usa13509_sol.html)) | 19,982,859 | Our best tour is about 23% above optimal |

The best final tour came from starting tour #4, not from the shortest nearest-neighbor tour. The best starting point is not always the best tour to improve, which is why the pipeline runs 2-opt on the top 10 rather than only the single best.

## Approach

1. **Formulation.** The report writes the TSP as a binary integer program (a binary variable per directed edge, enter-once and leave-once constraints, and subtour elimination) and explains why branch-and-bound is exact but impractical at this size.
2. **Nearest neighbor (NN) construction.** From a start city, repeatedly visit the closest unvisited city. One run is O(n²), about 15 seconds here.
3. **Multi-start with diverse starting cities.** Running NN from all 13,509 cities would take days of CPU time. Instead, the pipeline picks the 4 extreme cities (min/max x and y) plus the 25 cities farthest from each: 104 starts spread across the map.
4. **Best-improvement 2-opt.** Each pass checks every pair of edges and applies the single swap that shortens the tour most.

### A design decision worth noting

Uncapped best-improvement 2-opt ran for hours on this instance. A first-improvement variant was much faster, but even at 100 passes it converged to worse tours than 3 passes of best-improvement. The final design keeps best-improvement for quality and adds a `max_passes` cap (5) to control runtime, applied only to the 10 shortest NN tours.

## Repository structure

| Path | Contents |
| --- | --- |
| `src/nearest_neighbor.py` | Reads the instance, selects the 104 starts, writes all NN tours |
| `src/two_opt.py` | Best-improvement 2-opt with a pass cap on the 10 best NN tours |
| `src/run_pipeline.py` | Runs both steps in order |
| `data/usa13509.tsp` | TSPLIB instance (EUC_2D coordinates) |
| `results/nearest_neighbor_tours.txt` | All 104 NN tours with their lengths |
| `results/best_tour_found.txt` | Best tour after 2-opt |
| `docs/ISEN_320_Final_Project.pdf` | Full report: formulation, branch-and-bound, heuristic design, complexity |

## Running it

Python 3 standard library only; no packages to install.

```bash
python src/run_pipeline.py        # full pipeline
# or step by step
python src/nearest_neighbor.py
python src/two_opt.py
```

Expect roughly 25 minutes for the nearest-neighbor stage on a typical laptop, plus the 2-opt passes.

## What could make it better

- Or-opt or 3-opt moves, or Lin-Kernighan, to escape 2-opt local optima
- Neighbor lists (only testing swaps between nearby cities) to cut the O(n²) cost per pass
- A vectorized or compiled implementation (NumPy or Numba) for faster passes
