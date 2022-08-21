---
layout: post
title: "Google Sheets – financial data and custom scripts"
date: 2022-08-21 10:34:00 +0200
tags: [google sheets, finances, javascript, google cloud]
---

I’m not a hardcore user of spreadsheets. Usually, I just need to add some numbers up or track something (expenses, hours, income). I knew that spreadsheets are powerful and extensible, I just rarely needed these functionalities. Until now.

One thing that I was doing for quite some time, is tracking my investments. Each month or so, I was logging into all my investment accounts – and there are quite many of them, definitely too many – and manually copy-pasting values to track the current values of my wallets. Tedious and time-consuming. And yes, I know that there are specialized tools for that, but I don’t think I’m advanced enough (yet) to need them – a simple spreadsheet is enough for now.

Yesterday, I’ve had enough, and I thought about automating it somehow. I found out about [Google Finance](https://www.google.com/finance/) service and the related `GOOGLEFINANCE` function in Sheets. This greatly helped me to automate the process. Nothing advanced, but it should save me some hours in the long run, so I thought I can share it.

# Currencies

I invest using three currencies: PLN, EUR, and USD. Yet, I want to have one currency for tracking wallet values. I’ve chosen PLN. It’s not ideal, as a wallet value depends not only on the actual stock/fund/bond price, but also on the currency rate. But that’s the choice, for now.

It turns out, that I don’t need to update the currency rate manually. It can easily be done with the `GOOGLEFINANCE` function in Sheets. For example, to get the current rate of EUR to PLN, you can use `=GOOGLEFINANCE("CURRENCY:EURPLN")`.


# Stock Price

Similarly with stock prices. You can easily fetch many attributes about a stock using the `GOOGLEFINANCE` function in Sheets. I don’t have big needs here, and it’s enough for me to know the current price of a stock, and it’s as simple as `=GOOGLEFINANCE("NASDAQ:GOOG)"` (remember about the market prefix!). For more details, check the [docs](https://support.google.com/docs/answer/3093281?hl=en).

ETFs are also available, if you invest in them – like I do. For instance: `GOOGLEFINANCE("LON:IWDA")`.

# Custom functions

Automatically fetching stock prices is neat, but what if you own some mutual funds? Or you are in a market that isn’t supported by Google Finances?

Fortunately, Google provides a possibility to extend Sheets with custom functions, using App Script. With some JavaScript, you can create a function that can easily be called in Sheets and fetch a fund’s price.

It’s nicely documented how to create a function [here](https://developers.google.com/apps-script/guides/sheets/functions?hl=en), so I won’t be retyping it here. For fetching data from the Internet, you can use the [URL Fetch Service](https://developers.google.com/apps-script/reference/url-fetch?hl=en).

The hardest part might be finding a source of data/API that will contain what you need. At least that was my case (Polish market).

But, after some digging, I found out that there is an API on [analizy.pl](https://www.analizy.pl/). I don’t know if that’s an official API, as I didn’t find any documentation, but if you watch the traffic, you can figure out where the data comes from. I’ve created a [small script](https://github.com/a-mroz/google-sheet-helpers) for that:

```javascript
function FUND_PRICE(fundTicker) {
    const url = `https://www.analizy.pl/api/quotation/fio/${fundTicker}`
    const result = UrlFetchApp.fetch(url)
    const parsedData = JSON.parse(result.getContentText())
  
    const prices = parsedData.series[0].price
    prices.sort((a, b) => new Date(b.date) - new Date(a.date))
    console.log(prices[0])
    return prices[0].value;
}
```

Deploying and using it (e.g. `=FUND_PRICE("INK04")`) is a matter of seconds (see the docs). And yet, it’s quite powerful.

# Summary

I realize that these are just the basics of what can be achieved in spreadsheets, especially when extended with custom scripts. Nevertheless, I believe it will save me quite some time in the future, and I hope it will inspire someone else too.