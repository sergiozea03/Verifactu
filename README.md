# 🇪🇸 Sistema VERI*FACTU (AEAT)

Implementación y documentación técnica para la integración del sistema **VERI*FACTU** según el **Real Decreto 1007/2023**, que regula los **Sistemas Informáticos de Facturación (SIF)** en España.

---

## 📘 1. Descripción general

VERI*FACTU es un sistema desarrollado por la **Agencia Estatal de Administración Tributaria (AEAT)** que permite a los contribuyentes generar facturas verificables y enviar automáticamente los registros de facturación a la AEAT.

Su objetivo principal es garantizar la **integridad, trazabilidad e inalterabilidad** de los datos de facturación.

---

## 🧩 2. Tipos de registros

El sistema gestiona tres tipos de registros:

- **Registro de Alta de Factura**
- **Registro de Anulación**
- **Registro de Evento**

Cada registro genera una **huella criptográfica (hash)** que asegura la autenticidad y el encadenamiento entre facturas.

---

## ⚙️ 3. Especificaciones técnicas

| Elemento | Valor |
|-----------|--------|
| **Algoritmo de hash** | SHA-256 |
| **Codificación** | UTF-8 |
| **Salida** | Hexadecimal (64 caracteres, mayúsculas) |
| **Formato de concatenación** | `campo1=valor1&campo2=valor2&...` |
| **Campo de salida XML** | `<Huella>` o `<HuellaEvento>` |
| **Encadenamiento** | Cada registro incluye la huella del anterior |

---

## 🔐 4. Certificados digitales

El envío de registros requiere **autenticación mediante certificado digital**:

- Certificado del **obligado tributario**
- Certificado del **desarrollador**, si actúa como **colaborador social (tipo 017)** o tiene **apoderamiento**

📧 Contacto AEAT: `comunicacion.sepri@correo.aeat.es`  
📘 Normativa: Orden HAC/1398/2003, Resolución 18/12/2024, Reglamento 1065/2007

---

## 💾 5. Archivos técnicos (AEAT)

| Archivo | Descripción | Enlace |
|----------|-------------|--------|
| **WSDL VERI*FACTU** | Servicio web principal para el envío de registros | [SistemaFacturacion.wsdl](https://prewww2.aeat.es/static_files/common/internet/dep/aplicaciones/es/aeat/tikeV1.0/cont/ws/SistemaFacturacion.wsdl) |
| **XSD SuministroLR** | Define estructura del registro de facturación | Incluido en el WSDL |
| **XSD SuministroInformacion** | Define tipos de datos y validaciones | Incluido en el WSDL |
| **Manual técnico AEAT** | Guía de integración y mensajes SOAP | [Información técnica AEAT](https://sede.agenciatributaria.gob.es/Sede/iva/sistemas-informaticos-facturacion-verifactu/informacion-tecnica.html) |

---

## 🧮 6. Ejemplo de generación de huella (SHA-256)

```python
import hashlib

def generar_huella(cadena):
    return hashlib.sha256(cadena.encode('utf-8')).hexdigest().upper()

cadena = "IDEmisorFactura=89890001K&NumSerieFactura=12345678/G33&FechaExpedicionFactura=2025-11-06&Huella="
print(generar_huella(cadena))
# Verifactu
Contenido Verifactu

🌐 7. Entornos disponibles
Entorno	Descripción	URL
Pruebas (Sandbox)	Permite testear la integración sin utilizar certificados de empresa; útil para desarrollo y validación funcional.	https://prewww1.aeat.es/wlpl/TIKE-CONT/ws/SistemaFacturacion/VerifactuSOAP
Pruebas (WSDL / XSD preprod)	WSDL y XSDs publicados en preproducción para descargar esquemas y generar clientes SOAP.	https://prewww2.aeat.es/static_files/common/internet/dep/aplicaciones/es/aeat/tikeV1.0/cont/ws/SistemaFacturacion.wsdl
Producción	Entorno real para envíos oficiales. Requiere mTLS y certificado válido.	https://www1.aeat.es/wlpl/TIKE-CONT/ws/SistemaFacturacion/VerifactuSOAP

Notas prácticas

En sandbox puedes solicitar certificados de prueba a AEAT para titulares ficticios (contacto: catentidades@correo.aeat.es) y así emular envíos sin exponer certificados reales.

La URL del WSDL/XSD te sirve para generar stubs/clients y validar localmente contra los esquemas oficiales.

✅ 8. Validación y control (qué verifica AEAT y cómo manejarlo)
Lo que la AEAT comprueba a la recepción

Formato XML / XSD: que el XML cumple los esquemas oficiales (XSD).

Firma y mTLS: en producción, autenticación mediante certificado válido y correcto establecimiento de TLS mutuo.

Huella encadenada: coherencia del campo <Huella> con el registro anterior.

Campos obligatorios y reglas de negocio: tipos, formatos (fechas, decimales) y límites (máx. 1.000 registros por envío).

Respuesta de estado: EstadoEnvio, CSV, IdPeticion, TiempoEsperaEnvio, y respuestas por línea con códigos de error.

Estados y consecuencias

Correcto / Parcial / Incorrecto: estados globales de la remisión.

Si hay fallo en huella/estructura: la AEAT puede marcar la remisión como “Aceptada con errores” o rechazar líneas concretas.

TiempoEsperaEnvio: cuando la AEAT lo indica, debes esperar ese número de segundos antes de reintentar envíos para el presentador.

Recomendaciones de diseño para tu SIF / microservicio

Validar localmente antes de enviar (XSD, tipos de campo, normalización de valores).

Registrar todo: XML enviado, respuesta SOAP cruda, CSV, IdPeticion, tiempos y logs de usuario.

Implementar reintentos con backoff para errores temporales y respetar TiempoEsperaEnvio si se devuelve.

Mapear códigos de error a acciones (corregir datos, alertar al usuario, reintentar, descartar).

Auditoría: conservar registros y huellas hasta que prescriban obligaciones fiscales.

📂 9. Recursos adicionales (enlaces útiles)

Información técnica VERI*FACTU (AEAT)
https://sede.agenciatributaria.gob.es/Sede/iva/sistemas-informaticos-facturacion-verifactu/informacion-tecnica.html

Guía de integración / manual (PDF)
https://sede.agenciatributaria.gob.es/static_files/AEAT_Desarrolladores/EEDD/IVA/VERI-FACTU/Veri-Factu_Descripcion_SWeb.pdf

WSDL (preproducción)
https://prewww2.aeat.es/static_files/common/internet/dep/aplicaciones/es/aeat/tikeV1.0/cont/ws/SistemaFacturacion.wsdl

Endpoint sandbox (pruebas)
https://prewww1.aeat.es/wlpl/TIKE-CONT/ws/SistemaFacturacion/VerifactuSOAP

FAQ VERI*FACTU (preguntas frecuentes)
https://sede.agenciatributaria.gob.es/Sede/iva/sistemas-informaticos-facturacion-verifactu/preguntas-frecuentes/sistemas-verifactu.html

Contacto AEAT (colaboración social / apoderamientos)
comunicacion.sepri@correo.aeat.es

(para consultas sobre convenio tipo 017 y modelos de representación)

🧰 10. Estructura del proyecto y pasos rápidos para GitHub
Estructura recomendada
verifactu-integration/
│
├── README.md
├── docs/
│   ├── resumen-verifactu.md
│   ├── huella-verifactu.md
│   ├── estructura-xml.md
│   └── enlaces-oficiales.md
│
├── wsdl/
│   ├── SistemaFacturacion.wsdl
│   ├── SuministroLR.xsd
│   └── SuministroInformacion.xsd
│
├── ejemplos/
│   ├── ejemplo-alta-factura.xml
│   ├── ejemplo-anulacion.xml
│   └── ejemplo-evento.xml
│
└── scripts/
    ├── generar_huella.py
    └── pruebas_soap.py
