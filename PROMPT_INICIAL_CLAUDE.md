# PROMPT DE INICIALIZAÇÃO — Sistema Multi-Conta Twitch + Chat IA

Cole este prompt inteiro sempre que for iniciar uma nova sessão com o Claude (ou Cursor/Claude Code).

---

## CONTEXTO DO PROJETO

Você está implementando um sistema completo de multi-conta Twitch com as seguintes capacidades:

- Criação automatizada de contas (usando CapSolver para Kasada + Client-Integrity)
- Aquecimento de contas
- Follow em canal target
- Chat via IRC oficial
- Contagem de views via HLS low-quality
- **Camada de IA**: contas respondem usuários reais, conversam entre si e enviam mensagens aleatórias de forma natural
- Foco forte em economia de proxy/GB e baixa taxa de ban

O roadmap completo e a especificação técnica estão no arquivo:

**`twitch-multiaccount-ai-chat-roadmap-2026.md`**

Leia esse arquivo completamente antes de qualquer ação.

---

## REGRAS OBRIGATÓRIAS DE TRABALHO

1. **Sempre leia o arquivo de progresso** (`PROGRESS.md`) no início de cada sessão.
2. **Sempre atualize o `PROGRESS.md`** no final de cada sessão ou após completar uma etapa significativa.
3. Trabalhe de forma incremental. Nunca tente implementar tudo de uma vez.
4. Siga a ordem de implementação recomendada no roadmap (seção 9).
5. Priorize estabilidade e naturalidade acima de volume.
6. Não invente endpoints ou comportamentos da Twitch. Use apenas o que está documentado no roadmap ou em fontes oficiais.
7. Quando criar arquivos novos, explique o propósito de cada um.
8. Mantenha a arquitetura limpa e modular.
9. Sempre que possível, faça commits pequenos e descritivos.

---

## COMO SABER ONDE PARAMOS

O arquivo `PROGRESS.md` é a fonte única da verdade sobre o estado atual do projeto.

Estrutura esperada do `PROGRESS.md`:

```markdown
# Progresso do Projeto — Multi-Conta Twitch + IA Chat

**Última atualização:** [data e hora]
**Sessão atual:** [número ou descrição curta]
**Status geral:** [ex: Fase 1 em andamento / Pronto para testes de IRC / etc.]

## O que já foi concluído
- [lista clara do que está funcionando]

## O que está em andamento agora
- [o que está sendo feito nesta sessão]

## Próximos passos imediatos
1. ...
2. ...
3. ...

## Problemas / Bloqueios atuais
- [se houver]

## Decisões técnicas importantes tomadas
- [ex: escolhemos X biblioteca por Y motivo]

## Arquivos principais criados
- ...
```

No início de **toda** sessão você deve:
1. Ler `twitch-multiaccount-ai-chat-roadmap-2026.md`
2. Ler `PROGRESS.md`
3. Confirmar comigo qual é o próximo passo lógico
4. Só então começar a implementar

No final de **toda** sessão (ou quando eu pedir para parar):
- Atualize o `PROGRESS.md` com o que foi feito
- Deixe os próximos passos claros
- Não deixe código pela metade sem explicar o estado

---

## ORDEM DE IMPLEMENTAÇÃO (lembrete)

Siga esta ordem sempre que possível:

1. Sistema de criação de contas + CapSolver + catch-all
2. Storage de contas + mapa conta ↔ proxy
3. Aquecimento básico
4. Conexão IRC estável + envio de mensagens simples
5. Follow via GQL + Client-Integrity
6. View HLS low-quality
7. Camada de IA (ingestão + personas + geração de resposta)
8. Coordenação entre contas (conversas internas)
9. Monitoramento e métricas
10. Otimizações finais

---

## INSTRUÇÃO INICIAL DESTA SESSÃO

Leia os dois arquivos principais (`twitch-multiaccount-ai-chat-roadmap-2026.md` e `PROGRESS.md`).

Depois me diga:

1. Qual é o status atual do projeto segundo o PROGRESS.md
2. Qual é o próximo passo lógico e mais seguro agora
3. O que você pretende fazer nesta sessão

Só comece a escrever código depois que eu confirmar.

---

**Fim do prompt de inicialização.**  
Use este prompt toda vez que abrir uma nova sessão.