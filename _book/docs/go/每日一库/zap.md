### 一、是什么？
zap是一个结构化日志的库。

### 二、几个类和对象、方法的作用
* encoder 编码器。主要作用就是要说明，这个日志的一些信息
* encoderConfig 编码器的配置
* 

### 三、使用流程
#### 1、配置

代码
~~~go
package zap_test

import (
	"github.com/natefinch/lumberjack"
	"go.uber.org/zap"
	"go.uber.org/zap/zapcore"
	"net/http"
)

var logger *zap.Logger
var sugarLogger *zap.SugaredLogger
var sugarLoggerNew *zap.SugaredLogger

/**
	初始化日志记录器
 */
func InitLogger()  {
	
	logger,_ = zap.NewProduction()
	sugarLogger = logger.Sugar()
}

func SimpleHttpRequest(url string)  {
	resp, err := http.Get(url)
	if err != nil {
		logger.Error(
			"Error fetching url..",
			zap.String("url", url),
			zap.Error(err))
	} else {
		logger.Info("Success..",
			zap.String("statusCode", resp.Status),
			zap.String("url", url))
		resp.Body.Close()
	}
}


func SimpleHttpRequestOfSugarLogger(url string)  {
	resp, err := http.Get(url)
	if err != nil {
		sugarLogger.Errorf("Error fetching URL %s : Error = %s", url, err)
	} else {
		sugarLogger.Infof("Success! statusCode = %s for URL %s", resp.Status, url)
		resp.Body.Close()
	}
}


func SimpleHttpRequestOfSugarLoggerToFile(url string)  {
	resp, err := http.Get(url)
	if err != nil {
		sugarLoggerNew.Errorf("Error fetching URL %s : Error = %s", url, err)
	} else {
		sugarLoggerNew.Infof("Success! statusCode = %s for URL %s", resp.Status, url)
		resp.Body.Close()
	}
}

//修改输出内容
func getEncoder() zapcore.Encoder {
	encoderConfig := zap.NewProductionEncoderConfig()
	encoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder
	encoderConfig.EncodeLevel = zapcore.CapitalLevelEncoder
	return zapcore.NewJSONEncoder(encoderConfig)
}


func AddNotice()  {
	//编码器，如何写入文件。这个是log的使用配置使用项
	encoder := getEncoder()
	//指定写入位置,而且是怎么写入，是不是要分割，追加等等
	writeSync := getLogWrite()
	//指定log_level
	core := zapcore.NewCore(encoder,writeSync,zap.DebugLevel)
	loggerNew := zap.New(core,zap.AddCaller()) //创建logger
	sugarLoggerNew = loggerNew.Sugar()
}

//使用lumberjack分割日志
/**
Filename: 日志文件的位置
MaxSize：在进行切割之前，日志文件的最大大小（以MB为单位）
MaxBackups：保留旧文件的最大个数
MaxAges：保留旧文件的最大天数
Compress：是否压缩/归档旧文件
 */
func getLogWrite() zapcore.WriteSyncer  {
	lumberjackLogger := &lumberjack.Logger{
		Filename: "./lumberjack.log",
		MaxSize: 1,
		MaxBackups: 5,
		MaxAge: 30,
		Compress: false,
	}
	//如果不采用切割的话
	//file,_ := os.OpenFile("./test.log",os.O_CREATE | os.O_APPEND | os.O_RDWR,os.ModePerm)
	//return zapcore.AddSync(file) //file 实现了io.write

	return zapcore.AddSync(lumberjackLogger)
}
~~~



#### 思考：
* zap的速度快，是为什么？


项目地址
https://github.com/uber-go/zap

参考文献：
https://www.liwenzhou.com/posts/Go/zap/