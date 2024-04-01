## **SDN** 使用 Dijkstra演算法

## 電子碩一 **112368076**  **王識傑** 

### 內容描述:

此實驗為將 Dijkstra 演算法套入自製的網路架構中模擬，使其每次都走最短路徑。 

### 執行步驟: 

1. 利用 Ryu 控制器連線自製網路並使用基本 simple switch 規則。

![photo](https://github.com/NTUT-SDN-PLUS/Midterm-Project/blob/group09/photo/Aspose.Words.3efb8f9a-909a-49eb-a7e0-bb1ea19a544d.001_0.png)

2. 使用 pingall 指令對所有主機與伺服器接通(主機:hx、伺服器:sx、交換機:swx)確認全體接通。

![](https://github.com/NTUT-SDN-PLUS/Midterm-Project/blob/group09/photo/Aspose.Words.3efb8f9a-909a-49eb-a7e0-bb1ea19a544d.002_0.png)

3. 使用 GUI 觀看所有接通構造圖。

![](https://github.com/NTUT-SDN-PLUS/Midterm-Project/blob/group09/photo/Aspose.Words.3efb8f9a-909a-49eb-a7e0-bb1ea19a544d.003_0.jpg)

4. 使用 net 指令列出所有連線狀況。

![](https://github.com/NTUT-SDN-PLUS/Midterm-Project/blob/group09/photo/Aspose.Words.3efb8f9a-909a-49eb-a7e0-bb1ea19a544d.004_0.jpg)

5. 測試從 h1 ping h2 確保相互可以接通，並觀察執行時間。 

![](https://github.com/NTUT-SDN-PLUS/Midterm-Project/blob/group09/photo/Aspose.Words.3efb8f9a-909a-49eb-a7e0-bb1ea19a544d.005_0.png)

6. 再次利用 Ryu  控制器連線自製網路並使用 Dijkstra  演算法尋找最短路徑。 

![](https://github.com/NTUT-SDN-PLUS/Midterm-Project/blob/group09/photo/Aspose.Words.3efb8f9a-909a-49eb-a7e0-bb1ea19a544d.006_0.png)

7. 再次測試從 h1 ping h2 確保相互可以接通。

![](https://github.com/NTUT-SDN-PLUS/Midterm-Project/blob/group09/photo/Aspose.Words.3efb8f9a-909a-49eb-a7e0-bb1ea19a544d.007_0.png)

8. 此時我們發現有用 Dijkstra 演算法與沒用 Dijkstra 演算法存在著很明顯的差異，有用 Dijkstra 演算法的 time 明顯耗時較短。 

### 關鍵程式碼: 

#### Dijkstra  演算法

##### 起始節點和距離初始化：

選擇一個起始節點，將其到自身的距離設置為 0，表示到達起始節點的距離是 0。將所有其他節點的距離初始化為無窮大，表示初始情況下到達 這些節點的距離是未知的。

```
for dpid in switches: 

distance[dpid] = float('Inf') 

previous[dpid] = None distance[src] = 0
``` 

##### 優先級佇列（**Priority Queue**）：

使用一個優先級佇列（通常實現為最小堆或最小優先級隊列）來存儲節點和與它們關聯的當前最短距離。這使得我們可以高效地選擇當前最小距離的節點。

##### 選擇當前節點：

從優先級佇列中選擇當前距離最小的節點作為當前節點。

##### 更新相鄰節點的距離：
對於當前節點的每個相鄰節點，計算通過當前節點到達它們的距離。如果通過當前節點到達的距離比已知的距離小，則更新該節點的距離。

```
Q = set(switches)

while len(Q) > 0: 

     u = minimum_distance(distance, Q) 

     Q.remove(u) 

     for p in switches: 

         if adjacency[u][p] is not None: 

             w = 1 

             if distance[u] + w < distance[p]: 

                 distance[p] = distance[u] + w
                 previous[p] = u 

w = 1 

if distance[u] + w < distance[p]: 

distance[p] = distance[u] + w 

previous[p] = u 

```

###### 構建最短路徑:

這部分代碼用於構建最短路徑。它從目標節點（dst）開始，沿著前一節點的信息反向追溯，直到到達起始節點。最後，如果起始節點等於目標節點， 那麼路徑只包含起始節點，否則使用追溯的路徑。

```
r = [] 

p = dst 

r.append(p) 

q = previous[p]
while q is not None:
     if q == src: 
        r.append(q)
        break

     p = q 
     r.append(p) 
     q = previous[p] 

r.reverse() 

if src == dst: 

     path = [src]
else: 

     path = r
```

##### 構建路徑中的交換機之間的連接：

這部分代碼用於構建路徑中的交換機之間的連接。對於路徑中相鄰的兩個交換機，它使用相鄰矩陣（adjacency）來獲取它們之間的連接端口，最終返回連接的列表。

```
r = [] 

in\_port = first\_port 

for s1, s2 in zip(path[:-1], path[1:]):
out\_port = adjacency[s1][s2]
r.append((s1, in\_port, out\_port))
in\_port = adjacency[s2][s1]
r.append((dst, in\_port, final\_port))
return r 
```
### Topology 建立 

![](https://github.com/NTUT-SDN-PLUS/Midterm-Project/blob/group09/photo/Aspose.Words.3efb8f9a-909a-49eb-a7e0-bb1ea19a544d.008_0.jpg)

**↑  模擬網路架構圖** 

##### 以下為建立 topogoly 的程式碼 :

```
net = Mininet(controller=RemoteController, link=TCLink) 

- adding hosts 

h1 = net.addHost("h1", ip="10.0.0.1", mac="00:00:00:00:00:01") 

h2 = net.addHost("h2", ip="10.0.0.2", mac="00:00:00:00:00:02")

h3 = net.addHost("h3", ip="10.0.0.3", mac="00:00:00:00:00:03") 

- adding servers  

s1 = net.addHost("s1", ip="10.0.0.4", mac="00:00:00:00:00:04")  

s2 = net.addHost("s2", ip="10.0.0.5", mac="00:00:00:00:00:05")  

s3 = net.addHost("s3", ip="10.0.0.6", mac="00:00:00:00:00:06")  

s4 = net.addHost("s4", ip="10.0.0.7", mac="00:00:00:00:00:07")  

s5 = net.addHost("s5", ip="10.0.0.8", mac="00:00:00:00:00:08")  

s6 = net.addHost("s6", ip="10.0.0.9", mac="00:00:00:00:00:09")  

s7 = net.addHost("s7", ip="10.0.0.10", mac="00:00:00:00:00:10")

s8 = net.addHost("s8", ip="10.0.0.11", mac="00:00:00:00:00:11")

s9 = net.addHost("s9", ip="10.0.0.12", mac="00:00:00:00:00:12")

s10 = net.addHost("s10", ip="10.0.0.13", mac="00:00:00:00:00:13")

s11 = net.addHost("s11", ip="10.0.0.14", mac="00:00:00:00:00:14")

s12 = net.addHost("s12", ip="10.0.0.15", mac="00:00:00:00:00:15")  

- adding switches  

sw1 = net.addSwitch("sw1")

sw2 = net.addSwitch("sw2")

sw3 = net.addSwitch("sw3")

sw4 = net.addSwitch("sw4")  

sw5 = net.addSwitch("sw5")  

sw6 = net.addSwitch("sw6")  

sw7 = net.addSwitch("sw7")  

sw8 = net.addSwitch("sw8")  

sw9 = net.addSwitch("sw9")  

sw10 = net.addSwitch("sw10")  

sw11 = net.addSwitch("sw11")  

sw12 = net.addSwitch("sw12")  

sw13 = net.addSwitch("sw13")  

sw14 = net.addSwitch("sw14")  

sw15 = net.addSwitch("sw15")  

sw16 = net.addSwitch("sw16")
```  
```
- adding controller  

net.addController("C0", controller=RemoteController, ip="127.0.0.1", port=6633)  

- defining link options  

linkopt5 = dict(bw=5, delay='1ms', cls=TCLink)  

linkopt10 = dict(bw=10, delay='1ms', cls=TCLink)  

linkopt15 = dict(bw=15, delay='1ms', cls=TCLink)  

linkopt50 = dict(bw=50, delay='1ms', cls=TCLink)  

linkopt100 = dict(bw=100, delay='1ms', cls=TCLink)
```  
```
- adding links
- sw1  

net.addLink(sw1, s1, \*\*linkopt100, port1=12000, port2=12001)
net.addLink(sw1, s2, \*\*linkopt100, port1=12002, port2=12003)
net.addLink(sw1, sw3, \*\*linkopt50, port1=12004, port2=12005)
net.addLink(sw1, sw4, \*\*linkopt100, port1=12006, port2=12007)  

- sw2  

net.addLink(sw2, s3, \*\*linkopt100, port1=12008, port2=12009)
net.addLink(sw2, s4, \*\*linkopt100, port1=12010, port2=12011)  
net.addLink(sw2, sw3, \*\*linkopt100, port1=12012, port2=12013)
net.addLink(sw2, sw4, \*\*linkopt50, port1=12014, port2=12015)
 
- sw3

net.addLink(sw3, sw13, \*\*linkopt15, port1=12016, port2=12017)  
net.addLink(sw3, sw14, \*\*linkopt10, port1=12018, port2=12019)
 
- sw4  

net.addLink(sw4, sw14, \*\*linkopt5, port1=12020, port2=12021)  

- sw5  

net.addLink(sw5, s5, \*\*linkopt100, port1=12022, port2=12023)
net.addLink(sw5, s6, \*\*linkopt100, port1=12024, port2=12025)
net.addLink(sw5, sw7, \*\*linkopt50, port1=12026, port2=12027)
net.addLink(sw5, sw8, \*\*linkopt100, port1=12028, port2=12029)  

- sw6  

net.addLink(sw6, s7, \*\*linkopt100, port1=12030, port2=12031)
net.addLink(sw6, s8, \*\*linkopt100, port1=12032, port2=12033)  
net.addLink(sw6, sw7, \*\*linkopt100, port1=12034, port2=12035)
net.addLink(sw6, sw8, \*\*linkopt50, port1=12036, port2=12037)
 
- sw7  

net.addLink(sw7, sw13, \*\*linkopt15, port1=12038, port2=12039)  
net.addLink(sw7, sw14, \*\*linkopt20, port1=12040, port2=12041)  
net.addLink(sw7, sw15, \*\*linkopt5, port1=12042, port2=12043)
 
- sw8  

net.addLink(sw8, sw15, \*\*linkopt10, port1=12044, port2=12045)  
net.addLink(sw8, sw16, \*\*linkopt15, port1=12046, port2=12047)  
- sw9  

net.addLink(sw9, s9, \*\*linkopt100, port1=12048, port2=12049)
net.addLink(sw9, s10, \*\*linkopt100, port1=12050, port2=12051)
net.addLink(sw9, sw11, \*\*linkopt50, port1=12052, port2=12053)
net.addLink(sw9, sw12, \*\*linkopt100, port1=12054, port2=12055)  

- sw10  

net.addLink(sw10, s11, \*\*linkopt100, port1=12056, port2=12057)
net.addLink(sw10, s12, \*\*linkopt100, port1=12058, port2=12059)  
net.addLink(sw10, sw11, \*\*linkopt100, port1=12060, port2=12061)
net.addLink(sw10, sw12, \*\*linkopt50, port1=12062, port2=12063)
 
- sw11  

net.addLink(sw11, sw14, \*\*linkopt10, port1=12064, port2=12065)  

- sw12  

net.addLink(sw12, sw16, \*\*linkopt10, port1=12066, port2=12067)  

- sw13  

net.addLink(sw13, h1, \*\*linkopt100, port1=12068, port2=12069)  

- sw14  

net.addLink(sw14, h2, \*\*linkopt100, port1=12070, port2=12071)  

- sw15  

net.addLink(sw15, sw12, \*\*linkopt15, port1=12072, port2=12073)  

- sw16  

net.addLink(sw16, h3, \*\*linkopt100, port1=12074, port2=12075)  

# running network net.start() # starting cli CLI(net) # stopping network net.stop()
 if \_\_name\_\_ == "\_\_main\_\_": network = Network() network.run()
```

### 執行結果(最短路徑): 

##### Ex1、h1 ping h2 (紅框相加為最後路徑) 

![](https://github.com/NTUT-SDN-PLUS/Midterm-Project/blob/group09/photo/Aspose.Words.3efb8f9a-909a-49eb-a7e0-bb1ea19a544d.009_0.png)

![](https://github.com/NTUT-SDN-PLUS/Midterm-Project/blob/group09/photo/Aspose.Words.3efb8f9a-909a-49eb-a7e0-bb1ea19a544d.010_0.jpg)

![](https://github.com/NTUT-SDN-PLUS/Midterm-Project/blob/group09/photo/Aspose.Words.3efb8f9a-909a-49eb-a7e0-bb1ea19a544d.011_0.png)

![](https://github.com/NTUT-SDN-PLUS/Midterm-Project/blob/group09/photo/Aspose.Words.3efb8f9a-909a-49eb-a7e0-bb1ea19a544d.012_0.png)

##### Ex2、h1 ping h3 (紅框相加為最後路徑) 

![](https://github.com/NTUT-SDN-PLUS/Midterm-Project/blob/group09/photo/Aspose.Words.3efb8f9a-909a-49eb-a7e0-bb1ea19a544d.013_0.png)

![](https://github.com/NTUT-SDN-PLUS/Midterm-Project/blob/group09/photo/Aspose.Words.3efb8f9a-909a-49eb-a7e0-bb1ea19a544d.014_0.png)

![](https://github.com/NTUT-SDN-PLUS/Midterm-Project/blob/group09/photo/Aspose.Words.3efb8f9a-909a-49eb-a7e0-bb1ea19a544d.015_0.png)

##### Ex3、h1 ping s1 (紅框相加為最後路徑) 

![](https://github.com/NTUT-SDN-PLUS/Midterm-Project/blob/group09/photo/Aspose.Words.3efb8f9a-909a-49eb-a7e0-bb1ea19a544d.016_0.png)

![](https://github.com/NTUT-SDN-PLUS/Midterm-Project/blob/group09/photo/Aspose.Words.3efb8f9a-909a-49eb-a7e0-bb1ea19a544d.017_0.jpg)

![](https://github.com/NTUT-SDN-PLUS/Midterm-Project/blob/group09/photo/Aspose.Words.3efb8f9a-909a-49eb-a7e0-bb1ea19a544d.018_0.png)

### 結論: 

由於我們的網路架構較為複雜，且我們有相互打通交換機的通道，因此多出多種路徑，若沒使用演算法將隨機走路徑，會多耗費許多時間。故需套用演算法使其可以走最佳路徑以節省時間花費。在使用 Dijkstra 演算法後證實使用 Dijkstra 演算法尋找隻最短路徑較優於未使用演算法的情況，但在極少部分的情況下，未使用演算法隨機走的路徑會小於使用演算法的路徑，此舉證實 Dijkstra 演算法並非完美的演算法，還有改進的空間。 
