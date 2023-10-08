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

<code>
	
	#define PRIO_USER         40   //User Priority        
	int priority;
	int counter;    

</code>

<li>Na pm.c o processo é inicializado</li>

<p> Pela função pm_init(), os campos da struct processo são atualizados. Para essa implementação, os campos importantes são: </p>
<code>	
		
	IDLE->state = PROC_RUNNING;
	IDLE->counter = PROC_QUANTUM;
	IDLE->priority = PRIO_USER;
	 
</code> 
<p>Aqui temos o primeiro contato com as definições de um processo que será utilizado para inicialização do escalonamento, IDLE.</p>



 <li>Na Sched.c acontecem as operações relacionadas ao agendamento e ordem de execução da tabela de processos</li>
 <p>Podemos observar que no <code>Sched.c</code> existem algumas funções importantes como o sched em si que muda o estado do processo para para pronto e assim o processo na tabela de processos poderá ser executado quando o método yield fizer seu escalonamento após o termino de outro processo já escalonado. As alterações referentes ao trabalho foram implementadas dentro do método yield.</p>

 ### Yield ()
 <p>Dentro do método yield entendemos que havia implementado um algoritmo que selecionava o proximo processo com base em seu tempo de espera, sem levar em consideração sua prioridade efetiva. O trecho de código referente a essa implementação é esse:</p>
 <p><code> 
	 
	next = IDLE;
	for (p = FIRST_PROC; p <= LAST_PROC; p++)
	{
		/* Skip non-ready process. */
		if (p->state != PROC_READY)
			continue;

		/*
		 * Process with higher
		 * waiting time found.
		 */
		if (p->counter > next->counter)
		{
			next->counter++;
			next = p;
		}

		/*
		 * Increment waiting
		 * time of process.
		 */
		else
			p->counter++;
	}
 
</code></p>

<p>Então entendendo que esse trecho fazia a principal distinção entre qual seria o proximo processo executado, seguimos para a nossa solução para implementação de uma fila de prioridades</p>

### Yield ( ) - Com Fila de Prioridades
<p>Várias alterações foram feitas durante a decisão de qual forma seria implementada uma prioridade dinamica que se atualiza. A solução que mais fez sentido para o grupo foi a seguinte:</p>
<code>
	
	next = IDLE;
	for (p = FIRST_PROC; p <= LAST_PROC; p++)
	{
		/* Skip non-ready process. */
		if (p->state != PROC_READY)
			continue;

		// se prioridade e contador de tempo forem maiores, o processo é o proximo 
		if (p->priority >= next->priority)
		{	
			next->counter++;
			next = p;
			if(p->priority < PRIO_USER){
				p->priority --; //Aging 
			}
		}
		//caso não, contador incrementado
		else{
			p->counter++;
			if(p->counter >= MAX_WAITING && p->priority < PRIO_USER){ /**Max Waiting Time for process*/ 
				p->priority++;
			}
		}

	}
</code>
<p>No trecho condicional acima o proximo processo a ser executado é alterado sempre que ele possui a prioridade mais alta dos processos encontrados, para evitar inanição de processos foram implementados 2 técnicas vistas em sala de aula.
<li>Aging que faz uma diminuição da prioridade do processo sempre que o processo é executado com exeção de quando o processo tem uma prioridade de usuário (Foi levado em consideração que a prioridade PRIO_USER é a prioridade mais alta dentro do sistema)</li>
<li>Max-Waiting-Time, define um valor máximo de espera para que processos de prioridade baixa não fiquem infinitamente aguardando seu escalonamento</li>
</p>




 
