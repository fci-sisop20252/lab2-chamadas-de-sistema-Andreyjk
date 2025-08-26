Exercício 1a – printf

Comando: make ex1a_printf && ./ex1a_printf

Saída: (copiar as 3 mensagens)

Comando: strace -e write ./ex1a_printf

Observação: Várias mensagens, mas menos write() do que esperávamos → o printf usa buffer interno antes de chamar write.

Exercício 1b – write

Comando: make ex1b_write && ./ex1b_write

Saída: (as 3 mensagens)

Comando: strace -e write ./ex1b_write

Observação: Aqui cada chamada write() do código aparece 1:1 no strace. Ou seja, o write vai direto para o kernel sem buffering.

Exercício 2 – Leitura de arquivo

Comando: make ex2_leitura && ./ex2_leitura

Saída: Mostra o conteúdo de dados/teste2.txt.

Comando: strace -e openat,read,close ./ex2_leitura

Observação: Vemos openat() para abrir, várias read() até fim do arquivo e close() no final.

Exercício 3 – Contador de linhas

Comando: make ex3_contador && ./ex3_contador

Saída: mostra linhas, caracteres, nº de chamadas read, tempo, média de bytes por leitura.

Observação: Alterar BUFFER_SIZE muda drasticamente o número de syscalls — buffer menor → mais read(), buffer maior → menos chamadas → mais eficiente.

Exercício 4 – Cópia de arquivo

Comando: make ex4_copia && ./ex4_copia

Saída: "Cópia concluída! Verifique dados/destino.txt"

Comando: strace -o traces/ex4_trace.txt ./ex4_copia

Obs: O trace mostra a sequência de openat do arquivo de origem, read em blocos e write no destino, e depois close em ambos. É o fluxo de cópia de arquivo.
