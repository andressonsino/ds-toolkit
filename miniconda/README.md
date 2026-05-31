# 🐍 Miniconda

Guías de configuración y uso de Miniconda para entornos de ciencia de datos en Windows.

## Contenido

| Archivo | Descripción |
|---------|-------------|
| `miniconda_comandos.md` | Guía completa de comandos — gestión de entornos, paquetes y Jupyter |

## Entorno recomendado para ciencia de datos

```bash
conda create --prefix C:/datasci python=3.11
conda activate C:/datasci
pip install jupyter pandas numpy matplotlib scikit-learn tensorflow scipy seaborn
```

> ⚠️ Si tu usuario Windows tiene caracteres especiales (`é`, `ñ`, espacios), usá siempre `--prefix` con una ruta limpia para evitar errores de OpenSSL.

