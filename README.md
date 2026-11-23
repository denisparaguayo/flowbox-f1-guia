# FlowBox F1: modo depurador y limpieza

Guía práctica para dos tareas concretas en una **FlowBox F1 (HP40A2, SkyworthDigital)**:
activar el acceso por ADB y dejar el sistema limpio de apps innecesarias.

- **Qué cubre:** habilitar la depuración desde la caja y conectarse desde el PC; desactivar, suspender o
  quitar paquetes; verificar el resultado y restaurar los cambios.
- **Qué no cubre:** instalación de apps, temas, apariencia ni ajustes de red.
- **Tiempo estimado:** 10 a 15 minutos, más un reinicio para verificar.

> **Quick summary (English):** This guide covers only two tasks on a FlowBox F1: enabling ADB debugging
> and cleaning up apps. On the TV, enable *Settings → Developer options → USB debugging*. On the PC, run
> `adb connect <TV_IP>:5555`. Then use `pm disable-user` for regular packages, `pm suspend --user 0` for
> packages blocked by the firmware whitelist, and `pm uninstall -k --user 0` only for apps you are sure
> you do not want. The detailed steps below are in Spanish.

| Dato | Valor |
|---|---|
| Equipo | FlowBox F1 / HP40A2 (SkyworthDigital) |
| Sistema | Android TV 12 (API 31) |
| Firmware de referencia | `STTC.220815.001.C7.5.1_20260825 release-keys` (parche 2026-07-05) |
| ADB | `<TV_IP>:5555` |

Reemplaza los marcadores en tu copia:

| Marcador | Qué colocar |
|---|---|
| `<ADB>` | Ruta a `adb.exe`, por ejemplo `C:\...\platform-tools\adb.exe` |
| `<TV_IP>` | Dirección IP de la caja en tu red |

```powershell
$adb = '<ADB>'
```

---

## Antes de empezar

Necesitas:

- La caja encendida y conectada a la misma red que el PC.
- El control remoto (para aceptar los avisos en pantalla).
- Android `platform-tools` instalados en el PC.
- La IP de la caja. Si no la sabes, consúltala en *Ajustes → Red*.

### Tres formas de quitar una app

Elegí la correcta antes de ejecutar nada:

| Acción | Comando | ¿Se puede deshacer? | Cuándo usarla |
|---|---|---|---|
| Desactivar | `pm disable-user --user 0 <paquete>` | Sí: `pm enable <paquete>` | Ocultar una app sin borrarla |
| Suspender | `pm suspend --user 0 <paquete>` | Sí: `pm unsuspend --user 0 <paquete>` | Cuando el firmware bloquea `disable-user`; sobrevive al reinicio |
| Desinstalar | `pm uninstall -k --user 0 <paquete>` | **No**: hay que reinstalar el APK | Solo si estás seguro de no querer esa app |

> **Advertencia:** no desactives paquetes del núcleo de Android (`com.android.*`,
> `com.google.android.gms`, `com.google.android.gsf`, `com.android.tv.settings`). Una caja que pierde
> uno de esos puede dejar de arrancar.

### Glosario rápido

- **ADB:** herramienta de Android para enviar comandos a la caja desde el PC.
- **pm:** el administrador de paquetes de Android; `pm disable-user`, `pm suspend` y `pm uninstall` son
  sus comandos para retirar apps.
- **Whitelist:** lista del fabricante que prohíbe desactivar ciertos paquetes del operador. Si intentas
  desactivarlos, aparece `Info: <paquete> is in whitelist, there is no permission to excute.`
- **Launcher:** la pantalla de inicio. El launcher de stock puede volver a tomar el Home si se reactiva.

---

## Paso 1: Activar el modo depurador

### 1.1 En la caja

1. Abre *Ajustes → Preferencias del sistema → Opciones para desarrollador*.
2. Si no aparecen las opciones, entra en *Información* y toca repetidamente el número de compilación.
3. Activa **Depuración por USB**.
4. Acepta el aviso que aparece en la pantalla de la caja.

### 1.2 En el PC

```powershell
$adb = '<ADB>'
& $adb connect <TV_IP>:5555
& $adb devices
& $adb shell settings get global adb_enabled
```

**Resultado esperado:** `adb devices` lista `<TV_IP>:5555` con estado `device`, y `adb_enabled`
devuelve `1`.

### 1.3 Si no conecta

- Repite el paso 1.1 y acepta el aviso en pantalla.
- Comprueba que la IP sea correcta y que la caja esté en la misma red.
- Si la conexión se pierde más adelante, reactiva la depuración desde la caja; no insistas desde el PC.

> Nota: al terminar la sesión, el ADB de red se cerró por seguridad. Habrá que reactivarlo cada vez
> que se necesite.

---

## Paso 2: Dejar la caja limpia

Resumen de los tres grupos de paquetes:

| Grupo | Cantidad | Mecanismo | Resultado final |
|---|---|---|---|
| Desactivados | 25 | `pm disable-user` | 24 se reactivaron para probarlos; solo queda desactivado el launcher de stock |
| Suspendidos | 9 | `pm suspend` | Permanecen suspendidos después del reinicio |
| Desinstalados | 12 | `pm uninstall` | Opcional: requiere reinstalar el APK para volver atrás |

### 2.1 Desactivar los 25 paquetes

Estos paquetes se desactivaron primero para probar el efecto y después se reactivaron. Al final solo
se mantuvo desactivado `com.google.android.tvlauncher`, porque al reactivarse volvía a robar el Home.

```powershell
$disable = @(
    'com.google.android.tvlauncher'
    'com.amazon.amazonvideo.livingroom'
    'com.disney.disneyplus'
    'com.netflix.ninja'
    'com.google.android.youtube.tvmusic'
    'com.spotify.tv.android'
    'com.tiktok.tv'
    'tv.twitch.android.app'
    'py.tv.tigosports'
    'laola1.wrc'
    'live.playerpro'
    'com.worldtv2.android.exoplayer2.tv2'
    'com.magmaplayer'
    'com.dpsteam.filmplus'
    'com.mcdstore.playtv'
    'com.noobpy.tvpy'
    'com.omni.sci'
    'com.formulaone.production'
    'com.xtv.x1x'
    'com.harimurti.tv'
    'com.google.android.tvrecommendations'
    'com.google.android.backdrop'
    'com.google.android.play.games'
    'com.lbe.parallel.intl'
    'de.eye_interactive.atvl.parallelspace'
)

foreach ($p in $disable) { & $adb shell "pm disable-user --user 0 $p" }
```

Para reactivar todos y comprobar qué ocurre:

```powershell
foreach ($p in $disable) { & $adb shell "pm enable $p" }
```

Si todo funciona, deja desactivado solamente el launcher de stock:

```powershell
& $adb shell 'pm disable-user --user 0 com.google.android.tvlauncher'
```

### 2.2 Suspender los 9 paquetes de la whitelist

Estos rechazan `pm disable-user` por la whitelist del fabricante. La suspensión es reversible y
sobrevive al reinicio.

```powershell
$suspend = @(
    'ar.com.flow.provisioncheck'
    'ar.com.flow.radiosymusica'
    'com.skyworth.android.tr069'
    'com.skyworth.xmppservice'
    'com.skyworth.skylogcatservice'
    'com.skyworthdigital.diagnose'
    'com.sdt.android.tr369'
    'com.sdt.producttest'
    'ar.com.flow.androidtv_stb'
)

foreach ($p in $suspend) { & $adb shell "pm suspend --user 0 $p" }
```

Para revertir:

```powershell
foreach ($p in $suspend) { & $adb shell "pm unsuspend --user 0 $p" }
```

### 2.3 Desinstalar las 12 apps (opcional)

> **Advertencia:** este paso no se puede deshacer desde la lista; para volver hay que reinstalar el APK.
> Hazlo solo si estás seguro.

Las 12 devolvieron `Success`, incluidas las tres que eran del sistema: aceptaron
`pm uninstall -k --user 0` para el usuario 0.

| Nombre | Paquete | Origen |
|---|---|---|
| TV Abierta PY+ | `com.noobpy.tvpy` | usuario |
| TVEspaña+ | `com.google.android.exoplayer2.tvespana` | usuario |
| WorldTV2 | `com.worldtv2.android.exoplayer2.tv2` | usuario |
| Teleplay PRO+ | `com.harimurti.tv` | usuario |
| Spotify | `com.spotify.tv.android` | usuario |
| Rally TV | `laola1.wrc` | usuario |
| Play TV+ | `com.mcdstore.playtv` | usuario |
| Paramount+ | `com.cbs.ca` | `/system/app/paramount` |
| Magma Player+ | `com.magmaplayer` | usuario |
| HBOMax | `com.wbd.stream` | `/system/app/max` |
| F1 TV | `com.formulaone.production` | usuario |
| Disney+ | `com.disney.disneyplus` | `/vendor/app/Desney` |

Comando para cada paquete:

```powershell
& $adb shell 'pm uninstall -k --user 0 <paquete>'
```

---

## Paso 3: Verificación

Ejecuta estos comandos y compara con el resultado esperado:

| Comando | Resultado esperado |
|---|---|
| `& $adb devices` | La caja aparece como `device` |
| `& $adb shell settings get global adb_enabled` | `1` |
| `& $adb shell getprop sys.boot_completed` | `1` |
| `& $adb shell pm list packages -3` | Solo los paquetes de usuario que decidiste conservar |
| `& $adb shell 'dumpsys activity activities \| grep mResumedActivity'` | El Home abre en el launcher esperado |

Reinicia la caja y comprueba otra vez: los 9 paquetes suspendidos deben seguir suspendidos y el ADB
debe volver a conectar.

---

## Paso 4: Restaurar

El script `restore-flowbox-f1.ps1` quita las suspensiones, reactiva los 25 paquetes desactivados y
devuelve el Home al launcher de stock:

```powershell
powershell -ExecutionPolicy Bypass -File .\restore-flowbox-f1.ps1
```

> **Importante:** el script refleja el estado original, no el final. Las 25 apps ya se reactivaron,
> las 12 se desinstalaron y el Home se dejó en el launcher limpio. Revísalo antes de ejecutarlo tal cual.
> El launcher instalado no se borra: para quitarlo, descomenta su última línea `pm uninstall`.

---

## Problemas comunes

| Síntoma | Solución |
|---|---|
| ADB no conecta | Reactiva la depuración desde la caja y acepta el aviso en pantalla |
| `Info: <paquete> is in whitelist...` | Usa `pm suspend --user 0` en lugar de `pm disable-user` |
| Una actualización OTA reactiva apps | Vuelve a ejecutar los comandos de los grupos 2.1 y 2.2 |
| Una app del sistema parece inamovible | Prueba `pm uninstall -k --user 0`: algunas sí aceptan |

No se modificaron Private DNS, OTA ni Chromecast.

---

## Estado final de referencia

Después de este procedimiento, la caja quedó con **27 apps visibles**, **14 paquetes de usuario** y
aproximadamente **6,7 GB libres** en `/data`. El Home abría en `com.gathin.tvhome/.MainActivity` y los
9 paquetes del grupo 2.2 seguían suspendidos tras el reinicio.

Estos números son una referencia, no un objetivo: dependen de qué apps decidas conservar.
