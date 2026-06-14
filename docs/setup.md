# Guía de configuración — Entorno IA Local

## Requisitos previos

- GPU NVIDIA RTX (mínimo 8GB VRAM para SD 1.5, 16GB+ para SDXL)
- Python 3.10.x
- Git instalado
- 500GB+ de espacio en disco

---

## 1. Instalación de AUTOMATIC1111

```bash
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui
cd stable-diffusion-webui
./webui.sh  # Linux/Mac
webui-user.bat  # Windows
```

Accede en `http://127.0.0.1:7860`

---

## 2. Instalación de ComfyUI

```bash
git clone https://github.com/comfyanonymous/ComfyUI
cd ComfyUI
pip install -r requirements.txt
python main.py
```

Accede en `http://127.0.0.1:8188`

---

## 3. Estructura de carpetas

```
models/
├── checkpoints/     ← modelos principales (.safetensors)
│   ├── juggernautXL_ragnarokBy.safetensors
│   ├── realvisxlV50_v50Bakedvae.safetensors
│   └── flux_dev.safetensors
├── loras/           ← modelos LoRA
├── vae/             ← VAE
│   └── sdxl_vae.safetensors
├── controlnet/      ← modelos ControlNet
│   ├── control_openpose.safetensors
│   ├── control_depth.safetensors
│   └── control_canny.safetensors
├── ipadapter/       ← modelos IP-Adapter
└── upscale_models/  ← upscalers
    ├── 4x-UltraSharp.pth
    └── ESRGAN_4x.pth
```

---

## 4. ControlNet — Control de poses

ControlNet permite fijar poses exactas usando imágenes de referencia.

**Modelos necesarios:**
- `OpenPose` — control de pose corporal
- `Depth` — control de profundidad y estructura 3D
- `Canny` — control de bordes y geometría

**Uso básico:**
1. En AUTOMATIC1111 → pestaña **ControlNet**
2. Sube tu imagen de referencia de pose
3. Selecciona el preprocesador (`openpose`, `depth_anything`, `canny`)
4. Genera con tu prompt

---

## 5. IP-Adapter — Consistencia de vestuario e identidad

IP-Adapter transfiere el estilo visual de una imagen de referencia a la generación.

**Instalación en ComfyUI:**
```bash
cd ComfyUI/custom_nodes
git clone https://github.com/cubiq/ComfyUI_IPAdapter_plus
```

**Uso:**
- Imagen de referencia → IP-Adapter → mantiene vestuario, colores y estilo

---

## 6. Face-Swap — Consistencia facial

Para mantener la identidad facial idéntica en cada imagen:

**Instalación del módulo insightface:**
```bash
# Dentro del entorno de ComfyUI
pip install insightface
pip install onnxruntime-gpu
```

> ⚠️ Asegúrate de usar la misma versión de Python que usa ComfyUI para evitar conflictos de dependencias.

---

## 7. Entrenamiento de LoRAs con Florence-2

Para entrenar un LoRA específico (personaje, estilo, vehículo):

**Etiquetado automático del dataset:**
```python
# Florence-2 genera descripciones automáticas de cada imagen
# Esto elimina el etiquetado manual imagen por imagen
from transformers import AutoProcessor, AutoModelForCausalLM

model = AutoModelForCausalLM.from_pretrained("microsoft/Florence-2-large")
processor = AutoProcessor.from_pretrained("microsoft/Florence-2-large")
```

**Estructura del dataset:**
```
dataset/
├── imagen_001.jpg
├── imagen_001.txt   ← generado por Florence-2
├── imagen_002.jpg
└── imagen_002.txt
```

---

## 8. Vídeo con AnimateDiff y LTX2-3

**AnimateDiff** convierte cualquier workflow de imagen en animación.

**Text2Video con LTX2-3:**
- Modelo: `ltx-2.3-22b-dev-Q8_0.gguf`
- Permite generación de vídeo desde texto con alta coherencia temporal

---

## Solución de problemas frecuentes

| Error | Solución |
|---|---|
| `No module named 'insightface'` | `pip install insightface onnxruntime-gpu` |
| Conflicto de versiones Python | Usa el Python interno de ComfyUI, no el del sistema |
| OOM (Out of Memory) en GPU | Reduce resolución o usa `--lowvram` en los argumentos |
| LoRA no carga | Verifica que el LoRA es compatible con el checkpoint base |
