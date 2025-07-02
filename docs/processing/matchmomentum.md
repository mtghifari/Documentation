# Match Momentum 

Match momentum refers to the quantitatve representation that measures the swing of the match and which team is creating threatening situations at certain points in time. It also show important actions at certain time, example: goal, subs, and cards

Steps for python code that used to build match momentum graphic from timeline excel report in each round.

## Creating Momentum Function
```py linenums="1"
def genmomentum(data1, data2):
  '''
  Generating momentum based on passing and attacking thread

  Parameters:
    data1 (pd.DataFrame): First-half event data.
    data2 (pd.DataFrame): Second-half event data.

    Returns:
    A matplotlib figure containing momentum plots for both halves
  '''
  # duplicate to prevent direct modifying
  df1 = data1.copy()
  df2 = data2.copy()

  # label first and second half
  df1['Babak'] = 'Satu'
  df2['Babak'] = 'Dua'
  df = pd.concat([df1, df2], ignore_index=True)

  # extract match and team info
  temp = df1.copy()
  temp['Home'] = temp['Match'].str.split(' vs ').str[0]
  temp['Away'] = temp['Match'].str.split(' vs ').str[1]
  home = temp['Home'].unique().tolist()[0]
  away = temp['Away'].unique().tolist()[0]
  mtch = temp['Match'].unique().tolist()[0]

  # extract passing and zones (active-passive zone)
  '''
  df_match = df.copy()
  df_match = df_match[['Team','Action','Min','Babak','X1','Y1','X2','Y2']]
  df_match = df_match[(df_match['Action']=='passing')]
  '''
  df_match = df.copy()
  df_match = df_match[['Team','Action','Min','Babak','Act Zone','Pas Zone']]
  df_match = df_match[(df_match['Action']=='passing')]

  #Cleaning Data
  shots = df_match.copy()
  shots['Mins'] = shots['Min'].str.split(' : ').str[0]
  shots['Mins'].fillna(shots['Min'], inplace=True)
  shots['Mins'] = shots['Mins'].astype(float)
```
## Calculating expected threads (xT)
Continue from function above

```py linenums="44"
  '''
  Parameters:
    ht = dataframe from first half
    ft = dataframe from second half

    Returns:
    weighted xT momentum dataframe
  '''
  # split xT into first and second half
  ht = shots[shots['Babak']=='Satu'].reset_index(drop=True)
  ft = shots[shots['Babak']=='Dua'].reset_index(drop=True)

  # create each round dataframes and their labels
  data = [ht, ft]
  babak = ['Satu', 'Dua']
  momentum_df = pd.DataFrame()
  
  # loop through each round
  for fh, i in zip(data, babak):
    # get the maximum xT value
    max_xT_per_minute = fh.groupby(['Team', 'Mins'])['xT'].max().reset_index()
    minutes = sorted(fh['Mins'].unique())

    HOME_TEAM = home
    AWAY_TEAM = away
    # weighted xT values
    weighted_xT_sum = {HOME_TEAM: [], AWAY_TEAM: []}
    momentum = []

    # parameters for the function
    window_size = 4
    decay_rate = 0.25

    # calculate momentum for each minute
    for current_minute in minutes:
      for team in weighted_xT_sum.keys():
        # get recent xT values
        recent_xT_values = max_xT_per_minute[(max_xT_per_minute['Team'] == team) &
                                             (max_xT_per_minute['Mins'] <= current_minute) &
                                             (max_xT_per_minute['Mins'] > current_minute - window_size)]

        # apply exponential decay weights
        weights = np.exp(-decay_rate * (current_minute - recent_xT_values['Mins'].values))
        # compute weighted sum of xT values
        weighted_sum = np.sum(weights * recent_xT_values['xT'].values)
        #store the result
        weighted_xT_sum[team].append(weighted_sum)

      momentum.append(weighted_xT_sum[HOME_TEAM][-1] - weighted_xT_sum[AWAY_TEAM][-1])

    # create dataframe with momentum values for each round
    fh = pd.DataFrame({'minute': minutes,'momentum': momentum})
    fh['Babak'] = i
    # append to the full momentum dataframe
    momentum_df = pd.concat([momentum_df, fh], ignore_index=True)
```
## Create Momentum Graphics
Finishing function, return to figure
```py linenums="99"
  # calling visualization code
  from scipy.ndimage import gaussian_filter1d
  fig, axs = plt.subplots(nrows=1, ncols=2, figsize=(20, 10), dpi=500)
  fig.subplots_adjust(wspace=0.05)
  fig.patch.set_facecolor('#FFFFFF')
  axs = axs.flatten()

  # plot minute pole (every 15 minutes)
  m = [0, 15, 30, 45, 60, 75, 90]

  # plot each round
  for i in range(0,2):
    axs[i].set_ylim(-0.3, 0.3)
    axs[i].set_facecolor('#FFFFFF')
    if i == 0:
      auxdata = momentum_df[momentum_df['Babak']=='Satu'].reset_index(drop=True)
      test = aksi[aksi['Babak']=='Satu'].reset_index(drop=True)
      temp = test[test['Mins'].duplicated()==False].reset_index(drop=True)
      d1 = test[test['Mins'].duplicated()==True].reset_index(drop=True)
      d2 = d1[d1['Mins'].duplicated()==True].reset_index(drop=True)
      d3 = d2[d2['Mins'].duplicated()==True].reset_index(drop=True)
      d4 = d3[d3['Mins'].duplicated()==True].reset_index(drop=True)
      rlim = auxdata['minute'].max()
      axs[i].set_xticks(m[:4])
      axs[i].set_xlim(0, rlim)

      DC_to_FC = axs[0].transData.transform
      FC_to_NFC = fig.transFigure.inverted().transform
      DC_to_NFC = lambda x: FC_to_NFC(DC_to_FC(x))

      kons = 7.9
    else:
      auxdata = momentum_df[momentum_df['Babak']=='Dua'].reset_index(drop=True)
      test = aksi[aksi['Babak']=='Dua'].reset_index(drop=True)
      temp = test[test['Mins'].duplicated()==False].reset_index(drop=True)
      d1 = test[test['Mins'].duplicated()==True].reset_index(drop=True)
      d2 = d1[d1['Mins'].duplicated()==True].reset_index(drop=True)
      d3 = d2[d2['Mins'].duplicated()==True].reset_index(drop=True)
      d4 = d3[d3['Mins'].duplicated()==True].reset_index(drop=True)
      rlim = auxdata['minute'].max()
      axs[i].set_xticks(m[-4:])
      axs[i].set_xlim(45, rlim)

      DC_to_FC = axs[1].transData.transform
      FC_to_NFC = fig.transFigure.inverted().transform
      DC_to_NFC = lambda x: FC_to_NFC(DC_to_FC(x))

      kons = 8.1

    # create momentum curve
    auxdata['momentum_smooth'] = gaussian_filter1d(auxdata['momentum'], sigma=1)
    for j in range(len(auxdata)):
      if auxdata['momentum_smooth'][j] > 0:
        axs[i].plot(auxdata['minute'][j], auxdata['momentum_smooth'][j], color='#15AF15')
      else:
        axs[i].plot(auxdata['minute'][j], auxdata['momentum_smooth'][j], color='#AF15AF')

    # fill under momentum curve
    axs[i].fill_between(auxdata['minute'], auxdata['momentum_smooth'], where=(auxdata['momentum_smooth'] > 0),
                        color='#15AF15', alpha=0.5, interpolate=True)
    axs[i].fill_between(auxdata['minute'], auxdata['momentum_smooth'], where=(auxdata['momentum_smooth'] < 0),
                        color='#AF15AF', alpha=0.5, interpolate=True)
    axs[i].axhline(0, color='#000000', linewidth=.5)

    # load and plot for important events (goals, cards, subs)
    dupe = [temp, d1, d2, d3, d4]
    nilai_yh = [0.23-0.035*i for i in range(5)]
    nilai_ya = [-0.32+0.035*i for i in range(5)]
    for x, zh, za in zip(dupe, nilai_yh, nilai_ya):
      for j in range(len(x)):
        if x['Team'][j] == home:
          ymax = 0.945
          y1 = zh
          suf = '-H.png'
        else:
          ymax = 0.055
          y1 = za
          suf = '-A.png'

        if x['Action'][j] == 'goal':
          icon = '/content/gdrive/MyDrive/Data/Liga Indonesia 2024/Database/Icon/Goal'+suf
          axs[i].axvline(x['Mins'][j], color='#000000', lw=1, ymin=0.5, ymax=ymax, zorder=3, ls='--')
        elif x['Action'][j] == 'yellow card':
          icon = '/content/gdrive/MyDrive/Data/Liga Indonesia 2024/Database/Icon/YC'+suf
        elif x['Action'][j] == 'red card':
          icon = '/content/gdrive/MyDrive/Data/Liga Indonesia 2024/Database/Icon/RC'+suf
        elif x['Action'][j] == '2yellow':
          icon = '/content/gdrive/MyDrive/Data/Liga Indonesia 2024/Database/Icon/2Y'+suf
        elif x['Action'][j] == 'subs':
          icon = '/content/gdrive/MyDrive/Data/Liga Indonesia 2024/Database/Icon/Subs'+suf
        elif x['Action'][j] == 'own goal':
          icon = '/content/gdrive/MyDrive/Data/Liga Indonesia 2024/Database/Icon/OG'+suf
        elif x['Action'][j] == 'penalty goal':
          icon = '/content/gdrive/MyDrive/Data/Liga Indonesia 2024/Database/Icon/P'+suf
        else:
          icon = '/content/gdrive/MyDrive/Data/Liga Indonesia 2024/Database/Icon/PM'+suf
        ax_coords = DC_to_NFC([x['Mins'][j]-kons, y1])
        logo_ax = fig.add_axes([ax_coords[0], ax_coords[1], 0.125, 0.125], anchor = "C")
        club_icon = Image.open(icon)
        logo_ax.imshow(club_icon)
        logo_ax.axis('off')

    for k in m:
      axs[i].axvline(k, color='#000000', lw=2, ymin=-0.3, ymax=3, zorder=-2, ls='--', alpha=0.075)

    axs[i].spines['top'].set_visible(False)
    axs[i].spines['right'].set_visible(False)
    axs[i].spines['bottom'].set_visible(False)
    axs[i].spines['left'].set_visible(False)
    axs[i].set_yticks([])
    axs[i].xaxis.set_ticks_position('none')
    axs[i].tick_params(axis='x', colors='#000000')

    for tick in axs[i].get_xticklabels():
      tick.set_fontproperties(reg)
    axs[i].xaxis.set_tick_params(labelsize=15)

  # set title in each round
  axs[0].set_title('FIRST HALF', fontproperties=bold, size=20, alpha=0.35)
  axs[1].set_title('SECOND HALF', fontproperties=bold, size=20, alpha=0.35)
  #fig.suptitle('MATCH MOMENTUM', fontproperties=bold, size=30)

  # set name for x-label (minute) and y-label (attacking threat)
  fig.text(0.5, 0.025, 'Minute', ha='center', fontsize=18, fontproperties=bold)
  fig.text(0.1, 0.5, 'Attacking Threat', va='center', ha='center', fontsize=18, fontproperties=bold, rotation=90)

  # set title for match momentum
  fig_text(0.1, 1, '<'+home+'> vs <'+away+'>', fontproperties=bold, size=32,
           highlight_textprops=[{'color':'#15AF15'}, {'color':'#AF15AF'}], color='#000000')
  fig.text(0.1, 0.93, 'Match Momentum | Liga 1 2024/25', fontsize=22, fontproperties=reg)
  fig.savefig('Match Momentum - '+mtch+'.jpg', dpi=500, bbox_inches='tight', facecolor=fig.get_facecolor(), edgecolor='none')

  return fig
```

Calling match momentum after create function
```py
#excel file and path are adjust based on yours
df1 = pd.read_excel('tl1.xlsx', skiprows=[0])
df2 = pd.read_excel('tl2.xlsx', skiprows=[0])

genmomentum(df1, df2)
```