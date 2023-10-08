## Implementação de Fila de Prioridades para o sistema Nanvix

## Objetivo da implementação

O sistema Nanvix possui um sistema de escalonamento de processos simples baseado em Round-robin: atribuir a cada processo um quantum de tempo e então escaloná-los seguindo o critério first-in first-out. No entanto, esse tipo de escalonamento pode gerar esperas muito longas para processos executados por último e, por isso, buscamos implementar um algoritmo utilizando a fila de prioridades para realizar o escalonamento do sistema.

## O que é Fila de Prioridades

Uma fila de prioridades em um sistema de escalonamento de um sistema operacional é uma estrutura de dados usada para gerenciar os processos que estão prontos para serem executados pela CPU com base em suas prioridades. Ela permite que o sistema operacional selecione o próximo processo a ser executado com base em critérios de prioridade definidos.



## O sistema de Prioridades do Nanvix


#define PRIO_USER         40 /**< User priority.               */
