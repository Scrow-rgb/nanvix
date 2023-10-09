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

<p>A Struct Process implementa os campos de um processo. Nessa struct, presente na biblioteca pm.h vemos que o processo possui um campo priority e uma prioridade de usuário definida como 40 inicialmente, além de um counter que conta o tempo (quantum) faltante para o processo (que será utilizado para contar quanto tempo um processo está em espera com estado Ready). Adicionamos tambem uma definição para o tempo maximo de espera, para implementação da prioridade dinamica.

<code>

 	#define MAX_WAITING       30 
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
<p>Essas 2 implementações fazem as alterações de prioridade necessárias para que os processos mais prioritários sejam escalonados primeiro e conforme a tabela de processos, caso um processo tenha sua prioridade modificada pela função nice o algoritmo funcionará do mesmo jeito já que as prioridades estão sendo comparadas diretamente dos processos da tabela de processos</p>

<hr>

# Relatório

## Introdução

<hr>
<p>
Neste relatório, compartilhamos os resultados de uma série de testes base que conduzimos no sistema operacional Nanvix , utilizando o emulador QEMU. Nosso objetivo principal foi comparar o teste do arquivo base com nossas alterações nesse arquivo.
</p>

## Teste 1: Teste da Unidade de Ponto Flutuante (Test FPU)

<hr>
<ul>
<li>Resultado: PASSED
</ul>

<p>
Começamos nossos testes com o "Test FPU", focado na avaliação da unidade de ponto flutuante do Nanvix. Ficamos satisfeitos ao constatar que o teste passou com sucesso, indicando um bom desempenho nessa área.
</p>

## Teste 2: Teste de E/S (Test IO)

<ul>
<li>Resultado: PASSED
<li>Tempo decorrido: 600 unidades
</ul>

<p>
O segundo teste, chamado de "Test IO", concentrou-se na entrada e saída de dados. Este teste também foi bem-sucedido, com um tempo decorrido de 600 unidades.
</p>

## Teste 3: Teste de Swap (Swapping Test)

<hr>
<ul>
<li>Resultado: PASSED
<li>Tempo decorrido: 12000 unidades
</ul>

<p>
No terceiro teste, conhecido como "Swapping Test", exploramos a capacidade de troca de páginas do Nanvix. Este teste também foi concluído com sucesso, e o tempo decorrido foi de 12000 unidades.
</p>

## Teste 4: Teste de Agendamento (Scheduling Test)

<hr>
<ul>
<li>Waiting for child: PASSED
<li>Dynamic priorities: PASSED
<li>Sheduler stress: PASSED
</ul>

<p>
Em nosso quarto teste, o "Scheduling Test", examinamos vários aspectos do agendamento de processos. Todos os subtestes, incluindo "Espera por Filho", "Prioridades Dinâmicas" e "Estresse no Agendador", obtiveram êxito.
</p>

## Teste 5: Teste de Comunicação entre Processos (IPC)

<hr>
<ul>
<li>Resultado: Kernel Page Fault e Crash
</ul>

<p>
No quinto e último teste, o "IPC Test", enfrentamos um desafio. Infelizmente, o teste resultou em um "Kernel Page Fault" e no travamento do sistema.
</p>

# Teste com nossas alterações:

## Teste de agendamento que demos o nome de sched4 (Sheduling Test)

<hr>

<ul>
<li> Resultado: PASSED
</ul>

## Diferenças

<hr>

<p>
Com base no teste original feito acima, os resultados sofreram  diferenças com o teste  com nossas alterações, foi constatado duas mudanças,Teste IO (Entrada e Saída de Dados) e no teste de Swap(Swapping Test) e adicionamos mais um teste que chamamos de test sched4
</p>

## Conclusão

<hr>
 <p>
 No teste IO (Entrada e Saída de Dados) a variação de tempo decorrido de 600 unidades no teste base para 700 unidades no teste alterado pode ser devido a várias razões, como carga de trabalho do sistema, sobrecarga de processos em segundo plano ou até mesmo flutuações naturais na execução de tarefas.
No teste de Swap (Swapping Test) da mesma forma, a variação no tempo decorrido de 12000 unidades no teste base para 12300 unidades no teste com nossas alterações pode ser explicada pelo fato  de algoritmos que trabalham com prioridade terem mais tempo para selecionar qual processo vai ser executado, eles demandam um tempo maior, pois demora mais tempo para ver quais processos irão ser escalonados ou não e a capacidade de troca de páginas pode ser afetada pela carga de trabalho do sistema e pela disponibilidade de recursos de memória.
</p>





 
