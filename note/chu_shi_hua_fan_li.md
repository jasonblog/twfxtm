# 初始化範例


```cpp
// +------------------------------------------------------------------+
// |              第一步驟                                            |
// +------------------------------------------------------------------+
#include <cash.mqh>
cash cc;
// +------------------------------------------------------------------+
// |              第一步驟                                            |
// +------------------------------------------------------------------+
// +------------------------------------------------------------------+
// |              第二步驟                                            |
// +------------------------------------------------------------------+
double Ask, Bid, AccountBalance, Tickvalue;
// +------------------------------------------------------------------+
// |              第二步驟                                            |
// +------------------------------------------------------------------+
// 內部宣告---------------------------------------------------------------
int 買單單號, 賣單單號;
int 避免同根重複下單的風控變數 = 0;
int 避免同根重複平倉的風控變數 = 0;
int 每日下單筆數        = 0;
// 外部宣告---------------------------------------------------------------
input bool 自動計算手數開關   = true;
input double 每筆風險百分比  = 1;
input double 初始手數     = 0.1;
input int 每日僅下單次數     = 2;
input int MagicNumber = 100;
// +------------------------------------------------------------------+
// |                                                                  |
// +------------------------------------------------------------------+
int OnInit()
{
    Alert("此EA為教學範例檔,任何參數請自行設定");
    return (INIT_SUCCEEDED);
}

// +------------------------------------------------------------------+
// |                                                                  |
// +------------------------------------------------------------------+
void OnDeinit(const int reason)
{}

// +------------------------------------------------------------------+
// |                                                                  |
// +------------------------------------------------------------------+
void OnTick()
{
    // ==================================================================================================================================================//
    if (cc.TimeHour(TimeCurrent()) == 0 && cc.TimeMinute(TimeCurrent()) <= 5) {
        每日下單筆數 = 0;
    }

    // --------------------------------------------------------------------------------------------關鍵價位
    // +------------------------------------------------------------------+
    // |              第三步驟                                            |
    // +------------------------------------------------------------------+
    Ask = SymbolInfoDouble(Symbol(), SYMBOL_ASK);
    Bid = SymbolInfoDouble(Symbol(), SYMBOL_BID);
    AccountBalance = AccountInfoDouble(ACCOUNT_BALANCE);
    Tickvalue      = SymbolInfoDouble(Symbol(), SYMBOL_TRADE_TICK_VALUE);

    // +------------------------------------------------------------------+
    // |              第三步驟                                            |
    // +------------------------------------------------------------------+
    // ----------------------------------------------------------------------------------
    //                                          多單開始
    // ----------------------------------------------------------------------------------
    if (多單筆數() == 0 && 避免同根重複下單的風控變數 != iBars(Symbol(), 15) && 每日下單筆數 < 每日僅下單次數) {
        if (買進的條件() == true) {
            MqlTradeRequest request = { 0 };
            MqlTradeResult result   = { 0 };
            request.order     = 買單單號;
            request.action    = TRADE_ACTION_DEAL;
            request.symbol    = Symbol();
            request.type      = ORDER_TYPE_BUY;
            request.volume    = 多單資金控管();
            request.deviation = 100;
            request.price     = SymbolInfoDouble(Symbol(), SYMBOL_ASK);
            request.sl        = 0;
            request.tp        = 0;
            request.comment   = "HunterV B";
            request.magic     = MagicNumber;

            if (!OrderSend(request, result)) {
                PrintFormat("OrderSend error %d", GetLastError());
            }

            PrintFormat("retcode=%u  deal=%I64u  order=%I64u", result.retcode, result.deal, result.order);

            if (多單筆數() > 0) {
                每日下單筆數++;
                避免同根重複下單的風控變數 = iBars(Symbol(), 15);
            }
        }
    }

    // ---------------------------------------------------------多單出場
    if (多單筆數() > 0 && 多單出場條件() == true && 避免同根重複平倉的風控變數 != iBars(Symbol(), 15)) {
        多單平倉();

        if (多單筆數() == 0) {
            避免同根重複平倉的風控變數 = iBars(Symbol(), 15);
        }
    }

    // ----------------------------------------------------------------------------------
    //                                          空單開始
    // ----------------------------------------------------------------------------------
    if (空單筆數() == 0 && 避免同根重複下單的風控變數 != iBars(Symbol(), 15) && 每日下單筆數 < 每日僅下單次數) {
        if (賣進的條件() == true) {
            MqlTradeRequest request = { 0 };
            MqlTradeResult result   = { 0 };
            request.order     = 賣單單號;
            request.action    = TRADE_ACTION_DEAL;
            request.symbol    = Symbol();
            request.type      = ORDER_TYPE_SELL;
            request.volume    = 空單資金控管();
            request.deviation = 100;
            request.price     = SymbolInfoDouble(Symbol(), SYMBOL_BID);
            request.sl        = 0;
            request.tp        = 0;
            request.comment   = "HunterV S";
            request.magic     = MagicNumber + 77;

            if (!OrderSend(request, result)) {
                PrintFormat("OrderSend error %d", GetLastError());
            }

            PrintFormat("retcode=%u  deal=%I64u  order=%I64u", result.retcode, result.deal, result.order);

            if (空單筆數() > 0) {
                每日下單筆數++;
                避免同根重複下單的風控變數 = iBars(Symbol(), 15);
            }
        }
    }

    // ---------------------------------------------------------空單出場
    if (空單筆數() > 0 && 空單出場條件() == true && 避免同根重複平倉的風控變數 != iBars(Symbol(), 15)) {
        空單平倉();

        if (空單筆數() == 0) {
            避免同根重複平倉的風控變數 = iBars(Symbol(), 15);
        }
    }
} /* OnTick */

// ----------------------------------------------------------------------------------
//                                         函數庫
// ----------------------------------------------------------------------------------
// ---------------------------------------------------------多單筆數
int 多單筆數()
{
    int count = 0;

    for (int i = 0; i < PositionsTotal(); i++) {
        if (PositionGetTicket(i) > 0) {
            if (PositionGetString(POSITION_SYMBOL) == Symbol() &&
                PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_BUY &&
                PositionGetInteger(POSITION_MAGIC) == MagicNumber)
            {
                count++;
            }
        }
    }

    return (count);
}

// ---------------------------------------------------------空單筆數
int 空單筆數()
{
    int count = 0;

    for (int i = 0; i < PositionsTotal(); i++) {
        if (PositionGetTicket(i) > 0) {
            if (PositionGetString(POSITION_SYMBOL) == Symbol() &&
                PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_SELL &&
                PositionGetInteger(POSITION_MAGIC) == MagicNumber + 77)
            {
                count++;
            }
        }
    }

    return (count);
}

// ---------------------------------------------------------買進的條件
bool 買進的條件()
{
    if (iClose(Symbol(), PERIOD_CURRENT, 1) > iOpen(Symbol(), PERIOD_CURRENT, 1)) {
        return (true);
    } else {
        return (false);
    }
}

// ---------------------------------------------------------賣出的條件
bool 賣進的條件()
{
    if (iClose(Symbol(), PERIOD_CURRENT, 1) < iOpen(Symbol(), PERIOD_CURRENT, 1)) {
        return (true);
    } else {
        return (false);
    }
}

// ---------------------------------------------------------多單出場條件
bool 多單出場條件()
{
    if (iClose(Symbol(), PERIOD_CURRENT, 1) < iOpen(Symbol(), PERIOD_CURRENT, 1)) {
        return (true);
    } else {
        return (false);
    }
}

// ---------------------------------------------------------空單出場條件
bool 空單出場條件()
{
    if (iClose(Symbol(), PERIOD_CURRENT, 1) > iOpen(Symbol(), PERIOD_CURRENT, 1)) {
        return (true);
    } else {
        return (false);
    }
}

// ---------------------------------------------------------多單Close寫法
void 多單平倉()
{
    int t = PositionsTotal();

    for (int i = t - 1; i >= 0; i--) {
        if (PositionGetTicket(i) > 0) {
            if (PositionGetString(POSITION_SYMBOL) == Symbol() &&
                PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_BUY &&
                PositionGetInteger(POSITION_MAGIC) == MagicNumber)
            {
                MqlTradeRequest request = { 0 };
                MqlTradeResult result   = { 0 };
                request.action    = TRADE_ACTION_DEAL;
                request.symbol    = Symbol();
                request.volume    = PositionGetDouble(POSITION_VOLUME);
                request.type      = ORDER_TYPE_SELL;
                request.price     = SymbolInfoDouble(Symbol(), SYMBOL_BID);
                request.deviation = 10;
                request.position  = PositionGetTicket(i);

                if (!OrderSend(request, result)) {
                    PrintFormat("OrderSend error %d", GetLastError());
                }
            }
        }
    }
}

// ---------------------------------------------------------空單Close寫法
void 空單平倉()
{
    int t = PositionsTotal();

    for (int i = t - 1; i >= 0; i--) {
        if (PositionGetTicket(i) > 0) {
            if (PositionGetString(POSITION_SYMBOL) == Symbol() &&
                PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_SELL &&
                PositionGetInteger(POSITION_MAGIC) == MagicNumber + 77)
            {
                MqlTradeRequest request = { 0 };
                MqlTradeResult result   = { 0 };
                request.action    = TRADE_ACTION_DEAL;
                request.symbol    = Symbol();
                request.volume    = PositionGetDouble(POSITION_VOLUME);
                request.type      = ORDER_TYPE_BUY;
                request.price     = SymbolInfoDouble(Symbol(), SYMBOL_ASK);
                request.deviation = 100;
                request.position  = PositionGetTicket(i);

                if (!OrderSend(request, result)) {
                    PrintFormat("OrderSend error %d", GetLastError());
                }
            }
        }
    }
}

// +------------------------------------------------------------------+
// |                                                                  |
// +------------------------------------------------------------------+
double 多單資金控管()
{
    double ST = 0; // 進場-->停損的距離
    double x  = 0;
    double y  = (AccountBalance*每筆風險百分比) / 10;

    if (自動計算手數開關 == true) {
        x = (y / (SymbolInfoDouble(Symbol(), SYMBOL_TRADE_TICK_VALUE) * 10)) / (ST / Point());
    } else {
        x = 初始手數;
    }

    x = NormalizeDouble(x, 2);

    if (x <= 0.01) {
        x = 0.01;
    }

    if (x >= 0.29) {
        x = 0.3;
    }

    return (x);
}

// +------------------------------------------------------------------+
// |                                                                  |
// +------------------------------------------------------------------+
double 空單資金控管()
{
    double ST = 0; // 進場-->停損的距離
    double x  = 0;
    double y  = (AccountBalance*每筆風險百分比) / 10;

    if (自動計算手數開關 == true) {
        x = (y / (SymbolInfoDouble(Symbol(), SYMBOL_TRADE_TICK_VALUE) * 10)) / (ST / Point());
    } else {
        x = 初始手數;
    }

    x = NormalizeDouble(x, 2);

    if (x <= 0.01) {
        x = 0.01;
    }

    if (x >= 0.29) {
        x = 0.3;
    }

    return (x);
}

// +------------------------------------------------------------------+
```