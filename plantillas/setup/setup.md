## 1. Setup modular limpieza

```python
import pandas as pd
import numpy as np
import os
import warnings

# Supresión de warnings
warnings.filterwarnings('ignore')
pd.options.mode.chained_assignment = None

# Configuración de visualización del DataFrame
pd.set_option('display.max_columns', None)   # muestra todas las columnas
pd.set_option('display.max_rows', 100)
pd.set_option('display.float_format', lambda x: f'{x:,.2f}')

from pathlib import Path

# Rutas de salida — detectan automáticamente la raíz del proyecto (notebooks/, raíz en VS Code o Colab)
_cwd = Path.cwd()
if _cwd.name == 'notebooks':
    BASE_PATH = _cwd.parent
elif (_cwd / 'data').exists() or (_cwd / 'notebooks').exists() or (_cwd / 'src').exists():
    BASE_PATH = _cwd
elif (_cwd.parent / 'data').exists() or (_cwd.parent / 'notebooks').exists():
    BASE_PATH = _cwd.parent
else:
    BASE_PATH = _cwd

OUTPUT_PATH = str(BASE_PATH / 'outputs')
CLEAN_PATH  = str(BASE_PATH / 'data' / 'clean')
os.makedirs(OUTPUT_PATH, exist_ok=True)
os.makedirs(CLEAN_PATH, exist_ok=True)
```
## 2. Setup cuaderno monolítico (end to end)

```python
import pandas as pd
import numpy as np
import os
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
import seaborn as sns
import warnings

# Supresión de warnings
warnings.filterwarnings('ignore')
pd.options.mode.chained_assignment = None

# Configuración de visualización del DataFrame
pd.set_option('display.max_columns', None)   # muestra todas las columnas
pd.set_option('display.max_rows', 100)
pd.set_option('display.float_format', lambda x: f'{x:,.2f}')

# Configuración de gráficos
sns.set_theme(style='whitegrid')
sns.set_palette('husl')
plt.rcParams['figure.figsize'] = (12, 6)
plt.rcParams['axes.titlesize'] = 16
plt.rcParams['axes.labelsize'] = 13

from pathlib import Path

# Rutas de salida — detectan automáticamente la raíz del proyecto (notebooks/, raíz en VS Code o Colab)
_cwd = Path.cwd()
if _cwd.name == 'notebooks':
    BASE_PATH = _cwd.parent
elif (_cwd / 'data').exists() or (_cwd / 'notebooks').exists() or (_cwd / 'src').exists():
    BASE_PATH = _cwd
elif (_cwd.parent / 'data').exists() or (_cwd.parent / 'notebooks').exists():
    BASE_PATH = _cwd.parent
else:
    BASE_PATH = _cwd

OUTPUT_PATH = str(BASE_PATH / 'outputs')
CLEAN_PATH  = str(BASE_PATH / 'data' / 'clean')
os.makedirs(OUTPUT_PATH, exist_ok=True)
os.makedirs(CLEAN_PATH, exist_ok=True)
```
