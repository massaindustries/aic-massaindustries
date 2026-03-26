# Official Toolkit Bugs & Fixes (from intrinsic-dev/aic)

> Questi sono fatti oggettivi dal repo ufficiale - bug confermati e fix rilasciati.
> Influenzano direttamente il nostro sviluppo.

## Bug Aperti (da monitorare)

### Bug #424 - Coordinate Frame Mismatch Isaac Lab ↔ Gazebo
- Isaac Lab usa offset hard-coded (`SFP_ENTRANCE_OFFSET_W`, `INSERTION_DIR_W`) senza applicare rotazione NIC card
- Gazebo calcola pose da TF tree dinamicamente
- L'asse -Z del plug body in Isaac Lab può essere ~35° diverso dalla direzione reale di inserzione
- **Impatto**: Policy addestrate in Isaac Lab possono fallire in Gazebo

### Bug #434 - Gravity Compensation 2x tra Isaac Lab e Gazebo
- Stessa configurazione giunti produce torque gravitazionali diversi:
  - Gazebo: ~[0, 16.5, 27.5, 3.16, 0, 0] Nm
  - Isaac Sim: ~[0, 32, 46, 7, 0, 0] Nm (~2x)
- **Impatto**: Dinamica del controller diversa tra simulatori

### Bug #444 - Phantom Collision Plane in Gazebo
- Comandando moto Cartesiano diretto verso una posa ~10cm sopra `sc_port`, il braccio si ferma come se colpisse un muro invisibile ~19cm sopra la task board
- **Workaround**: Decomporre il moto in movimenti sequenziali su singolo asse invece di un unico `set_pose_target()`

### Bug #282 - Collisione end-effector ↔ braccio disabilitata
- Si può muovere la camera attraverso i giunti in simulazione
- Il controller segnala violazione limiti effort ma non previene il moto

### Bug #278 - NIC Collision Box più largo della geometria visuale
- La collision box del NIC card mount si estende oltre la geometria visibile
- Il robot non riesce ad arrivare completamente a destra della superficie NIC
- **Tip**: Abilitare visualizzazione collisioni in Gazebo per vedere i confini reali

### Bug #283 - Engine non fa shutdown completo del model
- Il modello viene solo deattivato (non shutdown) alla fine di tutti i trial
- Potrebbe influenzare cleanup tra run successivi

## Bug Risolti (Knowledge Base)

### Issue #209 - set_cartesian_mode() si blocca ~30% delle volte
- Il codice si ferma in `set_cartesian_mode()` dopo "Entering insert_cable_execute_callback()"
- **Workaround**: Retry - si risolve da solo senza modifiche al codice. Aggiungere logica di retry nella policy

### Issue #339 - Observation è None alla prima chiamata
- L'oggetto observation può essere `None` quando la policy viene chiamata per la prima volta
- **Fix**: SEMPRE controllare `if obs is None: continue` nel loop

### Issue #303 - Jerk inconsistente tra macchine
- Il jerk medio calcolato varia enormemente in base alla macchina (400 vs 28,283 m/s³)
- Causato da differenze di timing e Real-Time Factor (RTF)
- Macchine con ~70% RTF danno jerk più basso di quelle che girano più veloci

### Issue #318 - Tier-3 scoring riporta insertion fallita erroneamente
- Anche quando l'inserzione riesce, Tier-3 può riportare "Cable insertion failed. Incorrect Port"
- Era un bug nel sistema di scoring (corretto)

### Issue #320 - Static TF topic sovraccaricato
- PosePublisher di Gazebo sovrascrive `/tf_static` a frequenza fissa
- `gripper/tcp` static pose pubblicata solo una volta → subscriber in ritardo la perdono
- Aggiunto `tf_static_relay` ma inizialmente con QoS VOLATILE invece di TRANSIENT_LOCAL

### Issue #239 - Errore teleop si accumula durante collisione
- Durante collisione, i comandi si accumulano
- Quando la collisione si libera, tutti i comandi accumulati eseguono contemporaneamente → moto selvaggio

### Issue #396 - CheatCode fallisce nel trial 3 di valutazione
- La policy CheatCode può fallire nel trial 3 (SC insertion)
- Etichettato come "known issue" dai maintainer
- Non assumere che le policy di esempio funzionino al 100%

## Docker/Submission Issues

### Issue #338/#377/#397/#266 - POSIX SHM Error (il bug più comune)
- `Failed to create POSIX SHM provider (Error code: -1)` crasha `rclpy.init()`
- **Cause**: Zenoh shared memory abilitata, permessi, IPC namespace
- **Fix**: `transport/shared_memory/enabled=false` nel config Zenoh
- **Fix alternativo**: `sudo /entrypoint.sh` o fix IPC nel docker-compose

### Issue #415 - Variabili d'ambiente non sovrascritte con ROS di sistema
- Se hai un'installazione ROS di sistema, `pixi_env_setup.sh` non sovrascrive `RMW_IMPLEMENTATION`
- La policy non si connette anche se Gazebo e RViz funzionano
- **Fix**: Script corretto per forzare override delle env vars

### Issue #429 - /entrypoint.sh scompare dopo uscita dal distrobox
- Dopo uscita e rientro nel container distrobox, l'entrypoint sparisce
- **Fix**: `docker system prune -a --volumes` e ricreare il container

## Fix Rilasciati (da applicare)

### PR #405 - Task Board TF ora Statici
- Prima pubblicati a 1Hz causando race condition e lookup failure
- Ora pubblicati come static TF → lookup affidabili

### PR #430 - set_pose_target() con Stiffness/Damping Configurabili
```python
self.set_pose_target(move_robot, target_pose,
    stiffness=[90, 90, 90, 50, 50, 50],  # default
    damping=[50, 50, 50, 20, 20, 20])     # default
```

### PR #431 - OffLimitContactsPlugin ora World System
- Verificare collisioni con: `gz topic -e -t /aic/gazebo/contacts/off_limit`

### PR #432 - Fix docker compose up (Zenoh)
- `ZENOH_CONFIG_OVERRIDE` non deve essere sovrascritto nel compose

### PR #416 - RMW sempre Zenoh
- `RMW_IMPLEMENTATION=rmw_zenoh_cpp` fisso, no alternative DDS

### PR #411 - aic_engine exit code non-zero su errore
- Utile per testing automatizzato delle submission

### PR #419 - MuJoCo-to-Gazebo Dynamics Tuning
Se si usa MuJoCo per training, applicare:
- Damping viscoso per-giunto + armature (inerzia rotore) per UR5e
- Cable joint damping: 0.1 → 0.2
- SC port friction: [0.5, 0.5]
- NIC card mount friction: [0.1, 0.005, 0.1]
- Solver: implicitfast, Newton, 200 iter, 1e-10 tol, 2ms timestep
- **Risultato**: RMSE posizione da 0.064 rad a ~0.014 rad

### PR #406 - Fix Launch Crash su shutdown
- `on_exit` callback corretto, shutdown pulito dopo aic_engine

### PR #422 - Rinominato tare_ft_sensor → tare_force_torque_sensor nei docs

### PR #438 - LeRobot 0.5.0 in arrivo
- Bump da 0.4.3 a 0.5.0 in corso

## Dettagli Tecnici Non Ovvi (dalle Issues)

### Scoring
- 3 trial per evaluation run (trial_1, trial_2, trial_3)
- Task board e cavi ri-spawnati tra trial
- Score computation può sembrare bloccata → è normale (Issue #325)
- Verifica manuale delle top solution da parte degli organizzatori (no gaming)

### Controller
- Controller in modalità impedance: riporta "Control mode set to impedance"
- Joint effort limits enforced con warning throttled (es: wrist_1_joint effort: comandato 30.6, limitato a 28.0)
- 6 giunti braccio: shoulder_pan, shoulder_lift, elbow, wrist_1, wrist_2, wrist_3

### TF Frames chiave per SFP
- `sfp_port_0_link` - frame del port
- `sfp_port_0_link_entrance` - entrata del port (offset `(0, 0, -0.0458)` in frame locale)
- `cable_0/sfp_tip_link` - tip del plug
- NIC card ha rotazione `(-1.57, 0, 0)` nel suo mount

### LeRobot / ML
- LeRobot 0.4.3 ha conflitti NumPy 2.0 con stack ROS Kilted (Issue #437)
- RTX 50xx (Blackwell, sm_120) non supportate dal PyTorch pinnato (Issue #379)
- Camera image scaling default 0.25 per LeRobot
- `wrench_feedback_gains_at_tip` è un vettore di 6 elementi [Fx, Fy, Fz, Tx, Ty, Tz] (Issue #237)

## Prossime Feature (Draft/Open PRs)

### PR #441 - Insert Cable Skill per Flowstate (Phase 1)
- Nuovo pacchetto `insert_cable_skill` con state machine
- Non necessario per qualification

### PR #442 - Sideload aic_model in Flowstate
- Caricamento diretto del policy node in Flowstate

### PR #428 - Servizio ExpandXacro
- Servizio ROS 2 per espandere xacro senza accesso filesystem eval
