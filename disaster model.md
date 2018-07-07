# Disastor_Model

![disastor_data](https://github.com/chapman515/photo/blob/master/Figure_1.png) </br>
是次使用英國礦難發生的次數作例子。由此圖可見，在某年開始的死亡人數有下跌跡象。現希望透過貝葉斯機率估算出其開始改變的年份（後稱轉捩點），改變前的機率分佈的平均值 和 改變後的機率分佈的平均值。
<br/>

## python 代碼
	"""
	A model for the disasters data with a changepoint
	changepoint ~ U(0, 110)
	early_mean ~ Exp(1.)
	late_mean ~ Exp(1.)
	disasters[t] ~ Po(early_mean if t <= switchpoint, late_mean otherwise)
	"""
	
	from pymc import *
	from numpy import array, empty
	from numpy.random import randint
	
	disasters_array = array([4, 5, 4, 0, 1, 4, 3, 4, 0, 6, 3, 3, 4, 0, 2, 6,
	                         3, 3, 5, 4, 5, 3, 1, 4, 4, 1, 5, 5, 3, 4, 2, 5,
	                         2, 2, 3, 4, 2, 1, 3, 2, 2, 1, 1, 1, 1, 3, 0, 0,
	                         1, 0, 1, 1, 0, 0, 3, 1, 0, 3, 2, 2, 0, 1, 1, 1,
	                         0, 1, 0, 1, 0, 0, 0, 2, 1, 0, 0, 0, 1, 1, 0, 2,
	                         3, 3, 1, 1, 2, 1, 1, 1, 1, 2, 4, 2, 0, 0, 1, 4,
	                         0, 0, 0, 1, 0, 0, 0, 0, 0, 1, 0, 0, 1, 0, 1])
	
	
	# Define data and stochastics
	switchpoint = pymc.DiscreteUniform(
	    'switchpoint',
	    lower=0,
	    upper=110,
	    doc='Switchpoint[year]')
	
	early_mean = Exponential('early_mean', beta=1.)
	late_mean = Exponential('late_mean', beta=1.)
	
	
	
	@deterministic(plot=False)
	def rate(s=switchpoint, e=early_mean, l=late_mean):
	    ''' Concatenate Poisson means '''
	    out = empty(len(disasters_array))
	    out[:s] = e
	    out[s:] = l
	    return out
	
	disasters = Poisson('disasters', mu=rate, value=disasters_array, observed=True)

	from pymc import MCMC

	M = MCMC([switchpoint,early_mean,late_mean,rate,disasters])
	M.sample(iter=10000, burn=1000, thin=10)
	print switchpoint.value
	print rate.value

	
	print M.trace('switchpoint')[:]
	pymc.Matplot.plot(M)
	plt.show()
	
## 分行解釋
	switchpoint = pymc.DiscreteUniform(
		    'switchpoint',
		    lower=0,
		    upper=110,
		    doc='Switchpoint[year]')
假設switchpoint（轉捩點）是平均分布 ，是次例子共有111個數據樣本，設其下限為0，上限為110。
<br/>
***
    early_mean = Exponential('early_mean', beta=1.)
    late_mean = Exponential('late_mean', beta=1.)
   另外假設先驗概率(prior)是指數分布(exponential distribution )
   <br/>
***
   	@deterministic(plot=False)
	def rate(s=switchpoint, e=early_mean, l=late_mean):
	    ''' Concatenate Poisson means '''
	    out = empty(len(disasters_array))
	    out[:s] = e
	    out[s:] = l
	    return out
利用裝飾器來修飾函數 rate
   <br/>
***
	disasters = Poisson('disasters', mu=rate, value=disasters_array, observed=True)
定義disaters為泊松分布（Poisson distribution)，其分布的mean(mu)的數值是rate。value＝disasters_array，即是所觀測到的數字，所以把observed也定義為true.
  <br/>

***

	M = MCMC([switchpoint,early_mean,late_mean,rate,disasters])
	M.sample(iter=10000, burn=1000, thin=10)
把所有需要的變量都傳入MCMC()中，然後透過M.sample()為機率密度函數(Probability density function)進行採樣，模擬統計分佈。


- iter=10000代表是次估算重覆進行10000次。
- burn=1000即放棄頭1000個估算，因為一開始的數值沒有代表性，需要設定放棄一開始的估算。("burn” specifies the minimum iteration that we need before we are sure that we have achieved the “true” posterior distribution)
- thin=10即只取當中10分之1的數字作為最後的估算。




burn = 0 | burn = 1000
---- | ---
 ![em0](https://github.com/chapman515/photo/blob/master/em0.png)|  ![em](https://github.com/chapman515/photo/blob/master/em.png)
 ![lm0](https://github.com/chapman515/photo/blob/master/lm0.png)|  ![lm](https://github.com/chapman515/photo/blob/master/lm.png)
 ![sw0](https://github.com/chapman515/photo/blob/master/sw0.png)|  ![sw](https://github.com/chapman515/photo/blob/master/sw.png)
設定burn=0的時侯，前面的數據差異大，影響估算|設定burn=1000的時侯，前面的數據會被忽略，盡量避免對估算的影響。


## 輸出結果

early mean的估算：
![](https://github.com/chapman515/photo/blob/master/early_mean.png)
第一個分布的mean是~3.0
***
 late mean的估算：![](https://github.com/chapman515/photo/blob/master/late_mean.png)
 第二個分布的mean是~0.9
***
switchpoint的估算： ![](https://github.com/chapman515/photo/blob/master/switchpoint.png) 
轉捩點是在第40年發生
***
