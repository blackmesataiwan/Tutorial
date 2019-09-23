# 如何整合 Power BI 與 QIoT Suite

## 1. 取得您的第一個 Power BI 帳號

- 前往 https://powerbi.microsoft.com/en-us/ 免費註冊 Power BI 帳號
  ![](https://i.imgur.com/EjSaYPb.png)
  ![](https://i.imgur.com/znPJO6F.png)

- 註冊後，畫面將顯示以下頁面：
  ![](https://i.imgur.com/3XtwFn0.png)

## 2. 設定串流資料集 API

- 點選中間的 **\[My workspace\]**，然後點擊右上角的 **\[Create\]->\[Streaming datasets\]**
  ![](https://i.imgur.com/RHwCn7t.png)
  ![](https://i.imgur.com/A0bHYv4.png)


- 選取 **\[API\]** 作為資料來源，然後點擊 **\[Next\]**
  ![](https://i.imgur.com/qwAw3LR.png)

- 定義您的串流值，然後您將在文字方塊中得到 JSON 結果。 我們將使用此 JSON 碼推送資料至 IoT 應用。 點擊 **\[Create\]** 即可完成。
  ![](https://i.imgur.com/M0Yanqa.png)

- 一旦建立資料串流後，你會獲得 Push URL；IoT 應用可使用 POST 要求呼叫此 URL，將您的即時資料推送至您建立的串流資料集。
  ![](https://i.imgur.com/VxBXVDl.png)


## 3. 設定 IoT 應用中的 Node-RED 節點

- 在 QIoT Suite 中建立 IoT 應用。 以下是您的第一個 Node-RED 流程，接著您可以開始建立自己的 IoT 流程。 如需深入瞭解 Node-RED，請瀏覽 https://nodered.org/
  ![](https://i.imgur.com/GBmWP1i.png)

- 在您開始發佈即時資料至 Power BI 之前， 我們需要 **\[change\]** 節點以轉換 IoT 資料為串流資料集。參考下圖設定，您可以在此將 **temperature** 取代為您所設定的 value-key，或是使用 **\[function\]** 節點做更加彈性的設定。
  ![](https://i.imgur.com/qNpZTPR.png)

- 我們需要 **\[http request\]** 節點來協助推送即時資料至 Power BI。 您只需拖放 **\[http request\]** 節點，使其連至 **\[change\]** 節點節點的尾巴即可。
  ![](https://i.imgur.com/2NopiqU.png)

- 複製並貼上您從 Power BI dataset 取得的 Push URL，然後將 http 方法設為 POST。 點擊 **\[Save\]** 以儲存變更。
  ![](https://i.imgur.com/2BMFY5a.png)

- 您的 Node-RED 流程如下所示。
  ![](https://i.imgur.com/QgEHFD1.png)
  
- Node-Red 範例匯出參考 : 
    ```json
    [
    {
        "id": "2050223d.9dc9de",
        "type": "debug",
        "z": "c01d41a4.a2fbc",
        "name": "Click debug tab to watch data stream",
        "active": true,
        "console": "false",
        "complete": "false",
        "x": 494,
        "y": 157,
        "wires": []
    },
    {
        "id": "34caa285.75b72e",
        "type": "qiotbroker in",
        "z": "c01d41a4.a2fbc",
        "name": "MQTT Message In",
        "flow": "c01d41a4.a2fbc",
        "opt_customtopic": true,
        "customtopic": "USER_TOPIC_db514d80",
        "thing": "",
        "qtopic": "",
        "username": "admin",
        "rules": [],
        "outputs": 1,
        "key": "r:12907185af0147439eb2bda156cd814d",
        "x": 179,
        "y": 157,
        "wires": [
            [
                "2050223d.9dc9de",
                "a98aaab5.1d3208"
            ]
        ]
    },
    {
        "id": "a98aaab5.1d3208",
        "type": "change",
        "z": "c01d41a4.a2fbc",
        "name": "Convert",
        "rules": [
            {
                "t": "move",
                "p": "payload.value",
                "pt": "msg",
                "to": "temperature",
                "tot": "msg"
            },
            {
                "t": "delete",
                "p": "payload",
                "pt": "msg"
            },
            {
                "t": "move",
                "p": "temperature",
                "pt": "msg",
                "to": "payload[0].temperature",
                "tot": "msg"
            },
            {
                "t": "set",
                "p": "headers.Content-Type",
                "pt": "msg",
                "to": "application/json",
                "tot": "str"
            }
        ],
        "action": "",
        "property": "",
        "from": "",
        "to": "",
        "reg": false,
        "x": 400,
        "y": 200,
        "wires": [
            [
                "b1cf0fbc.4314e"
            ]
        ]
    },
    {
        "id": "6d513804.860e08",
        "type": "debug",
        "z": "c01d41a4.a2fbc",
        "name": "Click debug tab to watch data stream",
        "active": true,
        "tosidebar": true,
        "console": false,
        "tostatus": false,
        "complete": "payload",
        "x": 810,
        "y": 200,
        "wires": []
    },
    {
        "id": "b1cf0fbc.4314e",
        "type": "http request",
        "z": "c01d41a4.a2fbc",
        "name": "",
        "method": "POST",
        "ret": "txt",
        "url": "https://api.powerbi.com/beta/6eba8807-6ef0-4e31-890c-a6ecfbb98568/datasets/28c220f3-df96-41d4-a74e-a86db6743c49/rows?key=OzjegMmsmOm%2Bg6II%2Bnx1HTAio%2FhxCRZtKwt3TeA3uW4HyN2BT1SU2xJ%2F5RTq%2F2sLkm1sA5F02KxdilA3x41Phw%3D%3D",
        "tls": "",
        "x": 560,
        "y": 200,
        "wires": [
            [
                "6d513804.860e08"
            ]
        ]
    },
    {
        "id": "2e4aa64c.5fc46a",
        "type": "qiotbroker out",
        "z": "c01d41a4.a2fbc",
        "name": "MQTT Message Out & Dashboard",
        "opt_customtopic": true,
        "topic": "USER_TOPIC_db514d80",
        "qos": "",
        "retain": "",
        "thing": "",
        "qtopic": "",
        "username": "admin",
        "x": 609,
        "y": 320,
        "wires": []
    },
    {
        "id": "d9f122ca.132e9",
        "type": "function",
        "z": "c01d41a4.a2fbc",
        "name": "dummy data",
        "func": "var v = Math.floor(Math.random()*100+1);\nmsg.payload = {value:v};\nreturn msg;",
        "outputs": 1,
        "noerr": 0,
        "x": 373,
        "y": 320,
        "wires": [
            [
                "2e4aa64c.5fc46a"
            ]
        ]
    },
    {
        "id": "9c04dde3.36d24",
        "type": "inject",
        "z": "c01d41a4.a2fbc",
        "name": "Virtual Event",
        "topic": "",
        "payload": "",
        "payloadType": "date",
        "repeat": "",
        "crontab": "",
        "once": true,
        "x": 189,
        "y": 320,
        "wires": [
            [
                "d9f122ca.132e9"
            ]
        ]
    }
    ]
    ```


## 4. 新增圖磚以顯示即時資料

- 點擊右上角的 **\[Create\] -> \[Dashboards\]** 建立您的第一個儀表板，然後點擊 **\[Add tile\]** 以設定 Widget
  ![](https://i.imgur.com/mUgibMa.png)

- 選取 **\[Custom Streaming Data\]** 並點擊 **\[Next\]**
  ![](https://i.imgur.com/xRZ8VHp.png)
  
- 選取資料集並點擊 **\[Next\]**
  ![](https://i.imgur.com/gaNfSVb.png)

- 最後依據需求去設定標題等後並點擊 **\[Apply\]** ，您將擁有可供操作的串流資料集，並取得即時衡量工具（如下圖所示）
  ![](https://i.imgur.com/7XYV4AY.png)
  ![](https://i.imgur.com/aDoOBLS.png)


###### tags: `Tutorial`