# Difference report: `widget.html` vs `widgetold.html`

## Overview
`widget.html` is a simplified version of the search/filter pipeline compared with `widgetold.html`.

## Key differences

1. **Search index implementation changed**
   - `widgetold.html` builds tokenized search rows via `buildSearchRows` and includes a Web Worker-based inverted index/search pipeline.
   - `widget.html` replaces this with `buildSearchBlobs`, which precomputes lowercase text blobs (`questionBlob`, `optionsBlob`, `explanationBlob`, and `searchBlob`) and performs filtering directly without the worker pipeline.

2. **Web Worker search pipeline removed**
   - The large `SEARCH_WORKER_SCRIPT` and worker orchestration/caching logic present in `widgetold.html` are removed in `widget.html`.
   - `widget.html` now returns filtered records directly from indices (`indices.map((i) => ROWS[i].record)`) and renders synchronously via existing filtering functions.

3. **Rendering flow simplified**
   - `renderFilteredData(records, options = {})` in `widgetold.html` became `renderFilteredData(records)` in `widget.html`.
   - Progressive/partial-render options and `totalCount` handling were removed.
   - Render observer setup is now always executed after first render call in this path.

4. **Header count logic simplified**
   - `updateHeaderStats(filteredRecords = null, totalCount = null)` became `updateHeaderStats(filteredRecords = null)`.
   - Count is now always `filtered.length` (no separate total count override).

5. **Facet cache key no longer depends on search scopes**
   - `makeFacetCacheKey` in `widgetold.html` encoded query/options/explanation scope toggles into the cache key.
   - `widget.html` only uses `facetKey`, search text, and filter selections.

6. **Cache invalidation hooks removed in several UI handlers**
   - Calls like `invalidateSearchCache("filters-changed")` and `invalidateSearchCache("scope-changed")` were removed from filter/scope update handlers.

7. **Records update initialization simplified**
   - In `grist.onRecords`, `buildSearchRows(DATA)` changed to `buildSearchBlobs(DATA)`.
   - Worker init and search cache priming logic were removed.
   - Path now directly calls `renderFilteredData(getFilteredData())` after rebuilding filters and UI state.

## Functional implication
- `widgetold.html` favors a more complex, worker-assisted token-index search architecture with cache invalidation controls.
- `widget.html` favors a leaner in-thread blob-based matching pipeline with less moving parts.
