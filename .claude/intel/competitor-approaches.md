# Competitor Approaches & Tactics

> NOTA: Questo file documenta approcci osservati da altri partecipanti.
> NON usare questi approcci come vincoli per il nostro sviluppo.
> Sono riferimenti informativi, non best practice da seguire ciecamente.

## Landscape (Marzo 2026)
- ~157 fork del repo ufficiale, la maggior parte non modificati
- La maggior parte dei team tiene le strategie private
- Pochi repo pubblici con codice custom

## Partecipanti Documentati

### Rocky0Shao (Ohio State, freshman)
- **Score pubblico**: 216/300 (1 full insertion, 2 partial su 3 trial)
- **Approccio**: State machine con CheatCode per generare demo → LeRobot ACT training
- **Fasi**: INIT → APPROACH → ALIGN → INSERT → DONE
- **Controllo**: PI velocity control, hover a 20cm, allineamento XY per 2+ secondi
- **Parametri usati**: kp_linear=1.2, ki_linear=0.2, kp_angular=2.0, max_vel=0.08 m/s
- **Threshold forza**: 19.5N
- **Discesa**: 12mm/s costante con compliance gains laterale
- **Dati**: Target 50+ demo first-try per trial via LeRobot pipeline
- **Problemi incontrati**: velocità frame-rate dependent, naming cable diverso tra engine e manual spawn
- **Insight**: SC e SFP richiedono tuning separato

### No_Quarter_Robotics (Discourse)
- **Approccio**: Debugging sistematico con episode logging strutturato
- **Pattern**: Snapshot per fase (initial, hover, align, insert, final)
- **Scoperta**: Topic `/scoring/insertion_event` per conferma inserzione real-time
- **Filosofia**: Verificare sempre che il robot abbia raggiunto la posa
- **Score**: 216 con 3 trial (1 full, 2 partial)
- **Reverse engineering**: Container setup con GPU passthrough esplicito

### harilakshman-333
- **Approccio pianificato**: CV classica + pose estimation learned + MoveIt2
- **Training**: RL con domain randomization
- **Target**: <15s ciclo, <3mm errore, >85% success
- **Status**: Pianificazione, nessun risultato pubblico

### mattdamon1868 (Team Deep Dive)
- **Status**: Setup iniziale con bash scripts, nessun codice custom

### SujitMohite (Team Autoencoder)
- **Status**: Fork senza modifiche visibili

## Pattern Comuni Osservati
- Convergenza su: CheatCode per data collection → ACT/LeRobot per training
- State machine con fasi discrete
- Hover-then-descend come pattern di inserzione

## Fonti
- GitHub: intrinsic-dev/aic issues e fork
- Open Robotics Discourse: forum threads
- Repository pubblici dei partecipanti
