## 🔍 **Visão geral do que está sendo monitorado**

A ideia principal desse dashboard é **monitorar a sincronização entre a instância Blue (ativa) e a Green (cópia)** enquanto você faz modificações na Green (ex: `ALTER TABLE`, testes, novos índices, etc). Como a replicação é **lógica**, é fundamental saber:

- Se os eventos do Blue estão acumulando (binlog)
- Se a Green está aplicando esses eventos com atraso (lag)
- Se a Green está com recursos suficientes para processar tudo depois (CPU, memória, disco)

---

## 📊 **Explicação de cada widget**

---

### 1. **Binlog Disk Usage (Blue)**  
**Métrica:** `BinLogDiskUsage`  
**Fonte:** Cluster Aurora Blue

🧠 **O que mostra:**  
Quantidade de espaço em disco ocupada pelos **binlogs**, que armazenam os eventos de replicação lógica (inserts, updates, deletes, etc).

🔍 **Por que monitorar:**  
Se a Green não conseguir acompanhar as alterações (por estar travada, por exemplo), os binlogs vão **se acumular**.  
Um crescimento anormal aqui pode indicar que:
- A replicação está atrasada
- A Green está sobrecarregada ou travada
- Você está perto de estourar o limite de retenção

🟠 **Alerta:** Acima de 5 GB = pode sinalizar acúmulo de eventos

---

### 2. **Replica Lag (Green)**  
**Métrica:** `AuroraReplicaLag`  
**Fonte:** Instância Green

🧠 **O que mostra:**  
Tempo de atraso da instância Green para aplicar as mudanças do Blue (em segundos).

🔍 **Por que monitorar:**  
Esse é **o coração da replicação**. Se o lag está alto, significa que:
- A Green ainda não refletiu tudo que aconteceu no Blue
- A troca pode ser perigosa (você pode promover uma instância desatualizada)

🟠 **Alerta:** Acima de 30~60s = atenção. Acima de 300s = risco alto.

---

### 3. **CPU Usage (Green)**  
**Métrica:** `CPUUtilization`  
**Fonte:** Instância Green

🧠 **O que mostra:**  
Porcentagem de uso da CPU da instância Green.

🔍 **Por que monitorar:**  
Depois que a Green "destrava" (ex: após um ALTER), ela pode tentar aplicar muitos eventos de uma vez.  
Isso pode causar:
- Estouro de uso de CPU
- Lags contínuos
- Travamentos ou timeouts

🟠 **Alerta:** Acima de 80% de forma constante = pode travar

---

### 4. **Freeable Memory (Green)**  
**Métrica:** `FreeableMemory`  
**Fonte:** Instância Green

🧠 **O que mostra:**  
Memória RAM livre (em bytes).

🔍 **Por que monitorar:**  
Memória livre baixa pode levar a:
- Uso de swap (muito lento)
- Processos de replicação ficarem ainda mais lentos
- Crash do banco

🟠 **Alerta:** Abaixo de 500 MB = risco de lentidão ou falha

---

### 5. **Disk Queue Depth (Green)**  
**Métrica:** `DiskQueueDepth`  
**Fonte:** Instância Green

🧠 **O que mostra:**  
Número de operações de I/O em disco que estão **na fila**, esperando para serem processadas.

🔍 **Por que monitorar:**  
Após o término de um `ALTER TABLE`, a Green pode ter muitas alterações acumuladas no binlog.  
Se o disco não acompanhar a aplicação dos eventos, essa fila cresce e o lag também.

🟠 **Alerta:** Acima de 50 = pode indicar gargalo no disco

---

## ✅ **Resumo da utilidade**

| Widget                   | Monitorar o quê?                  | Indica o quê?                            |
|--------------------------|-----------------------------------|-------------------------------------------|
| Binlog Disk Usage        | Crescimento de eventos acumulados | Replicação travada ou atrasada            |
| Replica Lag              | Atraso da Green                   | Green desatualizada                       |
| CPU Usage                | Carga de processamento da Green   | Pode travar ao aplicar muitos eventos     |
| Freeable Memory          | Saúde de memória da Green         | Pode impactar replicação/performance      |
| Disk Queue Depth         | I/O em espera                     | Gargalo ao aplicar replicações acumuladas |
