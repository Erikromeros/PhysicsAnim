# 🐎 Doma Dinámica — Blueprint Design Document (BDD)
**Para:** Pasante de Unreal Engine  
**Versión:** 2.0 — PRODUCCIÓN (Arquitectura UHS Child Blueprint)  
**Meta:** Implementar el minijuego completo de doma de caballos salvajes usando el **Ultimate Horse System (UHS)** ya integrado en el proyecto Coleo.

> [!IMPORTANT]
> El proyecto ya tiene el caballo y el jinete integrados y funcionando con el **Ultimate Horse System** de Hivemind. Tu trabajo es agregar la lógica de Doma **encima** de ese sistema, NO reemplazarlo. **NUNCA modifiques los Blueprints padres**. Toda tu lógica va en los hijos que se describen en la Sección 2.

> [!CAUTION]
> Si el Lead te pasa los assets, verás los archivos `BP_Horse` y `BP_Rider` originales del UHS, más los archivos propios del proyecto ya integrados. **Solo toca los archivos que esta guía te dice que crees.** Si modificas un parent, contaminas el sistema.

---

## 1. VISIÓN GENERAL DEL MINIJUEGO

**Resumen en 1 línea:** El jugador lanza un lazo, monta un caballo salvaje y debe resistir sus embestidas con inputs de timing hasta domarlo.

**Bucle completo:**
```
Encuentro → Lanzar Lazo → Enganche → Doma (Core) → Éxito/Fallo → Recompensa
```

**Tiempo de cada sesión:** 30 a 90 segundos totales.  
**Plataforma:** Mobile (iOS/Android). Prioridad absoluta en controles táctiles.

---

## 2. ARQUITECTURA DE BLUEPRINTS (CHILD PATTERN)

### La regla de oro: NO tocar los padres

El UHS funciona a través de una **Blueprint Interface** llamada `BPI_HorseRiding`. El `BP_Horse` (padre del caballo) y el `BP_Rider` (padre del jinete) se comunican ÚNICAMENTE a través de esa interfaz. Esto significa que puedes crear hijos de esas clases y añadir lógica sin romper nada del sistema base.

### Blueprints que debes crear (solo estos 5):

| Blueprint a crear | Hereda de | Tipo | Propósito |
|---|---|---|---|
| `BP_ColeHorse_Wild` | ← **El horse BP ya integrado por el Lead** | Actor (Child BP) | Caballo salvaje con lógica de doma |
| `BP_ColeoRider_Taming` | ← **El rider BP ya integrado por el Lead** | Character (Child BP) | Jinete con lógica de lazo y balance |
| `BP_LassoComponent` | ActorComponent | Actor Component | Cálculo de trayectoria y hit del lazo |
| `BP_TamingComponent` | ActorComponent | Actor Component | Core loop: balance, patrones, timing |
| `WBP_TamingHUD` | UserWidget | Widget Blueprint | UI: barra de balance, timer, arrows |

> [!NOTE]
> Para crear un Child Blueprint: click derecho en el BP padre en el Content Browser → **Create Child Blueprint Class**. Dale el nombre exacto de la tabla.

### Cómo añadir lógica en un Child BP sin tocar el padre:

1. Abre tu Child BP (ej. `BP_ColeHorse_Wild`)
2. En el **Event Graph**, haz click derecho → busca `Add Event` → verás los eventos del padre que puedes **Override** (ej. `Event BeginPlay`, `Event Tick`)
3. Al hacer override, siempre añade nodo **`Add Call to Parent Function`** al inicio del evento. Esto ejecuta primero la lógica del padre y luego tu lógica custom.
4. Toda tu lógica de Doma va DESPUÉS del nodo de Parent Call.

```
[Event BeginPlay - Override]
    → [Parent: Begin Play]   ← SIEMPRE lo primero
    → [Tu lógica de spawn de TamingComponent]
    → [Inicializar variables de WildnessLevel]
```

### Relaciones entre Blueprints:
```
BP_ColeHorse_Wild
  ├── tiene componentes: BP_TamingComponent, BP_LassoComponent
  └── se comunica con BP_ColeoRider_Taming vía BPI_HorseRiding

BP_ColeoRider_Taming
  └── recibe eventos de montado/desmontado del BPI_HorseRiding
      └── muestra y controla WBP_TamingHUD
```

---

## 3. STATE MACHINE (ESTADOS DEL MINIJUEGO)

Implementar como un **Enum** llamado `ETamingState`:

```
Idle           → Caballo spawnado, esperando al jugador
Targeting      → Jugador apunta el lazo al caballo
LassoThrown    → El lazo está en el aire
MountedWild    → Jugador montado, caballo resistiendo
Taming         → Conteo de tiempo de doma activo (core loop)
Success        → Superó el tiempo requerido
Fail           → Balance llegó a límite o se cayó
```

**Variable principal (en BP_WildHorse):**
```
CurrentState (ETamingState) — ReplicatedUsing=OnRep_State
```

**Regla crítica:** NINGUNA lógica debe activarse si el estado no corresponde. Usa un `Switch on Enum` al inicio de cada lógica.

---

## 4. VARIABLES DEL SISTEMA

### 4.1 Variables del Caballo (`BP_WildHorse`)
| Variable | Tipo | Default | Descripción |
|---|---|---|---|
| `WildnessLevel` | Integer | 50 | Dificultad del caballo (1-100) |
| `HorseRarity` | Enum `EHorseRarity` | Common | Common / Rare / Epic / Legendary |
| `Temperament` | Enum `ETemperament` | Calm | Define el patrón de comportamiento |
| `TamingDuration` | Float | 10.0 | Segundos que se deben resistir |
| `CurrentState` | ETamingState | Idle | Estado actual |

### 4.2 Variables del TamingComponent (`BP_TamingComponent`)
| Variable | Tipo | Default | Descripción |
|---|---|---|---|
| `BalanceValue` | Float | 0.0 | El balance del jinete (-100 a +100) |
| `TamingProgress` | Float | 0.0 | Tiempo resistido (en segundos) |
| `ErrorTolerance` | Float | 60.0 | Límite antes de caer. Se calcula de WildnessLevel |
| `ReactionWindow` | Float | 1.0 | Ventana de tiempo para inputs de timing |
| `ForceIntensity` | Float | 1.0 | Fuerza con que el caballo empuja |
| `EventInterval` | Float | 2.0 | Tiempo entre eventos del caballo |
| `LastForceDirection` | Vector | (0,0,0) | Dirección del último empujón del caballo |
| `TimeSinceLastEvent` | Float | 0.0 | Timer interno entre eventos |

---

## 5. FÓRMULAS DE DIFICULTAD (COPIAR EXACTO)

Calcula estas variables en `BeginPlay` del `BP_TamingComponent` usando el `WildnessLevel` (1–100) del caballo padre.

> **Nota para el pasante:** En Blueprint, `Lerp(A, B, Alpha)` está disponible en la categoría Math.

```
// Ventana de reacción
ReactionWindow = Lerp(1.2, 0.25, WildnessLevel / 100.0)

// Intensidad de la fuerza del caballo
ForceIntensity = 1.0 + (WildnessLevel * 0.02)

// Intervalo entre eventos
EventInterval = Lerp(2.0, 0.4, WildnessLevel / 100.0)

// Tolerancia antes de caer
ErrorTolerance = Lerp(80.0, 20.0, WildnessLevel / 100.0)

// Tiempo de doma requerido
TamingDuration = Lerp(5.0, 15.0, WildnessLevel / 100.0)
```

### Tabla de referencia por rareza:
| Rareza | WildnessLevel | Duración | % de Éxito Esperado |
|---|---|---|---|
| Common | 1–25 | ~5s | 80–90% |
| Rare | 26–50 | ~7s | 65–75% |
| Epic | 51–75 | ~10s | 40–55% |
| Legendary | 76–100 | ~15s | 20–30% |

---

## 6. SISTEMA DE PATRONES DE COMPORTAMIENTO

**No uses random puro.** Usa tablas de patrones (arrays de structs).

### 6.1 Struct: `FHorseAction`
Crea un struct con estas variables:
```
Direction     (Enum: Left, Right, Jump, Spin, Fake)
Intensity     (Float: 0.1 – 1.0)
Duration      (Float: tiempo que dura el empujón)
IsTimingEvent (Bool: si requiere tap de timing del jugador)
```

### 6.2 Pattern Pools (Arrays de FHorseAction)
Define tres DataTables o variables de array por tier:

**Tier Bajo (Wildness 1–40):**
```
Pattern_A: [Left(0.5)]
Pattern_B: [Right(0.5)]
Pattern_C: [Left(0.6) → Right(0.6)]
```

**Tier Medio (Wildness 41–70):**
```
Pattern_D: [Left(0.7) → Right(0.8) → Jump(timing!)]
Pattern_E: [Right(0.6) → Fake(Left) → Right(0.9)]
```

**Tier Alto (Wildness 71–100):**
```
Pattern_F: [Left → Left → delay 0.3s → Right(1.0) → Jump → Spin]
Pattern_G: [Fake(Right) → Left(1.0) → Fake(Jump) → Spin(1.0)]
```

### 6.3 Selección de patrón:
```
SelectPattern():
  if WildnessLevel <= 40 → RandomFromArray(LowPatterns)
  elif WildnessLevel <= 70 → RandomFromArray(MidPatterns)
  else → RandomFromArray(HighPatterns)
```

---

## 7. LÓGICA TICK-BY-TICK (`BP_TamingComponent`)

### 7.1 Event Tick (ejecuta solo en estado `Taming`)
```
[Event Tick]
  → [Switch on CurrentState]
      ↓ (solo si == Taming)
  → TimeSinceLastEvent += DeltaTime
  → TamingProgress += DeltaTime
  
  → [If TimeSinceLastEvent >= EventInterval]
      → TimeSinceLastEvent = 0
      → SelectAndExecuteNextPattern()
  
  → [ApplyHorseForceToBalance(LastForceDirection, ForceIntensity * DeltaTime)]
  
  → [If Abs(BalanceValue) >= ErrorTolerance]
      → TriggerFail()
  
  → [If TamingProgress >= TamingDuration]
      → TriggerSuccess()
```

### 7.2 ApplyHorseForceToBalance()
```
// El caballo empuja, el jugador debe resistir
BalanceValue += HorseForceDirection * ForceIntensity * DeltaTime * 60

// Recuperación natural (el balance tiende a 0 sin input)
BalanceValue = Lerp(BalanceValue, 0.0, 0.02)  // suavizado natural
```

### 7.3 Input del Jugador (`ApplyPlayerCounter`)
```
// Llamado desde el widget/controller cuando el jugador hace swipe/joystick
PlayerInput (-1.0 a 1.0)

// La respuesta del jugador contrarresta la fuerza del caballo
BalanceValue -= PlayerInput * ReactionBonus * DeltaTime * 60
```

---

## 8. FASE 1: LANZAR EL LAZO (`BP_LassoComponent`)

### Variables:
```
LaunchForce (Float) — se acumula con Hold
IsCharging  (Bool)
MaxForce    (Float) — 1500.0
MinForce    (Float) — 400.0
HitRadius   (Float) — 150.0 (zona de acierto del lazo)
```

### Lógica Hold + Release:
```
[TouchPressed]
  → IsCharging = true
  → StartChargeAnimation()

[TouchHeld - Tick]
  → If IsCharging → LaunchForce = Min(LaunchForce + DeltaTime * 800, MaxForce)
  → UpdateLassoChargeFX(LaunchForce / MaxForce)  // escala visual del lazo

[TouchReleased]
  → IsCharging = false
  → LaunchLasso(LaunchForce)
  → LaunchForce = 0
```

### LaunchLasso():
```
// Proyecta hacia el caballo objetivo
Direction = (TargetHorse.Location - PlayerLocation).Normalized
HitResult = SphereTrace(Start=Player, End=Player + Direction * 1200, Radius=HitRadius)

If Hit.Actor == TargetHorse
  → PlayLassoHitFX()
  → TargetHorse.OnLassoHit()   // Cambia estado a MountedWild
Else
  → PlayLassoMissFX()
  → Reset (puede volver a intentar)
```

---

## 9. FASE DE ÉXITO Y FALLO

### TriggerSuccess():
```
CurrentState = Success
→ StopAllHorsePatterns()
→ PlaySuccessAnimation on Horse
→ PlaySlowMotionEffect(0.3, Duration=1.5)   // cámara lenta épica
→ PlayVictorySound()
→ ShowTamingResult(bSuccess=true, TimeResisted=TamingProgress)
→ GrantReward()
→ AddHorseToPlayerStable(HorseData)
```

### TriggerFail():
```
CurrentState = Fail
→ PlayFallAnimation on Rider
→ PlayHorseFreeFX()
→ CameraShake(Preset=Heavy, Duration=0.5)
→ ShowTamingResult(bSuccess=false, TimeResisted=TamingProgress)
→ OfferRetry()     // "Segunda oportunidad por 10 🟢"
```

---

## 10. HUD DEL MINIJUEGO (`WBP_TamingHUD`)

### Elementos de UI requeridos:

| Elemento | Tipo | Descripción |
|---|---|---|
| `BalanceBar` | Progress Bar | Centrada: 0 = centro, ±100 = extremos. Color: verde→amarillo→rojo |
| `DirectionalArrow` | Image | Flecha que indica hacia dónde empuja el caballo |
| `TimerText` | Text Block | Cuenta regresiva de segundos restantes |
| `WildnessIcon` | Image | Ícono de rareza del caballo |
| `TimingIndicator` | Image | Flash que aparece en eventos de timing (tap preciso) |
| `HeartbeatPulse` | Image/Anim | Pulsa rápido cuando Balance > 70% del límite |

### Binding de la BalanceBar:
```
GetPercent():
  → BalanceValue = TamingComp.BalanceValue
  → ReturnValue = (BalanceValue + 100) / 200.0   // normaliza -100/+100 a 0/1
  
Color:
  → If Abs(BalanceValue) < 40 → Verde
  → If Abs(BalanceValue) < 70 → Amarillo
  → Else → Rojo + Pulsing Animation
```

---

## 11. SISTEMA DE TIMING EVENTS (SKILL CEILING)

Cuando el patrón contiene `IsTimingEvent = true`:

```
OnTimingEventStart():
  → Show TimingIndicator (animación de flash en la UI)
  → Iniciar countdown de (ReactionWindow) segundos
  → EsperarInput del jugador
  
If Jugador toca dentro de ReactionWindow:
  → BalanceValue = Lerp(BalanceValue, 0, 0.7)    // corrección grande
  → Mostrar "¡PERFECTO!" en pantalla (texto efímero)
  → Bonus visual + sonido

Else (perdió la ventana):
  → BalanceValue += HorseForceIntensity * 1.5    // penalidad
  → CameraShake(Preset=Light)
```

---

## 12. SISTEMA DE AYUDA INVISIBLE (Anti-Frustration)

> ⚠️ El jugador **nunca debe saber** que esto existe.

```
// Checar en TriggerFail() antes de mostrar la pantalla de fallo:
If PlayerFailedConsecutively >= 3:
  → WildnessLevel = Max(WildnessLevel - 15, 1)   // reducir dificultad temporalmente
  → RecalcularTodasLasVariables()
  → PlayerFailedConsecutively = 0

// Checar en TriggerSuccess():
  → PlayerFailedConsecutively = 0
```

Guardar `PlayerFailedConsecutively` en el **GameInstance** para que persista entre intentos.

---

## 13. CÁMARA Y EFECTOS

### Cámara durante la doma:
```
Adjuntar SpringArmComponent al WildHorse con offset [X=0, Y=0, Z=300]
Durante Taming:
  → TargetArmLength = 450
  → Añadir CameraShake leve basado en Abs(BalanceValue) / 100
  → Intensidad del shake aumenta progresivamente
```

### En los últimos 3 segundos (Hype Moment):
```
If TamingDuration - TamingProgress <= 3.0
  → Set Global Time Dilation = 0.85   // ligero slow-mo
  → Intensificar música (Cambiar a track de alta intensidad)
  → Aumentar partículas de polvo
  → Heartbeat pulse en HUD
```

### En Success:
```
→ Set Global Time Dilation = 0.3   // 70% slow-mo por 1.5s
→ Camera Zoom-in en el caballo (FOV de 90 → 60)
→ Confetti particles (o polvo dorado para caballos Epic+)
```

---

## 14. AUDIO (REQUERIDO MÍNIMO)

Vincular estos eventos a sonidos:

| Evento | Sonido |
|---|---|
| Lasso cargando | SFX_LassoCharge (loop) |
| Lasso lanzado | SFX_LassoThrow |
| Lasso acertó | SFX_LassoHit |
| Caballo empuja Left | SFX_HorseBuck01 |
| Caballo empuja Right | SFX_HorseBuck02 |
| Caballo salta | SFX_HorseJump |
| Timing event perfecto | SFX_PerfectTap |
| Timing event fallido | SFX_FailThud |
| Jugador cae | SFX_PlayerFall |
| Éxito | SFX_TamingSuccess |
| Latido (balance crítico) | SFX_Heartbeat (loop, se activa > 70%) |

---

## 15. CONEXIÓN CON LA ECONOMÍA DEL JUEGO

Al completar `TriggerSuccess()`, llamar a `GrantReward()`:

```
GrantReward():
  → Calcular calidad de recompensa basada en (WildnessLevel + Rarity)
  → AddHorseToStable(HorseData)       // guarda el caballo ganado
  → Add SoftCurrency(amount)          // oro según dificultad
  → Registrar en Supabase (async):
      POST /rest/v1/inventory (item_id=horse_id)
      UPDATE /rest/v1/players SET soft_currency += amount
```

> 📝 Para el POST a Supabase, usar `GetGameInstance → GetSubsystem(GameServicesSubsystem) → BackendService → (hacer HTTP request)`. Pedir ayuda al lead si necesitas el patrón exacto.

---

## 16. CHECKLIST DE ENTREGA

Marca cada ítem antes de hacer merge:

- [ ] `ETamingState` enum creado  
- [ ] `EHorseRarity` enum creado  
- [ ] `ETemperament` enum creado  
- [ ] `FHorseAction` struct creado  
- [ ] `BP_WildHorse` con StateMachine básico funcionando  
- [ ] `BP_LassoComponent` con Hold/Release y esfera trace  
- [ ] `BP_TamingComponent` con balance, tick, y patrones de tier bajo  
- [ ] `WBP_TamingHUD` con BalanceBar binding correcto  
- [ ] TriggerSuccess y TriggerFail con animaciones placeholder  
- [ ] Timing Events funcionando (flash + ventana de reacción)  
- [ ] Sistema anti-frustración activado  
- [ ] Slow-mo al final (Hype Moment) funcionando  
- [ ] Probado en Mobile Preview con joystick virtual  
- [ ] **Todos los nombres de la Sección 18 verificados** (crítico para merge)

---

## 17. ERRORES COMUNES A EVITAR

| ❌ Error | ✅ Correcto |
|---|---|
| Usar random puro para el caballo | Usar Pattern Pools por tier |
| Hacer el minijuego muy largo | Max 90s. Sé estricto. |
| Poner mucha UI al mismo tiempo | Solo BalanceBar + Timer + DirectionalArrow |
| Olvidar el state machine | Toda lógica dentro de Switch on CurrentState |
| No ajustar para Mobile | TouchInput obligatorio, no solo teclado |
| Hacer ForceIntensity constante | Debe crecer con WildnessLevel usando la fórmula |
| Usar real-time multiplayer | Este sistema es 100% LOCAL |
| **Renombrar variables del Contrato** | **Respetar la Sección 18 al pie de la letra** |

---

## 18. CONTRATO DE INTEGRACIÓN (⚠️ OBLIGATORIO — NO MODIFICAR NOMBRES)

> [!CAUTION]
> Esta sección define el **contrato técnico** entre tu proyecto sandbox y el proyecto principal de Coleo. Si cambias **UN SOLO nombre** de esta lista, la integración fallará. Si necesitas más variables, agrégalas con otros nombres pero **nunca modifiques los que están aquí**.

El lead del proyecto tiene assets reales (modelo 3D del caballo, animaciones, físicas) que **van a reemplazar tus placeholders al hacer el merge**. Para que ese reemplazo sea quirúrgico, cada Blueprint debe exponer estas variables y eventos con **exactamente estos nombres**.

---

### 18.1 `BP_ColeHorse_Wild` (Child del Horse del Lead) — Variables que **tú añades**

> [!NOTE]
> El caballo padre ya tiene sus propias variables del UHS (velocidad, animaciones, física). Las que están aquí son **adicionales** que tú declaras en el Event Graph del hijo.

| Nombre exacto | Tipo | Descripción |
|---|---|---|
| `WildnessLevel` | Integer | Dificultad 1–100. Defínela como variable del Child BP. |
| `HorseRarity` | Enum `EHorseRarity` | Common / Rare / Epic / Legendary |
| `CurrentTamingState` | Enum `ETamingState` | Estado del minijuego (no confundir con estados del caballo del UHS) |
| `TamingDuration` | Float | Segundos a resistir |
| `HorseDisplayName` | FText | Nombre del caballo (ej. "El Diablo") |
| `TamingComp` | BP_TamingComponent Ref | Referencia al componente de doma que añadirás en el hijo |
| `LassoComp` | BP_LassoComponent Ref | Referencia al componente de lazo |

### 18.2 `BP_ColeHorse_Wild` — Event Dispatchers que **tú añades**

> Los Dispatchers son señales. El Rider Child se suscribe a estos para saber qué hace el caballo. Son ADICIONALES a los que ya existen en el padre del UHS.

| Nombre exacto | Parámetros | Cuándo llamarlo |
|---|---|---|
| `OnBuckLeft` | `Intensity (Float)` | Cuando el patrón de doma empuja izquierda |
| `OnBuckRight` | `Intensity (Float)` | Cuando el patrón de doma empuja derecha |
| `OnHorseJump` | ninguno | Cuando el patrón incluye salto |
| `OnHorseSpin` | ninguno | Cuando el patrón incluye giro |
| `OnTamingSuccess` | `HorseRef (self)` | Al superar el tiempo requerido |
| `OnTamingFail` | `TimeResisted (Float)` | Al caer el jugador |

### 18.3 Comunicación con el Rider: Usa la `BPI_HorseRiding` que ya existe

El UHS ya tiene una interfaz `BPI_HorseRiding` implementada en los padres. Para disparar eventos **hacia el Rider** desde el caballo, usa sus funciones de interfaz. Para comunicar eventos propios del minijuego, **suscribe el Rider Child a los Dispatchers de la sección 18.2** desde el `BeginPlay` del Rider.

```
[BP_ColeoRider_Taming — Event BeginPlay Override]
  → [Parent: Begin Play]
  → [Get Mounted Horse] → Cast to BP_ColeHorse_Wild
  → [Bind Event to OnBuckLeft]   → tu función AdjustBalance(Left)
  → [Bind Event to OnBuckRight]  → tu función AdjustBalance(Right)
  → [Bind Event to OnTamingSuccess] → tu función ShowSuccessScreen()
  → [Bind Event to OnTamingFail] → tu función ShowFailScreen()
```

### 18.4 `BP_TamingComponent` — Variables Públicas Obligatorias

| Nombre exacto | Tipo | Descripción |
|---|---|---|
| `BalanceValue` | Float | Balance actual (-100 a 100) |
| `TamingProgress` | Float | Segundos resistidos |
| `ErrorTolerance` | Float | Límite de caída |
| `IsTimingWindowOpen` | Bool | True cuando hay un evento de timing activo |

### 18.5 `BP_TamingComponent` — Funciones Públicas Obligatorias

| Nombre exacto | Input | Descripción |
|---|---|---|
| `ApplyPlayerInput` | `InputValue (Float -1 a 1)` | Llamar cada frame con el valor del joystick/swipe |
| `StartTaming` | ninguno | Inicia el core loop. Llamar cuando el UHS dispara el evento de montado. |
| `StopTaming` | ninguno | Para el loop. Llamar en Success o Fail. |

> [!IMPORTANT]
> `StartTaming` debe ser llamado desde el evento de **montado del jinete** que ya provee el UHS. Busca el evento en `BP_ColeoRider_Taming` que se dispara cuando el personaje monta el caballo (será algo como `On_Rider_Mounted` o el evento de interfaz equivalente del BPI_HorseRiding) y desde ahí llama a `StartTaming`.

### 18.6 Uso de Animaciones del Sistema (UHS)

> [!TIP]
> Dado que el paquete **Ultimate Horse System (UHS)** ya cuenta con todas las animaciones necesarias (Bucks, Rears, Jumps, etc.), **NO es necesario crear animaciones nuevas**.

1. **Usa el Skeletal Mesh** proporcionado en el FBX como base para tu `SkeletalMeshComp`.
2. **Identifica las animaciones de resistencia** (Bucking, Rearing, Shaking) en el Asset Browser. 
3. **Mapea tus eventos** (`OnBuckLeft`, `OnBuckRight`, etc.) a las animaciones reales del paquete.
4. Si necesitas probar lógica de balance sin ver la animación completa, puedes usar el `AnimBlueprint` temporal, pero la meta es que el sistema use las animaciones finales desde el día 1.

### 18.7 Carpeta de Asset Obligatoria

```
Content/
  Coleo/
    Minigames/
      Taming/
        BP_ColeHorse_Wild        ← Child del horse del Lead
        BP_ColeoRider_Taming     ← Child del rider del Lead
        BP_LassoComponent
        BP_TamingComponent
        WBP_TamingHUD
        Enums/
          ETamingState
          EHorseRarity
          ETemperament
        Structs/
          FHorseAction
```

### 18.8 Proceso de Merge (Lo hace el Lead, no tú)

```
1. Lead recibe tu proyecto sandbox
2. Selecciona Content/Coleo/Minigames/Taming/ completo
3. Click derecho → Asset Actions → Migrate → Proyecto Coleo principal
4. Los Child BPs ya referencian los padres correctos del proyecto REAL
   porque tú usaste el modelo 3D del caballo que el Lead te pasó
5. ✅ La integración es instantánea porque:
      - El Child Horse hereda las animaciones reales del padre
      - El BPI_HorseRiding ya conecta caballo y jinete
      - Solo los Dispatchers del minijuego son nuevos
```

### 18.9 Lo que el Lead te debe pasar para trabajar

Pide al Lead exactamente estos 2 archivos por correo/Drive:

1. **El FBX del caballo** con todas las animaciones (el Lead lo exporta con `Asset Actions → Export` incluyendo animations).
2. **El nombre exacto** de sus Blueprints de horse y rider ya integrados (ej. `BP_ColeoHorse`, `BP_ColeoRider`) para que puedas hacer el Child BP correctamente.

> [!NOTE]
> No necesitas los proyectos originales. Solo el FBX para poder crear el Child y ver cómo se ve, y los nombres para hacer la referencia de herencia correctamente.

---

*Documento generado por el equipo de Arquitectura de Coleo — Arkangel Games.*


