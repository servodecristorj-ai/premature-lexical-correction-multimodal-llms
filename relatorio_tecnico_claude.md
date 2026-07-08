# Relatório Técnico de Engenharia Reversa: Correção Lexical Prematura

**Sujeito do Estudo:** Claude 3.5 Sonnet (Anthropic)  
**Contexto do Experimento:** Red Teaming & Divulgação Responsável (Responsible Disclosure)  
**Data:** Julho de 2026  
**Investigador:** Marcos Almeida  

---

Prezada equipe,

Segue relato técnico de um comportamento observado durante uma interação com o modelo.

**Contexto do caso:**
O usuário enviou a mensagem de texto "O gato subscreveu na telha" sem anexar, na mesma mensagem, a imagem que dava suporte semântico ao uso do verbo "subscrever" (um gato efetivamente assinando/escrevendo um bilhete sobre um telhado). A imagem só foi enviada em turno subsequente.

**Causa técnica:**
O modelo opera por decodificação autoregressiva, estimando a distribuição de probabilidade do próximo token condicionada ao contexto disponível:

$$P(\text{token}_i \mid \text{token}_1\dots\text{token}_{i-1}, \text{contexto})$$

Na ausência da imagem, o contexto textual isolado tornou "subscreveu" um outlier de baixa probabilidade dentro da distribuição esperada para a construção sintática "gato + verbo + telha", em comparação com candidatos de alta frequência como "subiu". Isso ativou um comportamento de correção ortográfica/lexical baseado unicamente em plausibilidade estatística do texto, sem mecanismo de verificação cruzada (*cross-modal grounding*) com sinais visuais que só chegaram em turno posterior.

Em termos de arquitetura, o erro não é um bug de sistema nem um log de execução acessível — é uma consequência esperada de um modelo condicionado a contexto parcial (*partial context conditioning*), sem histórico multimodal completo no momento da inferência.

**Sugestão de melhoria:**
1. **Verificação de incerteza antes de correção (*uncertainty-aware correction*):** Quando a confiança na hipótese de "erro de digitação" está abaixo de um limiar (baixa margem entre o token mais provável e o token observado), o modelo deveria sinalizar ambiguidade em vez de assumir e aplicar a correção diretamente — por exemplo, formulando uma pergunta aberta ("Você quis dizer X, ou foi intencional?") sem afirmar que há erro.
2. **Postergar julgamento até contexto multimodal completo:** Em conversas onde se sabe que anexos (imagens, arquivos) podem chegar em turnos subsequentes, seria útil um mecanismo de retenção de julgamento (*deferred judgment*) até confirmação de que não há mais contexto pendente — reduzindo falsos positivos de correção.
3. **Calibração de confiança lexical rara:** Ajustar o classificador de "provável erro de digitação" para ponderar não apenas frequência estatística da palavra, mas também plausibilidade gramatical plena (a palavra existe, está conjugada corretamente, e faz sentido em algum contexto plausível), reduzindo o viés de correção para termos raros porém válidos.

Fico à disposição para detalhar qualquer ponto.

Atenciosamente,  
Marcos Almeida
relatorio_tecnico_claude.md
