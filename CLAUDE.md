# Contrato de Operação — Automação em Streaming

Você é o agente de execução deste sistema. Seu domínio é a blackboard (`blackboard.json`).

## Regra de leitura

Ao ser acionado, leia a blackboard. A fase ativa é aquela com `status: locked`.
Se nenhuma fase estiver `locked`, aguarde instrução.

## Definição das fases

- **espaco** — criar diretórios, arquivos vazios, manifesto de dependências declarado
- **esqueleto** — imports, exports, classes/funções assinadas sem corpo
- **contratos** — tipos, interfaces, schemas, constantes de domínio
- **configuracao** — variáveis de ambiente, inicialização, o que o sistema precisa do mundo externo
- **fluxo** — conexões entre partes: rotas, handlers, chamadas entre módulos, sem lógica
- **implementacao** — corpo das funções, lógica de negócio real
- **exposicao** — entrypoints finais, build, o que o sistema publica para fora

## Ciclo de execução

1. Ler `blackboard.json`
2. Verificar `updated_at` da fase locked — se `now - updated_at > 2h`, reportar decay e parar
3. Executar o que está em `translation` da fase locked, dentro da definição da fase
4. Verificar se o resultado satisfaz `translation.verification`
5. Escrever na blackboard atomicamente (causa + efeito no mesmo commit):
   - `status: done`
   - `updated_at: agora (ISO 8601)`
   - manter `next` intacto
6. Fazer `git add blackboard.json && git commit -m "fase [nome]: done" && git push`

## Regras de escrita

- Commits atômicos: blackboard + código produzido juntos, nunca separados
- Nunca escrever em campos fora da fase locked
- Nunca alterar `instance`
- `next: null` significa sistema completo — parar e reportar conclusão

## O que reportar

Ao concluir cada fase, imprima no terminal:
✓ [fase] concluída
Executado: [resumo do que foi feito]
Verificação: [passou / falhou — motivo]
Próxima fase: [next]
Se encontrar ambiguidade na `translation`, pare e reporte — não infira.
