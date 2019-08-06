---
layout: post
title: "Landsat 데이터 다운로드"
author: "JeonghyunGan"
categories: [Projects, HeatIsland]
---

>
Landsat은 인공 위성을 사용하여 지구를 지속적으로 관찰하기 위한 목적에 따라 USGS와 NASA가 공동으로 개발한 프로그램입니다. Landsat 프로그램은 지표면을 우주에서 가장 오랫동안 지속적으로 기록한 대장정으로서, 1972년 Landsat 1 위성부터 시작되었습니다. Landsat 4부터 시작하여 각 위성은 다중 스펙트럼 및 열 계측기를 사용하여 지표면을 2주에 한 번씩 30미터 해상도로 촬영했습니다. 이 컬렉션에는 Landsat 4, 5, 7, 8을 통해 얻은 전체 USGS 기록 자료가 포함되어 있습니다. 여기에는 전체 운영 기간 동안 35년에 걸쳐 400만장 이상의 고유 장면을 촬영한 데이터가 포함되어 있습니다. USGS 및 NASA의 개방 데이터 정책 덕분에 이 데이터세트는 Google Public Cloud Data 프로그램의 일부로 무료 제공됩니다. 즉, Google Cloud의 일부로 누구나 사용할 수 있습니다.

Landsat 4: 1982 - 1993
Landsat 5: 1984 - 2013
Landsat 7: 1999 - 현재
Landsat 8: 2013 - 현재

|Tier|Description|
|----|-----------|
|Tier 1 (T1) | Data that meets geometric and radiometric quality requirements|
|Tier 2 (T2) | Data that doesn't meet the Tier 1 requirements|
|Real Time (RT) | Data that hasn't yet been evaluated (it takes as much as a month)|

## 0. 패키지 및 설정

```python
from google.cloud import storage, bigquery
import os
import pandas as pd
import numpy as np
```


```python
# 구글 어플리케이션 계정 등록
os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = f"{os.getcwd()}/dsyonsei.json"
```

## 1. 데이터 쿼리

``google.cloud.bigquery``를 통해 필요한 데이터를 쿼리한다.

- WRS 116, 34
- 운량 10% 미만
- 6,7,8 월


```python
# 클라이언트 객체 생성
bigquery_client = bigquery.Client()

# 쿼리문 작성
# 모든 위성에서 wrs좌표와 운량, 계절 조건을 만족하는 영상 선택
QUERY = (
    'SELECT * '
    'FROM `bigquery-public-data.cloud_storage_geo_index.landsat_index` '
    'WHERE wrs_path=116 '
    'AND wrs_row=34 '
    'AND cloud_cover<10 '
    'AND REGEXP_CONTAINS(date_acquired, r"[0-9]{4}-0[678]-[0-9]{2}")'
)

# 쿼리
query_job = bigquery_client.query(QUERY)
```


```python
# 쿼리 결과를 데이터프레임으로 변환
landsat_df = bigquery.table.RowIterator.to_dataframe(query_job.result())
print(landsat_df.shape)
landsat_df.head()
```

    (139, 18)





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>scene_id</th>
      <th>product_id</th>
      <th>spacecraft_id</th>
      <th>sensor_id</th>
      <th>date_acquired</th>
      <th>sensing_time</th>
      <th>collection_number</th>
      <th>collection_category</th>
      <th>data_type</th>
      <th>wrs_path</th>
      <th>wrs_row</th>
      <th>cloud_cover</th>
      <th>north_lat</th>
      <th>south_lat</th>
      <th>west_lon</th>
      <th>east_lon</th>
      <th>total_size</th>
      <th>base_url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>LM51160341984180HAJ00</td>
      <td>None</td>
      <td>LANDSAT_5</td>
      <td>MSS</td>
      <td>1984-06-28</td>
      <td>1984-06-28T01:39:31.0760090Z</td>
      <td>PRE</td>
      <td>N/A</td>
      <td>L1G</td>
      <td>116</td>
      <td>34</td>
      <td>0.0</td>
      <td>38.53812</td>
      <td>36.39226</td>
      <td>125.27394</td>
      <td>128.22986</td>
      <td>9515887</td>
      <td>gs://gcp-public-data-landsat/LM05/PRE/116/034/...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>LM51160341989177HAJ00</td>
      <td>None</td>
      <td>LANDSAT_5</td>
      <td>MSS</td>
      <td>1989-06-26</td>
      <td>1989-06-26T01:39:12.0990000Z</td>
      <td>PRE</td>
      <td>N/A</td>
      <td>L1G</td>
      <td>116</td>
      <td>34</td>
      <td>0.0</td>
      <td>38.56167</td>
      <td>36.41129</td>
      <td>125.09895</td>
      <td>128.05886</td>
      <td>8771784</td>
      <td>gs://gcp-public-data-landsat/LM05/PRE/116/034/...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>LM51160341993236HAJ00</td>
      <td>None</td>
      <td>LANDSAT_5</td>
      <td>MSS</td>
      <td>1993-08-24</td>
      <td>1993-08-24T01:33:26.0240090Z</td>
      <td>PRE</td>
      <td>N/A</td>
      <td>L1G</td>
      <td>116</td>
      <td>34</td>
      <td>0.0</td>
      <td>38.51320</td>
      <td>36.36910</td>
      <td>125.27728</td>
      <td>128.22339</td>
      <td>6735940</td>
      <td>gs://gcp-public-data-landsat/LM05/PRE/116/034/...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>LM21160341981239HAJ00</td>
      <td>None</td>
      <td>LANDSAT_2</td>
      <td>MSS</td>
      <td>1981-08-27</td>
      <td>1981-08-27T00:37:44.0560070Z</td>
      <td>PRE</td>
      <td>N/A</td>
      <td>L1G</td>
      <td>116</td>
      <td>34</td>
      <td>0.0</td>
      <td>38.39297</td>
      <td>36.31878</td>
      <td>137.87572</td>
      <td>140.78645</td>
      <td>7639810</td>
      <td>gs://gcp-public-data-landsat/LM02/PRE/116/034/...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>LM51160341988207HAJ00</td>
      <td>None</td>
      <td>LANDSAT_5</td>
      <td>MSS</td>
      <td>1988-07-25</td>
      <td>1988-07-25T01:42:10.0530010Z</td>
      <td>PRE</td>
      <td>N/A</td>
      <td>L1G</td>
      <td>116</td>
      <td>34</td>
      <td>0.0</td>
      <td>38.53115</td>
      <td>36.38117</td>
      <td>125.13904</td>
      <td>128.09471</td>
      <td>8318018</td>
      <td>gs://gcp-public-data-landsat/LM05/PRE/116/034/...</td>
    </tr>
  </tbody>
</table>
</div>




```python
# date_acquired를 쪼개서 year, month 컬럼 생성
landsat_df = landsat_df.assign(year = [date.split("-")[0] for date in landsat_df['date_acquired']],
                              month = [date.split("-")[1] for date in landsat_df['date_acquired']])
```


```python
landsat_df.groupby(["year", "month", "collection_category", 'spacecraft_id']).size().reset_index()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>year</th>
      <th>month</th>
      <th>collection_category</th>
      <th>spacecraft_id</th>
      <th>0</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1972</td>
      <td>08</td>
      <td>T2</td>
      <td>LANDSAT_1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1976</td>
      <td>07</td>
      <td>N/A</td>
      <td>LANDSAT_2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1978</td>
      <td>07</td>
      <td>T2</td>
      <td>LANDSAT_3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1979</td>
      <td>07</td>
      <td>N/A</td>
      <td>LANDSAT_3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1979</td>
      <td>08</td>
      <td>N/A</td>
      <td>LANDSAT_2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1980</td>
      <td>06</td>
      <td>N/A</td>
      <td>LANDSAT_3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1980</td>
      <td>07</td>
      <td>N/A</td>
      <td>LANDSAT_3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1980</td>
      <td>08</td>
      <td>N/A</td>
      <td>LANDSAT_2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1981</td>
      <td>06</td>
      <td>N/A</td>
      <td>LANDSAT_2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1981</td>
      <td>07</td>
      <td>N/A</td>
      <td>LANDSAT_2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>10</th>
      <td>1981</td>
      <td>08</td>
      <td>N/A</td>
      <td>LANDSAT_2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>11</th>
      <td>1981</td>
      <td>08</td>
      <td>N/A</td>
      <td>LANDSAT_3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1982</td>
      <td>06</td>
      <td>N/A</td>
      <td>LANDSAT_3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>13</th>
      <td>1982</td>
      <td>07</td>
      <td>N/A</td>
      <td>LANDSAT_3</td>
      <td>2</td>
    </tr>
    <tr>
      <th>14</th>
      <td>1982</td>
      <td>08</td>
      <td>N/A</td>
      <td>LANDSAT_3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>15</th>
      <td>1983</td>
      <td>06</td>
      <td>N/A</td>
      <td>LANDSAT_4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>16</th>
      <td>1983</td>
      <td>06</td>
      <td>T2</td>
      <td>LANDSAT_4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>17</th>
      <td>1983</td>
      <td>08</td>
      <td>N/A</td>
      <td>LANDSAT_4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>18</th>
      <td>1983</td>
      <td>08</td>
      <td>T2</td>
      <td>LANDSAT_4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>19</th>
      <td>1984</td>
      <td>06</td>
      <td>N/A</td>
      <td>LANDSAT_4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>20</th>
      <td>1984</td>
      <td>06</td>
      <td>N/A</td>
      <td>LANDSAT_5</td>
      <td>2</td>
    </tr>
    <tr>
      <th>21</th>
      <td>1984</td>
      <td>07</td>
      <td>T1</td>
      <td>LANDSAT_5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>22</th>
      <td>1984</td>
      <td>07</td>
      <td>T2</td>
      <td>LANDSAT_5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>23</th>
      <td>1984</td>
      <td>08</td>
      <td>N/A</td>
      <td>LANDSAT_4</td>
      <td>2</td>
    </tr>
    <tr>
      <th>24</th>
      <td>1984</td>
      <td>08</td>
      <td>N/A</td>
      <td>LANDSAT_5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>25</th>
      <td>1985</td>
      <td>06</td>
      <td>N/A</td>
      <td>LANDSAT_4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>26</th>
      <td>1985</td>
      <td>06</td>
      <td>N/A</td>
      <td>LANDSAT_5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>27</th>
      <td>1985</td>
      <td>06</td>
      <td>T2</td>
      <td>LANDSAT_5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>28</th>
      <td>1985</td>
      <td>07</td>
      <td>N/A</td>
      <td>LANDSAT_4</td>
      <td>2</td>
    </tr>
    <tr>
      <th>29</th>
      <td>1985</td>
      <td>07</td>
      <td>N/A</td>
      <td>LANDSAT_5</td>
      <td>2</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>82</th>
      <td>1997</td>
      <td>07</td>
      <td>T2</td>
      <td>LANDSAT_5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>83</th>
      <td>1997</td>
      <td>08</td>
      <td>T2</td>
      <td>LANDSAT_5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>84</th>
      <td>1998</td>
      <td>06</td>
      <td>T2</td>
      <td>LANDSAT_5</td>
      <td>2</td>
    </tr>
    <tr>
      <th>85</th>
      <td>1998</td>
      <td>07</td>
      <td>T2</td>
      <td>LANDSAT_5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>86</th>
      <td>1998</td>
      <td>08</td>
      <td>T2</td>
      <td>LANDSAT_5</td>
      <td>2</td>
    </tr>
    <tr>
      <th>87</th>
      <td>1999</td>
      <td>06</td>
      <td>N/A</td>
      <td>LANDSAT_5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>88</th>
      <td>2002</td>
      <td>06</td>
      <td>T1</td>
      <td>LANDSAT_5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>89</th>
      <td>2002</td>
      <td>06</td>
      <td>T1</td>
      <td>LANDSAT_7</td>
      <td>1</td>
    </tr>
    <tr>
      <th>90</th>
      <td>2003</td>
      <td>06</td>
      <td>N/A</td>
      <td>LANDSAT_5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>91</th>
      <td>2003</td>
      <td>06</td>
      <td>T1</td>
      <td>LANDSAT_5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>92</th>
      <td>2004</td>
      <td>06</td>
      <td>N/A</td>
      <td>LANDSAT_5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>93</th>
      <td>2004</td>
      <td>06</td>
      <td>T1</td>
      <td>LANDSAT_5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>94</th>
      <td>2006</td>
      <td>06</td>
      <td>T1</td>
      <td>LANDSAT_7</td>
      <td>1</td>
    </tr>
    <tr>
      <th>95</th>
      <td>2006</td>
      <td>08</td>
      <td>T1</td>
      <td>LANDSAT_7</td>
      <td>1</td>
    </tr>
    <tr>
      <th>96</th>
      <td>2007</td>
      <td>08</td>
      <td>T1</td>
      <td>LANDSAT_7</td>
      <td>1</td>
    </tr>
    <tr>
      <th>97</th>
      <td>2009</td>
      <td>06</td>
      <td>T1</td>
      <td>LANDSAT_5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>98</th>
      <td>2010</td>
      <td>06</td>
      <td>N/A</td>
      <td>LANDSAT_5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>99</th>
      <td>2011</td>
      <td>06</td>
      <td>N/A</td>
      <td>LANDSAT_7</td>
      <td>1</td>
    </tr>
    <tr>
      <th>100</th>
      <td>2011</td>
      <td>06</td>
      <td>T1</td>
      <td>LANDSAT_7</td>
      <td>1</td>
    </tr>
    <tr>
      <th>101</th>
      <td>2012</td>
      <td>08</td>
      <td>N/A</td>
      <td>LANDSAT_7</td>
      <td>1</td>
    </tr>
    <tr>
      <th>102</th>
      <td>2012</td>
      <td>08</td>
      <td>T1</td>
      <td>LANDSAT_7</td>
      <td>1</td>
    </tr>
    <tr>
      <th>103</th>
      <td>2015</td>
      <td>07</td>
      <td>N/A</td>
      <td>LANDSAT_8</td>
      <td>1</td>
    </tr>
    <tr>
      <th>104</th>
      <td>2015</td>
      <td>07</td>
      <td>T1</td>
      <td>LANDSAT_8</td>
      <td>1</td>
    </tr>
    <tr>
      <th>105</th>
      <td>2016</td>
      <td>08</td>
      <td>T1</td>
      <td>LANDSAT_8</td>
      <td>1</td>
    </tr>
    <tr>
      <th>106</th>
      <td>2017</td>
      <td>06</td>
      <td>T1</td>
      <td>LANDSAT_7</td>
      <td>1</td>
    </tr>
    <tr>
      <th>107</th>
      <td>2017</td>
      <td>06</td>
      <td>T1</td>
      <td>LANDSAT_8</td>
      <td>1</td>
    </tr>
    <tr>
      <th>108</th>
      <td>2017</td>
      <td>08</td>
      <td>RT</td>
      <td>LANDSAT_8</td>
      <td>1</td>
    </tr>
    <tr>
      <th>109</th>
      <td>2017</td>
      <td>08</td>
      <td>T1</td>
      <td>LANDSAT_8</td>
      <td>1</td>
    </tr>
    <tr>
      <th>110</th>
      <td>2018</td>
      <td>06</td>
      <td>T1</td>
      <td>LANDSAT_7</td>
      <td>1</td>
    </tr>
    <tr>
      <th>111</th>
      <td>2019</td>
      <td>06</td>
      <td>T1</td>
      <td>LANDSAT_8</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>112 rows × 5 columns</p>
</div>



## 1. 데이터 다운로드

이제 걸러진 인덱스 데이터를 이용하여 실제로 위성사진을 다운로드하는 함수를 작성한다. 다운로드 함수의 개요는 다음과 같다.

1. 데이터에 해당하는 BaseURL의 리스트를 인자로 전달받는다.
2. 각 행에 해당하는 데이터의 prefix를 설정한다.
3. prefix를 통해 필요한 데이터를 다운로드한다.

먼저 google storage에 접근할 수 있는 클라이언트 객체를 생성하고, 랜샛 데이터가 담긴 버킷을 가져온다.


```python
storage_client = storage.Client()
bucket = storage_client.get_bucket('gcp-public-data-landsat')
```

 다음으로 필요한 데이터의 prefix를 설정해준다. prefix에 대한 정보는 `base_url` 컬럼으로부터 가져올 수 있다. 예를 들면 다음과 같은 형태이다. 'gcp-public-data-landsat'은 버킷 이름을 가리키고, 'gcp-public-data-landsat' 이하의 문자열이 해당 데이터의 prefix가 된다.


```python
landsat_df.base_url[0]
```




    'gs://gcp-public-data-landsat/LM05/PRE/116/034/LM51160341984180HAJ00'



 따라서 모든 베이스 URL에 대해서 prefix는 다음과 같은 형태로 표현할 수 있다.


```python
landsat_df.base_url[0].split("gcp-public-data-landsat/")[1]
```




    'LM05/PRE/116/034/LM51160341984180HAJ00'



지금까지 완성된 prefix에는 지표온도를 계산하는데 필요하지 않은 밴드들 역시 포함되어 있다. 따라서 지표온도를 계산하는데 필요하지 않은 밴드들을 걸러내는 작업이 필요하다. 위성별로 지표온도 계산에 필요한 밴드는 다음과 같다. 각 위성별로 필요한 밴드를 알 수 있으므로 이에 맞게 prefix를 조정한다.

|Spacecraft|Bands required|
|---|---|
|Landsat4||
|Landsat5||
|Landsat7||
|Landsat8|Band 4, 5, 10|


```python
def get_prefix(url):
    url = url.split("gcp-public-data-landsat/")[1]
    ID = url.split("/")[-1]        
    if url.split("/")[0] == "LC08":
        return pd.Series([f"{url}/{ID}_B4.TIF", f"{url}/{ID}_B5.TIF", f"{url}/{ID}_B10.TIF"])
    else: raise KeyError(f"{url}에 해당하는 데이터가 없습니다.")
```


```python
sample = get_prefix(landsat_df.base_url[137])
sample
```




    0    LC08/01/116/034/LC08_L1TP_116034_20160807_2017...
    1    LC08/01/116/034/LC08_L1TP_116034_20160807_2017...
    2    LC08/01/116/034/LC08_L1TP_116034_20160807_2017...
    dtype: object




```python
def download_prefix(prefix):
    directory = prefix.split('/')[-2]
    filename = prefix.split('/')[-1]
    if not os.path.exists(f"Downloads\\{directory}"):
        os.mkdir(os.path.join(f"Downloads\\{directory}"))
    blob = bucket.blob(prefix)
    blob.download_to_filename(f"Downloads\\{directory}\\{filename}")
```


```python
sample.map(download_prefix)
```




    0    None
    1    None
    2    None
    dtype: object




```python
def download_landsat(url_list):

    """
    google storage의 gcp-public-data-landsat으로부터 랜샛 데이터를 다운로드하는 함수이다.

    base_url을 인자로 넣어준다.
    """

    def get_prefix(url):
        url = url.split("gcp-public-data-landsat/")[1]
        ID = url.split("/")[-1]        

        if url.split("/")[0] == "LC08":
            return [f"{url}/{ID}_B4.TIF", f"{url}/{ID}_B5.TIF", f"{url}/{ID}_B10.TIF"]

        else: raise KeyError(f"{url}에 해당하는 데이터가 없습니다.")

    def download_prefix(prefix_list):
        for prefix in prefix_list:
            blob = bucket.blob(prefix=prefix)
            blob.download_to_filename(f"Downloads\\{}\\{prefix}")




    # 데이터를 저장할 다운로드 디렉토리 생성
    if not os.path.exists("Downloads"): os.mkdir("Downloads")

    # 스토리지 클라이언트/랜샛 버킷 객체 생성
    storage_client = storage.Client()
    bucket = storage_client.get_buccket('gcp-public-data-landsat')

    for url in url_list:
        prefix_list = get_prefix(url)   
```


```python

def download_landsat(product_id, WRSpath, WRSrow):

    """
    google storage의 Landsat8 > WRS(116,34) 데이터(사진과 메타데이터)를 다운로드

    WRS좌표를 세 자리 문자열로 입력

    prefix 수정해서 다른 밴드 다운로드
    """

    storage_client = storage.Client()
    bucket = storage_client.get_bucket('gcp-public-data-landsat')
    bands = ["_B4, _B5, _B10"]

    for band in bands:
        prefix=f"LC08/01/{WRSpath}/{WRSrow}/{product_id}/{product_id}{band}.TIF"
        blobs = bucket.list_blobs(prefix=prefix)

        for blob in blobs:
            if product_id not in os.listdir():
                os.mkdir(product_id)
            filename = blob.name.split("/")[-1]
            blob.download_to_filename(f"{os.getcwd()}\\{product_id}\\{filename}")
```


```python

```
