📘 1. Descripción general

VERI*FACTU es un sistema desarrollado por la Agencia Estatal de Administración Tributaria (AEAT) que permite a los contribuyentes generar facturas verificables y enviar automáticamente los registros de facturación a la AEAT.

Su objetivo principal es garantizar la integridad, trazabilidad e inalterabilidad de los datos de facturación.

🧩 2. Tipos de registros

El sistema gestiona tres tipos de registros:

Registro de Alta de Factura

Registro de Anulación

Registro de Evento

Cada registro genera una huella criptográfica (hash) que asegura la autenticidad y el encadenamiento entre facturas.

⚙️ 3. Especificaciones técnicas
Elemento	Valor
Algoritmo de hash	SHA-256
Codificación	UTF-8
Salida	Hexadecimal (64 caracteres, mayúsculas)
Formato de concatenación	campo1=valor1&campo2=valor2&...
Campo de salida XML	<Huella> o <HuellaEvento>
Encadenamiento	Cada registro incluye la huella del anterior
🔐 4. Certificados digitales

El envío de registros requiere autenticación mediante certificado digital:

Certificado del obligado tributario

Certificado del desarrollador, si actúa como colaborador social (tipo 017) o tiene apoderamiento

📧 Contacto AEAT: comunicacion.sepri@correo.aeat.es
📘 Normativa: Orden HAC/1398/2003, Resolución 18/12/2024, Reglamento 1065/2007

💾 5. Archivos técnicos (AEAT)
Archivo	Descripción	Enlace
WSDL VERI*FACTU	Servicio web principal para el envío de registros	SistemaFacturacion.wsdl

XSD SuministroLR	Define estructura del registro de facturación	Incluido en el WSDL
XSD SuministroInformacion	Define tipos de datos y validaciones	Incluido en el WSDL
Manual técnico AEAT	Guía de integración y mensajes SOAP	Información técnica AEAT
🧮 6. Ejemplo de generación de huella (SHA-256)
import hashlib

def generar_huella(cadena):
    return hashlib.sha256(cadena.encode('utf-8')).hexdigest().upper()

cadena = "IDEmisorFactura=89890001K&NumSerieFactura=12345678/G33&FechaExpedicionFactura=2025-11-06&Huella="
print(generar_huella(cadena))

🌐 7. Entornos disponibles
Entorno	Descripción	URL
Pruebas (Sandbox)	Permite testear la integración sin certificados reales.	Sandbox

Preproducción (WSDL/XSD)	WSDL y esquemas para generar clientes SOAP.	WSDL Preprod

Producción	Entorno oficial, requiere mTLS y certificado válido.	Producción

Notas prácticas:

En sandbox puedes solicitar certificados de prueba a AEAT (catentidades@correo.aeat.es).

El WSDL se usa para generar stubs/clients y validar los mensajes SOAP.

✅ 8. Validación y control (qué verifica AEAT)

Comprobaciones principales:

Formato XML/XSD: validación estructural.

Firma y mTLS: autenticación TLS mutua.

Huella encadenada: coherencia entre registros.

Campos obligatorios: fechas, decimales, tipos.

Límite: máx. 1000 registros por envío.

Estados posibles:

✅ Correcto

⚠️ Parcial

❌ Incorrecto

Si la AEAT detecta una huella incorrecta → “Aceptada con errores”.

Recomendaciones de diseño:

Validar localmente antes de enviar.

Registrar XML, respuestas SOAP, CSV e IDs.

Implementar reintentos con TiempoEsperaEnvio.

Mapear códigos de error a acciones concretas.

Guardar logs y huellas según los plazos fiscales.

📂 9. Recursos adicionales
Recurso	Enlace
Información técnica VERI*FACTU (AEAT)	AEAT - Información técnica

Guía de integración (PDF)	Manual Veri*Factu

WSDL Preproducción	SistemaFacturacion.wsdl

Endpoint Sandbox	VerifactuSOAP (Sandbox)

FAQ VERI*FACTU	Preguntas frecuentes AEAT

Contacto AEAT	comunicacion.sepri@correo.aeat.es
🧰 10. Estructura del proyecto (recomendada para GitHub)
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
