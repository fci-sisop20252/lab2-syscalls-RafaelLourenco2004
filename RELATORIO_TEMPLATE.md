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

```
A diferença se deve ao fato de que quando a função "write" é chamada a operação de saída é executada imediatamente, logo a quantidade de execuções é igual a quantidade de chamadas. Por outro lado, a função "printf", além de funcionar como uma camada de abstração acima de "write", tem um buffer onde os paramêmetros de saída são armazenados até uma certa condição ser atendida e todas as operações de saída serem executadas de uma vez. Como essa condição pode variar, a quantidade de operações de saída executadas nem sempre será igual a quantidade de chamadas da função "printf".
```

**3. Qual método é mais previsível? Por quê você acha isso?**

```
A função "write" é mais previsível porque a cada chamada uma operação de saída é executada, ao contrário de "printf" onde a quantidade de operações de saída executadas nem sempre se iguala a quantidade de chamadas, por conta do buffer de dados. 
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
O file descriptor 3 foi usado, pois ele era o primeiro livre, além dos files descriptors reservados 1, 2 e 3.
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
A única forma de saber que o arquivo foi lido completamente é se a função "read" retornar 0, enquanto o retorno não for igual a 0 ou -1 a função deve ser chamada repetidamente. 
```

**3. Por que verificar retorno de cada syscall?**

```
Deve-se vericar o retorno de cada syscall, pois seu comportamento pode variar. 
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
| 16          |          89       |     0,001290      |
| 64          |            23     |     0.000743      |
| 256         |             8    |     0.000317       |
| 1024        |             4    |        0.000708   |

### 🔍 Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
Quanto maior o tamanho do buffer mais bytes podem ser lidos a cada chamada `a "read", logo a quantidade de chamadas necessárias para ler todos os bytes do arquivo é menor.
```

**2. Todas as chamadas read() retornaram BUFFER_SIZE bytes? Discorra brevemente sobre**

```
Não, "read" pode retornar -1, sinalizando que houve um erro durante a leitura, 0, indicando que o conteúdo do arquivo foi lido completamente e um número inteiro positivo, que indica a quantidade de bytes lidos. A quantidade de bytes lidos nem sempre será igual ao tamanho do buffer, pois existem alguns cenários onde a chamada irá ler menos bytes do que o esperado como no caso de haver menos bytes não lidos no arquivo do que o tamanho do buffer.  
```

**3. Qual é a relação entre syscalls e performance?**

```
Sempre que ocorre uma chamada a uma syscall o precessador deve trocar o modo de execução de "User mode" para "Kernel" o que gera um overhead a cada chamada de syscall. 
```

---

## 4️⃣ Exercício 4 - Cópia de Arquivo

### 📈 Resultados:
- Bytes copiados: 1364
- Operações: 6
- Tempo: 0,000208 segundos
- Throughput: 6404,00 KB/s

### ✅ Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [ ] Idênticos [ ] Diferentes

### 🔍 Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
Se a quantidade de bytes escritos no arquivo de destino for diferente da quantidade de bytes lidos do arquivo original quer dizer que houve algum erro durante o processo de cópia, logo a intergridade da cópia gerada está comprometida.  
```

**2. Que flags são essenciais no open() do destino?**

```
 Apenas "O_WRONLY", pois é possível criar um arquivo de destino vazio manualmente.  
```

**3. O número de reads e writes é igual? Por quê?**

```
Para arquivos em disco sim, pois a quantidade de bytes lidos vai ser igual a quantidade de bytes escritos. 
```

**4. Como você saberia se o disco ficou cheio?**

```
Se o disco ficar cheio a função write irá retornar -1.
```

**5. O que acontece se esquecer de fechar os arquivos?**

```
Se alguém esquecer de fechar os arquivos eles ficarão reservados para o precesso que os abriu até o este finalizar sua execução, impossibilitando outro processo de acessar estes arquivos desnecessariamente.  
```

---

## 🎯 Análise Geral

### 📖 Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
Sempre que ocorre uma syscall o processador muda do estado de "usuário" para "kernel", permitindo que o processo em execução acesse o hardware e os recursos do sistema operacional através do kernel. 
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
Os files descriptors funcionam como uma identificação para um determinado arquivo o que possibilita o sistema operacional identificar quais arquivos estão sendo acessados em um determinado momento. 
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
Quanto maior o tamanho do buffer, menor a quantidade de syscalls chamadas. 
```

### ⚡ Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** O cp

**Por que você acha que foi mais rápido?**

```
Eu acho que a diferença se deve, principalmente, ao nível de abstração de cada uma. Meu programa é mais abtraído do que "cp" que funciona com chamadas diretas a funções do sistema operacional. 
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
