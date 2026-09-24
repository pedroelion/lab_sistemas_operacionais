# Relatório do Laboratório 01: Concorrência, Threads e Race Conditions

**Disciplina:** Sistemas Operacionais  
**Aluno:** [Seu Nome Aqui]

---

## 1. Evidências Práticas
<img width="1115" height="964" alt="Captura de tela 2026-09-24 194132" src="https://github.com/user-attachments/assets/570d2d92-b99e-4162-86c4-b1a14901621f" />


---

## 2. Troca de Contexto e Atomicidade

**Questão:** Explique por que a linha `temp = temp + 1` e a posterior atribuição não são executadas num único ciclo de CPU, permitindo que a troca de contexto cause inconsistência.

**Resposta:** 
A operação de incremento não é uma operação atómica. Ao nível do hardware, esta instrução divide-se em pelo menos três ciclos de CPU distintos:
1. **Leitura:** Ler o valor atual da variável da RAM para um registo da CPU.
2. **Modificação:** Incrementar o valor (+1) no registo.
3. **Escrita:** Gravar o novo valor de volta na memória.

Como o sistema operativo é preemptivo, o escalonador pode interromper uma thread exatamente a meio destes passos. Se a Thread A ler o saldo (ex: 0) e for interrompida, a Thread B pode executar, ler o mesmo saldo (0), incrementar para 1 e gravar. Quando a Thread A voltar, ela continuará com o valor antigo (0), incrementará para 1 e gravará por cima, anulando o trabalho da Thread B (Condição de Corrida).

---

## 3. Custo do Lock

**Questão:** Compare o tempo de execução entre as versões insegura e segura. Por que o Lock adiciona sobrecarga (overhead)?

**Resposta:**
A versão segura tem um tempo de execução superior à versão insegura. O uso da primitiva `Lock` adiciona sobrecarga (overhead) por dois motivos:
1. **Comunicação com o Sistema Operativo:** Solicitar e libertar um lock exige chamadas ao sistema (system calls), que são custosas em processamento.
2. **Espera e Troca de Contexto:** Se uma thread tenta aceder à Secção Crítica e o lock já está em uso, o sistema operativo tem de a suspender, gerando uma troca de contexto forçada e perda temporária do paralelismo.

---

## 4. Desafio Extra (Bônus)

**Questão:** Adapte o código para simular 3 threads: duas a realizar depósitos e uma a realizar saques, mantendo o saldo final correto.

**Resposta:**
Abaixo encontra-se a implementação utilizando o `threading.Lock()` para garantir a exclusão mútua.

```python
import threading
import time

saldo_conta = 0
NUM_OPERACOES = 100000
lock_bancario = threading.Lock()

def depositar():
    global saldo_conta
    for _ in range(NUM_OPERACOES):
        with lock_bancario:
            temp = saldo_conta
            temp = temp + 1
            saldo_conta = temp

def sacar():
    global saldo_conta
    for _ in range(NUM_OPERACOES):
        with lock_bancario:
            temp = saldo_conta
            temp = temp - 1
            saldo_conta = temp

def main():
    global saldo_conta
    inicio = time.time()
    
    t1 = threading.Thread(target=depositar, name="Thread-Dep1")
    t2 = threading.Thread(target=depositar, name="Thread-Dep2")
    t3 = threading.Thread(target=sacar, name="Thread-Saque1")

    t1.start()
    t2.start()
    t3.start()

    t1.join()
    t2.join()
    t3.join()

    fim = time.time()
    print(f"[*] Saldo Esperado: 100000")
    print(f"[*] Saldo Obtido: {saldo_conta}")

if __name__ == "__main__":
    main()
