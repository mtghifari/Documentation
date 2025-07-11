# Preparation Install Code

Prepare some code to install

## Install fundamental python library
```py linenums="1"
import os
import pandas as pd
import glob
from datetime import date
import numpy as np
from sklearn import preprocessing
```

## Install visualization library
```py linenums="1"
import matplotlib.pyplot as plt
from matplotlib.cm import ScalarMappable # creating scalar mappable objects for color mapping
from matplotlib.lines import Line2D # creating custom lines
import matplotlib.patches as mpatches # customize graphical shapes and patches
from matplotlib.patches import Patch
import matplotlib.font_manager as fm
import matplotlib.ticker as ticker
from matplotlib import rcParams
import matplotlib.patheffects as path_effects
from matplotlib.colors import ListedColormap, LinearSegmentedColormap
from matplotlib.offsetbox import (OffsetImage, AnnotationBbox)
import seaborn as sns
```

## Install football library (mplsoccer)
```py linenums="1"
!pip install mplsoccer
from mplsoccer import Pitch, VerticalPitch, PyPizza, Radar, grid
from matplotlib.legend_handler import HandlerLine2D
from matplotlib.patches import FancyArrowPatch
from matplotlib.patches import FancyBboxPatch
import matplotlib.patches as patches

from PIL import Image
from tempfile import NamedTemporaryFile
import urllib
import os

from textwrap import wrap # countries name lisibility
from tempfile import NamedTemporaryFile
import urllib

github_url = 'https://github.com/google/fonts/blob/main/ofl/poppins/Poppins-Bold.ttf'
url = github_url + '?raw=true'

response = urllib.request.urlopen(url)
f = NamedTemporaryFile(delete=False, suffix='.ttf')
f.write(response.read())
f.close()

bold = fm.FontProperties(fname=f.name)

github_url = 'https://github.com/google/fonts/blob/main/ofl/poppins/Poppins-Regular.ttf'
url = github_url + '?raw=true'

response = urllib.request.urlopen(url)
f = NamedTemporaryFile(delete=False, suffix='.ttf')
f.write(response.read())
f.close()

reg = fm.FontProperties(fname=f.name)

github_url = 'https://github.com/google/fonts/blob/main/ofl/poppins/Poppins-Italic.ttf'
url = github_url + '?raw=true'

response = urllib.request.urlopen(url)
f = NamedTemporaryFile(delete=False, suffix='.ttf')
f.write(response.read())
f.close()

ita = fm.FontProperties(fname=f.name)

path_eff = [path_effects.Stroke(linewidth=2, foreground='#ffffff'),
            path_effects.Normal()]
```

