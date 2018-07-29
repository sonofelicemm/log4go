# log4go
## 1.简介
log4go是对google的log4go的一个简单封装。在自己的go代码中，只需要配置简单的log的路径，以及需要打印的日志级相关信息，即可使用日志工具。该日志工具支持将日志文件按时间、文件大小、日志级别进行文件切分。

目前已经在sonofelicemm的goframe项目中使用。具体使用示例可以参考：https://github.com/sonofelicemm/goframe
## 2.详细配置
1.新建log配置文件：log.go

```
import (
"bytes"
"encoding/json"
"fmt"
"math"
"path"
"sync"
"runtime"

log "github.com/sonofelice/log4go"
)

```
2.使用细节

```
var (
logger log.Logger
bufP   sync.Pool
)

type Config struct {
    Dir string
}

```
3.配置日志初始化逻辑，以及日志级别

```
func Init(c *Config) {
    if c.Dir != "" {
        logger = log.Logger{}
        log.LogBufferLength = 10240
        // new info writer
        iw := log.NewFileLogWriter(path.Join(c.Dir, "info.log"), false)
        iw.SetRotateDaily(true)
        iw.SetRotateSize(math.MaxInt32)
        iw.SetRotate(true)
        iw.SetFormat("[%D %T] [%L] [%S] %M")
        logger["info"] = &log.Filter{
            Level:     log.INFO,
            LogWriter: iw,
        }
        // new warning writer
        ww := log.NewFileLogWriter(path.Join(c.Dir, "warning.log"), false)
        ww.SetRotateDaily(true)
        ww.SetRotateSize(math.MaxInt32)
        ww.SetRotate(true)
        ww.SetFormat("[%D %T] [%L] [%S] %M")
        logger["warning"] = &log.Filter{
            Level:     log.WARNING,
            LogWriter: ww,
        }
        // new error writer
        ew := log.NewFileLogWriter(path.Join(c.Dir, "error.log"), false)
        ew.SetRotateDaily(true)
        ew.SetRotateSize(math.MaxInt32)
        ew.SetRotate(true)
        ew.SetFormat("[%D %T] [%L] [%S] %M")
        logger["error"] = &log.Filter{
            Level:     log.ERROR,
            LogWriter: ew,
        }
    }
}

// Close close resource.
func Close() {
    if logger != nil {
        logger.Close()
    }
}

// Info write info log to file or elk.
func Info(format string, args ...interface{}) {
    if logger != nil {
        logger.Info(format, args...)
    }
}

// Warn write warn log to file or elk.
func Warn(format string, args ...interface{}) {
    if logger != nil {
        logger.Warn(format, args...)
    }
}

// Error write error log to file or elk.
func Error(format string, args ...interface{}) {
    if logger != nil {
        logger.Error(format, args...)
    }
}
```
4.配置文件中读取日志路径，并初始化日志模块

```
var (
confPath string
// Conf global
Conf = &Config{}
)
	
// Config .
type Config struct {
	Log *log.Config
}
	
func init() {
	flag.StringVar(&confPath, "conf", "", "config path")
}
	
// Init init conf
func Init() (err error) {
	_, err = toml.DecodeFile(confPath, &Conf)
	return
}

```
## 3.使用示例
```
log.Info("%+v test_endpoint url is: %+v", GetRequestId(r), url)
```
在上述代码中通过%+v进行参数格式化