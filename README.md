# 🇪🇸 Sistema VERI*FACTU (AEAT)

Implementación y documentación técnica para la integración del sistema **VERI*FACTU**, según el **Real Decreto 1007/2023**, que regula los **Sistemas Informáticos de Facturación (SIF)** en España.

---

## 📘 1. Descripción general

**VERI*FACTU** es un sistema desarrollado por la **Agencia Estatal de Administración Tributaria (AEAT)** que permite a los contribuyentes generar **facturas verificables** y enviar automáticamente los **registros de facturación** a la AEAT.

Su objetivo principal es garantizar la **integridad**, **trazabilidad** e **inalterabilidad** de los datos de facturación.

---

## 🧩 2. Tipos de registros

El sistema gestiona tres tipos de registros principales:

- 🧾 **Registro de Alta de Factura**  
- ❌ **Registro de Anulación**  
- ⚙️ **Registro de Evento**

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

- 🧑‍💼 **Certificado del obligado tributario**  
- 👨‍💻 **Certificado del desarrollador**, si actúa como **colaborador social (tipo 017)** o tiene **apoderamiento**

📧 **Contacto AEAT:** `comunicacion.sepri@correo.aeat.es`  
📘 **Normativa:** Orden HAC/1398/2003 · Resolución 18/12/2024 · Reglamento 1065/2007

---

## 💾 5. Archivos técnicos (AEAT)

| Archivo | Descripción | Enlace |
|----------|-------------|--------|
| 🧩 **WSDL VERI*FACTU** | Servicio web principal para el envío de registros | [SistemaFacturacion.wsdl](https://prewww2.aeat.es/static_files/common/internet/dep/aplicaciones/es/aeat/tikeV1.0/cont/ws/SistemaFacturacion.wsdl) |
| 🧱 **XSD SuministroLR** | Define estructura del registro de facturación | Incluido en el WSDL |
| 🧱 **XSD SuministroInformacion** | Define tipos de datos y validaciones | Incluido en el WSDL |
| 📘 **Manual técnico AEAT** | Guía de integración y mensajes SOAP | [Información técnica AEAT](https://sede.agenciatributaria.gob.es/Sede/iva/sistemas-informaticos-facturacion-verifactu/informacion-tecnica.html) |

--- 

## 🌐 6. Generar huella
import hashlib

def generar_huella(cadena):
    return hashlib.sha256(cadena.encode('utf-8')).hexdigest().upper()

cadena = "IDEmisorFactura=89890001K&NumSerieFactura=12345678/G33&FechaExpedicionFactura=2025-11-06&Huella="
print(generar_huella(cadena))

---

## 🌐 7. Entornos disponibles

El sistema VERI*FACTU dispone de varios **entornos de integración y producción** que permiten realizar pruebas, certificaciones y envíos reales.

| Entorno | Descripción | URL |
|----------|--------------|-----|
| 🧪 **Pruebas (Sandbox)** | Permite testear la integración sin necesidad de certificados reales ni datos fiscales válidos. Ideal para desarrollo inicial. | [Sandbox](https://prewww1.aeat.es/wlpl/TIKE-CONT/ws/SistemaFacturacion/VerifactuSOAP) |
| ⚙️ **Preproducción (WSDL/XSD)** | Contiene los ficheros WSDL y XSD para generar clientes SOAP y validar la estructura de mensajes. | [WSDL Preproducción](https://prewww2.aeat.es/static_files/common/internet/dep/aplicaciones/es/aeat/tikeV1.0/cont/ws/SistemaFacturacion.wsdl) |
| 🚀 **Producción** | Entorno oficial y definitivo. Requiere autenticación mTLS con certificado digital válido del emisor o desarrollador autorizado. | [Producción](https://www1.aeat.es/wlpl/TIKE-CONT/ws/SistemaFacturacion/VerifactuSOAP) |

> 💡 **Notas importantes:**
> - En el entorno *sandbox* puedes solicitar **certificados de prueba** a la AEAT → `catentidades@correo.aeat.es`  
> - El **WSDL** se usa para generar *stubs* o *clientes SOAP* y validar los mensajes XML antes del envío.  
> - Todos los mensajes deben estar firmados digitalmente y cumplir con los esquemas XSD oficiales.

---

## ✅ 8. Validación y control (qué verifica la AEAT)

Cuando se envía un registro a la AEAT, el sistema ejecuta una serie de **validaciones automáticas** que determinan si el envío es correcto o contiene errores.

### 🔍 Comprobaciones principales

- 📄 **Formato XML/XSD:** validación estructural y de tipos de datos.  
- 🔏 **Firma y mTLS:** comprobación de la autenticación TLS mutua.  
- 🔗 **Huella encadenada:** verificación del hash entre registros consecutivos.  
- 🧾 **Campos obligatorios:** control de fechas, decimales, claves de operación y valores.  
- ⚠️ **Límite técnico:** máximo **1000 registros por envío**.

### 📊 Estados posibles

| Estado | Descripción |
|--------|--------------|
| ✅ **Correcto** | El envío se ha procesado correctamente y todos los registros han sido aceptados. |
| ⚠️ **Parcial** | El envío se ha recibido, pero uno o varios registros contienen errores menores. |
| ❌ **Incorrecto** | Error general en el envío. Ningún registro ha sido aceptado. |

> Si la AEAT detecta una huella incorrecta, devuelve el estado **“Aceptada con errores”**.

### 💡 Recomendaciones de diseño

- ✅ Validar los XML localmente antes de enviarlos.  
- 🧾 Registrar todos los XML enviados, respuestas SOAP e identificadores devueltos.  
- 🔁 Implementar reintentos automáticos con un campo `TiempoEsperaEnvio`.  
- ⚙️ Mapear los **códigos de error AEAT** a mensajes y acciones comprensibles.  
- 🧠 Conservar **logs, XML y huellas hash** según los plazos fiscales establecidos por ley.

---

## 📂 9. Recursos adicionales

Enlaces útiles y documentación oficial proporcionada por la **AEAT** para el desarrollo y la certificación de sistemas VERI*FACTU.

| Recurso | Enlace |
|----------|--------|
| 🧠 **Información técnica VERI*FACTU (AEAT)** | [AEAT - Información técnica](https://sede.agenciatributaria.gob.es/Sede/iva/sistemas-informaticos-facturacion-verifactu/informacion-tecnica.html) |
| 📘 **Guía de integración (PDF)** | [Manual Veri*Factu](https://sede.agenciatributaria.gob.es/static_files/AEAT_Desarrolladores/EEDD/IVA/VERI-FACTU/Veri-Factu_Descripcion_SWeb.pdf) |
| 🧩 **WSDL Preproducción** | [SistemaFacturacion.wsdl](https://prewww2.aeat.es/static_files/common/internet/dep/aplicaciones/es/aeat/tikeV1.0/cont/ws/SistemaFacturacion.wsdl) |
| 🧪 **Endpoint Sandbox** | [VerifactuSOAP (Sandbox)](https://prewww1.aeat.es/wlpl/TIKE-CONT/ws/SistemaFacturacion/VerifactuSOAP) |
| ❓ **FAQ VERI*FACTU** | [Preguntas frecuentes AEAT](https://sede.agenciatributaria.gob.es/Sede/iva/sistemas-informaticos-facturacion-verifactu/preguntas-frecuentes/sistemas-verifactu.html) |
| ✉️ **Contacto AEAT** | `comunicacion.sepri@correo.aeat.es` |

---

## 🧰 10. Estructura del proyecto (recomendada para GitHub)

Se recomienda mantener una estructura de carpetas limpia y documentada para facilitar la integración y pruebas del sistema.

```bash
verifactu-integration/
│
├── README.md
│
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
