## Data visualization for analyzing loss ratio

> df = pd.read_csv('./HB_KDB_2.csv', header=None, index_col=[0], encoding='euc-kr')

raw data 에 해당하는 csv 파일을 읽어옴
첫번째 컬럼을 인덱스로 사용하며, 한글 인코딩은 euc-kr

> df_ml = df.iloc[0:2].to_numpy()

처음 2라인을 dataframe 의 multi index 로 사용하기 위하여 numpy array 로 저장

> index = pd.MultiIndex.from_arrays(df_ml)  
> index.set_levels(index.levels[0].astype(int), level=0, inplace=True)  
> df.columns = index  

저장된 numpy array 를 dataframe 의 multi index 로 변환후 index 로 정의함

> df = df.loc[df.index.dropna()]  
> df = df.drop(df.index[0])  

불필요한 내용 삭제

> df1 = df.stack()  
> df1.index.names = ['담보', '구분']  

dataframe stacking 및 인덱스 이름 지정


> def pick_data(toyr, items, ultyr, rprem, cumprofit, amounts):

interactive widget 구현을 위한 function
toyr; Text; amounts 에 적용되는 maximum UY 입력
items; SelectMultiple; 손해율 비교 plot 을 위한 CY 선택
ultyr; Text; ultimate year 입력
rprem; Checkbox; 위험보험료 plot on/off
cumprofit; Checkbox; 누적수지차 plot on/off
amounts; Checkbox; toyr 에 입력된 기간까지의 CY 별 위보 및 수지차 plot on/off


>     global df1, item_saved 
> 
>     # SelectMultiple 의 항목을 유지시켜주기 위한 처리 
>     if items != (2019,):
>         if item_saved in items:
>             item = item_saved
>         else:
>             item = int(items[0])
>     else:
>         item = int(items[0])
>     item_saved = item
>     
>     ult = int(ultyr)
> 
>     df2 = df1.copy()
>     df2 = df2.fillna(0)
>     df2 = df2.drop(('UV',))
>     df2 = df2.astype(float)
>     #df2 = df2.iloc[(df2.index.get_level_values('담보') == risktype) & (df2.index.get_level_values('구분') == item)]
>     
>     # 선택한 항목에 대해서만 필터링
>     # index.get_level_values 는 MultiIndex 처리 함수
>     df2 = df2.iloc[(df2.index.get_level_values('구분').isin(['위험P', '사고C', '수지차']))]
>     
>     # 행과 열을 모두 역순으로 정렬
>     df2 = df2.iloc[::-1, ::-1]
>     
>     # CY컬럼명을 문자에서 숫자로 변환
>     df2.columns = df2.columns.astype(int)
> 
>     df3 = df2.reset_index()
>     df3 = df3.sort_values(['구분', '담보'], ascending=[True, False])
>     
>     df4 = pd.DataFrame()
>     for itm in ['위험P', '사고C', '수지차']:
>         df3_1 = df3[(df3['구분']==itm)].reset_index()
>         for i, idx in enumerate(range(2019, 2014, -1)):
>             df4 = pd.concat([df4, df3_1[idx].shift(idx-2019).rename(itm+str(idx))], axis=1)
>         # NaN 값을 0 으로 변환 처리
>         df4 = df4.fillna(0)
>     
>     # df11; amounts plot 에 대한 dataframe
>     df11 = df[(df.index.values>='1980') & (df.index.values<=toyr)].iloc[:, df.columns.get_level_values(1).isin(['위험P', '수지차'])].astype(float).sum().unstack().reset_index()
> 
>     # Total sum per column: 
>     df4.loc[ult-1, :]= df4.iloc[ult-1:].sum(axis=0)
>     # To drop unnecessary rows
>     df4.drop(df4.index[ult:40], inplace=True)
>     
>     df4.update(df4.filter(regex='수지차').cumsum())
>     
>     # 손해율
>     for idx in range(0, 5):
>         df4[str(2019-idx)]= df4.iloc[:, idx+5] / df4.iloc[:, idx]
>     
>     # 컬럼명으로 정렬
>     df4 = df4.reindex(sorted(df4.columns), axis=1)
>     
>     # df5; plot 에 대한 dataframe
>     # 사고지급금 컬럼만 삭제
>     df5 = df4.iloc[:, :5]
>     df5 = df4[df4.columns.drop(list(df4.filter(regex='사고C')))]
>     #df5.columns = [format(x, 'd') for x in range(2015, 2020)]
>     
>     # 위험보험료 컬럼만 million 으로 나눔(자릿수)
>     df5.update(df5.filter(regex='위험P|수지차').div(1000000))
>     
>     # subplot 갯수 및 사이즈 지정; ax, ax2, ax3; 3개의 plot 동시 표현
>     fig, ax = plt.subplots(nrows=1, ncols=1, figsize=(12, 5))
>     ax2 = ax.twinx()
>     ax3 = ax.twinx()
> 
>     # x label rotation
>     ax.tick_params(axis='x', labelrotation=0)
>     ax.set_xticks(range(1, ult+1))
> 
>     # 0 부터 시작되는 index 를 1부터 시작하도록 인위적으로 증가
>     df5.index += 1
> 
>     # title setting
>     ax.set_title('CY '+str(item)+' / Ultimate year '+ultyr+'(+)', fontsize=15)
>     
>     # plotting
>     # 손해율
>     ax.bar(df5.index.values.tolist(), df5[str(item)], color=np.random.rand(3,), width=0.4)
>     
>     # 위험보험료
>     if rprem == True:
>         ax2.bar([x+0.4 for x in df5.index.values.tolist()], df5['위험P'+str(item)], color=np.random.rand(3,), width=0.4)
>         ax2.get_yaxis().set_major_formatter(mtick.FuncFormatter(lambda x, p: format(int(x), ',')))
>         ax2.grid(axis='y', alpha=0.3)
>         ax2.set_ylabel('Risk premium(KRW million)')
>     else:
>         ax2.yaxis.set_visible(False)
>     
>     # 누적 수지차
>     if cumprofit == True:
>         ax3.plot([x+0.2 for x in df5.index.values.tolist()], df5['수지차'+str(item)], color='r', linestyle=':', linewidth=2)
>         ax.get_yaxis().set_major_formatter(mtick.PercentFormatter(1.0))
>     
>     # 수지차에 대한 축은 숨김처리
>     ax3.yaxis.set_visible(False)
>     
>     # 선택된 모든 CY 위험보험료
>     df5[map(str, list(items))].plot(kind='bar', figsize=(15, 5), width=0.7).grid(axis='y', alpha=0.3)
>     
>     plt.title('Loss ratio comparison', size=15)
>     plt.xticks(rotation=0)
>     plt.xlabel('Policy year')
>     plt.ylabel('Loss ratio')
>     plt.gca().yaxis.set_major_formatter(mtick.PercentFormatter(1.0))
>     
>     ax.set_xlabel('Policy year')
>     ax.set_ylabel('Loss ratio')
>     
>     #df11.drop('level_1', axis=1, inplace=True)
>     df11.set_index('index', inplace=True)
>     df11.columns = ['Risk prem', 'Profit']
>     df11.update(df11.div(1000000))
>     if amounts == True:
>         ax10 = df11.plot(colormap='Pastel2', figsize=(10, 5), kind='bar')
>         plt.title('Upto year '+toyr, size=15)
>         plt.xticks(rotation=0)
>         plt.xlabel('Calendar year')
>         plt.ylabel('Amount(KRW million)')
>         ax10.grid(axis='y', alpha=0.3)
>         ax10.get_yaxis().set_major_formatter(mtick.FuncFormatter(lambda x, p: format(int(x), ',')))
>     
>     # 값에 대한 labeling
>     rects = ax.patches
>     for rect, label in zip(rects, df5[str(item)]):
>         height = rect.get_height()
>         ax.text(rect.get_x() + rect.get_width() / 2, height, "{:.1%}".format(label), ha='center', va='bottom', size=8)
> 
> 
> toyr = widgets.Text(value='2005', placeholder='Type something', description='To year')
> items = widgets.SelectMultiple(options=range(2019, 2014, -1), value=[2019], description='Calendar year')
> ultyr = widgets.Text(value='15', placeholder='Type something', description='Ultimate PY')
> rprem = widgets.Checkbox(value=False, description='Risk premiums')
> cumprofit = widgets.Checkbox(value=False, description='Cumulative profits')
> amounts = widgets.Checkbox(Value=False, description='Amounts')
> txtui = VBox([toyr, ultyr])
> subui = VBox([rprem, cumprofit, amounts])
> ui = HBox([txtui, items, subui])
> out = widgets.interactive_output(pick_data, {'toyr':toyr, 'items':items, 'ultyr':ultyr, 'rprem':rprem, 'cumprofit':cumprofit, 'amounts':amounts})
> 
> display(ui, out)