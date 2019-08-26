# gcp 與 MT5 透過socket


## MT5 新增webRequest
- gcp ip : 34.80.241.51


![](images/擷取選取區域_018.png)



## gcp 防火牆規則新增 9090 port

![](images/擷取選取區域_019.png)


## socket ip 要用gcp 內部ip 應該有mapping 關係

```py
# -*- coding: utf-8 -*-
"""
Created on Tue Feb 19 01:52:04 2019

@author: dmitrievsky
"""
import socket, numpy as np
from sklearn.linear_model import LinearRegression

class socketserver:
    def __init__(self, address = '', port = 9090):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.address = address
        self.port = port
        self.sock.bind((self.address, self.port))
        self.cummdata = ''
        
    def recvmsg(self):
        self.sock.listen(1)
        self.conn, self.addr = self.sock.accept()
        print('connected to', self.addr)
        self.cummdata = ''

        while True:
            data = self.conn.recv(10000)
            self.cummdata+=data.decode("utf-8")
            if not data:
                break    
            self.conn.send(bytes(calcregr(self.cummdata), "utf-8"))
            return self.cummdata
            
    def __del__(self):
        self.sock.close()
        
def calcregr(msg = ''):
    chartdata = np.fromstring(msg, dtype=float, sep= ' ') 
    Y = np.array(chartdata).reshape(-1,1)
    X = np.array(np.arange(len(chartdata))).reshape(-1,1)
        
    lr = LinearRegression()
    lr.fit(X, Y)
    Y_pred = lr.predict(X)

    P = Y_pred.astype(str).item(-1) + ' ' + Y_pred.astype(str).item(0)
    print(P)
    return str(P)
    
serv = socketserver('10.140.0.3', 9090)

while True:  
    msg = serv.recvmsg()
```