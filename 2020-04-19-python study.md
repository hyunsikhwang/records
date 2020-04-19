## A New Post

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
