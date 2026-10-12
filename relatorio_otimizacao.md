# Otimização da API — implementação e limites

## Estado real da entrega

O repositório usa React/Vite, não templates Bootstrap. A integração foi feita em
`frontend/src/api/client.ts`, preservando componentes, React Query e contratos.
Os caminhos solicitados `templates/*.html` e `static/js/*` foram substituídos pelos
equivalentes reais em `frontend/public/static/js/` e `frontend/index.html`.

**A migração completa de toda a aplicação para servidor exclusivamente de escrita
não está concluída.** As leituras de prioridade foram integradas à camada estática;
resultados eleitorais, filtros geográficos, detalhe histórico e Como agir ainda
dependem da API. O modo padrão é `transition`; desligá-los agora quebraria a UI.
O modo `minimal` existe, foi testado sem snapshot, mas só deve ser usado quando esses
consumidores adicionais forem migrados. Não foi feito deploy nem upload.

## Correção emergencial

Priority e Snapshot: limite DuckDB 2GB, spill 10GB, temporários nativos do sistema
operacional (não `/tmp` fixo no Windows). Snapshot limita concorrência a uma consulta.
Os allowlists, proibição de extensões e restrição de acesso externo foram preservados.
O health não executa SQL nem verifica hashes a cada chamada.

O ajuste reduz falhas de ordenação, mas **2GB não é garantia contra OOM do sistema**.
O host precisa de RAM e disco suficientes; não coloque esse modo numa VM de 256MB.
Não há isolamento subprocesso para consultas de fallback. A eliminação do risco
DuckDB no servidor só é efetiva no modo mínimo, sem inicialização analítica.

## Batch e pacote

```powershell
.venv/Scripts/python.exe scripts/gerar_parquets_derivados.py
.venv/Scripts/python.exe scripts/validar_derivados.py
.venv/Scripts/python.exe scripts/testar_otimizacao.py
node scripts/testar_wasm.mjs
```

Linux: `PYTHON=.venv/bin/python bash scripts/atualizar_derivados.sh`.
Windows: execute os dois scripts Python. O padrão exige 510.060 linhas; para um
snapshot diferente, passe `--expected-rows` aos dois scripts após conferir a fonte.

Geração ZSTD com row groups 2048; validação antes da substituição do pacote anterior.
Lookup mantém todas as 42 colunas e ordem por chave, particionado por UF; UFs grandes
são divididas novamente por faixa. Rankings e mapas usam faixas de ordenação global:
o navegador percorre arquivos na ordem do manifest, filtra e pagina sem ORDER BY.
Ordens alternativas possuem derivados próprios (troca espaço offline por resiliência).
`priority_top` contém exatamente 200 linhas, mas sozinho não atende filtros UF/faixa;
o cliente usa rankings completos para não esconder seções fora do top nacional.

Contagens municipais são as da fonte (códigos TSE), incluindo exterior; não são
forçadas a 5.570. ZZ fica separado do ranking doméstico de abstenção.

O validador verifica limites estritos <5.000.000 bytes, hashes, linhas, unicidade das
chaves, soma nacional e correspondência/ordem do top 200. Falhas abortam a publicação.
Nenhum DataFrame completo é impresso.

## Navegador

DuckDB WASM 1.28.0 importado como módulo, carregamento preguiçoso, worker isolado.
`registerFileURL(..., HTTP, false)` usa filesystem JavaScript com Range, sem instalar
httpfs nativo. Conexões fecham em finally, consultas são serializadas, timeout de 30s
desabilita WASM e ativa fallback. Valores SQL são escapados; modos/ordens vêm de listas
permitidas. A base R2 não tem valor público fictício como padrão.

Manifest serve versão e partições; base deve incluir prefixo imutável da publicação.
Copie `frontend/public/static/js/config.example.js` para `config.js`, preencha base
e publique o frontend. Sem esse arquivo, a requisição opcional pode registrar 404;
o aviso explica que a API de transição permanece ativa. Nunca colocar token de escrita
ou credenciais S3 nesse arquivo.

Não houve teste end-to-end em navegador com Range/CORS reais. Build, sintaxe e
fallback foram testados localmente; desempenho em celular e compatibilidade efetiva
WASM/Arrow/Parquets precisam de homologação após upload. Ranking nacional pode ler
muitas partições: limite por arquivo não equivale a limite de tráfego total.

## API mínima e escritas

```powershell
$env:ELEITORAL_API_MODE = 'minimal'
$env:ELEITORAL_WRITE_TOKEN = '<segredo forte somente no servidor>'
$env:ELEITORAL_WRITES_DB = '<volume persistente>/writes.sqlite3'
.venv/Scripts/python.exe -m uvicorn eleitoral.api.app:app --port 8000
```

Rotas mínimas: GET /health (+ alias /api/v1/health), GET /api/v1/manifest,
POST /api/v1/adotar-secao, POST /api/v1/reportar-campo.
POSTs exigem Authorization Bearer, validam UF/município/zona/seção e relato <=4000
caracteres; retornam 201 após commit SQLite. Sem token: 503. Token inválido: 401.
O recurso de adoção existente no frontend continua local; os novos POSTs não são
ligados à UI automaticamente, para não expor segredo compartilhado no navegador.
Para usuários públicos implementar autenticação individual e moderação é pendência.
Não gravar informação pessoal sensível em relatos. Volume persistente e backup são
obrigatórios em produção; SQLite local não é storage de Cloudflare Workers.

## Hospedagem: não declarar custo zero garantido

Frontend estático + R2 remove computação analítica do servidor. Workers Free tem
100.000 requests/dia e limites de CPU; portar esta API FastAPI/SQLite para Workers
requer adaptação do runtime e armazenamento (por exemplo D1). Não foi implementado
Worker nem D1, pois não existem no projeto e não se inventou deploy funcional.
Fly.io oferece trial/cobrança por uso, não uma hospedagem permanentemente gratuita
garantida. R2 também tem limites de armazenamento e operações; monitore consumo.

Referências oficiais consultadas: documentação DuckDB WASM (Import Data/Overview),
Cloudflare Workers Pricing, R2 Public buckets e Fly.io Free Trial/Resource Pricing.

## Pendências obrigatórias

### Publicação inicial reduzida

`scripts/preparar_upload.py --perfil essencial` prepara top, lookup e geo com
manifest reduzido em `data/upload-essencial`. `--perfil inicial` acrescenta o
ranking padrão e agregações em `data/upload-inicial` (opção recomendada).
Os pacotes são locais, não executam upload; cada manifest referencia somente seus
arquivos e possui versão com sufixo do perfil. Datasets deliberadamente ausentes
usam fallback seletivo sem desativar WASM para as outras consultas.

As 30 partições alternativas (taxa, score, abstenção e mapa por taxa) podem ficar
para uma segunda publicação em novo prefixo. Embora os dados básicos permitam
recalcular essas ordens, não foi implementada ordenação on-the-fly: reintroduziria
o risco de memória/latência e exigiria homologação de dispositivos reais.
No perfil essencial, Home/Ação Imediata continuam consultando ranking filtrável na
API: top nacional não é equivalente ao ranking completo por UF/faixa.

As medidas por perfil estão na auditoria; comandos e detalhes no README de upload.

1. Upload manual e CORS conforme `data/derived/README_UPLOAD.md`.
2. Configurar base pública versionada; domínio customizado para produção.
3. Homologar Range 206, cache, filtros e equivalência no navegador.
4. Migrar demais leituras antes de habilitar modo mínimo na aplicação real.
5. Definir runtime/armazenamento persistente e autenticação das escritas.

Resultados reais, e não números do exemplo do enunciado, estão em
`auditoria_otimizacao.json`.