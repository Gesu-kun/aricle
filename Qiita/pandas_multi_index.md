# はじめに
複数のキーを使って行を抽出する際に便利なMultiIndexですが、私はその存在を知らずに複数キーを検索していたので、MultiIndexを使う場合と使わない場合の実行速度を比較してみました。

# 実行環境
- Python 3.7.4
- pandas 0.25.1
- Jupyter Notebook 6.0.1  

コードの実行と速度測定はJupyter Notebook上で行いました。  
また、以下のコードは

```Python
import pandas as pd
```

が実行されている前提とします。

# データ準備

```sample.csv
city,year,number,dividion
Tokyo,2019,1,A
Tokyo,2019,2,B
Tokyo,2020,1,C
Tokyo,2020,2,D
Tokyo,2018,1,E
Tokyo,2018,2,F
Kyoto,2019,1,G
Kyoto,2019,2,H
Kyoto,2020,1,I
Kyoto,2020,2,J
Kyoto,2018,1,K
Kyoto,2018,2,L
```

sample.csvをDataFrameに取り込んでみる。

```Python
df1 = pd.read_csv("sample.csv")
```

するとこのように読み込まれます。
<table border="1" class="dataframe">  <thead>    <tr style="text-align: right;">      <th></th>      <th>city</th>      <th>year</th>      <th>number</th>      <th>dividion</th>    </tr>  </thead>  <tbody>    <tr>      <th>0</th>      <td>Tokyo</td>      <td>2019</td>      <td>1</td>      <td>A</td>    </tr>    <tr>      <th>1</th>      <td>Tokyo</td>      <td>2019</td>      <td>2</td>      <td>B</td>    </tr>    <tr>      <th>2</th>      <td>Tokyo</td>      <td>2020</td>      <td>1</td>      <td>C</td>    </tr>    <tr>      <th>3</th>      <td>Tokyo</td>      <td>2020</td>      <td>2</td>      <td>D</td>    </tr>    <tr>      <th>4</th>      <td>Tokyo</td>      <td>2018</td>      <td>1</td>      <td>E</td>    </tr>    <tr>      <th>5</th>      <td>Tokyo</td>      <td>2018</td>      <td>2</td>      <td>F</td>    </tr>    <tr>      <th>6</th>      <td>Kyoto</td>      <td>2019</td>      <td>1</td>      <td>G</td>    </tr>    <tr>      <th>7</th>      <td>Kyoto</td>      <td>2019</td>      <td>2</td>      <td>H</td>    </tr>    <tr>      <th>8</th>      <td>Kyoto</td>      <td>2020</td>      <td>1</td>      <td>I</td>    </tr>    <tr>      <th>9</th>      <td>Kyoto</td>      <td>2020</td>      <td>2</td>      <td>J</td>    </tr>    <tr>      <th>10</th>      <td>Kyoto</td>      <td>2018</td>      <td>1</td>      <td>K</td>    </tr>    <tr>      <th>11</th>      <td>Kyoto</td>      <td>2018</td>      <td>2</td>      <td>L</td>    </tr>  </tbody></table>
このテーブルではdivisionがcity, year, numberの3つのキーによって一意に定められています。
# 手法1：何も工夫しないで複数条件指定する
## csv読み込みも含めた場合の処理速度

```Python
%%timeit
df1 = pd.read_csv("sample.csv")
df1[(df1["city"] == "Tokyo")&(df1["year"] == 2019)&(df1["number"] == 1)].iloc[0]
```

<b>実行結果</b>
2.65 ms ± 48.9 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)  

## csv読み込みを事前に行っていた場合

```Python
%timeit df1[(df1["city"] == "Tokyo")&(df1["year"] == 2019)&(df1["number"] == 1)].iloc[0]
```

<b>実行結果</b>
1.44 ms ± 101 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)  
手法1では大量のデータを処理する場合にかなりの実行時間を必要としたので、手法2を使って処理をしていました。

# 手法2：複合キーのカラムを追加して検索する
city, year, numberの3つの要素を文字列として足し合わせてユニークな要素を作る。

```Python
df2 = pd.read_csv("sample.csv")
# city, year, numberカラムをリストとして取り出す（要素はstrに変換）
cities = list(map(str, df2["city"].values.tolist()))
years = list(map(str, df2["year"].values.tolist()))
numbers = list(map(str, df2["number"].values.tolist()))
# 3つの文字列を足し合わせてユニークなキーを生成する
keys = [city+year+number for city, year, number in zip(cities, years, numbers)]
df2["key"] = keys
```

するとこのようなDataFrameができる。
<table border="1" class="dataframe">  <thead>    <tr style="text-align: right;">      <th></th>      <th>city</th>      <th>year</th>      <th>number</th>      <th>dividion</th>      <th>key</th>    </tr>  </thead>  <tbody>    <tr>      <th>0</th>      <td>Tokyo</td>      <td>2019</td>      <td>1</td>      <td>A</td>      <td>Tokyo20191</td>    </tr>    <tr>      <th>1</th>      <td>Tokyo</td>      <td>2019</td>      <td>2</td>      <td>B</td>      <td>Tokyo20192</td>    </tr>    <tr>      <th>2</th>      <td>Tokyo</td>      <td>2020</td>      <td>1</td>      <td>C</td>      <td>Tokyo20201</td>    </tr>    <tr>      <th>3</th>      <td>Tokyo</td>      <td>2020</td>      <td>2</td>      <td>D</td>      <td>Tokyo20202</td>    </tr>    <tr>      <th>4</th>      <td>Tokyo</td>      <td>2018</td>      <td>1</td>      <td>E</td>      <td>Tokyo20181</td>    </tr>    <tr>      <th>5</th>      <td>Tokyo</td>      <td>2018</td>      <td>2</td>      <td>F</td>      <td>Tokyo20182</td>    </tr>    <tr>      <th>6</th>      <td>Kyoto</td>      <td>2019</td>      <td>1</td>      <td>G</td>      <td>Kyoto20191</td>    </tr>    <tr>      <th>7</th>      <td>Kyoto</td>      <td>2019</td>      <td>2</td>      <td>H</td>      <td>Kyoto20192</td>    </tr>    <tr>      <th>8</th>      <td>Kyoto</td>      <td>2020</td>      <td>1</td>      <td>I</td>      <td>Kyoto20201</td>    </tr>    <tr>      <th>9</th>      <td>Kyoto</td>      <td>2020</td>      <td>2</td>      <td>J</td>      <td>Kyoto20202</td>    </tr>    <tr>      <th>10</th>      <td>Kyoto</td>      <td>2018</td>      <td>1</td>      <td>K</td>      <td>Kyoto20181</td>    </tr>    <tr>      <th>11</th>      <td>Kyoto</td>      <td>2018</td>      <td>2</td>      <td>L</td>      <td>Kyoto20182</td>    </tr>  </tbody></table></tbody></table>
以下、手法1と同様に速度計測を行う。
## csv読み込みも含めた場合の処理速度

```Python
df2 = pd.read_csv("sample.csv")
cities = list(map(str, df2["city"].values.tolist()))
years = list(map(str, df2["year"].values.tolist()))
numbers = list(map(str, df2["number"].values.tolist()))
keys = [city+year+number for city, year, number in zip(cities, years, numbers)]
df2["key"] = keys
df2[df2["key"] == "Tokyo20191"].iloc[0]
```

<b>実行結果</b>
2.5 ms ± 136 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)

## csv読み込みと前処理を行っていた場合

```Python
%timeit df2[df2["key"] == "2019Tokyo1"].iloc[0]
```

<b>実行結果</b>
569 µs ± 5.39 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)

csv読み込みも含めた実行速度は変わらないものの、検索のみの実行速度はマイクロ秒オーダーまで高速化していることがわかります。
# MultiIndexを利用する場合
この方法を今回初めて知ったので、手法2と比べてどちらが早いか検証します。
MultiIndexについて詳しくは[公式ドキュメント](https://pandas.pydata.org/pandas-docs/stable/user_guide/advanced.html)を参照。

city, year, numberの3つをインデックスとして指定すると次のように読み込まれる。

```Python
df3 = pd.read_csv("sample.csv",index_col=["city","year","number"])
```

<table border="1" class="dataframe">  <thead>    <tr style="text-align: right;">      <th></th>      <th></th>      <th></th>      <th>dividion</th>    </tr>    <tr>      <th>city</th>      <th>year</th>      <th>number</th>      <th></th>    </tr>  </thead>  <tbody>    <tr>      <th rowspan="6" valign="top">Tokyo</th>      <th rowspan="2" valign="top">2019</th>      <th>1</th>      <td>A</td>    </tr>    <tr>      <th>2</th>      <td>B</td>    </tr>    <tr>      <th rowspan="2" valign="top">2020</th>      <th>1</th>      <td>C</td>    </tr>    <tr>      <th>2</th>      <td>D</td>    </tr>    <tr>      <th rowspan="2" valign="top">2018</th>      <th>1</th>      <td>E</td>    </tr>    <tr>      <th>2</th>      <td>F</td>    </tr>    <tr>      <th rowspan="6" valign="top">Kyoto</th>      <th rowspan="2" valign="top">2019</th>      <th>1</th>      <td>G</td>    </tr>    <tr>      <th>2</th>      <td>H</td>    </tr>    <tr>      <th rowspan="2" valign="top">2020</th>      <th>1</th>      <td>I</td>    </tr>    <tr>      <th>2</th>      <td>J</td>    </tr>    <tr>      <th rowspan="2" valign="top">2018</th>      <th>1</th>      <td>K</td>    </tr>    <tr>      <th>2</th>      <td>L</td>    </tr>  </tbody></table>
検索を行って速度計測を行う。
## csv読み込みも含めた場合の処理速度

```Python
df3 = pd.read_csv("sample.csv",index_col=["city","year","number"])
df3.loc[("Tokyo",2019,1)]
```

<b>実行結果</b>
2.64 ms ± 148 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
## csv読み込みを事前に行っていた場合

```Python
%timeit df2[df2["key"] == "2019Tokyo1"].iloc[0]
```

<b>実行結果</b>
151 µs ± 12.1 µs per loop (mean ± std. dev. of 7 runs, 10000 loops each)

csv読み込みを含めた実行速度に大きな差はないものの、事前に読み込みを行っていた場合の実行速度は手法2の3.8倍ほど早いという結果になりました。

# まとめ
これまでの結果を表にまとめると以下のようになります。
<table border="1" class="dataframe">  <thead>    <tr style="text-align: right;">      <th></th>      <th>手法1</th>      <th>手法2</th>      <th>手法3</th>    </tr>  </thead>  <tbody>    <tr>      <th>csv読込含む</th>      <td>2.65 ms</td>      <td>2.5 ms</td>      <td>2.64 ms</td>    </tr>    <tr>      <th>検索のみ</th>      <td>1.44 ms</td>      <td>569 µs</td>      <td>151 µs</td>    </tr>  </tbody></table>
私が使っていた手法2はcsv読み込みを含んだ実行時間で言えば若干早いものの、一度読み込んでしまえばMultiIndexを使った方が早いという結果になりました。
ついでに言えば手法2はコードが長くなる上にDataFrameに保持するデータが増えてしまうので、基本的にはMultiIndexを使った方が良さそうです。

# 参照
[pandasのMultiIndexについて](https://qiita.com/ryskchy/items/59028f1a3ed1d433b1a8#multiindex%E3%81%A3%E3%81%A6%E4%BD%95)
[jupyterでコードのスピードを測定する](https://qiita.com/shama/items/980776a6292f3b2e768e)
# はじめに
複数のキーを使って行を抽出する際に便利なMultiIndexですが、私はその存在を知らずに複数キーを検索していたので、MultiIndexを使う場合と使わない場合の実行速度を比較してみました。

# 実行環境
- Python 3.7.4
- pandas 0.25.1
- Jupyter Notebook 6.0.1  

コードの実行と速度測定はJupyter Notebook上で行いました。  
また、以下のコードは

```Python
import pandas as pd
```

が実行されている前提とします。

# データ準備

```sample.csv
city,year,number,dividion
Tokyo,2019,1,A
Tokyo,2019,2,B
Tokyo,2020,1,C
Tokyo,2020,2,D
Tokyo,2018,1,E
Tokyo,2018,2,F
Kyoto,2019,1,G
Kyoto,2019,2,H
Kyoto,2020,1,I
Kyoto,2020,2,J
Kyoto,2018,1,K
Kyoto,2018,2,L
```

sample.csvをDataFrameに取り込んでみる。

```Python
df1 = pd.read_csv("sample.csv")
```

するとこのように読み込まれます。
<table border="1" class="dataframe">  <thead>    <tr style="text-align: right;">      <th></th>      <th>city</th>      <th>year</th>      <th>number</th>      <th>dividion</th>    </tr>  </thead>  <tbody>    <tr>      <th>0</th>      <td>Tokyo</td>      <td>2019</td>      <td>1</td>      <td>A</td>    </tr>    <tr>      <th>1</th>      <td>Tokyo</td>      <td>2019</td>      <td>2</td>      <td>B</td>    </tr>    <tr>      <th>2</th>      <td>Tokyo</td>      <td>2020</td>      <td>1</td>      <td>C</td>    </tr>    <tr>      <th>3</th>      <td>Tokyo</td>      <td>2020</td>      <td>2</td>      <td>D</td>    </tr>    <tr>      <th>4</th>      <td>Tokyo</td>      <td>2018</td>      <td>1</td>      <td>E</td>    </tr>    <tr>      <th>5</th>      <td>Tokyo</td>      <td>2018</td>      <td>2</td>      <td>F</td>    </tr>    <tr>      <th>6</th>      <td>Kyoto</td>      <td>2019</td>      <td>1</td>      <td>G</td>    </tr>    <tr>      <th>7</th>      <td>Kyoto</td>      <td>2019</td>      <td>2</td>      <td>H</td>    </tr>    <tr>      <th>8</th>      <td>Kyoto</td>      <td>2020</td>      <td>1</td>      <td>I</td>    </tr>    <tr>      <th>9</th>      <td>Kyoto</td>      <td>2020</td>      <td>2</td>      <td>J</td>    </tr>    <tr>      <th>10</th>      <td>Kyoto</td>      <td>2018</td>      <td>1</td>      <td>K</td>    </tr>    <tr>      <th>11</th>      <td>Kyoto</td>      <td>2018</td>      <td>2</td>      <td>L</td>    </tr>  </tbody></table>
このテーブルではdivisionがcity, year, numberの3つのキーによって一意に定められています。
# 手法1：何も工夫しないで複数条件指定する
## csv読み込みも含めた場合の処理速度

```Python
%%timeit
df1 = pd.read_csv("sample.csv")
df1[(df1["city"] == "Tokyo")&(df1["year"] == 2019)&(df1["number"] == 1)].iloc[0]
```

<b>実行結果</b>
2.65 ms ± 48.9 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)  

## csv読み込みを事前に行っていた場合

```Python
%timeit df1[(df1["city"] == "Tokyo")&(df1["year"] == 2019)&(df1["number"] == 1)].iloc[0]
```

<b>実行結果</b>
1.44 ms ± 101 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)  
手法1では大量のデータを処理する場合にかなりの実行時間を必要としたので、手法2を使って処理をしていました。

# 手法2：複合キーのカラムを追加して検索する
city, year, numberの3つの要素を文字列として足し合わせてユニークな要素を作る。

```Python
df2 = pd.read_csv("sample.csv")
# city, year, numberカラムをリストとして取り出す（要素はstrに変換）
cities = list(map(str, df2["city"].values.tolist()))
years = list(map(str, df2["year"].values.tolist()))
numbers = list(map(str, df2["number"].values.tolist()))
# 3つの文字列を足し合わせてユニークなキーを生成する
keys = [city+year+number for city, year, number in zip(cities, years, numbers)]
df2["key"] = keys
```

するとこのようなDataFrameができる。
<table border="1" class="dataframe">  <thead>    <tr style="text-align: right;">      <th></th>      <th>city</th>      <th>year</th>      <th>number</th>      <th>dividion</th>      <th>key</th>    </tr>  </thead>  <tbody>    <tr>      <th>0</th>      <td>Tokyo</td>      <td>2019</td>      <td>1</td>      <td>A</td>      <td>Tokyo20191</td>    </tr>    <tr>      <th>1</th>      <td>Tokyo</td>      <td>2019</td>      <td>2</td>      <td>B</td>      <td>Tokyo20192</td>    </tr>    <tr>      <th>2</th>      <td>Tokyo</td>      <td>2020</td>      <td>1</td>      <td>C</td>      <td>Tokyo20201</td>    </tr>    <tr>      <th>3</th>      <td>Tokyo</td>      <td>2020</td>      <td>2</td>      <td>D</td>      <td>Tokyo20202</td>    </tr>    <tr>      <th>4</th>      <td>Tokyo</td>      <td>2018</td>      <td>1</td>      <td>E</td>      <td>Tokyo20181</td>    </tr>    <tr>      <th>5</th>      <td>Tokyo</td>      <td>2018</td>      <td>2</td>      <td>F</td>      <td>Tokyo20182</td>    </tr>    <tr>      <th>6</th>      <td>Kyoto</td>      <td>2019</td>      <td>1</td>      <td>G</td>      <td>Kyoto20191</td>    </tr>    <tr>      <th>7</th>      <td>Kyoto</td>      <td>2019</td>      <td>2</td>      <td>H</td>      <td>Kyoto20192</td>    </tr>    <tr>      <th>8</th>      <td>Kyoto</td>      <td>2020</td>      <td>1</td>      <td>I</td>      <td>Kyoto20201</td>    </tr>    <tr>      <th>9</th>      <td>Kyoto</td>      <td>2020</td>      <td>2</td>      <td>J</td>      <td>Kyoto20202</td>    </tr>    <tr>      <th>10</th>      <td>Kyoto</td>      <td>2018</td>      <td>1</td>      <td>K</td>      <td>Kyoto20181</td>    </tr>    <tr>      <th>11</th>      <td>Kyoto</td>      <td>2018</td>      <td>2</td>      <td>L</td>      <td>Kyoto20182</td>    </tr>  </tbody></table></tbody></table>
以下、手法1と同様に速度計測を行う。
## csv読み込みも含めた場合の処理速度

```Python
df2 = pd.read_csv("sample.csv")
cities = list(map(str, df2["city"].values.tolist()))
years = list(map(str, df2["year"].values.tolist()))
numbers = list(map(str, df2["number"].values.tolist()))
keys = [city+year+number for city, year, number in zip(cities, years, numbers)]
df2["key"] = keys
df2[df2["key"] == "Tokyo20191"].iloc[0]
```

<b>実行結果</b>
2.5 ms ± 136 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)

## csv読み込みと前処理を行っていた場合

```Python
%timeit df2[df2["key"] == "2019Tokyo1"].iloc[0]
```

<b>実行結果</b>
569 µs ± 5.39 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)

csv読み込みも含めた実行速度は変わらないものの、検索のみの実行速度はマイクロ秒オーダーまで高速化していることがわかります。
# MultiIndexを利用する場合
この方法を今回初めて知ったので、手法2と比べてどちらが早いか検証します。
MultiIndexについて詳しくは[公式ドキュメント](https://pandas.pydata.org/pandas-docs/stable/user_guide/advanced.html)を参照。

city, year, numberの3つをインデックスとして指定すると次のように読み込まれる。

```Python
df3 = pd.read_csv("sample.csv",index_col=["city","year","number"])
```

<table border="1" class="dataframe">  <thead>    <tr style="text-align: right;">      <th></th>      <th></th>      <th></th>      <th>dividion</th>    </tr>    <tr>      <th>city</th>      <th>year</th>      <th>number</th>      <th></th>    </tr>  </thead>  <tbody>    <tr>      <th rowspan="6" valign="top">Tokyo</th>      <th rowspan="2" valign="top">2019</th>      <th>1</th>      <td>A</td>    </tr>    <tr>      <th>2</th>      <td>B</td>    </tr>    <tr>      <th rowspan="2" valign="top">2020</th>      <th>1</th>      <td>C</td>    </tr>    <tr>      <th>2</th>      <td>D</td>    </tr>    <tr>      <th rowspan="2" valign="top">2018</th>      <th>1</th>      <td>E</td>    </tr>    <tr>      <th>2</th>      <td>F</td>    </tr>    <tr>      <th rowspan="6" valign="top">Kyoto</th>      <th rowspan="2" valign="top">2019</th>      <th>1</th>      <td>G</td>    </tr>    <tr>      <th>2</th>      <td>H</td>    </tr>    <tr>      <th rowspan="2" valign="top">2020</th>      <th>1</th>      <td>I</td>    </tr>    <tr>      <th>2</th>      <td>J</td>    </tr>    <tr>      <th rowspan="2" valign="top">2018</th>      <th>1</th>      <td>K</td>    </tr>    <tr>      <th>2</th>      <td>L</td>    </tr>  </tbody></table>
検索を行って速度計測を行う。
## csv読み込みも含めた場合の処理速度

```Python
df3 = pd.read_csv("sample.csv",index_col=["city","year","number"])
df3.loc[("Tokyo",2019,1)]
```

<b>実行結果</b>
2.64 ms ± 148 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
## csv読み込みを事前に行っていた場合

```Python
%timeit df2[df2["key"] == "2019Tokyo1"].iloc[0]
```

<b>実行結果</b>
151 µs ± 12.1 µs per loop (mean ± std. dev. of 7 runs, 10000 loops each)

csv読み込みを含めた実行速度に大きな差はないものの、事前に読み込みを行っていた場合の実行速度は手法2の3.8倍ほど早いという結果になりました。

# まとめ
これまでの結果を表にまとめると以下のようになります。
<table border="1" class="dataframe">  <thead>    <tr style="text-align: right;">      <th></th>      <th>手法1</th>      <th>手法2</th>      <th>手法3</th>    </tr>  </thead>  <tbody>    <tr>      <th>csv読込含む</th>      <td>2.65 ms</td>      <td>2.5 ms</td>      <td>2.64 ms</td>    </tr>    <tr>      <th>検索のみ</th>      <td>1.44 ms</td>      <td>569 µs</td>      <td>151 µs</td>    </tr>  </tbody></table>
私が使っていた手法2はcsv読み込みを含んだ実行時間で言えば若干早いものの、一度読み込んでしまえばMultiIndexを使った方が早いという結果になりました。
ついでに言えば手法2はコードが長くなる上にDataFrameに保持するデータが増えてしまうので、基本的にはMultiIndexを使った方が良さそうです。

# 参照
[pandasのMultiIndexについて](https://qiita.com/ryskchy/items/59028f1a3ed1d433b1a8#multiindex%E3%81%A3%E3%81%A6%E4%BD%95)
[jupyterでコードのスピードを測定する](https://qiita.com/shama/items/980776a6292f3b2e768e)
