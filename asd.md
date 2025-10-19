# 📢 Sistema de Broadcast Timer

Sistema de mensajes programados automáticos para Discord.

## 🎯 Características

- **Mensajes automáticos programados** que se envían a intervalos específicos
- **Embeds personalizados** con título, descripción e imagen opcional
- **Selección de canal** donde se enviará el mensaje
- **Control de permisos** (solo administradores y usuarios autorizados)
- **Gestión completa** con comandos para crear, listar y detener broadcasts
- **Almacenamiento en MySQL** para persistencia

## 📋 Comandos Disponibles

### `/broadcasttimer`
Crea un nuevo broadcast timer programado.

**Permisos requeridos:** Administrador o ID autorizado

**Pasos:**
1. Ejecuta el comando `/broadcasttimer`
2. Se abre un modal donde debes ingresar:
   - **Título**: Título del mensaje (máx. 255 caracteres)
   - **Descripción**: Contenido del mensaje (máx. 2000 caracteres)
   - **URL de Imagen**: URL de una imagen (opcional)
3. Selecciona el canal donde se enviará el mensaje
4. Presiona el botón "Configurar Intervalo"
5. Ingresa el intervalo en minutos (mínimo 1, máximo 43200)
6. El sistema confirmará la creación y mostrará el próximo envío

### `/listbroadcasts`
Muestra todos los broadcast timers activos del servidor.

**Permisos requeridos:** Administrador o ID autorizado

Muestra:
- ID del broadcast
- Título
- Canal de destino
- Intervalo configurado
- Próximo envío (timestamp relativo)

### `/stopbroadcast <broadcast_id>`
Detiene un broadcast timer específico.

**Permisos requeridos:** Administrador o ID autorizado

**Parámetros:**
- `broadcast_id`: ID del broadcast a detener (obtenido con `/listbroadcasts`)

## 🔧 Configuración

### Agregar Usuarios Autorizados

Edita el archivo `src/commands/broadcast_timer.py` y modifica la lista `AUTHORIZED_USER_IDS`:

```python
AUTHORIZED_USER_IDS = [
    123456789012345678,  # ID del usuario 1
    987654321098765432,  # ID del usuario 2
    # Agrega más IDs aquí
]
```

Para obtener el ID de un usuario:
1. Activa el Modo Desarrollador en Discord (Ajustes > Avanzado)
2. Click derecho en el usuario > Copiar ID

## 💾 Base de Datos

El sistema crea automáticamente la tabla `broadcastdbtimer` con la siguiente estructura:

```sql
CREATE TABLE broadcastdbtimer (
    id INT AUTO_INCREMENT PRIMARY KEY,
    guild_id BIGINT NOT NULL,
    channel_id BIGINT NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT NOT NULL,
    image_url VARCHAR(500),
    interval_minutes INT NOT NULL,
    last_sent DATETIME,
    next_send DATETIME NOT NULL,
    active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_by BIGINT NOT NULL
);
```

## ⚙️ Funcionamiento

1. **Loop de verificación**: Cada 60 segundos, el bot verifica si hay broadcasts listos para enviar
2. **Envío automático**: Cuando llega el momento, el mensaje se envía al canal configurado
3. **Actualización**: Después de enviar, se calcula el próximo envío sumando el intervalo
4. **Persistencia**: Todo se guarda en MySQL, por lo que sobrevive a reinicios del bot

## 📝 Ejemplos de Uso

### Ejemplo 1: Anuncio de evento cada hora
```
/broadcasttimer
Título: 🎉 Evento de Doble EXP Activo
Descripción: ¡Aprovecha el evento de doble experiencia! Está activo las 24 horas.
Imagen: https://ejemplo.com/evento.png
Canal: #anuncios
Intervalo: 60 minutos
```

### Ejemplo 2: Recordatorio diario
```
/broadcasttimer
Título: 📅 Recordatorio Diario
Descripción: No olvides reclamar tus recompensas diarias en el juego!
Imagen: (vacío)
Canal: #recordatorios
Intervalo: 1440 minutos (24 horas)
```

### Ejemplo 3: Mensaje cada 5 minutos
```
/broadcasttimer
Título: 💎 Visita nuestra Tienda
Descripción: Tenemos nuevos items disponibles. Usa /shop para ver el catálogo.
Imagen: https://ejemplo.com/tienda.png
Canal: #general
Intervalo: 5 minutos
```

## ⚠️ Notas Importantes

- El intervalo mínimo es **1 minuto**
- El intervalo máximo es **43200 minutos (30 días)**
- Solo los administradores y usuarios autorizados pueden usar los comandos
- Los broadcasts se envían automáticamente sin necesidad de que el bot esté en el canal
- Si el canal es eliminado, el broadcast seguirá intentando enviarse (generará errores en logs)
- Usa `/stopbroadcast` para detener broadcasts no deseados

## 🔒 Seguridad

- Todas las operaciones de base de datos usan `asyncio.to_thread()` para no bloquear el event loop
- Los permisos se verifican en cada comando
- Los datos temporales del modal se limpian después de usarse
- Los timestamps usan la zona horaria del servidor

## 🐛 Troubleshooting

### El broadcast no se envía
- Verifica que el canal todavía existe con `/listbroadcasts`
- Comprueba que `active = TRUE` en la base de datos
- Revisa los logs del bot para errores

### Error al crear broadcast
- Verifica que la conexión a MySQL esté activa
- Comprueba que la tabla `broadcastdbtimer` existe
- Verifica que el intervalo esté en el rango permitido (1-43200)

### Usuarios no autorizados pueden usar los comandos
- Verifica que los IDs estén correctamente agregados en `AUTHORIZED_USER_IDS`
- Los IDs deben ser enteros (sin comillas)
- Reinicia el bot después de modificar la lista

## 📊 Monitoreo

Los logs del bot mostrarán:
- ✅ Cuando se crea un nuevo broadcast
- ✅ Cuando se envía un broadcast exitosamente
- ❌ Errores al enviar broadcasts
- ⚠️ Advertencias si un canal no se encuentra

## 🚀 Próximas Mejoras Posibles

- [ ] Editar broadcasts existentes
- [ ] Pausar/reanudar broadcasts
- [ ] Roles mencionables en el mensaje
- [ ] Múltiples canales para el mismo broadcast
- [ ] Estadísticas de envíos
- [ ] Exportar/importar configuraciones
