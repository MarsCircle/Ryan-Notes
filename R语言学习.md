
# R绘图基础
```R
setwd(dir="E:/Rdata2020")   //进入文件夹
getwd()
data<-read.csv("data2.csv") //读取
head(data)  //显示表格

data1<-data[,2]
head(data1) //显示第二列内容
mean(data1) //求均值
plot(data1) //画图 但是没搞懂画的是个啥玩意

data2<-data[2,]
head(data2) //显示第二行内容

data3<-data[,2:13]
data4<-apply(data3[,1:12],2,mean)  求所有列的均值 ，2表示列 1表示行
polt(data4 , type="b" , col="red" , main="Province GDP" ,xlab="number" ,ylab="GDP" , bty="l")
```



```R
library(ggplot2) //调用ggplot2包

//mtcars只是一个例子数据集
plot(mtcars$wt ,mtcars$mpg) //散点图   $是用于调用数据集的某一列

plot(pressure$temperature , pressure$pressure ，type="l") //曲线图
points(pressure$temperature , pressure$pressure)  //给上面的线图 标出点

lines(pressure$temperature , pressure$pressure/2 ,type="l" ,col="red")
//在上图中画出另外一条线
points(pressure$temperature , pressure$pressure/2,col="red") //给另外一条线也加上点

barplot(table(mtcars$cyl))  //柱状图
hist(mtcars$mpg,breaks=10) //直方图
boxplot(mtcars$mpg) //箱图

curve(x^3 - 5*x ,from=-4,to=4) // 给函数画图
myfun <- function(xvar){         //自定义函数 有xvar这一个参数
    1/(1 + exp(-xvar + 10))
}
curve(myfun(x) , from=0,to=20)
```

```R
//绘图基础
dat=diamonds // 用R自带的一个数据集
qplot(carat , price ,data = diamonds) //用自带的数据集画出散点图
qplot(log(carat)  , log(price) , data = diamonds) //进行一些数据的处理再散点
qplot(carat ,x * y * z,data = diamonds)

//改进
dsmall = diamonds[sample(nrow(diamonds),100) , ] //对数据集做一个简化 ，方便练习而已
qplot(carat,price,data = dsamll ,colour = color) 
//这里的color只是数据集的一个变量，这样写会给color里的不同值 随机划分颜色，再散点
qplot(carat,price,data = dsamll ,shape = cut)
//cut也只是一个变量，这样写会给 cut不同值 随机划分不同的形状

qplot(carat , price, data = diamonds ,alpha = I(1/10)) 
qplot(carat , price, data = diamonds ,alpha = I(1/100)) //数越小越透明
qplot(carat , price, data = diamonds ,alpha = I(1/200))//可以使点变得半透明

qplot(carat , price ,data = dsamll , geom = c("point" , "smooth"))


```

```R
library(splines)   
qplot(carat , price , data = dsamll , geom = c("point" , "smooth") ,method = "lm")
//定义了lm 变成一个直线 ，阴影处为standard error标准误差
qplot(carat , price , data = dsamll , geom = c("point" , "smooth") ,method = "lm" ,formula = y ~ ns(x,5))

//箱型图 color是边界线的颜色 ， fill是填充的颜色
qplot(color , price ,data = dsamll ,geom = "boxplot")

qplot(color , price ,data = dsamll ,geom = "boxplot" ,size = 2 ) //size是线的粗细

qplot(color , price ,data = dsamll ,geom = "boxplot" ,fill="blue") //这里虽然是blue但颜色不是                                                  //blue,下面是解决办法
qplot(color , price ,data = dsamll ,geom = "boxplot" ,fill=I("blue"))//这才是蓝色
qplot（color，price,data=dsmall ,geom ="boxplot")  +geom_boxplot(outlier.colour = "green",
                                                              outlier.size = 10,
                                                              fill = "red",
                                                              colour = "blue",
                                                              size = 2)


qplot(carat , data = diamonds ,geom = "histogram" ,colour = color)//color还只是数据集的一个变量
qplot(carat , data = diamonds ,geom = "density") //密度曲线
qplot(carat , data = diamonds ,geom = "density" ,colour = color)  
qplot(carat , data = diamonds ,geom = "histogram" ,fill = color)

qplot(carat , data = diamonds ,geom = "bar" ,fill = color)  //柱状图

//折线图，这里换了一个数据集
qplot(date , unemploy / pop ,data = economics ,geom = "line")
qplot(date , uempmed , data = econmics ,geom = "line")

//path和line的区别
qplot(unemploy / pop ,uempmed , data = economics ,geom = c("point","path"))
//把year用颜色深浅标记
year = funtion(x) as.POSIXlt(x)$year + 1900
qplot(unemploy /pop,uempmed ,data =economics ,geom = "path" ,colour =year(date))
```



```R
setwd("~/documents/R programming/ggplot2") //加载工作地址 文件夹
library(ggplot2)
attach(mpg) 
head(mpg)  // 这个可以看数据

qplot（displ , hwy ,data = mpg ,colour = factor(cyl))  //factor(cyl)、displ 、hwy是变量
qplot(displ , hwy ,data = mpg ,colour = factor(cyl) ,
     geom=c("smooth","point"),
     method="lm")
qplot(displ,hwy,data=mpg , facets = . ~ year) + geom_smooth() //分年Smooth

```

```R
//存储
qplot（displ , hwy ,data = mpg ,colour = factor(cyl)) 
summary(p)
save(p , file ="plot.rdata")
load("plot.rdata")
ggsave("plot.png" ,width = 5 ,height =5)
```

```R
p = ggplot(diamonds ,aes(carat ,price ,colour = cut))
p  //这里缺东西 会报错
p = p + geom_point() //p = p + geom_smooth()或者smooth也可以

p <- p + layer(
	geom = "bar" ,
	geom_params = list(fill = "steelblue"),
	stat = "bin",
    stat_params = list(binwidth = 0.5)
)
```

```R
//封装
bestfit = geom_smooth(method = "lm" ,
                     se = T ,
                     colour = "steelblue" , 
                     alpha =0.5,
                     size = 2)
qplot(sleep_rem ,sleep_total ,data = msleep) + bestfit
qplot(displ , hwy,data=mpg,facets = .~year) + bestfit
```

```R
p = ggplot(mtcars ,aes(mpg ,wt ,colour = cyl)) +geom_point()
p
mtcars = transform(mtcars , mpg = mpg^2) //修改数据后绘图
p %+% mtcars
```

```R
library(ggplot2)
p = ggplot(mtcars ,aes(x = mpg , y = wt))
p + geom_point()
p + geom_point(aes(colour = factor(cyl))) //加颜色
p + geom_point(aes(y = disp)) //替换y
p + geom_point(aes(y = NULL)) //取消y

p + geom_point(colour = "green")
p + geom_point(aes(colour = I("blue"))
```

```R
library(nlme)
data(package ="nlme")
head(Oxboys)  //Oxboys是一个数据集的名字 在nlme这个包中
str(Oxboys)
p = ggplot(Oxboys,aes(age,height,group =Subject)) + geom_line()//以subject这个变量来分类

p + geom_smooth(aes(group=Subject) , method="lm" ,se = F)
p + geom_smooth(aes(group = 1) ,method = "lm" ,se = F ,size=2)
```

```R
ggplot(diamonds,
      aes(clarity,fill = cut)) + geom_bar(position = "stack") 
ggplot(diamonds,
      aes(clarity,fill = cut)) + geom_bar(position = "fill")  //反映比例
ggplot(diamonds,
      aes(clarity,fill = cut)) + geom_bar(position = "dodge")  //每一种的内部分类
```

