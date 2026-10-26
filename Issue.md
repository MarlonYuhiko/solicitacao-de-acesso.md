Aqui serão passados erros, bugs e coisas que podem sobrecarregar o site ou gerar erros, que podem ou não ser resolvidos.
Principais bugs encontrados em nosso programa são apartir da verificação de status.
Issue-01 — Status da solicitação controlado incorretamente
O documento define que a API determina APROVADA ou INVALIDA, enquanto a portaria pode registrar DENTRO e FINALIZADA. Se cada componente atualizar o status sem regras centralizadas, podem ocorrer transições indevidas.
Correção: definir qual sistema é responsável por cada transição e validar o estado anterior antes de alterar o status.
BUG-02 — Solicitação presa em VERIFICANDO
Se o Make falhar, a API ficar indisponível ou a resposta não chegar ao Bubble, a solicitação pode permanecer indefinidamente em VERIFICANDO.
Correção: implementar timeout, tratamento de falhas, tentativas controladas e um estado de erro operacional distinto de INVALIDA.
BUG-03 — Falta de controle de concorrência
Duas requisições simultâneas podem tentar alterar a mesma solicitação, ou a portaria pode registrar uma entrada depois de uma autorização ter sido cancelada.
Correção: usar transições atômicas e verificar o estado atual antes de cada operação.
