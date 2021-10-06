# static threshold readme

## 程式介紹
### start.java
主程式
參數:
iter:訓練的epoch數(一次訓練多個epoch時資料會存在同一個txt當中，要自行分開)
max_iter:每個epoch的學習步數
input_file:訓練用的workload file

### restart.java
每個epoch開始時更新Docker環境 
1.docker stck rm app(移除docker stack)
2.docker stack deploy --compose-file docker-compose.yml(重新部署docker環境)

### get_all_use.java
取得學習過程中各個service的cpu utilization
add_cons():新增各個service name
add_machine():新增各個docker machine的ID
get_use1():取得各個service的cpu utlization
read():讀取系統狀態，判斷系統是否正常

### get_all_res.java
取得學習過程中各個service的response time
add_cons():新增各個service name
add_machine():新增各個docker machine的ID
response_time():向service發送request獲取responnse time，我們用3次request的平均做為這次的response time
send_request():對各個service發送request，這邊的IP指定swarm manager的IP，需要手動修改
print_res_time():輸出並記錄response time

### new_send_request.java
作為傳送requests的程式
read():讀取要做為workload的檔案
add_stage():加入要傳送的First level MNCSE的stage
send():執行send_req()程式傳送request，這邊使用FixedThreadPool=40，讓他可同時傳送40個request
### send_request.java
將new_send_request設定好的request傳送出去的程式，
RFID():隨機產生RFID，避免request在傳送到MNCSE時service name重複
send():傳送request到MNCSE，其中，path中的IP要手動修改，修改為Docker swarm的IP(192.168.99.130:666)，Docker swarm microservice 建立方式會另外說明

### health_check.java
負責確認service是否正常運作
add_c():紀錄service的IP位置，這邊的IP會隨著docker 分配到的IP而改變
check():傳送request到service判斷是否正常運作，出現異常時透過改變signal.txt作為service異常的訊號，暫停系統運作並利用restart.java重新部署service
write_error_count():記錄錯誤次數(不影響系統運作)

### service.java
static threshold的核心程式，判斷service需不需要scaling
1.讀取signal.txt，判斷系統是否正常運作
2.執行get_state2()，獲取service的cpu utilization
3.判斷service的cpu utilization有沒有超過80%，若超過80%，執行get_cons()或取當下service的replicas，並執行add_cons()，向swarm manager調整service的replicas數量
4.判斷service的cpu utilization有沒有低於10%，若低於10%，執行get_cons()或取當下servuce的replicas，並執行add_cons()，向swarm manager調整service的replicas數量
get_state2():或取當下service的cpu utilization
get_cons():或取當下service的replicas
add_cons():調整service的replicas數量

### 啟動步驟
1.建立Docker 環境[參考這篇](https://hackmd.io/bMI9DG9ASmmdvG3h0kN2lA?edit)
2.依照docker machine IP修改程式中的IP，需要修改的有
get_all_res.java、send_req.java、health_check.java
3.執行start.java啟動程式




























