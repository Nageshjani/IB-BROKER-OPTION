
## Atm For Single Stock/Single Expiry

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
