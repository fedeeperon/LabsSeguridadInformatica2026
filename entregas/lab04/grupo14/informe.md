# Informe — Laboratorio 04 · Marcos normativos y gestión

**Grupo:** 14 · **Integrantes:** Fede Peron — @fedeeperon · Laureano Gomez Redondo — @Laureano-GR · Martín Cesano — @____ · Martín Gaido — @martingaidoo

## 0. Declaración de uso de IA

- **Herramienta:** Claude (Anthropic), a través de Claude Code.
- **Para qué:** proponer la implementación de `ale`, `roi_control` y `priorizar`;
  armar un primer borrador del mapeo debilidad → subcategoría NIST CSF, de los
  valores de SLE/ARO de `riesgos.json` y del mini-research; revisar la redacción.
- **Qué partes:** `src/riesgo.py` (los tres TODO), `riesgos.json`, `research.md`
  y las secciones 1 y 2 de este informe.
- **Cómo lo verificamos:**
  - Corrimos `python3 src/verificar.py` (3 OK) y los ejemplos del enunciado
    (`ale` → 20000.00, `roi` → 0.875).
  - Cada ID de subcategoría citado (PR.AA-03, PR.DS-11, etc.) se buscó y se
    comparó con el texto oficial de NIST CSWP 29 (CSF 2.0), para no citar
    controles que no existen.
  - Las citas del research se compararon con NIST SP 800-30 Rev. 1, §2.3.2.
  - Los valores de SLE/ARO son estimaciones nuestras para un escenario ficticio.
    Discutimos los supuestos de cada uno (ver Parte B).

## 1. Parte A — Marco aplicado

### A.1 — Marco elegido: NIST Cybersecurity Framework 2.0

Elegimos **NIST CSF 2.0** (NIST CSWP 29, 2024) por tres razones:

1. **Está pensado para cualquier tamaño de organización y no es certificable.**
   PhantomCorp parece una pyme sin área de seguridad. Lo que necesita primero es
   un diagnóstico de brechas y un plan, no una auditoría de certificación como
   la de ISO/IEC 27001.
2. **Está organizado por resultados** (*outcomes*), en seis funciones: Govern,
   Identify, Protect, Detect, Respond y Recover. Las debilidades del escenario se
   reparten justo entre esas funciones. Así se ve de un vistazo que PhantomCorp
   casi no tiene nada en Respond ni en Recover.
3. **Incorpora la gestión de riesgo dentro del marco:** GV.RM-04 (opciones de
   respuesta al riesgo), GV.RM-06 (método estandarizado para calcular y
   priorizar) e ID.RA-06 (elegir y seguir respuestas). Eso conecta directo con la
   Parte B.

> Nota: el enunciado habla de "cinco funciones", que es el CSF 1.1. La versión
> 2.0 agregó **Govern (GV)** como sexta función. Usamos los IDs de la 2.0.

### A.2 — Cinco debilidades → funciones y subcategorías del marco

| # | Debilidad del escenario | Función | Subcategoría CSF 2.0 (texto oficial resumido) | Control concreto que proponemos |
|---|---|---|---|---|
| D1 | Empleados con acceso remoto **sin MFA** | Protect | **PR.AA-03**: usuarios, servicios y hardware están autenticados · **PR.IR-01**: redes y entornos protegidos de acceso lógico no autorizado | VPN con MFA obligatorio (app TOTP o llave FIDO2); cerrar todo acceso remoto directo (RDP/SSH expuestos) |
| D2 | **Sin política de contraseñas** | Govern + Protect | **GV.PO-01**: política de gestión de riesgos de ciberseguridad establecida, comunicada y aplicada · **PR.AA-01**: identidades y credenciales gestionadas por la organización | Política escrita (largo mínimo, no reutilizar, bloqueo ante filtradas), gestor de contraseñas corporativo, baja de cuentas al irse un empleado |
| D3 | **Sin plan de respuesta a incidentes** | Identify + Respond | **ID.IM-04**: planes de respuesta a incidentes establecidos, comunicados, mantenidos y mejorados · **RS.MA-01**: el plan se ejecuta en coordinación con terceros relevantes | Plan escrito con roles, contactos y pasos; ejercicio de mesa una vez por año; contrato de respuesta con un tercero |
| D4 | **Backups en un disco en la oficina** | Protect + Recover | **PR.DS-11**: backups creados, protegidos, mantenidos y probados · **RC.RP-03**: se verifica la integridad de los backups antes de restaurar | Esquema 3-2-1: copia fuera del sitio e inmutable (nube con versionado/bloqueo) y prueba de restauración trimestral |
| D5 | **Datos de clientes (nombres, DNI, tarjetas) en el servidor web público** | Identify + Protect | **ID.AM-07**: inventario de datos y metadatos para tipos de datos designados · **PR.DS-01**: confidencialidad, integridad y disponibilidad de los datos en reposo protegidas | Inventario de qué datos sensibles hay y dónde; no almacenar tarjetas (tokenizar con un procesador de pagos); cifrar los DNI en reposo y sacar la base del servidor expuesto |

### A.3 — Respuesta al riesgo para cada debilidad

La respuesta se elige por el costo del control frente a la pérdida que evita
(ID.RA-06 / GV.RM-04), no por reflejo. Por eso no todo es "mitigar":

| # | Respuesta | Justificación |
|---|---|---|
| D1 | **Mitigar** | El MFA es barato (pocos dólares por usuario al mes) y corta el vector de entrada más común del ransomware, el riesgo #2 de la Parte B (ALE USD 45.000). No hay forma razonable de evitarlo, porque el negocio necesita acceso remoto. Tampoco conviene transferirlo: el seguro igual exigiría MFA. |
| D2 | **Mitigar** | Una política y un gestor de contraseñas cuestan casi nada frente al ALE de USD 30.000 por toma de cuentas. Además, el MFA de D1 baja el impacto de una contraseña filtrada. |
| D3 | **Transferir** (en parte) + mitigar lo mínimo | Una pyme no puede sostener un equipo de respuesta propio. Contratamos un **ciberseguro que incluya un equipo de respuesta a incidentes**: así transferimos el costo financiero y conseguimos expertos. El contrato tiene que dejar los requisitos por escrito (GV.SC-05). Lo que no se transfiere es tener un plan propio (ID.IM-04) que diga a quién llamar en la primera hora: eso lo hacemos nosotros. |
| D4 | **Mitigar** | Una copia fuera del sitio cuesta poco y es lo que convierte un ransomware en "restauramos en un día" en vez de "perdimos todo". Aceptar este riesgo sería un error, aunque su ALE solo sea bajo (ver B.1). |
| D5 | **Evitar** (tarjetas) + mitigar (DNI) | La forma más segura de proteger números de tarjeta es **no guardarlos**: los pagos pasan por un procesador certificado PCI DSS y PhantomCorp solo guarda un token. Así eliminamos la actividad riesgosa, no solo la reducimos. Los nombres y DNI sí hacen falta para el negocio: esos se mitigan cifrándolos y sacándolos del servidor público. |

La cuarta respuesta posible, **aceptar**, la aplicamos a la caída del sitio web
(ver B.3), donde el control cuesta más que la pérdida que evita.

## 2. Parte B — Riesgo cuantitativo

*(en curso)*

## 3. Anexo — tu riesgos.json

*(en curso)*
