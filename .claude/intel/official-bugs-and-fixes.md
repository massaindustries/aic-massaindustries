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

## Prossime Feature (Draft/Open PRs)

### PR #441 - Insert Cable Skill per Flowstate (Phase 1)
- Nuovo pacchetto `insert_cable_skill` con state machine
- Non necessario per qualification

### PR #442 - Sideload aic_model in Flowstate
- Caricamento diretto del policy node in Flowstate

### PR #428 - Servizio ExpandXacro
- Servizio ROS 2 per espandere xacro senza accesso filesystem eval
