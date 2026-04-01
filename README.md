# deepdarkCTI Dashboard

Dashboard de Threat Intelligence que carga y visualiza diariamente las fuentes del repositorio [fastfire/deepdarkCTI](https://github.com/fastfire/deepdarkCTI).

## Estructura del repositorio

```
mi-cti-dashboard/
├── .github/
│   └── workflows/
│       └── daily-rebuild.yml    ← GitHub Actions (rebuild diario)
├── docs/
│   ├── index.html               ← Dashboard (publicado en GitHub Pages)
│   └── metadata.json            ← Stats generadas por el workflow
└── README.md
```

## Configuración inicial (5 pasos)

### 1. Crear el repositorio en GitHub
```bash
# Crear repo nuevo en github.com → "New repository"
# Nombre sugerido: cti-dashboard
# Visibilidad: Public (necesario para GitHub Pages gratis)
```

### 2. Subir los archivos
```bash
git clone https://github.com/TU_USUARIO/cti-dashboard.git
cd cti-dashboard

# Crear carpeta docs y copiar el dashboard
mkdir -p docs .github/workflows
cp deepdarkcti-dashboard.html docs/index.html
cp daily-rebuild.yml .github/workflows/

git add .
git commit -m "Initial CTI dashboard setup"
git push
```

### 3. Activar GitHub Pages
1. Ir a **Settings → Pages**
2. Source: **Deploy from a branch**
3. Branch: `main` / Folder: `/docs`
4. Guardar

Tu dashboard estará en: `https://TU_USUARIO.github.io/cti-dashboard/`

### 4. (Opcional) Configurar webhook de n8n
En **Settings → Variables → Actions**, crear una variable:
- Nombre: `N8N_WEBHOOK_URL`
- Valor: URL de tu webhook en n8n

Esto enviará una notificación a tu SOC cada vez que el dashboard se actualice.

### 5. Verificar que funciona
- El workflow corre automáticamente a las **06:00 UTC** (01:00 AM Colombia)
- Para forzar un rebuild: **Actions → Daily CTI Dashboard Rebuild → Run workflow**

## Cómo funciona

```
06:00 UTC (diario)
     │
     ▼
GitHub Actions corre
     │
     ├── git clone fastfire/deepdarkCTI   ← baja los .md actualizados
     ├── Genera metadata.json             ← estadísticas de entradas
     ├── Actualiza docs/index.html        ← inyecta timestamp
     ├── git push                         ← publica en GitHub Pages
     └── Notifica webhook n8n             ← alerta al SOC (opcional)
     │
     ▼
El HTML en el browser carga los .md
via GitHub Raw API en tiempo real
```

> **Nota:** El HTML también carga los datos en vivo desde GitHub Raw cada vez que un usuario abre el dashboard, por lo que siempre muestra la versión más reciente incluso sin rebuild.

## Integración con n8n (SOC Autónomo)

El `metadata.json` generado puede ser consumido por tu agente de Threat Intelligence:

```javascript
// En n8n - HTTP Request Node
// GET https://TU_USUARIO.github.io/cti-dashboard/metadata.json
// Retorna: total de entradas, online/offline por categoría, timestamp
```
