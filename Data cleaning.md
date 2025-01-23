``` python
import numpy as np # linear algebra
import pandas as pd # data processing, CSV file I/O (e.g. pd.read_csv)

df = pd.read_csv('downloads/dirty_cafe_sales.csv')

pd.options.display.max_columns=None
pd.options.display.expand_frame_repr = False

from IPython.core.display import display, HTML
display(HTML("<style>div.output_area pre {white-space: pre;}</style>"))
```
