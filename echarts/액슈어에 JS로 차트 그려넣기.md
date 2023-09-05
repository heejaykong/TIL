# 액슈어에 JS로 차트 그려넣기

### 이 모든 것의 시작을 부른 하나의 블로그 글: [Axure Tutorial: Use JavaScript in Axure to Make Data Visualization Chart](https://axureboutique.com/en-kr/blogs/axure-tutorials/use-javascript-in-axure-to-call-echarts-charts)

## 참고 자료
⚠⚠⚠⚠ 액슈어에 코드 복붙할 때는 반드시 주석 제거하기 ⚠⚠⚠⚠

![image](https://github.com/heejaykong/TIL/assets/18097984/95bda9e3-0b0b-4b8e-aa55-4e72937d9c3c)

* 파이차트
  * [공식문서](https://echarts.apache.org/en/option.html#series-pie)
  * [연습장](https://echarts.apache.org/examples/en/editor.html?c=pie-simple&code=PYBwLglsB2AEC8sDeBYAULWBTAtiAFgIYDOExAXMuppgCZmEBGANlrZWAE4CuW1sAXwA06fsSycIWCrADaojDVSKasaIRxZKAcgCS0aBNgAFKbADCRTmG0iVNMAE8QW2NpBTb_VZ0L1uMtoAHAAMAKRe9pgunADGWNBgxpxYsWRQ0DoAjJGqmMxMWMyUynk0IMCkkDA6ENCktFi5ZbAAZsCcOIRgYBI6SLQCEXYtsLHAzB06AO74EL3NZe2JAMoQAF6uWVkjecLeNLTdhJTyUUqwAG6EzLyUAKwAzEJqGq7aACoAatqCu2VIK43O6wR47V6aHQAWRuzF--3OmEB11urgATM8Ie8AKK0bixboZeH_PLI4Gue4vdSQtwAKWAjGJB1UZNRlEx1Pe-l6kg6TPOAF0SQiFCzmVidKYsBYrDYSZgnC4dB4mvLYAVGEUSuLMO1Ot0ef1GBNaB8sAAPMAAHyQjAEgCx_gRIEAFOpmy022ICASLPKMQixADWAHNOMBuNBaOYJlM3HV8BJ5r7VMbOI1OAAlPwQAKUNFq6J-ejQYOnABsLyCLwALJWhTrYJJYvhtYjyq7oO6wK3RqpxpNODoAMStUdj5MtZZgNabShZfMNzAI3uwY3MU0W7tUNt9mODtwjsejidLGDTjZbBc7mhTgDqWAgwfwW-0a9oJ5oy5aAnFX9URzAE45AbVkQSeKk3h0b5-VGUDXDBCCaW0GFmDhP4QKBNlYAxRCcTxAlqmgGCWjgh5cJ0elGXQttSNBci3G5RM-Wosp63sL8BXQAQAG4gA)
  * <details>
      <summary>코드</summary>
    
    ```javascript
    option = {
      emphasis: {
        disabled: true
      },
    
      series: [
    
        {
          name: 'Inner Pie Chart',
          type: 'pie',
          radius: '80%',
          percentPrecision: '1',
          label: {
            position: 'inside',
            formatter: '{d}%',
            color: 'white',
            fontSize: 11,
          },
          data: [
            { value: 53, name: 'TV' },
            { value: 31, name: 'Mall' },
            { value: 23, name: 'Education' },
            { value: 5, name: 'Job' },
            { value: 3, name: 'Interior' },
          ],
        },
    
    
        {
          name: 'Pie Chart',
          type: 'pie',
          label: {
            formatter: '{boldText|{b}：}{plainText|{c}}',
            backgroundColor: 'inherit',
            borderRadius: 2,
            padding: [6, 8, 4, 8],
            rich: {
              plainText: {
                color: '#ffffff',
                fontSize: 12,
              },
              boldText: {
                color: '#ffffff',
                fontSize: 12,
                fontWeight: 'bold',
              },
            }
          },
          data: [
            { value: 53, name: 'TV' },
            { value: 31, name: 'Mall' },
            { value: 23, name: 'Education' },
            { value: 5, name: 'Job' },
            { value: 3, name: 'Interior' },
          ],
        },
      ]
    };
    ```
    </details>

---

![image](https://github.com/heejaykong/TIL/assets/18097984/f8dfaa18-1d75-4349-a616-ed5d7bc49094)

* 막대차트
  * [공식문서](https://echarts.apache.org/en/option.html#series-bar)
  * [연습장](https://echarts.apache.org/examples/en/editor.html?c=bar-y-category-stack&code=PYBwLglsB2AEC8sDeBYAULWBzAThAJgFzLqaZijEDkADAKRUA0pZeWAFmNQKwPMZkARsDAUAttQDMfFpgA2AUwBmXWLRkDMAYxhgAhhGgAZPYIVziYHAFcFLAL79MADwCCziAGdiqTbDAAniAK1ABuenK2VA5OsAHuXj6y_kEhalp6YApYwDgBTMn4mXrEANpUACoAakxqgC7NgDRjgA1dtVSACrWAhOOtdYCF462AHuOAMouANZ2ALaNUALqxmIahCjieaVRWUbGOLIt4Ct6wpcm-ZGTQemJLAMoKejha7LAAotBYhgoFfuSp1IJXr4eYnvpaADW1Ao-jkP1-clM5iSbzInnYwAA7pYbAppr8lLkxJksjhiEprNAtJAYAAKIr6ACUB1-h081jECFg3EksAA1LBJABGDmwABMbM53D5kgA3Mk6cFrgpoGBmZS9AA6cKRBSwAD0sAZTIAVLBuTQaBK4YccAowNYcHAAAYAEiQipVEVsSooRmAGUUZyshiwZKp9lgZId0q0srAbuAADEIM4FPgydzA3QqTbJWR1nCs3SFGIQOw9J5EiRTbAsVprDsqJsINtotmMZhFWUM5haXTMKrbMRWU3fhAsmIfQFFD4dHJcoQqABibgAFgA7DQtAA2Kg5zubukdzvdtI8_uHQd5kdj5ATqezgCc3K0WkX3A3R9g29-u7p--IgpfMyHZ7SJBL3xWclD0LQAA5uAg5821fF8P1-L8WV_WAT2HQJzyA4BJxAmcFAUVciPXN9DlIshEMOZDJFQ9CAPHHCrxnRdJC0Gh8BecjMHIiZknsFheLQewxSAA)
  * <details>
      <summary>코드</summary>
    
    ```javascript
    option = {
      grid: {
        top: '0%',
        right: '5%',
        bottom: '3%',
        left: '0%',
        containLabel: true
      },
      xAxis: {
        type: 'value'
      },
      yAxis: {
        type: 'category',
        data: ['TV', '덴올몰', '교육', '덴잡', '인테리어'],
        inverse: 'true',
      },
      series: [
        {
          name: 'Search Engine',
          type: 'bar',
          stack: 'total',
          label: {
            show: true,
            formatter: function(data){
              sum = 53 + 31 + 23 + 5 + 3;
              percent = data.value / sum * 100;
              return `${data.value.toLocaleString()} (${percent.toFixed(1)}%)`
            },
          },
          emphasis: {
            focus: 'series'
          },
          data: [
            {
              value: 53,
              itemStyle: {color:'#5470c6'},
            },
            {
              value: 31,
              itemStyle: {color:'#91cc75'},
            },
            {
              value: 23,
              itemStyle: {color:'#fac858'},
            },
            {
              value: 5,
              itemStyle: {color:'#ee6666'},
            },
            {
              value: 3,
              itemStyle: {color:'#73c0de'},
            },
          ]
        }
      ]
    };
    ```
    </details>

---

![image](https://github.com/heejaykong/TIL/assets/18097984/2227ec0b-b42e-4f9d-85cb-1b2d4236ba9c)

* 히트맵
  * [공식문서](https://echarts.apache.org/en/option.html#series-heatmap)
  * [연습장](https://echarts.apache.org/examples/en/editor.html?c=heatmap-cartesian&code=MYewdgzgLgBFCGAnA5gUyhGBeGBtAsAFAwkwDkgHuOAyi4DWdgLaMC0gO0OCvNYDUDgAh1kA0Rp5AFQBqDQCHjgGFXAAe2AOpZiAMIcAEQz0EiJMmIBeewAUzyssIaAcCcARK4BjBwDg1gQDHAgwO79gDTnAPzWBemou2RgBVrAqBM8-pMoAauhgEAZRgASQiYYIBBXUDggHkwuIZohgTg4J9iP0AXZsBC8YZAG1qKIsBByd0ChkARGcBKsZZAA5rKwopAGvHPDWz-AIZcAGF4ADdUAA8AXRhogBtkAEsweChUGAA6NZSACgBxPoBKVMQQAFt4GAAxedQ1leyxgG4iIlBIWHmlxFRoTBwCHJIyRwuGAbACMACZdro1LINgBmACckO45AoVgA9NpgQB2JHkLzAsEgpG-f4lcrAxG6aj0Am4sjBADqtN0gA1xwC-o0ZgUTdO5AITjFLpZkAC2NcyFEe6PQhotEwQAio4AM9pgAA1AKmzgBXRuWAFm7AME1gBHmmCAHRWmIANVcAA5MwQBSo4AGOpggBBxwA6qzBACSDnkAoeOACabACdNKxgAB4GDAAITB4OAQ1HABFDdBggFeewCZvTAmDBAD8TgFxBmCAWnqYKbAA1jVpDwae4GgMAAJgBXI5HACeAH0AGYgRA1gBGICgUGONZGNfgI1m3zwJPIuhHyLIo-HY4n46nvEIEsIRGleFV8xLo2RAE0VWuNzBAA7NgBdxwAMixNADhDgB5xwA6HTBD4ALjrogBHJmBgwAUMyLAAw9gBfRj3ev0B_NAwjKM4wTZM00zHM83zQsXlLRZTh-XxcAABm4AAWbhwTGOcSFwEEsO4eEcLwAiwQwkj8O4WEsIAVkogiADYiIY7gsW4MEcOQ8jwW4RjKJ4miQS4nJcBo2isM45ExOoljcLwTD4SwyjMJBND6Ok1SCNhES8IktCpOQ5iaMM0T2KE3S8AADkInTpJs8j-Pk3AlKU4S5zGFYTgABw2OsyzAYAoFmcBgVmJYjl2GAAG8hw-KAy0QMA8HC1AjlQkjUvS9yYCy3BOJgAAfQryAYMgFwAX12B5CBAbzgtCnBYpyYKoCmVAAC4YqHdq6ygLqyGAVAwHeLpSCWEZ-vIFCrJ9EFaJgAA_GAZp9MFYTIXwKvkjsQCmYLvK65r-G8kAIHCkKwAGjtvM2nJtt8ZBEFmEsjoenIRmiPsIC63BjvGqtvM68hgEWVBkEbKsxpIEsEK68tK1rBsm1bdtO27Xt-3kkgIG8qZwuiD54COoccYACxAAB3LqoEQMtUCHCqtvk_6SCgQHgcGsGIcQKHsZgTGIAAGXgZtUCmI7cpG1BEEGeAJZW5FDgQJYupQmB3v4WGEC6t4Zc-DB-dx_GoEJ1Bie6v5SAgCnqbgOmGatpn7ssqsvv7EmrfZoGBtBpYeb5odBZFsWFeiqX3jlhX1c10htYthAUHQCAjbxgmic9_hyapmmHcZ5nfEGfsy3lgBZeBDst_gjnmNX-ZOEYuvm_nQamYAyymUX2tz-n-cbWZhqmsgKeegAvcAECmaG4DqgaGFolCAFIxtjiAZYHn7Byt1nSAWI5OcARAnAFjB-Rp7ZjmBrJ82oB8s_4J1-_4H5khO9DzOs-zu3afp0mNefmA0reTJvAc6m8d78CysEdm3cq4f2tsAksVMABCUxEpNzQr_fgNt4CIMpn0PajYBooGbPADYaFFYUJQisWiYorb8Gdh_BhpAmFjCIBVO4QA)
  * <details>
      <summary>코드</summary>
    
    ```javascript
    const targets = [
        '인테리어-시공사례',
        'TV-임플란트 수술', 'TV-임플란트 보철', 'TV-제품소개영상', 'TV-치과경영', 'TV-교정',
        '몰-TS III SA', '몰-SOI', '몰-A-OSS',
        '덴잡-구인구직', '덴잡-채용공고', '덴잡-인재정보',
        '몰-[Cavex] Alginate ...', '몰-(GC)-Aroma Fine...',
    ];
    
    const interests = [
        '치과경영 (12)', '임플란트 (39)', '인상/보철 (7)', '교정 (21)',
        '구인구직 (9)', '인테리어 (2)', 'SW (2)', '의약품 (1)', '교육 (9)', '개원 (1)'
    ];
    
    // 아래 X축엔 아무것도 표시하지 않기 위한 설정입니다. <- !!!!!액슈어 복붙 시 주석 꼭 지우기!!!!!
    const dummy_for_bottom_x_axis = [
        '', '', '', '', '', '', '', '', '', '',
    ];
    
    // [X축index, Y축index, 데이터] 순으로 이루어진 2차원 배열입니다. <- !!!!!액슈어 복붙 시 주석 꼭 지우기!!!!!
    const data = [
      [0,4,12],
      [1,1,9], [1,2,4], [1,3,15], [1,6,9], [1,7,2],
      [2,12,6], [2,13,1],
      [3,5,12], [3,3,9],
      [4,9,1], [4,10,5], [4,11,3],
      [5,0,2],
      [6,3,2],
      [7,3,1],
      [8,1,3], [8,2,6],
      [9,9,1],
    ].map(function (item) {
        return [item[0], item[1], item[2] || '-'];
    });
    option = {
      title: {
        left: 'center',
        text: '08. 15 ~ 08. 23'
      },
      tooltip: {
        position: 'top'
      },
      grid: {},
      xAxis: [{
        type: 'category',
        data: dummy_for_bottom_x_axis,
        splitArea: {
          show: true
        }
      },
      {
        type: 'category',
        axisLabel: { interval: 0, rotate: 0 },
        data: interests,
        splitArea: {
          show: true
        }
      }],
      yAxis: {
        type: 'category',
        axisLabel: { interval: 0 },
        data: targets,
        splitArea: {
          show: true
        }
      },
      visualMap: {
        min: 0,
        max: 15,
        calculable: true,
        orient: 'horizontal',
        top: '-50%',
      },
      series: [
        {
          name: '접속 수',
          type: 'heatmap',
          data: data,
          label: {
            show: true
          },
          emphasis: {
            itemStyle: {
              shadowBlur: 10,
              shadowColor: 'rgba(0, 0, 0, 0.5)'
            }
          }
        }
      ]
    };
    ```
    </details>

---

![image](https://github.com/heejaykong/TIL/assets/18097984/bbd32fce-9730-45cd-813b-fa658686807c)

* 다중꺾은선그래프
  * [공식문서](https://echarts.apache.org/en/option.html#series-line)
  * [연습장](https://echarts.apache.org/examples/en/editor.html?c=dataset-link&code=PYBwLglsB2AEC8sDeBYAULWAbApgcx2gBMAuZAXwBp1MxhgtIQzUNNYwAnCPAzsgOQBDAB4QAzgOptM4gBbAA7gGEYYQmDJcArjhqwq-okLBDxOTcn2zg2zgGMcZANrX2zgSE7Ai2-2ClYAQAGAA4AWgBGAFYACkAVNYBKQJCIyIA2WMAMIeTKILCogHZYwEqu3Py00NjACDry1KiATljAAsW6goAmYNjAH3G2iPbI2MAVseSAXWl2dwFnAGkASQAVUdgAFlDo-YWAfQBlBU4wWDmAWxAsIWhDwF2hwF7OwBFx2EAQccAdVdhAHaHAAiHAEoXADqXA4Ly7TykWBeQBsHBwXGbkwHgACpwcOEALK2S5HU7ndG3QAANYAXccAOy2wQAxg4AcGv-YMBoIhlIh0JkU0AIeOAGFXAAHtP1gt1ggBAJvKAH4msrBAJQ9gFSeh6AGVHACQdBJugAmmwAnTQA6CmwEGwIE0zWQ-mTWECXzHY4AT1g4lsDhwKshVNVNqhE3Yo30hjYIgAgmJxCwOEaQE4gvYTPhgJwjQIDA6jR6JN68NwiHNiDgRGRghH9HGIKQKA7zNwcF7YK4GaxdT6_YIsBBoJaHZNxMd6GA5FpOLo6-w8xACwAZIRG2xgABCRsE3kUUhhsBwpzkZhjyFgADNgPZtIWBF2C-HyG4XZNS7qwL7_QIqzXJwzZI3gM3W-2p1vxH2B9ph6OguPL2WZyA5-IFyQZdV3XQQnx3PcO0PSZjwrIJz1rR8bzvDg2xwDtZBwfNn37QcRzHJRv11X9_0A4C1w3cCDEgtxoPYWDTwQoj62QltUIfK9TSw7scNfd8CInDDp1nedCyAlcKLA7jt2ohl91gJ00HIABuIA)
  * <details>
      <summary>코드</summary>
    
    ```javascript
    option = {
      legend: {},
      tooltip: {
        trigger: 'axis',
        showContent: true
      },
      dataset: {
        source: [
          ['product', '08-15(화)', '08-16(수)', '08-17(목)', '08-18(금)', '08-19(토)', '08-20(일)', '08-21(월)'],
          ['[KIT] 485KIT_Short Implant 식립을 위한 시술키트', 0, 2, 1, 1, 0, 0, 0],
          ['Pre-Mount Implant 식립가이드 소개', 0, 0, 2, 1, 0, 0, 0],
          ['임플란트 식립 전, 주수 방법을 알려드립니다.', 0, 1, 2, 0, 0, 0, 0],
          ['dummy source', 0, 0, 0, 2, 1, 2, 0],
        ]
      },
      xAxis: { type: 'category' },
      yAxis: { gridIndex: 0 },
      grid: {},
      series: [
        {
          type: 'line',
          smooth: true,
          seriesLayoutBy: 'row',
          emphasis: { focus: 'series' }
        },
        {
          type: 'line',
          smooth: true,
          seriesLayoutBy: 'row',
          emphasis: { focus: 'series' }
        },
        {
          type: 'line',
          smooth: true,
          seriesLayoutBy: 'row',
          emphasis: { focus: 'series' }
        },
        {
          type: 'line',
          smooth: true,
          seriesLayoutBy: 'row',
          emphasis: { focus: 'series' }
        },
      ]
    };
    ```
    </details>

---

![image](https://github.com/heejaykong/TIL/assets/18097984/90dd557d-450b-413f-89a4-fa4b8a63ca3f)

* 스캐터차트
  * [공식문서](https://echarts.apache.org/en/option.html#series-scatter)
  * [연습장](https://echarts.apache.org/examples/en/editor.html?c=scatter-simple&code=MYewdgzgLgBAygWQIIBkUH0AiB5AKuuASQC0BRGAXhgA4BuAWACgQAHKAS3EpgG8mYYADySD2EAFy9-AmAFt2YSQFoAjAAYANNIGyAhoMkAmNdIC-WxgICeIsZL6WZ8xTCUBWTdrn6jHsxYEIAFMAJ3YgiRgAbS8HGRkoKxYgyQByCGBdKChQ1ID4mAATLN1JGMcCqQrKmAA3XQAbAFcU6JUAZg0YFQBOHq6VAHY1AF18mpgwXVlW1Lh2vK948yWZOIm6xpaylQA2AYAWTXhkNCw8AhJSMdX4qZm0xerl8YL1ifrm1qj97o8Bjw3Z4Fe6zBALV4vW5VDabL47FRdajHDqjSEg6azAAKEOhK2BAneNU-2zaBwG_ROqAwOHwRDIQNhoMe6IE-ImRMqJO-KjcXVUXUQ1POdKujI2zJgqSeE3ZNU5BW5O0GA12hkFpxpF3p11ZMkl0r1csqCviSuifJoxyFZ1plwZeoEBplNWNb2hAnNURVrl5GuFdp14omzqNetNMi9Apghk6VNt2rFjsmmJZePDHrhpKi7n57WtmpF9t1mdD6ehEc9W2-Sl-6n9CdFDtLqalLsqbvilazNd-qgLAcTzYJ-tbhvLI-7UZ9SnaiPjWqbJZHTrH7YKnbWmaj1H5xgbi-LwZqZZHm-PAgaugARkEGvZoRAABYgADukigIRaeoAZiAQno2ShJIP5NGAwAcOAAAUxRQLoACUMIbCEQRQE0IRgEUJQAHSggwZ6_uAUAAOpBOwADmT5QJIABE14gA0hQ0YRYBQHA7AAF6tIYfKrJuEBWLI9ENOxXEgWBEGcJhMElIhEYoWhGFYXBUSGCM-EduioANP-ABCVi0bBujMV4pjSECpi0EAA)
  * <details>
      <summary>코드</summary>
    
    ```javascript
    const SMALL_DOT_SIZE = 8;
    option = {
      xAxis: {
        min: -10,
        max: 20
      },
      yAxis: {
        min: -50,
        max: 250
      },
      series: [
        {
          type: 'scatter',
          data: [
            {
              value: [13, 199, 170],
              name: 'S3',
            },
            {
              value: [16, 140, SMALL_DOT_SIZE],
              name: '',
            },
            {
              value: [6, 150, 150],
              name: 'M3',
            },
            {
              value: [11, 80, 130],
              name: 'P3',
            },
            {
              value: [14, 19, SMALL_DOT_SIZE],
              name: '',
            },
            {
              value: [15, -1, SMALL_DOT_SIZE],
              name: '',
            },
            {
              value: [17, 162, SMALL_DOT_SIZE],
              name: '',
            },
            {
              value: [5, 80, SMALL_DOT_SIZE],
              name: '',
            },
            {
              value: [7, -15, SMALL_DOT_SIZE],
              name: '',
            },
            {
              value: [-1, 23, SMALL_DOT_SIZE],
              name: '',
            },
            {
              value: [-5, -30, SMALL_DOT_SIZE],
              name: '',
            },
            {
              value: [-6, 10, SMALL_DOT_SIZE],
              name: '',
            },
            {
              value: [-6, -10, SMALL_DOT_SIZE],
              name: '',
            },
            {
              value: [-7, -31, SMALL_DOT_SIZE],
              name: '',
            },
            {
              value: [-8, -20, SMALL_DOT_SIZE],
              name: '',
            },
          ],
          label: {
            show: true,
            formatter: function(data) {
              return data.name;
            },
            fontWeight: "bold",
            fontSize: 25,
          },
          symbolSize: function (data) {
            return data[2];
          },
          colorBy: "data",
        }
      ],
    };
    ```
    </details>

---

### 기타
* 구글링요령: `how to [찾는 내용] with echarts`
* echarts 예시모음: [echarts - Examples](https://echarts.apache.org/examples/en/index.html)
