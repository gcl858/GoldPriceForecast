

```R
#載入資料
library("AzureML")
ws <- workspace()
dat <- download.datasets(ws, "Roc2010_t1_omit0.csv") 
```


```R
#將原本漲跌幅換算成百分比
dat<-dat*100 
head(dat)
```


<table>
<thead><tr><th scope=col>Ut</th><th scope=col>St</th><th scope=col>Ot</th><th scope=col>Gt</th><th scope=col>Ut1</th><th scope=col>St1</th><th scope=col>Ot1</th><th scope=col>Gt1</th></tr></thead>
<tbody>
	<tr><td>-0.50</td><td>1.59 </td><td> 2.67</td><td> 1.09</td><td>-0.07</td><td>-1.01</td><td> 0.10</td><td> 0.81</td></tr>
	<tr><td> 0.03</td><td>0.31 </td><td> 0.32</td><td>-0.02</td><td>-0.50</td><td> 1.59</td><td> 2.67</td><td> 1.09</td></tr>
	<tr><td>-0.25</td><td>0.05 </td><td> 1.71</td><td> 0.51</td><td> 0.03</td><td> 0.31</td><td> 0.32</td><td>-0.02</td></tr>
	<tr><td> 0.58</td><td>0.40 </td><td>-0.63</td><td>-0.80</td><td>-0.25</td><td> 0.05</td><td> 1.71</td><td> 0.51</td></tr>
	<tr><td>-0.58</td><td>0.29 </td><td> 0.11</td><td> 3.18</td><td> 0.58</td><td> 0.40</td><td>-0.63</td><td>-0.80</td></tr>
	<tr><td>-0.64</td><td>0.17 </td><td>-0.28</td><td>-0.45</td><td>-0.58</td><td> 0.29</td><td> 0.11</td><td> 3.18</td></tr>
</tbody>
</table>




```R
#安裝neuralnet套件
install.packages("neuralnet")
```

    Installing package into '/home/nbcommon/R'
    (as 'lib' is unspecified)


    
    The downloaded source packages are in
    	'/tmp/RtmpTF0Maf/downloaded_packages'



```R
#載入library
library(neuralnet) # for neuralnet(), nn model
```


```R
#由當日即前一日相關資料預測當日黃金單價
formula.bpn <- Gt ~ Ut + St + Ot + Ut1 + St1 + Ot1 + Gt1

bpn <- neuralnet(formula = formula.bpn, 
                 data = dat,
                 hidden = c(8,4,2),       # 三個隱藏層：8個nodes-4個nodes-2個nodes
                 learningrate = 0.01, # learning rate
                 threshold = 0.2,    # partial derivatives of the error function, a stopping criteria
                 stepmax = 5e4        # 最大的ieration數 = 50000(5*10^5)
                 
)
```


```R
# bpn模型
print(bpn)
plot.nn(bpn, rep="best")
```

    Call: neuralnet(formula = formula.bpn, data = dat, hidden = c(8, 4,     2), threshold = 0.2, stepmax = 50000, learningrate = 0.01)
    
    1 repetition was calculated.
    
            Error Reached Threshold Steps
    1 724.0702033      0.1997624943 35405
    



![png](output_5_1.png)



```R
#以訓練模型預測
pred <- compute(bpn, dat[,c(1:3,5:8)])  
head(pred$net.result)
```


<table>
<tbody>
	<tr><td>-0.2677838890</td></tr>
	<tr><td>-0.1164178906</td></tr>
	<tr><td> 0.4838770102</td></tr>
	<tr><td>-0.3176532768</td></tr>
	<tr><td> 0.4665877344</td></tr>
	<tr><td> 0.1769698168</td></tr>
</tbody>
</table>




```R
#預測及實際檔跌幅
max(pred$net.result)
max(dat$Gt)
min(pred$net.result)
min(dat$Gt)
plot(dat$Gt,type="l")
lines(pred$net.result,col="red")
```


4.18866554123433



4.79



-2.12395504904789



-8.91



![png](output_7_4.png)



```R
#將漲跌幅回推成黃金歷史牌價，黑色為訓練資料，紅色為預測資料
init_GP<-1113

for( i in c(1:length(pred$net.result)))
{
  if(i==1)  
  {
    pred_GP=c(init_GP+init_GP*(pred$net.result[i]/100))    
  }
  else
    pred_GP= c(pred_GP,pred_GP[i-1]+pred_GP[i-1]*(pred$net.result[i]/100))    
}

for( i in c(1:length(dat$Gt)))
{
  if(i==1)  
  {
    train_GP=c(init_GP+init_GP*(dat$Gt[i]/100))    
  }
  else
    train_GP= c(train_GP,train_GP[i-1]+train_GP[i-1]*(dat$Gt[i]/100))    
}
plot(pred_GP,ylim=c(min(min(pred_GP),min(train_GP)),max(max(pred_GP),max(train_GP))),type="l",col="red")
lines(train_GP)
```


![png](output_8_0.png)

