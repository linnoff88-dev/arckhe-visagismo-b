# Visagismo B · v1.1.0

**🇧🇷 [Português](README.md) · 🇺🇸 [English](README.en.md) · 🇪🇸 [Español](README.es.md)**

Consultoría de estilo para barberías, en portugués, con análisis facial real, revisión profesional e implementación en **VPS**.

**[Guía pública](https://inematds.github.io/visagismo-b/guia/es/)** · [Instalación en la VPS](VPS.md) · [Plan original](PLANO.md)

## Qué está implementado

- FastAPI, interfaz adaptable, página pública y panel por barbería.
- Inicio de sesión con contraseña protegida por scrypt, sesiones revocables y protección CSRF.
- Cuestionario, carga de imágenes JPEG/PNG/WEBP/HEIC y validación de tamaño/resolución.
- MediaPipe Tasks con 478 puntos detectados; cálculo y clasificación heredados del kit.
- Recomendaciones en portugués, preferencias del cliente y decisión final del barbero.
- Cola persistente en SQLite y worker separado, con reanudación del procesamiento interrumpido.
- Revisión obligatoria, notas profesionales, seguimiento e informe imprimible/PDF desde el navegador.
- Historial aislado por barbería, eliminación y retención automática.
- Enlaces revocables de 7 días para informes **sin fotos**.
- Simulación frontal opcional con flux2-klein vía fal.ai o inemaimg remoto; filtro geométrico y alternativa sin imagen.
- Cobro en caja e integración opcional con Mercado Pago Checkout Pro, con firma del webhook, consulta del pago en el servidor y verificación del monto.
- Docker Compose, Caddy/HTTPS, volumen persistente, copia de seguridad y workflow de pruebas.

El recorrido público es **página de la barbería → pago → cuestionario/foto → análisis → revisión profesional → informe → seguimiento**. El profesional también puede iniciar la consulta desde el panel. Comparte la página disponible en Configuración. El cliente sigue la consulta mediante un enlace privado de 7 días; las fotos y los borradores no aparecen en el informe público. El barbero puede habilitar el reenvío tras un fallo sin volver a cobrar.

La suscripción automática B2B y la agenda con recordatorios no forman parte de esta versión; la mensualidad de la barbería se administra comercialmente fuera del sistema.

## Instalación

El destino es una VPS Linux con Docker Compose, dominio y puertos 80/443 disponibles. Se recomienda comenzar con 2 vCPUs, 4 GB RAM y 20 GB de disco, y validar la carga real. No se necesita GPU cuando la simulación usa fal.ai; sin proveedor, la consulta funciona sin imagen generada.

```bash
git clone https://github.com/inematds/visagismo-b.git
cd visagismo-b
cp .env.example .env
# Edite DOMAIN e PUBLIC_URL; configure las integraciones opcionales.
chmod 600 .env
docker compose up -d --build
docker compose exec app python -m app.manage create-user \
  --email voce@barbearia.com.br --barbearia 'Sua Barbearia' --profissional 'Seu nome'
```

La contraseña se solicita sin mostrarse en la terminal. No hay un inicio de sesión predeterminado. Accede al dominio configurado y completa los datos de la barbería. Consulta los detalles, cobros, copias de seguridad y actualizaciones en [VPS.md](VPS.md).

## Verificación

```bash
python3 -m venv .venv
.venv/bin/pip install --no-deps -r requirements.txt
.venv/bin/pip install -r requirements-dev.txt
.venv/bin/python -m pytest -q tests
```

Las dependencias están fijadas; `--no-deps` evita instalar un segundo paquete OpenCV mediante MediaPipe. La prueba usa una imagen pública de prueba de scikit-image, nunca fotos de clientes. No uses el `requisitos.txt` antiguo para la aplicación nueva.

## Estructura

- `app/`: aplicación actual, interfaz pt-BR, autenticación, cola, pagos y administración.
- `03-motor/`: motor facial original reutilizado; detector ajustado para rechazar más de un rostro.
- `tests/`: flujo completo, aislamiento, acceso, cargas, retención, firma y conciliación.
- `guia/` y `capa/`: página pública y portada del proyecto.
- `06-app/`, `01-estrategia/`, `configura.py`, `marca.json`: referencia del kit heredado en español, no utilizada por el Compose actual.
- `docs/legado/`: instrucciones originales preservadas como referencia; no son la ruta de implementación actual.

## Límites y tratamiento de datos

Las proporciones son estimaciones 2D influenciadas por la captura; no validan la belleza, la personalidad ni la salud. Comparar las proporciones de la imagen generada **no garantiza que se preserve la identidad**. La simulación lateral está deshabilitada. La persona revisa el resultado antes de la entrega.

Las fotos se eliminan después de 24 horas y los informes, después de 90 días (configurable); el worker verifica cada minuto. Las copias de seguridad necesitan su propio período de retención. El administrador debe completar los datos del responsable y la política de privacidad antes de atender a clientes.

Mercado Pago usa la cuenta configurada durante la instalación: no existe una división automática de los ingresos entre barberías. Para operadores comerciales independientes, usa una instalación y credenciales por empresa. El pago en línea requiere credenciales y validación en sandbox en la cuenta de destino; las pruebas automatizadas usan respuestas controladas.

## Origen

Adaptado del ZIP `visagismo-barberias.zip` proporcionado por el usuario. El kit no incluye un archivo de licencia; esta adaptación no declara una licencia sobre el material original. Verifica los términos de origen antes de redistribuir o comercializar el kit.
