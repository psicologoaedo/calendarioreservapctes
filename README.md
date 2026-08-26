[INSTRUCCIONES.md](https://github.com/user-attachments/files/31478341/INSTRUCCIONES.md)
# calendarioreservapctes# Agenda de pacientes — instalación y uso

Tres piezas:

| Pieza | Dónde vive | Qué hace |
|---|---|---|
| `index.html` | GitHub Pages | Lo que ven el paciente y el administrador |
| `Codigo.gs` | Google Apps Script | Guarda, envía correos y crea los Meet |
| Hoja `Pacientes` | Google Sheets | La base de datos |

> **Ojo con la cuenta.** Todo (Apps Script + Sheet + Calendar) tiene que estar en la
> cuenta Google del **psicólogo**, porque las sesiones se crean en *su* calendario y
> los Meet se generan con *su* cuenta. Para la prueba lo montas en la tuya y después
> se repite el mismo procedimiento en la de él.

---

## 1. La hoja de cálculo

Ya existe: `11YC2OclBaR5nH_8ThjRmOE8h8ntZjy3t8zbAIhrKZ6c`, pestaña `Pacientes`.

No hay que preparar nada a mano: el script arma los encabezados solo. Estas son las
columnas que va a crear:

`id · fecha · inicio · fin · publico · modalidad · estado · nombre · email · telefono · notas · meet · eventoId · creado · actualizado`

- **publico**: `Todos`, `Adulto`, `NNA`
- **modalidad**: `Online`, `Presencial`, `Ambas` (el paciente elige al reservar)
- **estado**: `disponible`, `reservado`, `confirmado`, `bloqueado`

La hoja **no se comparte con nadie**. Déjala privada: el sitio nunca la lee directo,
siempre pasa por el script.

---

## 2. Apps Script

1. Abre la planilla → **Extensiones → Apps Script**.
2. Borra lo que venga y pega todo `Codigo.gs`.
3. **⚙ Configuración del proyecto** → Zona horaria → `(GMT-04:00) Santiago`.
4. Panel izquierdo → **Servicios** → **+** → busca **Google Calendar API** → *Agregar*.
   (El identificador tiene que quedar como `Calendar`.)
5. Arriba, en `CONFIG`, ajusta:
   - `PROFESIONAL` — el nombre que firma los correos
   - `EMAIL_AVISOS` — dónde llegan los avisos de reserva nueva
   - `DIRECCION`, `TELEFONO`
   - `ADMINS` — los correos que pueden administrar (hoy: `...`)
   - `DURACION_MIN` (50), `MESES_ADELANTE` (3), `RETENCION_DIAS` (15)
6. Selecciona la función **`instalar`** y dale ▶ *Ejecutar*.
   Google va a pedir permisos: *Configuración avanzada → Ir a (proyecto) → Permitir*.
   Sale la advertencia de "app no verificada" porque el proyecto es tuyo — es normal.

Con eso quedan creados: los encabezados, la pestaña `Historial`, la semana tipo
(la de tu imagen) y las horas de los próximos 3 meses. También quedan andando dos
tareas automáticas: mantención a las 3 AM y recordatorios a las 9 AM.

---

## 3. Publicar la API

**Implementar → Nueva implementación → ⚙ → Aplicación web**

- Descripción: `agenda v1`
- **Ejecutar como: Yo**
- **Quién tiene acceso: Cualquier usuario** ← imprescindible, si no, los pacientes no pueden reservar

Copia la URL que termina en `/exec`.

> Cada vez que edites el código tienes que **volver a implementar** (Implementar →
> Administrar implementaciones → ✏️ → Versión: Nueva). Si no, sigue corriendo la versión vieja.

---

## 4. Conectar el sitio

En `index.html`, arriba del todo:

```js
window.CONFIG = {
  API       : 'https://script.google.com/macros/s/AKfy…/exec',   // ← pega acá
  CLIENT_ID : '',
  NOMBRE    : 'Nombre de la consulta',
  BAJADA    : 'Elige el día y la hora que te acomode…'
};
```

Es lo único del archivo que se toca.

---

## 5. Subirlo a GitHub

1. Repositorio nuevo, público.
2. Sube `index.html` (`Codigo.gs` e `INSTRUCCIONES.md` déjalos ahí de respaldo, no molestan).
3. **Settings → Pages → Source: Deploy from a branch → main / (root)** → *Save*.
4. En un par de minutos queda en `https://TUUSUARIO.github.io/REPO/`.

- Sitio del paciente: `https://TUUSUARIO.github.io/REPO/`
- Panel del psicólogo: `https://TUUSUARIO.github.io/REPO/#admin`

---

## 6. Ingreso del administrador

Hay dos formas. Elige una.

### A) Con la cuenta de Google (recomendada)

1. [console.cloud.google.com](https://console.cloud.google.com) → mismo proyecto que
   creó el Apps Script (aparece en Configuración del proyecto → *Proyecto de Google Cloud*).
2. **Pantalla de consentimiento de OAuth** → Externo → completa nombre y correo de soporte.
   En *Usuarios de prueba* agrega el correo del psicólogo.
3. **Credenciales → Crear credenciales → ID de cliente de OAuth → Aplicación web**
   - Orígenes de JavaScript autorizados: `https://TUUSUARIO.github.io`
4. Copia el ID (`…apps.googleusercontent.com`) y pégalo en dos lugares:
   - `index.html` → `CONFIG.CLIENT_ID`
   - `Codigo.gs` → `CONFIG.CLIENT_ID` (y vuelve a implementar)

Ese ID es público, no es un secreto. La seguridad está en que el script solo deja
pasar los correos de `CONFIG.ADMINS`.

### B) Con una clave numérica (más simple, menos seguro)

En Apps Script, edita la función `definirPin()`, pon la clave y ejecútala una vez.
Deja `CLIENT_ID` vacío en `index.html` y aparecerá el campo de clave.

Sirve para probar rápido. Para producción, usa la opción A.

---

## 7. Cómo lo usa el psicólogo

Entra a `…/#admin` y tiene cuatro pestañas:

**Reservas** — quién viene, cuándo, con qué teléfono y qué escribió al reservar.
Botones para *Confirmar* (le llega correo al paciente) y *Cancelar* (pide un motivo,
borra el evento del calendario, avisa al paciente y deja la hora libre otra vez).

**Horas de la semana** — el mapa de la semana. Toca cualquier hora para dejarla
disponible, bloquearla o eliminarla. También hay *Bloquear todo el día* (para un
feriado o un imprevisto) y un formulario para agregar una hora suelta fuera del horario normal.

**Semana tipo** — los bloques que se repiten. Es la traducción de tu planilla de colores:

| Día | Horario | Para | Modalidad |
|---|---|---|---|
| Lunes | 14:00–17:00 | Niños y adolescentes | Presencial |
| Lunes | 18:00–21:00 | Cualquiera | Online |
| Martes | 09:00–12:00 | Cualquiera | Online o presencial |
| Martes | 14:00–18:00 | Cualquiera | Online |
| Miércoles | 14:00–17:00 | Niños y adolescentes | Presencial |
| Miércoles | 18:00–21:00 | Adultos | Presencial |
| Jueves | 09:00–12:00 | Cualquiera | Online o presencial |
| Jueves | 14:00–18:00 | Cualquiera | Online |
| Viernes | 09:00–12:00 | Adultos | Presencial |
| Viernes | 13:00–18:00 | Niños y adolescentes | Presencial |

Si cambia el horario, edita ahí y guarda: se crean las horas nuevas hasta 3 meses
adelante y **no se toca ninguna hora ya reservada**.

**Mantención** — cuántas horas hay, y los botones para forzar la limpieza o la
generación de horas (que igual corren solas cada madrugada).

---

## 8. Qué pasa cuando alguien reserva

1. La fila pasa a `reservado` con los datos del paciente.
2. Se crea el evento en el calendario del psicólogo, con el paciente como invitado.
   Si la sesión es online, el evento nace con link de Meet.
3. Al paciente le llegan **dos** correos: el del sistema (con el link y el código) y
   la invitación de Google Calendar.
4. Al psicólogo le llega el aviso con nombre, correo, teléfono y motivo de consulta.
5. El día anterior a las 9 AM sale el recordatorio automático.

Si dos personas hacen clic en la misma hora al mismo tiempo, gana la primera: la
segunda ve *"Alguien tomó esa hora recién"*. Está resuelto con bloqueo del script.

---

## 9. La limpieza automática

Cada madrugada, `mantenimientoDiario()`:

- borra de `Pacientes` las horas anteriores a hace 15 días (`RETENCION_DIAS`);
- antes de borrar, copia a la pestaña `Historial` **solo** las que tuvieron paciente
  (las horas libres que nadie tomó se van sin dejar rastro, que es lo que ocupa espacio);
- crea las horas que falten para completar los 3 meses hacia adelante.

Con este horario la hoja se estabiliza en unas 300 filas. No es un volumen que preocupe.

---

## 10. Límites que conviene tener claros

**Correos.** Una cuenta gratuita de Gmail permite ~100 correos al día desde Apps Script
(una cuenta Workspace, 1.500). Cada reserva gasta 2. No vas a llegar al techo, pero si
algún día pasa, los correos dejan de salir en silencio hasta el otro día.

**Google Meet en cuenta gratuita.** Sí crea los links solo. Con dos participantes la
sesión no tiene límite práctico (24 h). Si entra un tercero, la llamada se corta a los
60 minutos. Para terapia 1 a 1 no es problema; para sesión familiar o de pareja, sí.

**La sala de espera.** En cuentas gratuitas, el invitado externo queda "tocando la
puerta" hasta que el anfitrión lo admite, y si el psicólogo no ha entrado, el paciente
no puede entrar solo. El correo ya se lo advierte, pero conviene que él entre primero.

**Quién puede reservar.** El sitio es público: cualquiera con el link puede tomar una
hora. Hay tope de 3 horas activas por correo y un campo trampa contra bots, pero no hay
verificación de identidad. Si aparece spam, lo siguiente sería pedir confirmación por
correo antes de bloquear la hora.

**Datos sensibles.** El campo "motivo de consulta" puede terminar con información
clínica en una planilla. Yo lo dejaría explícitamente como campo de contacto ("¿algo que
quieras contar antes?" es voluntario y breve), y la ficha clínica en otra parte. Con la
Ley 21.719 entrando en vigencia, es mejor que la planilla tenga lo mínimo.

**Lo que este sistema no hace:** cobrar, mandar WhatsApp o SMS, ficha clínica,
recurrencia automática de pacientes fijos, ni lista de espera. Todo eso se puede agregar
después; nada de esto lo bloquea.
