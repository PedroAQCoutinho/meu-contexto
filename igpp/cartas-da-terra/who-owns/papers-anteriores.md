# Who Owns — papers anteriores (linha de base)

Os dois artigos que originaram a ideia. PDFs: Drive (link a definir; não versionar no repo).

## 1. Freitas et al. — *Who owns the Brazilian carbon?*
- **Ref.:** Freitas, F.L.M.; Englund, O.; Sparovek, G.; Berndes, G.; Guidotti, V.; Pinto, L.F.G.; Mörtberg, U.
  Global Change Biology 24:2129–2142 (2018; aceito/online 2017). doi:10.1111/gcb.14011
  (PDF disponível é o *accepted manuscript*; a nota técnica cita como Freitas et al. 2017a.)
- **Instituições:** KTH, Chalmers, USP/ESALQ, Imaflora.
- **Pergunta:** quanto do estoque de carbono acima do solo (AGC) está sob mecanismos públicos de proteção, por posse da terra.
- **Malha:** primeira malha fundiária nacional integrada (base do Atlas Agropecuário/Imaflora), integrando TI, UC,
  áreas militares, assentamentos, imóveis privados (INCRA e CAR), Terra Legal e terras não destinadas → ~670 Mha (79% do país).
  - CAR limpo manualmente: removidos 1.404 polígonos inconsistentes e recortadas sobreposições (~70 Mha a menos).
  - Hierarquia: INCRA > CAR; TI > UC em sobreposição; terra pública > privada em conflito (testado em sensibilidade).
  - Os 21% restantes foram **simulados** como imóveis privados (Censo Agro 2006, por município e faixa de tamanho,
    alocação com Voronoi, 10 repetições). Validação com CAFIR agregado.
  - Terra não destinada "regularizada" por simulação: vegetação nativa > 95% → conservação pública; < 95% → privada.
- **Modelo:** LUGA (Land Use Governance Assessment; Sparovek et al. 2010, 2015), uso/cobertura 2008 a 50 m,
  APP e RL modeladas pelo Código Florestal (Lei 12.651/2012), mapa de AGC 50 m (Englund et al. 2017). ArcGIS + Python + MySQL.
- **Resultados-chave:** ~70% do AGC sob proteção legal, +20% após regularização das terras não destinadas na Amazônia;
  ~30% do AGC em terra privada, ~2/3 protegido. AGC desprotegido: Cerrado ~40%, Amazônia ~30%, Caatinga ~20%.
  Vegetação nativa privada sem obrigação legal: ~101 Mha, ~3,4 Gt (6,5% do AGC). 2% dos proprietários detêm metade
  da terra privada e do AGC privado.

## 2. Sparovek et al. — *Who owns Brazilian lands?*
- **Ref.:** Sparovek, G.; Reydon, B.P.; Pinto, L.F.G.; Faria, V.; Freitas, F.L.M.; Azevedo-Ramos, C.; Gardner, T.;
  Hamamura, C.; Rajão, R.; Cerignoni, F.; Siqueira, G.P.; Carvalho, T.; Alencar, A.; Ribeiro, V.
  Land Use Policy 87:104062 (2019). doi:10.1016/j.landusepol.2019.104062
- **Formato:** artigo curto (3 p.), primeiro mapa fundiário "wall-to-wall" do Brasil.
- **Malha:** evolui a de Freitas et al.; CAR de 18/12/2018 + 17 outras bases oficiais → 14 categorias, com regras de
  hierarquia definidas com especialistas (nota técnica: Freitas et al. 2018b, malha v.170321, Atlas Agropecuário).
  APAs excluídas das UCs. Florestas públicas tipo B com baixa prioridade.
- **Resultados (área total 850,3 Mha):**

| Grupo | % do Brasil |
|---|---|
| Terra pública | 36,1% (TI 13,2%; UC 11,0%; assentamentos 4,9%; não destinadas 6,4%; militares 0,4%; comunitárias 0,2%) |
| Terra privada | 44,2% (CAR 20,4%; SIGEF 22,2%; Terra Legal 1,2%; quilombolas 0,4%) |
| Sem registro | 16,6% |
| Transporte, urbano, água | 3,1% |

  - Sobreposições somam 354,6 Mha = 50% da área registrada (41% do país): 48% entre públicas, 50% público×privado, 2% entre privadas.
  - Imóveis grandes (CAR+SIGEF, ~97 mil, média 1.876 ha) cobrem 21,5% do país; 48% da terra privada em < 100 mil imóveis (2%).
  - Não destinadas: mínimo de 54,6 Mha (vs 65,5 Mha de Azevedo-Ramos & Moutinho 2018, sem hierarquia).

## O que mudou na Coleção 1 (para comparar ao publicar a série)
| Aspecto | Papers (2017–2019) | [Coleção 1](../malha-fundiaria-colecao-1.md) (2024–2026) |
|---|---|---|
| CAR | entra como terra privada (abaixo do INCRA) | **não** é categoria fundiária; vira camada declaratória |
| Áreas sem dado | simuladas (2017) ou "sem registro" (2019) | ASRFG, sem simulação |
| Sobreposições | resolvidas por hierarquia | rastreadas; 6 zonas explícitas + OUTROS |
| Florestas públicas tipo B | baixa prioridade | GPFPND (gleba pública com FP tipo B) |
| Quilombolas | privada (2019) | TQR / TQI, separadas por fase |
| Série | foto única | anual, metodologia única |
| Stack | ArcGIS/MySQL | PostGIS |

Implicação (inferência minha): números da Coleção 1 não são diretamente comparáveis aos de 2019 — sobretudo
"terra privada", que em 2019 incluía o CAR. Ao publicar a série, vale uma tabela de reconciliação 2019 × 2024–2026.
