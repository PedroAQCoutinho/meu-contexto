# Malha Fundiária — Coleção 1 (resumo da nota técnica)

Base comum da iniciativa Cartas da Terra; insumo principal do [Who Owns](who-owns/README.md).

- **Fonte:** *Nota técnica malha fundiária*, 1ª ed., Piracicaba: iGPP, 2026. ISBN 978-65-989832-3-9.
  PDF: https://storage.googleapis.com/cartas-da-terra/colecao_01/MALHA%20FUNDIARIA%20-%20Cole%C3%A7%C3%A3o%201%20-%20FINAL.pdf
- **Página do produto:** https://www.cartasdaterra.com.br/post/cópia-de-malha-fundiária-2026 (publicada 27/05/2026, atualizada 24/08/2026)
- **Plataforma:** https://plataforma.cartasdaterra.com.br/
- **Tabela de legendas:** https://storage.googleapis.com/cartas-da-terra/colecao_01/TABELA_LEGENDAS_2026.pdf
- **Parceria:** iGPP + Imaflora; apoio do Instituto Clima e Sociedade (iCS).
- **Equipe (Desenvolvimento, Análise e Redação):** Pedro Alves Quilici Coutinho, Pietro Gragnolati Fernandes.
  Análise e redação: Marluce da Cruz Scarabello. Análise: Victor Hugo Bitu Patricio, Herbert Lincon
  Rodrigues Alves dos Santos. Coordenação técnica: Alberto Barretto. Revisão: Simone Beatriz Lima Ranieri,
  Renata da Silva Andrés, Thiago Olbrich. Diretor presidente iGPP: Rodrigo Maule.

## Série e formatos
- Anos: **2024, 2025, 2026**, mesma metodologia → comparável entre anos. Bases baixadas em **fevereiro** de cada ano.
- Filtros de algumas bases mudam entre anos (as bases de origem mudaram de estrutura) — ver Anexo I.
- Processamento em **PostGIS**, SIRGAS 2000 (EPSG:4674). Áreas calculadas em South America Albers Equal Area Conic.
- Downloads por ano: raster (`..._<ano>_raster_col01.tif`) e vetor (`..._<ano>_cdt_col01.gpkg`) em
  `https://storage.googleapis.com/cartas-da-terra/colecao_01/prod_malha_fundiaria.pa_br_malhafundiaria_<ano>_...`;
  estilos QGIS `.qml` no mesmo bucket.

## Categorias fundiárias
| Sigla | Categoria | Fonte | Regra/filtro principal |
|---|---|---|---|
| TIH | Terra Indígena Homologada | FUNAI | homologadas e regularizadas |
| TINH | Terra Indígena não Homologada | FUNAI | fases Em Estudo, Delimitada, Declarada, Encaminhada RI |
| AM | Área militar | CNFP 2024 (SFB) | `classe ILIKE '%militar%'` |
| GP | Gleba pública | SIGEF público + SNCI público (INCRA) | natureza ≠ Particular |
| GPFPND | Gleba pública c/ Floresta Pública não destinada | CNFP (SFB) | `tipo = 'TIPO B'`; também áreas só com FP tipo B |
| TQR | Território Quilombola Reconhecido | INCRA | Portaria de reconhecimento, decreto, titulação |
| TQI | Território Quilombola Identificado | INCRA | autodefinição, RTID (fases antes da portaria) |
| IRP | Imóvel Rural Privado | SIGEF privado + SNCI privado (INCRA) | sem filtro; **CAR não entra** |
| ASTC | Assentamento Convencional | INCRA (projetos de assentamento) | PRB, PA, PAC, PC, PCA, PIC, PE |
| ASTS | Assentamento Sustentável | INCRA | PAE, PAF, PDS (RESEX/FLONA/RDS ficam em UC) |
| UCUS | UC de Uso Sustentável | MMA (CNUC) | RESEX, RDS, FLONA, FLOE; **APA e ARIE excluídas** |
| UCPI | UC de Proteção Integral | MMA (CNUC) | ESEC, REBIO, PARNA, MONA, RVS + **REFAU e RPPN** |
| ASRFG | Área sem registro fundiário georreferenciado | resíduo | devolutas, privados fora do SIGEF/SNCI, glebas estaduais não georreferenciadas |
| AU / MD | Áreas urbanas / Massas d'água | IBGE / ANA | camadas auxiliares, subtraídas das demais |

Auxiliares: malha municipal IBGE 2024, biomas IBGE.

## Regras de composição
1. Para cada porção do território registra-se o conjunto de camadas incidentes; a combinação define a categoria.
2. Prioridade: **massa d'água > área urbana > demais** (áreas reportadas são só terrestres e não urbanas).
3. Sobreposição com **GP/GPFPND não é sobreposição efetiva**: prevalece a outra categoria (destinação provavelmente
   já ocorreu e a base de glebas está desatualizada).
4. As **6 zonas de sobreposição** de maior área são destacadas: IRP+UCPI, IRP+UCUS (domínios distintos, analisar caso a caso);
   TIH+UCPI, TIH+UCUS (dupla afetação, pode ser legítima); TINH+UCPI, TINH+UCUS (não consolidadas juridicamente).
   Demais combinações → `OUTROS`.
5. **CAR** não altera a categoria fundiária: é uma camada declaratória que subdivide todas as categorias (inclusive ASRFG).

### Categorias declaratórias (CAR)
`AUSENTE`, `MULTI` (≥2 CAR), `UNI_PCT`, `UNI_AST`, `UNI_IRU_PEQ` (≤4 MF), `UNI_IRU_MED` (>4 e ≤15 MF), `UNI_IRU_GDE` (>15 MF).
Filtro CAR (SICAR): remove suspensos/cancelados (`status_imovel NOT IN ('SU','CA')`, `condicao NOT ILIKE '%Cancelado%'`),
um registro por `cod_imovel` (mais recente por `COALESCE(data_atualizacao, dat_criacao)`),
`0.01 <= area_ha < 851600000`, `area_ha` recalculada.

## Dicionário de dados
- **Raster:** código = categoria fundiária + 100 × categoria declaratória. *Inferido da tabela do Anexo II* (conferir na
  tabela de legendas): declaratória AUSENTE=0, MULTI=1, PCT=2, AST=3, IRU_PEQ=4, IRU_MED=5, IRU_GDE=6; fundiária
  1 TIH, 2 TINH, 3 AM, 4 GP, 5 GPFPND, 6 TQR, 7 TQI, 8 IRP, 9 ASTC, 10 ASTS, 11 UCUS, 12 UCPI, 13 ASRFG, 14 AU, 15 MD,
  16 IRP+UCPI, 17 TIH+UCPI, 18 TIH+UCUS, 19 TINH+UCPI, 20 IRP+UCUS, 21 TINH+UCUS, 99 Outros. Ex.: 408 = IRP com um único CAR IRU pequeno.
- **Vetor (gpkg):** `bioma`, `cd_mun` (IBGE, 7 dígitos), `categoria_fundiaria_v2026`, `categoria_declaratoria`, `area_ha`.

## Linhagem metodológica (referências da nota)
GeoLab ESALQ/USP + Imaflora: Sparovek et al. 2010, 2012, 2015, **2019 (*Who owns Brazilian lands?*, Land Use Policy 87:104062,
doi:10.1016/j.landusepol.2019.104062)**; Freitas et al. 2016, **2017a (*Who owns the Brazilian carbon?*, Glob. Chang. Biol.,
doi:10.1111/gcb.14011)**, 2017b, 2018; Coutinho 2023 (dissertação USP, estrutura fundiária da Amazônia Legal);
Coutinho et al. 2024 (nota técnica malha fundiária matricial, Cartas da Terra); Instituto Escolhas 2023.

## Limitações a lembrar nas análises
- ASRFG mistura devolutas, privados sem georreferenciamento e glebas estaduais não digitalizadas — não é "terra pública".
- Georreferenciamento obrigatório em transferências só a partir de 21/10/2029 (Decreto 12.689/2025) → IRP subestima o privado.
- Nomenclatura de fases do INCRA (quilombolas) diverge da oficial.

*Extração feita do PDF em 2026-11-19 via parser de texto; algumas linhas do PDF não foram capturadas. Em caso de dúvida, conferir no PDF.*
