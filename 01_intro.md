# Concorrenza - Introduzione

## Concetti Fondamentali

### Assioma di Finite Progress
Ogni processo viene eseguito ad una velocità **finita, non nulla, ma sconosciuta**.

### Stati dei Processi (versione semplice)
- `Running`: In esecuzione.
- `Waiting`: In attesa di un evento esterno (es. I/O). Non può essere eseguito.
- `Ready`: Può essere eseguito, ma la CPU è occupata.

### Interleaving vs Overlapping
- **Interleaving**: 1 processore. Un solo processo in esecuzione in un dato istante (alternanza nel tempo).
- **Overlapping**: Sistema multiprocessore. Più processi eseguiti simultaneamente su CPU diverse (alternanza nello spazio).

> **Problema comune**: Non si può predire la velocità relativa dei processi. Le problematiche di multiprogramming e multiprocessing sono sostanzialmente le stesse.

---

## Problemi e Proprietà

### Race Condition
> Il risultato finale dell'esecuzione dipende dalla **temporizzazione** (ordine) con cui vengono eseguiti i processi.

### Interazione tra Processi
- **Memoria Condivisa**: Comunicazione tramite sincronizzazione.
- **Memoria Privata**: Sincronizzazione tramite comunicazione.

### Proprietà Chiave
- **Safety**: I processi non devono interferire tra loro nell'accesso a risorse condivise. *Cosa non deve succedere.*
- **Liveness**: I meccanismi di sincronizzazione non devono bloccare l'avanzamento del programma. *Cosa deve succedere.*

---

## Patologie della Concorrenza

### Deadlock
- **Causa**: La mutua esclusione, se mal gestita, può portare a un blocco permanente dei processi.
- **Caratteristiche**:
    - È una condizione da evitare.
    - È **definitiva**.
    - Si risolve solo con metodi distruttivi (kill processo, reboot).
- **Nota**: L'assenza di deadlock è una proprietà di **Safety**.

### Starvation
- **Esempio**: Processi P1 e P2 si alternano su una risorsa R, impedendo a P3 di accedervi. Si dice che P3 è in *starvation*.
- **Caratteristiche**:
    - **Non è definitiva** (a differenza del deadlock).
    - Risolvibile con un'opportuna politica di assegnamento (scheduling).
    - È comunque una situazione da evitare.

---

## Atomicità e Sezioni Critiche (CS)

### Azioni Atomiche (ipotesi per l'esame)
- **NON sono atomiche**:
    - Aggiornamento di una variabile (`x = x + 1`)
    - Valutazione di espressioni
- **SONO atomiche**:
    - Assegnamento di un valore costante a una variabile (`x = 5`)
    - Lettura/scrittura di una singola cella di memoria alla volta.

**Notazione**: `<S>` indica che lo statement `S` è eseguito in modo atomico.
- Esempio: `< x = x + 1; >`

### Sezione Critica (CS)
> La parte di un programma che utilizza una o più risorse condivise.

**Sintassi per l'esame**:
- `[enter cs]`: Inizio della sezione critica.
- `[exit cs]`: Fine della sezione critica.

**Esempio**:
```c
// Process P
[enter cs]
a = a + val;
b = b + val;
[exit cs]

// Process Q
[enter cs]
a = a * val;
b = b * val;
[exit cs]
```

### Requisiti per le CS
1.  **Mutua Esclusione**: Un solo processo nella CS alla volta.
2.  **Assenza di Deadlock**.
3.  **Assenza di Delay non necessari**: Se la CS è libera, un processo che vuole entrare deve poterlo fare.
4.  **Eventual Entry**: Assenza di Starvation.

> La responsabilità di garantire la mutua esclusione è del **S.O.** o del **linguaggio** (es. Semafori, Monitor, Message Passing).

---

## Soluzioni al Problema della CS

### Approcci Software (e loro fallimenti)
- **Tentativo 1/2**: "Verifica di una variabile + aggiornamento di un'altra" **non è atomico** e fallisce.
- **Tentativo 3 (Turni)**: Evita deadlock e race condition, ma introduce *delay non necessari*.
- **Tentativo 4 (Bandierine)**: Può causare **deadlock** (se entrambi insistono) o **livelock** (se entrambi sono "gentili" e si ritirano).

**Soluzioni software note**: Algoritmo di **Dekker**, Algoritmo di **Peterson**.
- **Problema**: Sono basate su **busy waiting** (il processo consuma CPU controllando una condizione in un loop).

### Approcci Hardware

#### 1. Disabilitare gli Interrupt
```c
while (true) {
  disable_interrupts();
  // critical section
  enable_interrupts();
  // non-critical section
}
```
- **Contro**:
    - Pericoloso lasciare ai processi la gestione degli interrupt.
    - **Non funziona su sistemi multiprocessore**.

#### 2. Istruzioni Atomiche Speciali (Test & Set)
Realizzano due azioni in modo atomico (es. lettura+scrittura).

**Spinlock**: Una (quasi) sezione critica realizzata con istruzioni speciali.

**Istruzione Test & Set**: `TS(x, y)`
- **Definizione**: `< y = x; x = 1; >`
- **Spiegazione**: Ritorna in `y` il valore precedente di `x` e imposta `x` a `1`, il tutto in un'unica operazione atomica.

- **Pro**: Applicabile a N processi.
- **Contro**: Utilizza **busy-waiting**, non elimina la **starvation**.
