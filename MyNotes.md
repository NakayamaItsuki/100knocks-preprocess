
# Dockerの起動
```bash
docker compose up -d
```

# Jupyterサーバの環境を利用
1. 既存のJupyterサーバを選択し，URLに下記を入力
    ```
    http://localhost:8888/
    ```
2. /opt/conda/bin/python (localhost)を選択

# CursurのCopilotを停止
左下のCursur TabからDisableを選択

# 重要そうなもの
- カラム名の変更
    ```
    df.rename{'before_name', 'after_name'}
    df.rename(columns={'before_name':'after_name'}, inplace=True)
    ```
    または
    ```
    df.columns = ['column_name1', 'column_name2', ...]
    ```

- SQLのクエリを実行
    ```
    df.query('column_name == "aaa"')
    ```

    query内での変数へのアクセス<br>
    @でアクセス可能
    ```
    df.query('column_name >= @variable')
    ```
- pythonの機能を活用しつつSQLのクエリを実行
    ```
    df.query('column_name.str.startswith("aaa")', engine='python')
    ```

- contain関数
    ```
    df.query('column_name.str.contain(r'^[A-F]')', engine='python')
    ```

    正規表現を活用して，startswithやendswithも代替できる

    正規表現は()で挟むと，配列でアクセスできるようになる．


- apply機能
    ```
    df['column_name'].apply(lambda x:np.mean(x))
    ```
    lambdaだけでなく，一般の関数もOK
    ```
    def func(x):
        return np.mean(x)
    df['column_name'].apply(func)
    ```
    
    rowでのアクセス
    ```
    df['column_name'] = df.apply(lambda row: row['a']+row['b'])
    ```
    関数ならaxis=1がいるっぽい

    複数要素のアクセス，xにリストが入る
    ```
    df['column_name1'] = df.apply['column_name2', 'column_name3'](lambda x: x[0]+x[1])
    ```

- ソート機能
    ```
    df.sort_values(by='column_name')
    ```
    デフォルトでは降順．
    ascending=Falseにすれば昇順になる

    複数要素によるソート
    ```
    df.sort_values(by=['column_name1', 'column_name1'], ascending=[False, True])
    ```

- rank機能<br>
    順位に変換できる

- agg機能
    ```
    df.agg({'column_name_1':'max', 'column_name_2':'min'})
    df.agg({'column_name_1':['max', 'min']}) # カラム名が階層構造になる
    df.agg(column_name=('column_name_1':'max')) # カラム名を指定できる
    ```

<!-- - query内での変数へのアクセス<br>
    @でアクセス可能
    ```
    df.query('column_name >= @variable')
    ``` -->

- JOINについて<br>
    種類
    - inner...どちらにもあるデータ
    - outer...どちらかにあるデータ，片側にしか無いデータはNaNが入る
    - left...左側にあるデータ，右側になければNaN
    - right...右側にあるデータ，左側になければNaN
    - cross...外積，全パターンを作成

    NaNの対応<br>
    ```df.fillna(0)```でNaNに0を代入する

    NaNのあるレコードの削除
    ```
    df.dropna(inplace=True)
    ```

    key名が異なる場合のマージ
    ```
    df_tmp = pd.concat(df1, df2, left_on='key1', right_on='key2')
    ```

- 重複の削除
    ```
    df[~df.duplicated(subset=['column_name1', 'column_name2'])]
    ``` 

    subsetを指定して重複を削除
    ```
    df_tmp = df.drop_duplicates(subset=['column_name1', 'column_name1'], keep=first)
    ```

- レコードごとに処理
    ```
    for index, row in df.iterrows():
        # dfを更新
        df.at[index, column_name] = 処理
    ```
    可能ならapplyの方を使う方が良さそう

- shift機能<br>
    prediodsがズラす幅，axis=0で行，axis=1で列
    ```
    df.shift(periods=1)
    ```

- 集計機能
    行にcolumn_name1，列にcolumn_name2で集計する
    ```
    df_aggregated = pd.pivot_table(
                        df, index='column_name1',
                        columns='column_name2', 
                        values='value_name',
                        aggfunc='sum'
                    ).reset_index()
    ```

- replace機能
    テーブルの全要素で文字の変換を行う
    ```
    df.replace( {'a':'b', 'c':'d'} )
    ```

- stack，unstack機能<br>
    列を行に持っていく
    ```
    df.stack
    ```

    行を列に持っていく
    ```
    df.unstack
    ```

    詳しくは，[pandasでstack, unstack, pivotを使ってデータを整形](https://note.nkmk.me/python-pandas-stack-unstack-pivot/)

- 日付型内での変換
    ```
    df['date'].dt.strftime('%Y%m%d')
    ```

- 文字列型から日付型への変換
    ```
    df['date'] = pd.to_datetime(df['date'], axis=1)
    ```

    UNIX秒の場合はsやmsなどを指定する
    ```
    df['date'] = pd.to_datetime(df['date'], unit='s', axis=1)
    ```

- cut機能
    指定した値で区切って，順にカテゴリ(0,1,2...)を割り振る
    ```
    pd.cut(df['column_name'],[0.0, 5, 10, 15])
    ```

- one-hot encoding
    ```
    df = pd.get_dummies(df, columns=['column_name'])
    ```

- NaNの対応
    ```
    df['column_name'].mean(skipna=True)
    ```

- relativedelta (python)
    差分
    years, months, days
    ```
    relativedelta(date型，date型).years
    ```

    特定の期間を生成
    ```
    relativedelta(days=3)
    ```

- レコードのサンプリング
    ```
    df.sample(frac=0.01)
    ```

- 指定レコードの取り出し
    ```
    df.iloc[index or index_list]
    ```

- 特定の割合に基づくサンプリング
    ```
    _, df_tmp = train_test_split(df, test_size=0.1, 
                                    stratify=df['column_name'])
    ```

- 欠損数のカウント
    ```
    df.isnull().sum()
    ```

- 欠損値の平均値による保管<br>
    pandas
    ```
    df = df.fillna({'column_name1': value1, 'column_name2': value2})
    ```

    sklearn
    ```
    imp_mean = SimpleImputer(missing_values=np.nan, strategy='mean')
    imp_values = imp_mean.fit_transform(df[['column_name']])
    df[['column_name']] = imp_values.round() # 欠損値以外はそのままにしてくれる
    ```

- mask機能による置換
    ```
    df['column_name'] = (df['column_name'].mask(条件，値))
    ```



- 時系列データのsplit
    TimeSeriesSplitを利用
    ```
    tscv = TimeSeriesSplit(gap=0, max_train_size=12, n_splits=3, test_size=6)

    series_list = []
    for train_index, test_index in tscv.split(df):
        series_list.append((df.loc[train_index], df.loc[test_index]))
        
    df_train_1, df_test_1 = series_list[0]
    df_train_2, df_test_2 = series_list[1]
    df_train_3, df_test_3 = series_list[2]
    ```

- アンダーサンプリング
    ```
    # column_nameがnullなら0を，そうでないなら1を格納する
    df['flag'] = np.where(df['column_name'].isnull(), 0, 1)

    rs = RandomUnderSampler(random_state=42)
    df_down_sampling, _ = rs.fit_resample(df, df.flag) # _には
    ```

- 第三正規形への変換
    ```
    df_ = df[['column_name1', 'column_name2']].drop_duplicates()
    df_new = df.drop('column_name2')
    ```

- ファイルの保存と読み込み<br>
    保存
    ```
    df.to_csv(file_path, encoding='UTF-8', index=False)
    ```

    読み込み
    ```
    df_product_full = pd.read_csv(file_path, dtype={'column_name':str})
    ```

    headerがない場合はnamesを指定する必要がある
    ```
    pd.read_csv(file_path, header=None, names=df_product_full.columns, dtype={'category_major_cd':str, 'category_medium_cd':str, 'category_small_cd':str})
    ```

