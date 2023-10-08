## Implementação de Fila de Prioridades para o sistema Nanvix
Por:
Caio Silva
Mateus Gontijo
Milleny Teixeira

## Objetivo da implementação

O sistema Nanvix possui um sistema de escalonamento de processos simples baseado em Round-robin: atribuir a cada processo um quantum de tempo e então escaloná-los seguindo o critério first-in first-out. No entanto, esse tipo de escalonamento pode gerar esperas muito longas para processos executados por último e, por isso, buscamos implementar um algoritmo utilizando a fila de prioridades para realizar o escalonamento do sistema.

## O que é Fila de Prioridades

Uma fila de prioridades em um sistema de escalonamento de um sistema operacional é uma estrutura de dados usada para gerenciar os processos que estão prontos para serem executados pela CPU com base em suas prioridades. Ela permite que o sistema operacional selecione o próximo processo a ser executado com base em critérios de prioridade definidos.



## O sistema de Prioridades do Nanvix

As prioridades do sistema Nanvix obedecem os seguintes critérios:

<li>Prioridade estática, que é atribuída pelo sistema operacional com base na atividade de um processo:

<p>A Struct Process implementa os campos de um processo. Nessa struct, presente na biblioteca pm.h vemos que o processo possui um campo priority e uma prioridade de usuário definida como 40 inicialmente, além de um counter que conta o tempo (quantum) faltante para o processo.

<code>#define PRIO_USER         40 /**< User priority.               *
int priority;
int counter;    </code>

<li>Na pm.c o processo é inicializado</li>

<p> Pela função pm_init(), os campos da struct processo são atualizados. Para essa implementação, os campos importantes são:
<code>	IDLE->state = PROC_RUNNING;
	IDLE->counter = PROC_QUANTUM;
	IDLE->priority = PRIO_USER;</code> </p>

 
