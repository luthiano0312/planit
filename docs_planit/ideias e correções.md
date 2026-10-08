Inbox
	vamos manter a ideia eu so acho que o termo não é tão adequado, mas isso é discutil, pois o que eu penso sobre essa palavra é que inbox seria algo diferente das outras tarefas, então eu cadastaria uma tarefa e ela iria pra uma area separada de todas as tarefas e quando eu fizer a triagem dela, ela iria se juntar com todas as tarefas, so que eu acho que isso é um fluxo desnecessario, mas se o que voce queis dizer com inbox foi so "cadastro", e que apos o cadastro ela ja se junta a todas as tarefas, então ok era esse fluxo que eu estava pensando
	é melhor o termo inbox ou so cadastro? (eu pensei em manter inbox, pois assim eu poderia reaproveitar esse cadastro tanto pra tarefas quanto pra eventos, existe a possibilidade de expandir isso para outros cadastros tambem, mas por agora o que temos de cadastravel é so tarefas e eventos); ou não é nenhum dos dois que eu falei?
prazo
	ate que faz sentido, mas isso é exatamente o que? um atributo da tarefa? so que se isso for um atributo isso sera mais um campo (poderia ser opcional) pro cadastro, e na pratica isso não tem um impcato tão grande ter ou não ter esse atributo
	vale a pena ou não ter esse atributo?
Evento
	pode ou não ter periodicidade
	pode ou não ter hora marcada (pro caso de não ter é pq é o dia todo. tipo um feriado ou o aniversario de alguem, ou uma comemoração)
	pode ou não ter exceções pra eventos periodicos
Recorrência
	deixa esse conceito mais explicito e amplo tipo, eu naõ entendi isso de regra e ocorrencia?
	eu acho que Recorrência não deveria ser um atributo de tarefa, pois se uma tarefa tem periodicidade ela deveria virar um evento
Implementation intention
	isso vai ser algo da minha parte, e no software isso vai estar na mistura de data planejada e ordem de execução, na visualização do dia
Revisão semanal
	não necessariamente eu vou ter um dia especifico pra fazer isso e eu posso fazer isso mais de uma vez na semana
isso são conceitos, e pro modelo do sistema muitas vezes uma parte tera varios conceitos envolvidos, tipo, concordamos da imlpementation intention ser uma camada, so que isso vai misturar data planejada e ordem de execução, na visualização do dia 

lembretes
	informações volateis 
	informação é diferente de tarefa, tarefa é algo executavel, informação não, é algo que deve ser lembrado, 
	vai ter um lfuxo parecido com tarefa, eu cadastro, e obrigatoriamente definir um tempo limite (data de quando vai sumir ou tempo de duração, tipo 7 dias), se acso for necessario ele pode virar uma informação permanente, mais pra frente vamos discutir como isso vai funcionar

filosofias
o mais desacoplado o possivel