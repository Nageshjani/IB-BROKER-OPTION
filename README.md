
## ATM/Single Ticker/Single Week

```python
#important : We Are Fetching Current Month Data , Adjust Expiry according to yout need


from ibapi.client import EClient
from ibapi.wrapper import EWrapper
from ibapi.contract import Contract
import threading
import time
import pandas as pd

tickers = ["AAPL"]

class TradingApp(EWrapper, EClient):
    def __init__(self):
        EClient.__init__(self,self)
        self.data = {}
        self.df_data = {}
        
    def contractDetails(self, reqId, contractDetails):
        # print("extracting data for redID: {}".format(reqId))
        if reqId not in self.data:
            self.data[reqId] = [{"expiry":contractDetails.contract.lastTradeDateOrContractMonth,
                                 "strike":contractDetails.contract.strike,
                                 "call/put":contractDetails.contract.right}]
        else:
            self.data[reqId].append({"expiry":contractDetails.contract.lastTradeDateOrContractMonth,
                                     "strike":contractDetails.contract.strike,
                                     "call/put":contractDetails.contract.right})
    
    def contractDetailsEnd(self, reqId):
        super().contractDetailsEnd(reqId)
        print("Downloaded Data For ReqId:", reqId)
        self.df_data[tickers[reqId]] = pd.DataFrame(self.data[reqId])
        contract_event.set()
              
def websocket_con():
    app.run()
 
app = TradingApp()      
app.connect("127.0.0.1", 7497, clientId=1)

# starting a separate daemon thread to execute the websocket connection
con_thread = threading.Thread(target=websocket_con, daemon=True)
con_thread.start()
time.sleep(1) # some latency added to ensure that the connection is established

contract_event = threading.Event()

def contractOpt(symbol,sec_type="OPT",currency="USD",exchange="BOX"):
    contract = Contract()
    contract.symbol = symbol
    contract.secType = sec_type
    contract.currency = currency
    contract.exchange = exchange
    # contract.lastTradeDateOrContractMonth = time.strftime("%Y%m")
    contract.lastTradeDateOrContractMonth = "20230505"
    contract.right="C"
    return contract 

for ticker in tickers:
    contract_event.clear() #clear the set flag for the event object
    app.reqContractDetails(tickers.index(ticker), contractOpt(ticker)) # EClient function to request contract details
    contract_event.wait() #waiting for the set flag of the event object
    
    
df_dict = app.df_data  
currMonth=time.strftime("%Y%m")
print(f'---------Printing Data for Current Month : {currMonth}')  
# print(sorted(df_dict['INTC']))


def atm(df,ltp):
    strikes=df['strike']
    strikes=sorted(strikes)
    atm=None

    for strike in strikes:

        if strike>ltp:
            break
        atm=strike

    if strike==None:
        atm=strikes[-1]  

    return atm


print(atm(df_dict['AAPL'],168.32))

```

# ATM/Single Ticker/Single Month/All Weeks

```python
from ibapi.client import EClient
from ibapi.wrapper import EWrapper
from ibapi.contract import Contract
import threading
import time
import pandas as pd

tickers = ["AAPL"]
expiry_dates = ["20230505", "20230512", "20230519", "20230526"]  # Weekly expiry dates for May 2023

class TradingApp(EWrapper, EClient):
    def __init__(self):
        EClient.__init__(self,self)
        self.data = {}
        self.df_data = {}
        
    def contractDetails(self, reqId, contractDetails):
        if reqId not in self.data:
            self.data[reqId] = [{"expiry":contractDetails.contract.lastTradeDateOrContractMonth,
                                 "strike":contractDetails.contract.strike,
                                 "call/put":contractDetails.contract.right}]
        else:
            self.data[reqId].append({"expiry":contractDetails.contract.lastTradeDateOrContractMonth,
                                     "strike":contractDetails.contract.strike,
                                     "call/put":contractDetails.contract.right})
    
    def contractDetailsEnd(self, reqId):
        super().contractDetailsEnd(reqId)
        print("Downloaded Data For ReqId:", reqId)
        self.df_data[reqId] = pd.DataFrame(self.data[reqId])
        contract_event.set()
              
def websocket_con():
    app.run()
 
app = TradingApp()      
app.connect("127.0.0.1", 7497, clientId=1)

# starting a separate daemon thread to execute the websocket connection
con_thread = threading.Thread(target=websocket_con, daemon=True)
con_thread.start()
time.sleep(1) # some latency added to ensure that the connection is established

contract_event = threading.Event()

def contractOpt(symbol, expiry_date, sec_type="OPT", currency="USD", exchange="BOX"):
    contract = Contract()
    contract.symbol = symbol
    contract.secType = sec_type
    contract.currency = currency
    contract.exchange = exchange
    contract.lastTradeDateOrContractMonth = expiry_date
    contract.right="C"
    return contract 

reqid=0
id_list=[]
for ticker in tickers:
    for expiry_date in expiry_dates:
        contract_event.clear() 
        app.reqContractDetails(reqid,contractOpt(ticker, expiry_date)) 
        id_list.append(reqid)
        reqid+=1
        contract_event.wait() 
    
    
df_dict = app.df_data  
currMonth=time.strftime("%Y%m")
print(f'---------Printing Data for Current Month : {currMonth}')  

def atm(df,ltp):
    strikes=df['strike']
    strikes=sorted(strikes)
    atm=None

    for strike in strikes:
        if strike>ltp:
            break
        atm=strike

    if strike==None:
        atm=strikes[-1]  

    return atm


for reqid in id_list:
    expiry=expiry_dates[reqid]
    print(f'Atm For Expiry Date {expiry} : {atm(df_dict[reqid],168.30)}')


```

## ATM/Multi Tickers/Single Month/All Weekly

```python
from ibapi.client import EClient
from ibapi.wrapper import EWrapper
from ibapi.contract import Contract
import threading
import time
import pandas as pd

tickers = ["AAPL","INTC","AMZN"]
expiry_dates = ["20230505", "20230512", "20230519", "20230526"]  # Weekly expiry dates for May 2023

class TradingApp(EWrapper, EClient):
    def __init__(self):
        EClient.__init__(self,self)
        self.data = {}
        self.df_data = {}
        
    def contractDetails(self, reqId, contractDetails):
        if reqId not in self.data:
            self.data[reqId] = [{"expiry":contractDetails.contract.lastTradeDateOrContractMonth,
                                 "strike":contractDetails.contract.strike,
                                 "call/put":contractDetails.contract.right}]
        else:
            self.data[reqId].append({"expiry":contractDetails.contract.lastTradeDateOrContractMonth,
                                     "strike":contractDetails.contract.strike,
                                     "call/put":contractDetails.contract.right})
    
    def contractDetailsEnd(self, reqId):
        super().contractDetailsEnd(reqId)
        print("Downloaded Data For ReqId:", reqId)
        self.df_data[reqId] = pd.DataFrame(self.data[reqId])
        contract_event.set()
              
def websocket_con():
    app.run()
 
app = TradingApp()      
app.connect("127.0.0.1", 7497, clientId=1)

# starting a separate daemon thread to execute the websocket connection
con_thread = threading.Thread(target=websocket_con, daemon=True)
con_thread.start()
time.sleep(1) # some latency added to ensure that the connection is established

contract_event = threading.Event()

def contractOpt(symbol, expiry_date, sec_type="OPT", currency="USD", exchange="BOX"):
    contract = Contract()
    contract.symbol = symbol
    contract.secType = sec_type
    contract.currency = currency
    contract.exchange = exchange
    contract.lastTradeDateOrContractMonth = expiry_date
    contract.right="C"
    return contract 

reqid=0
id_list=[]
for ticker in tickers:
    for expiry_date in expiry_dates:
        contract_event.clear() 
        app.reqContractDetails(reqid,contractOpt(ticker, expiry_date)) 
        id_list.append(reqid)
        reqid+=1
        contract_event.wait() 
    
    
df_dict = app.df_data  
currMonth=time.strftime("%Y%m")
print(f'---------Printing Data for Current Month : {currMonth}')  

def atm(df,ltp):
    strikes=df['strike']
    strikes=sorted(strikes)
    atm=None

    for strike in strikes:
        if strike>ltp:
            break
        atm=strike

    if strike==None:
        atm=strikes[-1]  

    return atm

ltp_list=[168.70,29.93,103.37]
t=0
i=0
d={
    0:168.70,
    1:168.70,
    2:168.70,
    3:168.70,

    4:29.93,
    5:29.93,
    6:29.93,
    7:29.93,

    8:103.37,
    9:103.37,
    10:103.37,
    11:103.37,
    

}
for reqid in id_list:
  
    1   
    ltp=d[reqid]
    Atm=atm(df_dict[reqid],ltp)
    
    print(f'Atm : {Atm}')



```


## Getting current price of the ATM call and put options closest to expiry

```python
from ibapi.client import EClient
from ibapi.wrapper import EWrapper
from ibapi.contract import Contract
import threading
import time
import pandas as pd

tickers = ["INTC","AMZN","MSFT"]

class TradingApp(EWrapper, EClient):
    def __init__(self):
        EClient.__init__(self,self)
        self.data = {}
        self.df_data = {}
        self.underlyingPrice = {}
        self.atmCallOptions = {}
        self.atmPutOptions = {}
        self.optionPrice = {}
        
    def tickPrice(self, reqId, tickType, price, attrib):
        super().tickPrice(reqId, tickType, price, attrib)
        #print("TickPrice. TickerId:", reqId, "tickType:", tickType, "Price:", price)
        if tickType == 4:
        #tickType 4 corresponds to LTP
            if reqId >=100:
                self.optionPrice[options[reqId-100]] = price
            else:
                self.underlyingPrice[tickers[reqId]] = price
            
    def contractDetails(self, reqId, contractDetails):
        print("extracting data for redID: {}".format(reqId))
        if reqId not in self.data:
            self.data[reqId] = [{"expiry":contractDetails.contract.lastTradeDateOrContractMonth,
                                 "strike":contractDetails.contract.strike,
                                 "call/put":contractDetails.contract.right,
                                 "symbol":contractDetails.contract.localSymbol}]
        else:
            self.data[reqId].append({"expiry":contractDetails.contract.lastTradeDateOrContractMonth,
                                     "strike":contractDetails.contract.strike,
                                     "call/put":contractDetails.contract.right,
                                     "symbol":contractDetails.contract.localSymbol})
    
    def contractDetailsEnd(self, reqId):
        super().contractDetailsEnd(reqId)
        print("ContractDetailsEnd. ReqId:", reqId)
        self.df_data[tickers[reqId]] = pd.DataFrame(self.data[reqId])
        contract_event.set()
              
def websocket_con():
    app.run()

def specificOpt(local_symbol,sec_type="OPT",currency="USD",exchange="BOX"):
    contract = Contract()
    contract.symbol = local_symbol.split()[0]
    contract.secType = sec_type
    contract.currency = currency
    contract.exchange = exchange
    contract.right = local_symbol.split()[1][6]
    contract.lastTradeDateOrContractMonth = "20"+ local_symbol.split()[1][:6]
    contract.strike = float(local_symbol.split()[1][7:])/1000
    return contract    
    
def usTechOpt(symbol,sec_type="OPT",currency="USD",exchange="BOX"):
    contract = Contract()
    contract.symbol = symbol
    contract.secType = sec_type
    contract.currency = currency
    contract.exchange = exchange
    contract.lastTradeDateOrContractMonth = time.strftime("%Y%m")
    return contract

def usTechStk(symbol,sec_type="STK",currency="USD",exchange="ISLAND"):
    contract = Contract()
    contract.symbol = symbol
    contract.secType = sec_type
    contract.currency = currency
    contract.exchange = exchange
    return contract 

def streamSnapshotData(tickers):
    """stream tick leve data"""
    for ticker in tickers:
        app.reqMktData(reqId=tickers.index(ticker), 
                       contract=usTechStk(ticker),
                       genericTickList="",
                       snapshot=False,
                       regulatorySnapshot=False,
                       mktDataOptions=[])
        time.sleep(1) 
        
def streamSnapshotDataOpt(opt_symbols):
    """stream tick leve data"""
    for opt in opt_symbols:
        app.reqMktData(reqId=100+opt_symbols.index(opt), 
                       contract=specificOpt(opt),
                       genericTickList="",
                       snapshot=False,
                       regulatorySnapshot=False,
                       mktDataOptions=[])
        time.sleep(1) 
        
def atm_call_option(contract_df,stock_price):
    temp = contract_df[contract_df["call/put"]=="C"].sort_values(by="expiry",ascending=True)
    temp2 = temp[temp["expiry"] == temp["expiry"].iloc[0]]
    atm_option = temp2.iloc[(temp2["strike"] - stock_price).abs().argsort()[:1]]["symbol"].values[0]
    return atm_option

def atm_put_option(contract_df,stock_price):
    temp = contract_df[contract_df["call/put"]=="P"].sort_values(by="expiry",ascending=True)
    temp2 = temp[temp["expiry"] == temp["expiry"].iloc[0]]
    atm_option = temp2.iloc[(temp2["strike"] - stock_price).abs().argsort()[:1]]["symbol"].values[0]
    return atm_option
 
app = TradingApp()      
app.connect("127.0.0.1", 7497, clientId=1)

# starting a separate daemon thread to execute the websocket connection
con_thread = threading.Thread(target=websocket_con, daemon=True)
con_thread.start()
time.sleep(1) # some latency added to ensure that the connection is established

contract_event = threading.Event()

#streaming underlying price on a separate thread
streamThread = threading.Thread(target=streamSnapshotData, args=(tickers,))
streamThread.start()
time.sleep(3) #some lag added to ensure that streaming has started

for ticker in tickers:
    contract_event.clear() #clear the set flag for the event object
    app.reqContractDetails(tickers.index(ticker), usTechOpt(ticker)) # EClient function to request contract details
    contract_event.wait() #waiting for the set flag of the event object
    app.atmCallOptions[ticker] = atm_call_option(app.df_data[ticker],app.underlyingPrice[ticker])
    app.atmPutOptions[ticker] = atm_put_option(app.df_data[ticker],app.underlyingPrice[ticker])

#storing selected option contracts in a list    
options = list(app.atmCallOptions.values()) + list(app.atmPutOptions.values())

#streaming option contracts' current price on a separate thread
optStreamThread = threading.Thread(target=streamSnapshotDataOpt, args=(options,))
optStreamThread.start()
time.sleep(3) #some lag added to ensure that streaming has started

#utility function to get the option contract local symbol
def optIdentifier(symbol,right):
    for opt in options:
        if opt.split()[0]==symbol and opt.split()[1][6] == right:
            return opt

#get the current price of the option
app.optionPrice[optIdentifier("INTC","P")]  
   
```


    
                    
    


 


