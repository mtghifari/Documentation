# Create data frame from excel

All file's path and name are adjusted based on what you've got on your drive or storage

## Data Frame from Excel Report
```py
#mount to gdrive
from google.colab import drive
drive.mount('/content/gdrive', force_remount=True)

#loading all the files
path = '/content/gdrive/MyDrive/Data/Liga Indonesia 2024/Liga 1/Match'
all_files = glob.glob(os.path.join(path, "*.xlsx"))

#load all the files to a dataframe
df = pd.concat((pd.read_excel(f, skiprows=[0]) for f in all_files), ignore_index=True)
#filter data based on gameweek if you want specific data
#df = df[df['Gameweek'](spesific game week)].reset_index(drop=True)
#example: df = df[df['Gameweek']<18], if you want filter data based on first 18 gameweek
df
```

## Data Frame from Timeline Report
```py
#mount to gdrive
from google.colab import drive
drive.mount('/content/gdrive', force_remount=True)

#loading all the files
path = '/content/gdrive/MyDrive/Data/Liga Indonesia 2024/Liga 1/Timeline'
all_files = glob.glob(os.path.join(path, "*.xlsx"))
#load all the files to a dataframe
dx = pd.DataFrame()
for f in all_files:
  temp = pd.read_excel(f, skiprows=[0])
  dx = pd.concat([dx, temp], ignore_index=True)
#filter data based on gameweek if you want specific data
#df = df[df['Gameweek'](spesific game week)].reset_index(drop=True)
#example: df = df[df['Gameweek']<18], if you want filter data based on first 18 gameweek
dx
```

## Expected Goal (xG) Data Frame
```py
xg = pd.DataFrame()
for i in range(1,28):
  path = '/content/gdrive/MyDrive/xG/GW '+str(i)
  all_files = glob.glob(os.path.join(path, "*.csv"))
  #load all the files to a dataframe
  dy = pd.DataFrame()
  for f in all_files:
    temp = pd.read_csv(f)
    dy = pd.concat([dy, temp], ignore_index=True)
  xg = pd.concat([xg, dy], ignore_index=True)
xg
```

## Player Data Base
```py
db = pd.read_excel('player.xlsx')
db
```

