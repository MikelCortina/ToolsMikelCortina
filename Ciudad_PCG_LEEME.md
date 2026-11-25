# Ciudad PCG (Guía PCG para generar una ciudad)

Todo está en `Content/City`. Abre `Maps/L_City` (es el mapa de inicio): la ciudad ya está generada con la seed 1234.

## Cómo se usa
- **Herramienta:** menú **Tools > Ciudad PCG** (o clic derecho en `Blueprints/EUW_CityTool` > Run Editor Utility Widget).
  Ajusta los parámetros y pulsa **Generar**, **Limpiar** o **Probar determinismo** (seed 7 dos veces = igual, seed 8 = distinta).
- **Sin la herramienta:** selecciona el actor `City` (BP_City) y usa los botones **Generate** / **Cleanup** de su panel Details.
- **Play:** empiezas en una vista aérea (PlayerStart_Overview) y puedes volar por la ciudad.

## Parámetros (Graph Parameters de PCG_City + Seed del PCG Component)
| Parámetro | Por defecto | Efecto |
|---|---|---|
| CitySize | 400 m | lado de la ciudad → nº de cruces y manzanas |
| BlockSize | 60 m | lado de la manzana → nº de manzanas, parcelas y farolas |
| BuildingDensity | 0.6 | 0 = sin edificios, 1 = todas las parcelas con fachada |
| BuildingHeight | 30 m | altura media |
| BuildingVariation | 0.5 | 0 = edificios iguales, 1 = alturas y huellas muy distintas |
| Seed | 1234 | misma seed, misma ciudad |

Constantes internas (en `SG_Params`): calle 8 m, acera 3 m, parcela 20 m, mobiliario cada ~12 m.

## Los 11 pasos → assets
| Paso | Asset | Nodos principales |
|---|---|---|
| 01 Params | `PCG/SG_Params` | Get Graph Parameter, Add Attribute, Attribute Maths Op, Make Vector |
| 02 Subgraphs | `PCG/PCG_City` | solo cablea un subgrafo por capa |
| 03 Grid | `PCG/SG_Grid` | Create Points Grid (con overrides), Col = $Index % Count, Row, Sanity Check |
| 04 Streets | `PCG/SG_Streets` | Attribute Filter (Col/Row < count-1), Maths Op, Transform Points, Bounds Modifier |
| 05 Blocks | `PCG/SG_Blocks` | filtros, BlockID = Row × (count-1) + Col, aceras escaladas |
| 06 Lots | `PCG/SG_Lots` | rejilla de parcelas + Copy Points en cada manzana, Frontage (Compare Op), StreetYaw (Atan2 → múltiplo de 90°) |
| 07 Density | `PCG/SG_Density` | Attribute Noise sobre $Density (seed) + filtro < BuildingDensity |
| 08 Buildings | `PCG/SG_Buildings` | altura y lado con R1/R2 de la seed, Make Vector → $Scale, mirando a la calle |
| 09 Props | `PCG/SG_Props` | anillo de la acera, farola/árbol alternos, Distance a cruces > 12 m |
| 10 Overlaps | `PCG/SG_Overlaps` | Difference binaria contra la calzada y edificios, Self Pruning, Sanity Check |
| — Spawn | `PCG/SG_Spawn` | Static Mesh Spawner (datos primero, mallas al final) |
| 11 Tool | `Blueprints/BP_City`, `EUW_CityTool`, `EUM_CityToolMenu` | Set Double Parameter, Seed, Generate Local, Cleanup Local |

Comprobado sin abrir el editor: con 400 m / 60 m salen 36 cruces + 60 tramos, 25 manzanas, 200 farolas y 200 árboles;
densidad 0 → 0 edificios y 1 → 200; 800 m → 121 manzanas; manzana de 40 m → 64; tras Limpiar no queda ninguna instancia.
