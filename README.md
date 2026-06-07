# Health Facility Proximity to Waterways Risk Screen

## Project Overview

This project screens all health facilities in Ghana for flood corridor exposure by evaluating their proximity to the national waterway and river network. Facilities located within 500 metres of any waterway or river are classified as flood-risk assets. The output is a fully tagged point layer distinguishing At Risk from Safe facilities, alongside a filtered layer of at-risk facilities only — enabling health sector planners, disaster risk reduction teams, and infrastructure managers to rapidly prioritise facilities that may be disrupted during flood events.

---

## Coordinate Reference System

All layers are in **EPSG:25000 (Accra / Ghana National Grid)**. No reprojection was required as all source layers were already in this CRS. Distance thresholds are applied in metres, consistent with the metric projection.

---

## Input Layers

| Layer | File | Geometry | CRS | Features |
|---|---|---|---|---|
| Waterways | waterways_rivers_merged.gpkg | Line | EPSG:25000 | 9,159 |
| Health facilities | health_facilities_reproj.gpkg | Point | EPSG:25000 | 1,905 |

The waterway network was produced by merging the `waterways` and `rivers` layers using **Merge Vector Layers** (native:mergevectorlayers), consolidating all watercourse features into a single unified line dataset before buffering.

---

## Processing Workflow

### Step 1 — Merge waterway layers
`waterways` and `rivers` were merged into `waterways_rivers_merged.gpkg` to ensure the buffer encompasses every watercourse type present in the dataset, regardless of its original source layer.

### Step 2 — Generate 500 m flood corridor buffer
A dissolved buffer of **500 metres** was applied to `waterways_rivers_merged.gpkg` using **Buffer** (native:buffer) with Dissolve result enabled. This produces a single unified polygon representing the full flood proximity corridor. Output saved as `flood_corridor_500m.gpkg`.

The 500-metre threshold is a standard flood-risk proximity convention for infrastructure screening and must not be altered without explicit justification from the commissioning authority.

### Step 3 — Tag health facilities by flood risk
Each health facility was evaluated against the flood corridor using **Join Attributes by Location** (native:joinattributesbylocation) with a geometric intersection predicate. A **Field Calculator** pass then added the field `flood_risk` (text, 10 characters), populated as:

```
if(intersects($geometry, geometry(get_feature_by_id('flood_corridor_500m', 1))), 'At Risk', 'Safe')
```

Output saved as `health_facilities_risk_tagged.gpkg`. Feature count is identical to the input (1,905), preserving full dataset integrity.

### Step 4 — Extract at-risk facilities
**Extract by Attribute** (native:extractbyattribute) filtered `health_facilities_risk_tagged.gpkg` on `flood_risk = 'At Risk'`, saving the subset as `health_facilities_at_risk.gpkg`.

---

## Output Layers

| Layer | File | Features | Key Fields |
|---|---|---|---|
| Merged waterway network | waterways_rivers_merged.gpkg | 9,159 | name, waterway |
| Flood corridor buffer | flood_corridor_500m.gpkg | 1 | (dissolved polygon) |
| All facilities risk-tagged | health_facilities_risk_tagged.gpkg | 1,905 | name, adm1_name, flood_risk |
| At-risk facilities only | health_facilities_at_risk.gpkg | 770 | name, adm1_name, flood_risk |

---

## Key Findings

| Category | Count | Share of total |
|---|---|---|
| At Risk (within 500 m of a waterway) | 770 | 40.4% |
| Safe (beyond 500 m) | 1,135 | 59.6% |
| Total health facilities | 1,905 | 100% |

Over 40% of all health facilities in Ghana fall within the 500-metre flood corridor. This represents a significant systemic vulnerability that requires targeted resilience planning, particularly in regions with dense waterway networks such as Volta and Western North.

---

## Symbology

`health_facilities_risk_tagged.gpkg` is styled using a **categorised renderer** on `flood_risk`: orange circles for At Risk facilities and green circles for Safe facilities. The flood corridor buffer is displayed as a semi-transparent blue fill to provide visual context for the proximity threshold.

---

## Project File

`Health-Facility-Waterway-Risk.qgz` — saved with relative paths. All layers load without broken links on any machine that preserves the folder structure.
