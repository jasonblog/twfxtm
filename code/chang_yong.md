# 範例程式

## 常用API

```c
void getinfo(string s)
{
    // 打印現在時間
    datetime time[];
    int copied = sj.gettime(time, 1);

    if (copied > 0) { // 数据已经被成功复制
        Print(time[0]);
    }
    // 打印帳戶貨幣
    string account_currency = AccountInfoString(ACCOUNT_CURRENCY);
    Print(account_currency);
    // 打印貨幣對
    Print(s + Symbol());
    // 打印下單時候的買賣價
    printf("ask:%f, bid:%f", sj.getask(), sj.getbid());
    // 抓取前幾4根k棒, 開盤價, 收盤價,最高最低價
    MqlRates rates[];
    sj.getrates(rates, 4);
    printf("open=%f, close=%f, high=%f, low=%f", rates[0].open, rates[0].close, rates[0].high, rates[0].low); // 目前ｋ棒
    printf("open=%f, close=%f, high=%f, low=%f", rates[1].open, rates[1].close, rates[1].high, rates[1].low); // 前一根
    printf("open=%f, close=%f, high=%f, low=%f", rates[2].open, rates[2].close, rates[2].high, rates[2].low); // 前兩根
    printf("open=%f, close=%f, high=%f, low=%f", rates[3].open, rates[3].close, rates[3].high, rates[3].low); // 前三根
}
```