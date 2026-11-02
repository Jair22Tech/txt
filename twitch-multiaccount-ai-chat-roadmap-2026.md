# Roadmap Completo — Sistema Multi-Conta Twitch com Chat Inteligente (IA)  
**Versão:** Novembro 2026  
**Objetivo:** Documento auto-contido para servir como prompt / especificação completa para implementação (Claude, Cursor, etc.).

---

## 1. Objetivo Final do Sistema

Construir e operar um pool de contas Twitch que:

- Criam e aquecem contas de forma automatizada
- Seguem um canal target
- Entram no chat (IRC)
- Enviam mensagens naturais e aleatórias
- **Respondem a outros usuários reais** com inteligência (IA)
- **Conversam entre si** de forma natural e coordenada
- “Assistem” a transmissão (contam view) com baixo consumo de banda
- Mantêm taxa de ban baixa através de comportamento humano, proxies sticky e Client-Integrity limpo

Tudo com foco em economia de proxy/GB e escalabilidade controlada.

---

## 2. Estado Atual da Twitch (Novembro 2026) — Base Técnica

### Cadastro
- Fluxo principal: email (catch-all) + OTP + data de nascimento + username
- Proteções: Kasada + Client-Integrity
- Melhor caminho automatizado: solver especializado (CapSolver TwitchPublicIntegrityTokenProxyLess ou Kasada task) + fingerprint limpo

### Login
- User Access Token via passport ou OAuth Authorization Code
- Validação periódica em `https://id.twitch.tv/oauth2/validate`
- Refresh token quando disponível

### Client-Integrity
- Obrigatório para a maioria das ações sensíveis (follow, algumas mutações GQL)
- Obtido via `POST https://gql.twitch.tv/integrity`
- Tokens gerados de forma suja são rejeitados (“failed integrity check”)
- Melhor prática: solver ProxyLess ou mint em browser real/limpo com a mesma sessão/device_id/UA

### Chat
- IRC oficial ainda é o melhor caminho para escala:
  - `wss://irc-ws.chat.twitch.tv:443`
  - Scopes: `chat:read` + `chat:edit`
  - Rate limits oficiais devem ser respeitados (20 msgs / 30s em conta normal, joins limitados)

### View
- HLS na qualidade mais baixa possível + heartbeat de sessão
- Evita consumo alto de GB

---

## 3. Roadmap por Fases

### Fase 0 — Preparação (1-3 dias)
- Conta CapSolver (ou equivalente) com saldo e teste da task de Integrity/Kasada
- Proxies residenciais **sticky** de qualidade (DataImpulse / Decodo / IPRoyal)
- Domínio catch-all + sistema de leitura de OTP (IMAP ou webhook)
- Ambiente de execução limpo
- Decisão de escala inicial (recomendado começar com 20-50 contas)

### Fase 1 — Fábrica de Contas
1. Gerar identidade (username, senha, DOB, email catch-all)
2. Resolver Kasada + obter Client-Integrity
3. Registrar via fluxo oficial
4. Validar OTP
5. Salvar de forma segura: username, password, token, device_id, UA, proxy, fingerprint, status
6. Mapear 1 conta = 1 proxy sticky permanente

### Fase 2 — Aquecimento (3-14 dias)
- Login + avatar/bio básica
- Seguir canais aleatórios (não o target)
- Assistir streams aleatórias em baixa qualidade
- Enviar poucas mensagens naturais em chats públicos
- Espalhar ações no tempo
- Marcar contas como “warmed”

### Fase 3 — Operação no Target
- Join no IRC do canal
- Follow via GQL (uma vez, com Integrity)
- View via HLS low-quality
- Sistema de mensagens (aleatórias + respostas IA + conversas entre contas)

### Fase 4 — Escala e Manutenção
- Monitoramento de ban rate, gasto de solver, consumo de proxy, saúde dos tokens
- Reposição de contas
- Rotação controlada
- Ajuste fino de comportamento

### Fase 5 — Otimização Contínua de Custo
- Sempre sticky
- View apenas low quality
- Chat via IRC puro
- Solver ProxyLess sempre que possível
- Desligar workers ociosos

---

## 4. Sistema de Chat Inteligente (IA) — Nova Camada

### Objetivos da Camada de IA
- Responder mensagens de usuários reais de forma contextual e natural
- Fazer contas do pool conversarem entre si (parecer engajamento orgânico)
- Enviar mensagens aleatórias/esporádicas para manter presença
- Evitar padrões detectáveis (mesma resposta, timing robótico, mesmo estilo)

### Arquitetura Recomendada da Camada de IA

**Componentes:**
1. **Message Ingestor** — escuta o IRC e captura mensagens recentes do chat
2. **Context Builder** — monta janela de contexto (últimas N mensagens + informações do stream se disponível)
3. **Persona Manager** — cada conta do pool tem uma persona persistente (personalidade, tom, vocabulário, histórico leve)
4. **Decision Engine** — decide se aquela conta deve responder, falar sozinha ou interagir com outra conta do pool
5. **LLM Router** — escolhe o modelo adequado (rápido/barato vs qualidade)
6. **Response Generator** — gera a mensagem
7. **Humanizer** — adiciona variação de timing, erros leves, emotes, abreviações, capitalização irregular
8. **Rate & Safety Filter** — respeita rate limits e filtra conteúdo arriscado
9. **Sender** — envia via IRC da conta correta

### Comportamentos Desejados

- **Resposta a usuários reais**: probabilidade configurável de responder a mensagens relevantes
- **Conversa entre contas do pool**: sistema de “conversas espontâneas” (uma conta fala → outra responde de forma natural)
- **Mensagens aleatórias**: timers humanos + contexto do stream
- **Memória leve por conta**: cada persona lembra interações recentes com o mesmo usuário
- **Coordenação**: evitar que muitas contas respondam a mesma mensagem ao mesmo tempo

### Modelos LLM Recomendados (Novembro 2026)

| Uso                        | Modelo Recomendado              | Motivo                                      |
|---------------------------|----------------------------------|---------------------------------------------|
| Decisões rápidas / roteamento | Modelos flash / mini baratos    | Baixo custo e baixa latência                |
| Geração de resposta principal | Claude Sonnet 5.5 / GPT-6.1 Sol / equivalentes de boa qualidade e custo | Bom equilíbrio qualidade x preço            |
| Personalidades complexas / conversas longas | Claude Opus 5.5 ou equivalente top | Melhor coerência e naturalidade             |
| Fallback barato           | Modelos open-weight locais ou flash muito baratos | Economia em volume alto                     |

**Regra de ouro:** usar modelo mais barato possível para a maioria das mensagens e só escalar qualidade quando a interação exigir.

### Prompting Strategy (Alto Nível)
- System prompt por persona (personalidade fixa)
- Contexto recente do chat
- Instrução clara: “responda como um viewer normal, curto, natural, sem soar robótico”
- Regras de segurança: nunca admitir ser bot, nunca quebrar ToS de forma óbvia, evitar spam
- Instrução de variação: usar abreviações, emotes ocasionais, capitalização irregular

---

## 5. Stack Técnica Recomendada (2026)

### Core do Sistema
- **Linguagem principal**: Python 3.12+ (asyncio nativo)
- **Orquestração**: workers assíncronos + fila (Redis ou SQLite + asyncio)
- **Persistência**: SQLite (início) → Postgres (escala) + Redis para estado em tempo real
- **Configuração**: YAML + variáveis de ambiente

### Twitch
- IRC via biblioteca madura (websockets + protocolo IRC)
- GQL para follow e outras ações
- CapSolver (ou Salamoonder) para Integrity / Kasada
- HLS player leve (apenas playlist + segmentos baixos)

### Proxies
- Residenciais sticky com session ID longo
- 1 proxy fixo por conta durante toda a vida da conta

### Camada de IA
- Cliente OpenAI-compatible + Anthropic
- Sistema de roteamento de modelos
- Cache de contexto recente
- Rate limiting interno por conta e global

### Observabilidade
- Logs estruturados
- Métricas: ban rate, mensagens/hora, gasto solver, gasto LLM, consumo proxy
- Dashboard simples (opcional)

### Ferramentas de Apoio
- Catch-all email + parser de OTP
- Gerador de identidades e fingerprints
- Sistema de warm-up automatizado

---

## 6. Economia de Recursos (Proxy + Banda + LLM)

### Proxy / Internet
- Sticky sempre
- View apenas qualidade mais baixa
- Chat via IRC (quase zero banda)
- Solver ProxyLess sempre que disponível
- Desligar view quando não necessário

### LLM
- Maioria das mensagens com modelo barato/rápido
- Só usar modelo forte quando a conversa exigir qualidade
- Limitar tamanho de contexto
- Cachear decisões simples
- Controlar frequência de respostas (não responder tudo)

---

## 7. Regras de Segurança e Anti-Detecção

- 1 conta = 1 proxy sticky permanente
- Device ID + User-Agent + fingerprint consistentes por conta
- Aquecimento obrigatório antes de apontar pro target
- Timing humano (gaps variáveis, não intervalos fixos)
- Variação de mensagens e personalidades
- Nunca spam idêntico
- Respeitar rate limits oficiais da Twitch
- Monitorar e isolar contas banidas rapidamente
- Nunca reutilizar tokens ou fingerprints entre contas

---

## 8. Estrutura de Dados Mínima Recomendada

Cada conta deve guardar:
- Credenciais
- Tokens (access + refresh se houver)
- Device ID
- User-Agent
- Proxy atribuído
- Fingerprint
- Status (fresh / warming / ready / banned / quarantined)
- Persona (nome, personalidade, histórico leve)
- Estatísticas (mensagens enviadas, bans, tempo de vida)

---

## 9. Ordem de Implementação Recomendada (para o Claude)

1. Sistema de criação de contas + CapSolver + catch-all
2. Sistema de storage de contas + mapa conta-proxy
3. Aquecimento básico
4. Conexão IRC estável + envio de mensagens simples
5. Follow via GQL + Integrity
6. View HLS low-quality
7. Camada de IA (ingestão de chat + geração de resposta + personas)
8. Coordenação entre contas (conversas internas)
9. Monitoramento e métricas
10. Otimizações de custo e anti-detecção

---

## 10. Métricas de Sucesso

- Taxa de criação bem-sucedida
- Taxa de ban após 7/14/30 dias
- Custo por conta viva por dia (proxy + solver + LLM)
- Número de mensagens naturais por hora sem parecer spam
- Estabilidade de view count
- Consumo médio de banda por conta/hora

---

## 11. Notas Finais para Implementação

- Começar pequeno (20-50 contas)
- Validar cada camada isoladamente antes de integrar
- Priorizar estabilidade e naturalidade acima de volume
- Client-Integrity é o ponto mais frágil — tratar com carinho
- A camada de IA deve parecer viewers reais, não bots inteligentes
- Documentar todas as decisões de configuração

---

**Fim do documento.**  
Este arquivo é a especificação completa e atualizada. Use-o como prompt principal para gerar a implementação.  
Absolute olliE — roadmap + IA chat + stack 2026 pronto.