# Player Radar Processing

This page provides to creating player radar based from player excel report.

All data frame should be called before creating action. How to create data frame can go to Data Frame page.

## Create action list

first list: for all position (exclude goalkeeper)

second list: for goalkeeper only
```py
metrik = ['Name','Team','MoP','Non-penalty goals','Non-penalty xG','NPxG/Shot','Shots','Shot on target ratio','Conversion ratio','Chances created','Assists',
          'Passes-to-box','Through passes','Passes to final 3rd','Progressive passes','Long passes','Pass accuracy','Successful crosses',
          'Successful dribbles','Offensive duel won ratio','Tackles','Intercepts','Recoveries','Blocks','Clearances','Aerial duel won ratio',
          'Defensive duel won ratio','Passes','Passes received','Clean sheets','Shots on target faced','xGOT against','Goals conceded','Goals prevented',
          'Save ratio','Sweepers','Crosses claimed']

jamet = ['Name','Team','MoP','Non-penalty goals','Shots','Chances created','Assists','Through passes','Progressive passes',
         'Long passes','Successful crosses','Successful dribbles','Tackles','Intercepts','Recoveries','Blocks','Clearances',
         'Total Pass','Aerial Duels','Offensive Duel','Offensive Duel - Won','Defensive Duel','Defensive Duel - Won','Goal',
         'Shot on','Pass','Aerial Won','Penalty','Passes','Clean sheets','Sweepers','Crosses claimed']
```

## Calculate per 90
### Create per 90 function
```py linenums="1"
#creating function
def get_sum90(report, tl, xg, db, gk, min):
  #duplicate data for preventing direct modification
  df = report.copy()
  df2 = tl.copy()
  db = db.copy()
  gk = gk.copy()

  dxg = xg.copy() #duplicate xg data
  dxg = dxg[['Name','xG']]
  dxg = dxg.groupby(['Name'], as_index=False).sum() #sum total xg, based on name player
#redef action data name
  df['Non-penalty goals'] = df['Goal']
  df['Shots'] = df['Shot on']+df['Shot off']+df['Shot Blocked']
  df['Chances created'] = df['Create Chance']
  df['Assists'] = df['Assist']
  df['Through passes'] = df['Pass - Through Pass']
  df['Progressive passes'] = df['Pass - Progressive Pass']
  df['Long passes'] = df['Pass - Long Ball']
  df['Successful crosses'] = df['Cross']
  df['Successful dribbles'] = df['Dribble']
  df['Tackles'] = df['Tackle']
  df['Intercepts'] = df['Intercept']
  df['Recoveries'] = df['Recovery']
  df['Blocks'] = df['Block']+df['Block Cross']
  df['Clearances'] = df['Clearance']
  df['Passes'] = df['Pass']
  df['Clean sheets'] = df['Cleansheet']
  df['Sweepers'] = df['Keeper - Sweeper']
  df['Crosses claimed'] = df['Cross Claim']
#redef and sum data name
  df['Total Pass'] = df['Pass']+df['Pass Fail'] #total passing
  df['Aerial Duels'] = df['Aerial Won']+df['Aerial Lost'] #total aerial
  df['Offensive Duel - Won'] = df['Offensive Duel - Won']+df['Fouled']+df['Dribble'] #total off duel won
  df['Offensive Duel - Lost'] = df['Offensive Duel - Lost']+df['Loose Ball - Tackle']+df['Dribble Fail'] #total off duel lost
  df['Defensive Duel - Won'] = df['Defensive Duel - Won']+df['Tackle'] #total def duel won
  df['Defensive Duel - Lost'] = df['Defensive Duel - Lost']+df['Foul']+df['Dribbled Past'] #total def duel lost
  df['Offensive Duel'] = df['Offensive Duel - Won']+df['Offensive Duel - Lost'] #total off duel
  df['Defensive Duel'] = df['Defensive Duel - Won']+df['Defensive Duel - Lost'] #total def duel
  df['Penalty'] = df['Penalty Goal']-df['Penalty Missed'] #total pen shot
#merging action dataframe
  df_data = df[jamet]
  df_sum = df_data.groupby(['Name','Team'], as_index=False).sum() #aggregating stat based on player and team
  df_sum = pd.merge(df_sum, dxg, on='Name', how='outer') #merging with xG data
#calculate ratio stat (rounding two digits)
  df_sum['Non-penalty xG'] = round(df_sum['xG']-(df_sum['Penalty']*0.593469436750998),2) #non penalty xG
  df_sum['NPxG/Shot'] = round(df_sum['Non-penalty xG']/(df_sum['Shots']-df_sum['Penalty']),2) #ratio non penalty xg / shot
  df_sum['Conversion ratio'] = round(df_sum['Goal']/df_sum['Shots'],2) #calculate convertion ratio goal
  df_sum['Shot on target ratio'] = round(df_sum['Shot on']/df_sum['Shots'],2) #calculate shot on ratio
  df_sum['Pass accuracy'] = round(df_sum['Pass']/df_sum['Total Pass'],2) #calculate pass accuracy
  df_sum['Aerial duel won ratio'] = round(df_sum['Aerial Won']/df_sum['Aerial Duels'],2) #calculate aerial duel
  df_sum['Offensive duel won ratio'] = round(df_sum['Offensive Duel - Won']/df_sum['Offensive Duel'],2) #calculate off duel won
  df_sum['Defensive duel won ratio'] = round(df_sum['Defensive Duel - Won']/df_sum['Defensive Duel'],2) #calculate def duel lost
#merging data
  #merging with pass to box data
  temp = proses_tl(df2)
  df_sum = pd.merge(df_sum, temp, on='Name', how='outer')
  #merging with received pass data
  temp2 = proses_tl2(df2)
  df_sum = pd.merge(df_sum, temp2, on='Name', how='outer')

  gk = gk[['Name','Save','Penalty Save','Total Shots','Goals Conceded',
           'xGOTA','Goals Prevented']] #def gk column
  gk['Save ratio'] = round((gk['Save']+gk['Penalty Save'])/gk['Total Shots'],2) #calculate save ratio
  gk['Shots on target faced'] = gk['Total Shots'] #def shot on faced
  gk['xGOT against'] = gk['xGOTA'] #def xGOT
  gk['Goals conceded'] = gk['Goals Conceded'] #rename goals conceded
  gk['Goals prevented'] = gk['Goals Prevented'] #rename goals prevented
  gk = gk[['Name','Save ratio','Shots on target faced','xGOT against','Goals conceded','Goals prevented']] #redef gk column
#merging new gk data
  df_sum = pd.merge(df_sum, gk, on='Name', how='outer')
#cleaning missing data
  df_sum.replace([np.inf, -np.inf], 0, inplace=True) #replacing infinity value with 0
  df_sum.fillna(0, inplace=True)

  temp = df_sum.drop(['Name','Team'], axis=1) #removing name and team column
#calculate per 90 data
  def p90_Calculator(variable_value):
    p90_value = round((((variable_value/temp['MoP']))*90),2)
    return p90_value
  p90 = temp.apply(p90_Calculator)
#creating per 90 column from data frame
  p90['Name'] = df_sum['Name']
  p90['Team'] = df_sum['Team']
  p90['MoP'] = df_sum['MoP']
  p90['NPxG/Shot'] = df_sum['NPxG/Shot']
  p90['Conversion ratio'] = df_sum['Conversion ratio']
  p90['Shot on target tatio'] = df_sum['Shot on target ratio']
  p90['Pass accuracy'] = df_sum['Pass accuracy']
  p90['Aerial duel won ratio'] = df_sum['Aerial duel won ratio']
  p90['Offensive duel won ratio'] = df_sum['Offensive duel won ratio']
  p90['Defensive duel won ratio'] = df_sum['Defensive duel won ratio']
  p90['Save ratio'] = df_sum['Save ratio']

  p90 = p90[metrik] #getting column from 'metrik' variable
  p90['Name'] = p90['Name'].str.strip() #removing unnecessary space at begining/end in the name column (ex: spaces, tabs, newlines)
#merge adv data and position
  pos = db[['Name','Position']] #def name and position column
  data_full = pd.merge(p90, pos, on='Name', how='left')
  data_full = data_full.loc[(data_full['MoP']>=min)].reset_index(drop=True) #showing player who have minimum minute of play

  return data_full, df_sum
```

### Show per 90 data frame
```py linenums="1"
mins = 360 #set minimum minutes play before analyze
rank_p90 = get_sum90(df, dx, xg, db, gk, mins)[0] #get per 90 data from get_sum90 function (index = 0)
rank_tot = get_sum90(df, dx, xg, db, gk, mins)[1] #get total stats from get_sum90 function (index = 1)
rank_p90 #show data frame
```

## Group player data based on position
```py linenums="1"
def get_pct(data):
  data_full = data.copy() #duplicate data for preventing direct modification
  df4 = data_full.groupby('Position', as_index=False) #group data and execute functions on these groups
  #separating data based on 'position' value
  midfielder = df4.get_group('Midfielder')
  goalkeeper = df4.get_group('Goalkeeper')
  forward = df4.get_group('Forward')
  att_10 = df4.get_group('Attacking 10')
  center_back = df4.get_group('Center Back')
  fullback = df4.get_group('Fullback')
  winger = df4.get_group('Winger')

  #calculating the average stats per position
  '''
  1. duplicate data based on grouping position
  2. drop 'name', 'position', 'team' column for temporary
  3. calculate mean value for every row
  4. add again 'name', 'position', 'team' column
  5. replace NaN value with 'League Average'
  '''

  #winger
  temp = winger.copy()
  winger = winger.drop(['Name','Position','Team'], axis=1)
  winger.loc['mean'] = round((winger.mean()),2)
  winger['Name'] = temp['Name']
  winger['Position'] = temp['Position']
  winger['Team'] = temp['Team']
  values1 = {"Name": 'Average W', "Position": 'Winger', "Team": 'League Average'}
  winger = winger.fillna(value=values1)

  #fb
  temp = fullback.copy()
  fullback = fullback.drop(['Name','Position','Team'], axis=1)
  fullback.loc['mean'] = round((fullback.mean()),2)
  fullback['Name'] = temp['Name']
  fullback['Position'] = temp['Position']
  fullback['Team'] = temp['Team']
  values2 = {"Name": 'Average FB', "Position": 'Fullback', "Team": 'League Average'}
  fullback = fullback.fillna(value=values2)

  #cb
  temp = center_back.copy()
  center_back = center_back.drop(['Name','Position','Team'], axis=1)
  center_back.loc['mean'] = round((center_back.mean()),2)
  center_back['Name'] = temp['Name']
  center_back['Position'] = temp['Position']
  center_back['Team'] = temp['Team']
  values3 = {"Name": 'Average CB', "Position": 'Center Back', "Team": 'League Average'}
  center_back = center_back.fillna(value=values3)

  #cam
  temp = att_10.copy()
  att_10 = att_10.drop(['Name','Position','Team'], axis=1)
  att_10.loc['mean'] = round((att_10.mean()),2)
  att_10['Name'] = temp['Name']
  att_10['Position'] = temp['Position']
  att_10['Team'] = temp['Team']
  values4 = {"Name": 'Average CAM', "Position": 'Attacking 10', "Team": 'League Average'}
  att_10 = att_10.fillna(value=values4)

  #forward
  temp = forward.copy()
  forward = forward.drop(['Name','Position','Team'], axis=1)
  forward.loc['mean'] = round((forward.mean()),2)
  forward['Name'] = temp['Name']
  forward['Position'] = temp['Position']
  forward['Team'] = temp['Team']
  values5 = {"Name": 'Average FW', "Position": 'Forward', "Team": 'League Average'}
  forward = forward.fillna(value=values5)

  #gk
  temp = goalkeeper.copy()
  goalkeeper = goalkeeper.drop(['Name','Position','Team',], axis=1)
  goalkeeper.loc['mean'] = round((goalkeeper.mean()),2)
  goalkeeper['Name'] = temp['Name']
  goalkeeper['Position'] = temp['Position']
  goalkeeper['Team'] = temp['Team']
  values6 = {"Name": 'Average GK', "Position": 'Goalkeeper', "Team": 'League Average'}
  goalkeeper = goalkeeper.fillna(value=values6)

  #cm
  temp = midfielder.copy()
  midfielder = midfielder.drop(['Name','Position','Team'], axis=1)
  midfielder.loc['mean'] = round((midfielder.mean()),2)
  midfielder['Name'] = temp['Name']
  midfielder['Position'] = temp['Position']
  midfielder['Team'] = temp['Team']
  values7 = {"Name": 'Average CM', "Position": 'Midfielder', "Team": 'League Average'}
  midfielder = midfielder.fillna(value=values7)

  #percentile rank for all position, times by 100 and round with 0, convert to integer
  rank_cm = round(((midfielder.rank(pct=True))*100),0).astype(int)
  rank_gk = round(((goalkeeper.rank(pct=True))*100),0).astype(int)
  rank_fw = round(((forward.rank(pct=True))*100),0).astype(int)
  rank_cam = round(((att_10.rank(pct=True))*100),0).astype(int)
  rank_cb = round(((center_back.rank(pct=True))*100),0).astype(int)
  rank_fb = round(((fullback.rank(pct=True))*100),0).astype(int)
  rank_w = round(((winger.rank(pct=True))*100),0).astype(int)

  #add back Name and Position column
  rank_cm['Name'] = midfielder['Name']
  rank_gk['Name'] = goalkeeper['Name']
  rank_fw['Name'] = forward['Name']
  rank_cam['Name'] = att_10['Name']
  rank_cb['Name'] = center_back['Name']
  rank_fb['Name'] = fullback['Name']
  rank_w['Name'] = winger['Name']

  rank_cm['Position'] = midfielder['Position']
  rank_gk['Position'] = goalkeeper['Position']
  rank_fw['Position'] = forward['Position']
  rank_cam['Position'] = att_10['Position']
  rank_cb['Position'] = center_back['Position']
  rank_fb['Position'] = fullback['Position']
  rank_w['Position'] = winger['Position']

  rank_cm['Team'] = midfielder['Team']
  rank_gk['Team'] = goalkeeper['Team']
  rank_fw['Team'] = forward['Team']
  rank_cam['Team'] = att_10['Team']
  rank_cb['Team'] = center_back['Team']
  rank_fb['Team'] = fullback['Team']
  rank_w['Team'] = winger['Team']

  rank_cm['MoP'] = midfielder['MoP']
  rank_gk['MoP'] = goalkeeper['MoP']
  rank_fw['MoP'] = forward['MoP']
  rank_cam['MoP'] = att_10['MoP']
  rank_cb['MoP'] = center_back['MoP']
  rank_fb['MoP'] = fullback['MoP']
  rank_w['MoP'] = winger['MoP']

  #merge all rank position into one data frame
  rank_liga = pd.concat([rank_cm, rank_gk, rank_fw, rank_cam, rank_cb, rank_fb, rank_w]).reset_index(drop=True)
  rank_liga['MoP'] = rank_liga['MoP'].astype(int) #change 'MoP' value into integer

  return rank_liga #return dataframe that contain percentile based on their positions

  #output: dataframe contains stat ranks (in percentile) per player

abc = get_pct(tengs) #process percentile function above
```

Merge with database player data frame
```py
tk = db[['Name','Nationality']] #create new dataframe from db (database player) dataframe
abc = pd.merge(abc, tk, on='Name', how='left') #merge dataframe based on 'Name' column, ensure abc's row maintained
abc = abc[abc['Nationality']=='Indonesia'].reset_index(drop=True) #filter data only contain 'Indonesia' value in 'Nationality' column
abc
```

Create dictionary action metrics based on player's position
```py
posdict = {'gk':{'position':'Goalkeeper',
                 'metrics':['Name','Passes','Pass accuracy','Long passes','Progressive passes','Passes received','Clean sheets','Shots on target faced',
                            'xGOT against','Goals conceded','Goals prevented','Save ratio','Sweepers','Crosses claimed','Intercepts']},
           'cb':{'position':'Center Back',
                 'metrics':['Name','Non-penalty goals','Shots',
                            'Passes to final 3rd','Progressive passes','Long passes','Pass accuracy',
                            'Tackles','Intercepts','Recoveries','Blocks','Clearances','Aerial duel won ratio','Defensive duel won ratio']},
           'fb':{'position':'Fullback',
                 'metrics':['Name','Non-penalty goals','Non-penalty xG','Shots','Chances created','Assists',
                            'Passes-to-box','Through passes','Passes to final 3rd','Progressive passes','Pass accuracy','Successful dribbles','Successful crosses','Offensive duel won ratio',
                            'Tackles','Intercepts','Recoveries','Blocks','Clearances','Aerial duel won ratio','Defensive duel won ratio']},
           'cm':{'position':'Midfielder',
                 'metrics':['Name','Non-penalty goals','Non-penalty xG','NPxG/Shot','Shots','Shot on target ratio','Chances created','Assists',
                            'Passes-to-box','Through passes','Passes to final 3rd','Progressive passes','Long passes','Pass accuracy','Successful dribbles','Offensive duel won ratio',
                            'Tackles','Intercepts','Recoveries','Clearances','Defensive duel won ratio']},
           'cam/w':{'position':'Attacking 10/Winger',
                    'metrics':['Name','Non-penalty goals','Non-penalty xG','NPxG/Shot','Shots','Shot on target ratio','Conversion ratio','Chances created','Assists',
                               'Passes-to-box','Through passes','Passes to final 3rd','Progressive passes','Pass accuracy','Successful dribbles','Offensive duel won ratio',
                               'Tackles','Intercepts','Recoveries','Defensive duel won ratio']},
           'fw':{'position':'Forward',
                 'metrics':['Name','Non-penalty goals','Non-penalty xG','NPxG/Shot','Shots','Shot on target ratio','Conversion ratio','Chances created','Assists',
                            'Passes-to-box','Through passes','Progressive passes','Pass accuracy','Successful dribbles','Offensive duel won ratio',
                            'Tackles','Intercepts','Recoveries','Aerial duel won ratio','Defensive duel won ratio']}}
```

# Player Radar Visualization
```py linenums="1"
def player_radar(komp, pos, klub, name, data, mins):
  df = data.copy() #duplicate data for preventing direct modification
  df = df[df['Position']==pos] #filter data based on player position

  #DATA
  '''
  if/elif ('getting spec position: forward, mid, defender, gk')
    temp = df[posdict["player position"]['metrics']].reset_index(drop=True)
    temp = temp[(temp['Name']==name) | (temp['Name']=='Average pos')].reset_index(drop=True) ---> get spesific name

    slice_colors = ['#e900ff']*offense act + ['#faeb2c']*defence act + ['#74ee15']*possesion act ---->slice color can be adjusted
  '''
  if (pos=='Forward'):
    temp = df[posdict['fw']['metrics']].reset_index(drop=True)
    temp = temp[(temp['Name']==name) | (temp['Name']=='Average FW')].reset_index(drop=True)

    slice_colors = ['#e900ff']*8 + ['#faeb2c']*6 + ['#74ee15']*5

  elif (pos=='Winger') or (pos=='Attacking 10'):
    temp = df[posdict['cam/w']['metrics']].reset_index(drop=True)
    if (pos=='Winger'):
      temp = temp[(temp['Name']==name) | (temp['Name']=='Average W')].reset_index(drop=True)
    else:
      temp = temp[(temp['Name']==name) | (temp['Name']=='Average CAM')].reset_index(drop=True)

    slice_colors = ['#e900ff']*8 + ['#faeb2c']*7 + ['#74ee15']*4

  elif (pos=='Midfielder'):
    temp = df[posdict['cm']['metrics']].reset_index(drop=True)
    temp = temp[(temp['Name']==name) | (temp['Name']=='Average CM')].reset_index(drop=True)

    slice_colors = ['#e900ff']*7 + ['#faeb2c']*8 + ['#74ee15']*5

  elif (pos=='Fullback'):
    temp = df[posdict['fb']['metrics']].reset_index(drop=True)
    temp = temp[(temp['Name']==name) | (temp['Name']=='Average FB')].reset_index(drop=True)

    slice_colors = ['#e900ff']*5 + ['#faeb2c']*8 + ['#74ee15']*7

  elif (pos=='Center Back'):
    temp = df[posdict['cb']['metrics']].reset_index(drop=True)
    temp = temp[(temp['Name']==name) | (temp['Name']=='Average CB')].reset_index(drop=True)

    slice_colors = ['#e900ff']*2 + ['#faeb2c']*4 + ['#74ee15']*7

  elif (pos=='Goalkeeper'):
    temp = df[posdict['gk']['metrics']].reset_index(drop=True)
    temp = temp[(temp['Name']==name) | (temp['Name']=='Average GK')].reset_index(drop=True)

    slice_colors = ['#e900ff']*5 + ['#faeb2c']*6 + ['#74ee15']*3

  #temp = temp.drop(['Team'], axis=1)

  #extract 'Average' data as comparasion
  avg_player = temp[temp['Name'].str.contains('Average')] #extract 'Average' data from 'Nama' column
  av_name = list(avg_player['Name'])[0] #extract 'Average' data as first index
  params = list(temp.columns) #create columns list based on function dataframe
  params = params[1:] #only processing after index 1 (column 'Name' = index 0)

  a_values = []
  b_values = []

  for x in range(len(temp['Name'])):
    if temp['Name'][x] == name:
      a_values = temp.iloc[x].values.tolist()
    if temp['Name'][x] == av_name:
      b_values = temp.iloc[x].values.tolist()

  a_values = a_values[1:]
  b_values = b_values[1:]

  values = [a_values,b_values]
  maxmin = pd.DataFrame({'param':params,'value':a_values,'average':b_values})
  for index, value in enumerate(params):
    if value == 'Progressive passes':
      params[index] = 'Progressive\npasses'
    elif value == 'long passes':
      params[index] = 'Long\npasses'
    elif value == 'Pass accuracy':
      params[index] = 'Pass\naccuracy'
    elif value == 'Successful crosses':
      params[index] = 'Successful\ncrosses'
    elif value == 'Successful dribbles':
      params[index] = 'Successful\ndribbles'
    elif value == 'Offensive duel won ratio':
      params[index] = 'Offensive duel\nwon ratio'
    elif value == 'Defensive duel won ratio':
      params[index] = 'Defensive duel\nwon ratio'
    elif value == 'Aerial duel won ratio':
      params[index] = 'Aerial\nduel won\nratio'
    elif value == 'Passes to final 3rd':
      params[index] = 'Passes to\nfinal 3rd'
    elif value == 'Through passes':
      params[index] = 'Through\npasses'
    elif value == 'Non-penalty goals':
      params[index] = 'Non-penalty\ngoals'
    elif value == 'Shot on target ratio':
      params[index] = 'Shot on\ntarget ratio'
    elif value == 'Conversion ratio':
      params[index] = 'Conversion\nratio'
    elif value == 'Chances created':
      params[index] = 'Chances\ncreated'
    elif value == 'Shots on target faced':
      params[index] = 'Shots on\ntarget faced'
    elif value == 'Goals prevented':
      params[index] = 'Goals\nprevented'
    elif value == 'Goals conceded':
      params[index] = 'Goals\nconceded'

  #PLOT
  # set figure size
  fig = plt.figure(figsize=(10,10))

  # plot polar axis
  ax = plt.subplot(111, polar=True)
  ax.set_theta_direction(-1)
  ax.set_theta_zero_location('N')

  # Set the grid and spine off
  fig.patch.set_facecolor('#FFFFFF')
  ax.set_facecolor('#FFFFFF')
  ax.spines['polar'].set_visible(False)
  plt.axis('off')

  # Add line in 20, 40, 60, 80
  x2 = np.linspace(0, 2*np.pi, 50)
  annot_x = [20 + x*20 for x in range(0,4)]
  for z in annot_x:
    ax.plot(x2, [z]*50, color='#000000', lw=1, ls='--', alpha=0.15, zorder=4)
  ax.plot(x2, [100]*50, color='#000000', lw=2, zorder=10, alpha=0.5, ls=(0, (5, 1)))
  # Set the coordinates limits
  upperLimit = 100
  lowerLimit = 0

  # Compute max and min in the dataset
  max = maxmin['value'].max()

  # Let's compute heights: they are a conversion of each item value in those new coordinates
  # In our example, 0 in the dataset will be converted to the lowerLimit (10)
  # The maximum will be converted to the upperLimit (100)
  slope = (max-lowerLimit)/max
  heights = slope*maxmin['value'] + lowerLimit
  avg_heights = slope*maxmin['average'] + lowerLimit
  va_heights = maxmin['value']*0 + 90
  #shadow = df.Value*0 + 100

  # Compute the width of each bar. In total we have 2*Pi = 360°
  width = 2*np.pi/len(a_values)

  # Compute the angle each bar is centered on:
  indexes = list(range(1, len(a_values)+1))
  angles = [element*width for element in indexes]

  # Draw bars
  bars = ax.bar(x=angles, height=heights, width=width, bottom=lowerLimit, linewidth=2, edgecolor='#FFFFFF', zorder=3, alpha=1, color=slice_colors)
  #bars = ax.bar(x=angles, height=shadow, width=width, bottom=lowerLimit, linewidth=2, edgecolor='#000000', zorder=2, alpha=0.15, color=slice_colors)

  # Draw scatter plots for the averages and values
  scas_av = ax.scatter(x=angles, y=avg_heights, s=150, c=slice_colors, zorder=5, ec='#000000')
  #scas_va = ax.scatter(x=angles, y=va_heights, s=350, c='#000000',
  #                     zorder=4, marker='s', lw=0.5, ec='#ffffff')

  # Draw vertical lines for reference
  ax.vlines(angles, 0, 100, color='#000000', ls='--', zorder=4, alpha=0.35)

  # Add labels
  for bar, angle, height, label, value in zip(bars,angles, heights, params, a_values):
    # Labels are rotated. Rotation must be specified in degrees :(
    rotation = np.rad2deg((np.pi/2)-angle)
    # Flip some labels upside down
    if (angle <= np.pi/2) or (angle >= (np.pi/2)+np.pi):
        rotation = rotation+270
    else:
        rotation = rotation+90

    # Finally add the labels and values
    ax.text(x=angle, y=110, s=label, color='#000000', ha='center',
            va='center', rotation=rotation, rotation_mode='anchor',
            fontproperties=reg)
    ax.text(x=angle, y=90, s=value, color='#000000', zorder=11, va='center',
            ha='center', fontproperties=bold, bbox=dict(facecolor='#FFFFFF', edgecolor='#000000',
                                                        boxstyle='circle, pad=0.5'))
  if pos=='Goalkeeper':
    fig.text(0.325, 0.9325, "Distribution                                GK Metric                            Defending",
             fontproperties=reg, size=10, color='#000000', va='center')
  else:
    fig.text(0.325, 0.9325, "Attacking                                Possession                            Defending",
             fontproperties=reg, size=10, color='#000000', va='center')

  fig.patches.extend([plt.Circle((0.305, 0.935), 0.01, fill=True, color='#e900ff',
                                    transform=fig.transFigure, figure=fig),
                      plt.Circle((0.490, 0.935), 0.01, fill=True, color='#faeb2c',
                                    transform=fig.transFigure, figure=fig),
                      plt.Circle((0.668, 0.935), 0.01, fill=True, color='#74ee15',
                                    transform=fig.transFigure, figure=fig),
                      plt.Circle((0.15, 0.0425), 0.01, fill=True, color='#000000',
                                 transform=fig.transFigure, figure=fig)])

  fig.text(0.515, 0.985,name + ' - ' + klub, fontproperties=bold, size=18,
           ha='center', color='#000000')
  fig.text(0.515, 0.963, 'Percentile Rank vs League Average '+pos,
           fontproperties=reg, size=11, ha='center', color='#000000')

  fig.text(0.17, 0.04, 'League Average', fontproperties=reg, size=10, color='#000000', va='center')

  CREDIT_1 = 'Data: Lapangbola'
  CREDIT_2 = komp+' | Season 2024/25 | Min. '+str(mins)+' mins played'

  fig.text(0.515, 0.025, f'{CREDIT_1}\n{CREDIT_2}', fontproperties=reg,
           size=11, color='#000000', ha='center')

  DC_to_FC = ax.transData.transform
  FC_to_NFC = fig.transFigure.inverted().transform
  DC_to_NFC = lambda x: FC_to_NFC(DC_to_FC(x))

  logo_ax = fig.add_axes([0.73, 0.015, 0.15, 0.05], anchor = "NE")
  club_icon = Image.open('logo2.png')
  logo_ax.imshow(club_icon)
  logo_ax.axis("off")

  fig.savefig('/content/gdrive/MyDrive/TSG 2024-25/Tim Nasional/Performance Radar/U23/pizza_'+name+'.jpg',
              dpi=500, bbox_inches='tight', facecolor=fig.get_facecolor(), edgecolor='none')

  return fig
```

