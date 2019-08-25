# python 與MT5 透過 socket 溝通


- server 端

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
        print("address: " + address + ", port: " + str(port))
        
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
    
serv = socketserver('127.0.0.1', 9090)

while True:  
    msg = serv.recvmsg()
```


- mq5 client

只會跑一次OnInit , 如果爬歷史資料應該要在OnInit 建 socket 傳資料

```cpp
//+------------------------------------------------------------------+
//|                                               socketclientEA.mq5 |
//|                        Copyright 2018, MetaQuotes Software Corp. |
//|                                             https://www.mql5.com |
//+------------------------------------------------------------------+
#property copyright "Copyright 2018, MetaQuotes Software Corp."
#property link      "https://www.mql5.com"
#property version   "1.00"

sinput int lrlenght = 150;
int socket;
//+------------------------------------------------------------------+
//| Expert initialization function                                   |
//+------------------------------------------------------------------+
int OnInit()
{
    socket = SocketCreate();

    if (socket != INVALID_HANDLE) {
        if (SocketConnect(socket, "localhost", 9090, 1000)) {
            Print("Connected to ", " localhost", ":", 9090);
            MqlRates rates[];
            double Avg = 0;
            int copied = CopyRates(Symbol(), 0, 1, 3, rates); //獲取指定數量的指定符號周期的MqlRates結構的歷史數據到rates_array數組

            //商品名稱,期間,開始位置,要復制的數據計數,要復制的目標數組
            int totalk = fmin(copied, 10);//獲取數量若超過10，則僅計算10

            for (int i = 0; i < totalk; i ++) { //計算總數之開盤價並計算平均值
                Avg += rates[i].open;//將開盤價累加至Avg變數中
            }

            Print(Avg);
            string received = socksend(socket, string(Avg)) ? socketreceive(socket, 10) : "";
            drawlr(received);
        } else {
            Print("Connection ", "localhost", ":", 9090, " error ", GetLastError());
        }    
    }
    return (INIT_SUCCEEDED);
}

//+------------------------------------------------------------------+
//| Expert deinitialization function                                 |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
{
    SocketClose(socket);
}
//+------------------------------------------------------------------+
//| Expert tick function                                             |
//+------------------------------------------------------------------+
void OnTick()
{
    socket = SocketCreate();

    if (socket != INVALID_HANDLE) {
        if (SocketConnect(socket, "localhost", 9090, 1000)) {
            Print("Connected to ", " localhost", ":", 9090);

            double clpr[];
            int copyed = CopyClose(_Symbol, PERIOD_CURRENT, 0, lrlenght, clpr);

            string tosend;

            for (int i = 0; i < ArraySize(clpr); i++) {
                tosend += (string)clpr[i] + " ";
            }

            string received = socksend(socket, tosend) ? socketreceive(socket, 10) : "";
            drawlr(received);
        }

        else {
            Print("Connection ", "localhost", ":", 9090, " error ", GetLastError());
        }

        SocketClose(socket);
    } else {
        Print("Socket creation error ", GetLastError());
    }
}

bool socksend(int sock, string request)
{
    char req[];
    int  len = StringToCharArray(request, req) - 1;

    if (len < 0) {
        return (false);
    }

    return (SocketSend(sock, req, len) == len);
}

string socketreceive(int sock, int timeout)
{
    char rsp[];
    string result = "";
    uint len;
    uint timeout_check = GetTickCount() + timeout;

    do {
        len = SocketIsReadable(sock);

        if (len) {
            int rsp_len;
            rsp_len = SocketRead(sock, rsp, len, timeout);

            if (rsp_len > 0) {
                result += CharArrayToString(rsp, 0, rsp_len);
            }
        }
    } while ((GetTickCount() < timeout_check) && !IsStopped());

    return result;
}

void drawlr(string points)
{
    string res[];
    StringSplit(points, ' ', res);

    if (ArraySize(res) == 2) {
        Print(StringToDouble(res[0]));
        Print(StringToDouble(res[1]));
        datetime temp[];
        CopyTime(Symbol(), Period(), TimeCurrent(), lrlenght, temp);
        ObjectCreate(0, "regrline", OBJ_TREND, 0, TimeCurrent(), NormalizeDouble(StringToDouble(res[0]), _Digits), temp[0], NormalizeDouble(StringToDouble(res[1]), _Digits));
    }
}
//+------------------------------------------------------------------+
```