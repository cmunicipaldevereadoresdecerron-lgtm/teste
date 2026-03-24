# 📡 API REST - PTZ Camera Control System

## Base URL

```
http://localhost:5000/api
```

## Autenticação

Todas as requisições devem incluir token JWT no header:

```
Authorization: Bearer <seu-jwt-token>
```

---

## 🎥 Câmeras

### Descobrir Câmeras ONVIF

Escaneia a rede para descobrir câmeras com suporte ONVIF.

```
GET /cameras/discovery/scan
```

**Response:**
```json
{
  "success": true,
  "devices": [
    {
      "hostname": "IPC-D100",
      "ip": "192.168.1.100",
      "manufacturer": "Hikvision",
      "model": "DS-2CD2143G0-I"
    }
  ]
}
```

---

### Listar Câmeras

Obtém todas as câmeras conectadas.

```
GET /cameras
```

**Response:**
```json
{
  "success": true,
  "cameras": [
    {
      "id": "cam_1234567890_abc123",
      "name": "Câmera 1",
      "ip": "192.168.1.100",
      "status": "online"
    }
  ]
}
```

---

### Adicionar Câmera

Adiciona uma nova câmera ao sistema.

```
POST /cameras
```

**Body:**
```json
{
  "name": "Câmera Principal",
  "ip": "192.168.1.100",
  "port": 8080,
  "username": "admin",
  "password": "senha123",
  "manufacturer": "Hikvision",
  "model": "DS-2CD2143G0-I"
}
```

**Response:**
```json
{
  "success": true,
  "camera": {
    "id": "cam_1234567890_abc123",
    "name": "Câmera Principal",
    "ip": "192.168.1.100",
    "status": "online"
  }
}
```

---

### Remover Câmera

Remove uma câmera do sistema.

```
DELETE /cameras/:cameraId
```

**Response:**
```json
{
  "success": true,
  "message": "Câmera removida com sucesso"
}
```

---

### Obter Status da Câmera

Obtém posição atual (Pan, Tilt, Zoom) e status.

```
GET /cameras/:cameraId/status
```

**Response:**
```json
{
  "success": true,
  "status": {
    "pan": 0.5,
    "tilt": 0.3,
    "zoom": 2.0,
    "moving": false
  }
}
```

---

## 🎮 Controle PTZ

### Mover Câmera

Controla Pan, Tilt e Zoom da câmera.

```
POST /cameras/:cameraId/move
```

**Body:**
```json
{
  "action": "pan",
  "direction": "left",
  "speed": 0.5
}
```

**Ações disponíveis:**
- `pan` - Movimento horizontal
  - Direções: `left`, `right`
- `tilt` - Movimento vertical
  - Direções: `up`, `down`
- `zoom` - Aproximação/afastamento
  - Direções: `in`, `out`
- `stop` - Parar movimento
  - Direction: `null`

**Velocidade:** 0 a 1 (0 = parado, 1 = máximo)

**Response:**
```json
{
  "success": true,
  "message": "Comando pan enviado"
}
```

---

## 📍 Presets

### Listar Presets

Obtém todos os presets de uma câmera.

```
GET /presets/:cameraId
```

**Response:**
```json
{
  "success": true,
  "presets": [
    {
      "id": "preset_1234567890_abc123",
      "cameraId": "cam_1234567890_abc123",
      "name": "Palco",
      "position": {
        "pan": 0.5,
        "tilt": 0.3,
        "zoom": 2.0
      }
    }
  ]
}
```

---

### Criar Preset

Salva posição atual como um novo preset.

```
POST /presets/:cameraId
```

**Body:**
```json
{
  "name": "Palco",
  "position": {
    "pan": 0.5,
    "tilt": 0.3,
    "zoom": 2.0
  },
  "description": "Visão do palco principal"
}
```

**Response:**
```json
{
  "success": true,
  "preset": {
    "id": "preset_1234567890_abc123",
    "name": "Palco",
    "position": {
      "pan": 0.5,
      "tilt": 0.3,
      "zoom": 2.0
    }
  }
}
```

---

### Atualizar Preset

Modifica um preset existente.

```
PUT /presets/:cameraId/:presetId
```

**Body:**
```json
{
  "name": "Palco Principal",
  "position": {
    "pan": 0.6,
    "tilt": 0.4,
    "zoom": 2.5
  }
}
```

**Response:**
```json
{
  "success": true,
  "preset": {
    "id": "preset_1234567890_abc123",
    "name": "Palco Principal"
  }
}
```

---

### Deletar Preset

Remove um preset.

```
DELETE /presets/:cameraId/:presetId
```

**Response:**
```json
{
  "success": true,
  "message": "Preset deletado com sucesso"
}
```

---

## 🎥 Vídeo / RTSP

### Testar Conexão RTSP

Testa se consegue conectar ao stream RTSP.

```
POST /cameras/rtsp/test
```

**Body:**
```json
{
  "rtspUrl": "rtsp://admin:senha@192.168.1.100:554/stream1"
}
```

**Response:**
```json
{
  "success": true,
  "connected": true
}
```

---

### Capturar Screenshot

Captura uma imagem do stream RTSP.

```
POST /cameras/:cameraId/snapshot
```

**Body:**
```json
{
  "rtspUrl": "rtsp://admin:senha@192.168.1.100:554/stream1",
  "outputPath": "./snapshots"
}
```

**Response:**
```json
{
  "success": true,
  "path": "./snapshots/snapshot.png"
}
```

---

## ⚠️ Códigos de Erro

| Código | Significado |
|--------|------------|
| 200 | ✅ Sucesso |
| 400 | ❌ Requisição inválida |
| 401 | 🔐 Não autenticado |
| 404 | 🔍 Não encontrado |
| 500 | 💥 Erro no servidor |

---

## 🔌 WebSocket Events

### Cliente para Servidor

```javascript
// Conectar câmera
socket.emit('camera:select', { id: 'cam_123' })

// Controlar câmera
socket.emit('camera:pan', { direction: 'left', speed: 0.5 })
socket.emit('camera:tilt', { direction: 'up', speed: 0.5 })
socket.emit('camera:zoom', { direction: 'in', speed: 0.5 })

// Presets
socket.emit('preset:save', { cameraId: 'cam_123', name: 'Palco', position: {} })
socket.emit('preset:load', { cameraId: 'cam_123', presetId: 'preset_123' })
```

### Servidor para Cliente

```javascript
// Atualização de status
socket.on('camera:update', (data) => {
  console.log('Status atualizado:', data)
})

// Lista de câmeras
socket.on('camera:list', (data) => {
  console.log('Câmeras:', data.cameras)
})
```

---

## 📚 Exemplos

### Exemplo: Mover câmera com CURL

```bash
curl -X POST http://localhost:5000/api/cameras/cam_123/move \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer seu-token-jwt" \
  -d '{
    "action": "pan",
    "direction": "left",
    "speed": 0.5
  }'
```

### Exemplo: JavaScript/Fetch

```javascript
const moveCamera = async (cameraId, action, direction, speed) => {
  const response = await fetch(
    `http://localhost:5000/api/cameras/${cameraId}/move`,
    {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`
      },
      body: JSON.stringify({ action, direction, speed })
    }
  )
  return response.json()
}

// Uso
moveCamera('cam_123', 'pan', 'left', 0.5)
```

---

**Última atualização:** 24/03/2026