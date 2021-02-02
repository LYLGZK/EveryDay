
* 读取yaml类型配置文件

```go
type Mysql struct {
	Port int `json:"port" mapstructure:"port"`
	Host string `json:"host" mapstructure:"host"`
	IsSsl bool `json:"is_ssl" mapstructure:"is_ssl"`
}
// 这里必须在外部包装一层，要不解析不了
type Server struct {
	Mysql Mysql
}


func testConfigYaml()  {
	config := "config.yaml"

	v := viper.New()
	v.SetConfigType("yaml")
	v.SetConfigFile(config)

	err := v.ReadInConfig()
	if err != nil {
		panic("error:")
	}

	//解析成struct
	var server Server
	err1 := v.Unmarshal(&server)
	if err1 != nil {
		panic("error1")
	}
	fmt.Printf("%v",server.Mysql)
}  
```

* 读取ini类型的config

```go
func testConfigIni()  {
	config := "config.ini"

	v := viper.New()
	v.SetConfigType("ini")
	v.SetConfigFile(config)

	err := v.ReadInConfig()
	if err != nil {
		panic("error:")
	}

	fmt.Println(v.GetString("default.mysql_address"))

}
```

#### 官方文档
https://pkg.go.dev/go.uber.org/zap#pkg-constants

#### 参考文献
* https://www.liwenzhou.com/posts/Go/viper_tutorial/

