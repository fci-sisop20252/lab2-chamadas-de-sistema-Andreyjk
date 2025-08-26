# 📝 Relatório do Laboratório 2 - Chamadas de Sistema

---

## 1️⃣ Exercício 1a - Observação printf() vs 1b - write()

### 💻 Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### 🔍 Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: 8 syscalls
- ex1b_write: 7 syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**
Há diferença, pois o printf é capaz de escrever textos para o usuário, formatar dados e é utilizado para casos que não seja necessário uma performance crítica.
E o write é capaz de escrever, além de texto, dados binários, comtrolar quando enviar os dados e possui um comportamento previsível.

**3. Qual método é mais previsível? Por quê você acha isso?**

```
O método write() é mais previsível, porque ele faz exatamente uma chamada de sistema para cada solicitação de escrita, sem depender de buffers internos ou formatação extra. Já o printf() utiliza buffering e pode gerar mais chamadas de sistema dependendo do tamanho da saída ou da forma como os dados são formatados. Isso torna o comportamento do printf() menos direto, enquanto o write() é consistente e previsível.
```

---

## 2️⃣ Exercício 2 - Leitura de Arquivo

### 📊 Resultados da execução:
- File descriptor: _____
- Bytes lidos: _____

### 🔧 Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### 🔍 Análise

**1. Qual file descriptor foi usado? Por que não começou em 0, 1 ou 2?**

```
O file descriptor usado foi o 3, porque os descritores 0, 1 e 2 já são reservados pelo sistema para stdin (entrada padrão), stdout (saída padrão) e stderr (saída de erro). Assim, o próximo descritor disponível é o 3.
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
Porque a última chamada de read() retornou 0, o que indica EOF (End of File) — ou seja, não havia mais dados a serem lidos.
```

**3. Por que verificar retorno de cada syscall?**

```
Porque o retorno mostra se a chamada foi bem-sucedida. Em caso de erro, o retorno é -1 e o programa pode reagir (ex.: tentar novamente, liberar recursos ou exibir uma mensagem).
```

---

## 3️⃣ Exercício 3 - Contador com Loop

### 📋 Resultados (BUFFER_SIZE = 64):
- Linhas: _____ (esperado: 25)
- Caracteres: _____
- Chamadas read(): _____
- Tempo: _____ segundos

### 🧪 Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |                 |           |
| 64          |                 |           |
| 256         |                 |           |
| 1024        |                 |           |

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
Quanto menor o buffer, mais chamadas read() são necessárias para percorrer o arquivo inteiro. Buffers maiores reduzem a quantidade de syscalls.
```

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre**

```
Não. Na maioria das vezes sim, mas a última chamada pode retornar menos bytes, dependendo do tamanho do arquivo, ou até 0 no final (EOF)
```

**3. Qual é a relação entre syscalls e performance?**

```
Syscalls têm custo alto, pois exigem transição do modo usuário para modo kernel. Menos syscalls, menos overhead, melhor desempenho.
```

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: _____
- Operações: _____
- Tempo: _____ segundos
- Throughput: _____ KB/s

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [ ] Idênticos [ ] Diferentes

### 🔍 Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
Para garantir a integridade da cópia. Se menos bytes forem escritos do que lidos, parte do arquivo de destino ficará corrompida ou incompleta
```

**2. Que flags são essenciais no open() do destino?**

```
As flags O_WRONLY | O_CREAT | O_TRUNC:

O_WRONLY: abre para escrita.

O_CREAT: cria o arquivo se não existir.

O_TRUNC: zera o conteúdo se já existir.
```

**3. O número de reads e writes é igual? Por quê?**

```
Sim, pois para cada leitura realizada deve haver uma escrita correspondente, mantendo a proporção entre o que foi lido e o que foi copiado.
```

**4. Como você saberia se o disco ficou cheio?**

```
O write() retornaria menos bytes escritos do que o solicitado, ou retornaria -1 com erro ENOSPC.
```

**5. O que acontece se esquecer de fechar os arquivos?**

```
O sistema mantém os descritores abertos até o processo terminar, desperdiçando recursos. Além disso, dados podem não ser realmente gravados no disco, já que ficam no buffer.
```

---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
Cada syscall (read, write, open, close) faz o programa sair do modo usuário e pedir ao kernel que execute a operação. Isso é necessário porque só o kernel tem acesso direto ao hardware.
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
São referências numéricas para identificar arquivos abertos. Sem eles, o kernel não teria como saber em qual arquivo realizar as operações de leitura/escrita
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
Buffers maiores → menos chamadas de sistema → menos overhead. Mas buffers gigantes podem desperdiçar memória. Existe um equilíbrio ideal dependendo do tamanho do arquivo e do sistema.
```

### ⚡ Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** _____
````
O comando cp do sistema.
````
**Por que você acha que foi mais rápido?**

```
Porque o cp é altamente otimizado, usando rotinas eficientes do kernel, buffers ajustados e até chamadas especiais como sendfile(), reduzindo o número de cópias de dados entre usuário e kernel.
```

---

## 📤 Entrega
Certifique-se de ter:
- [ ] Todos os códigos com TODOs completados
- [ ] Traces salvos em `traces/`
- [ ] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```
# Bom trabalho!
