

代码
~~~go
package main

import (
	"encoding/json"
	"fmt"
	"github.com/mitchellh/mapstructure"
	"github.com/spf13/viper"
	"log"
	"os"
)
//"github.com/mitchellh/mapstructure"

//type Person struct {
//	Name string
//	Age  int
//	Job  string
//}
//
//type Cat struct {
//	Name  string
//	Age   int
//	Breed string
//}

//func main() {
//	datas := []string{`
//    {
//      "type": "person",
//      "name":"dj",
//      "age":18,
//      "job": "programmer"
//    }
//  `,
//		`
//    {
//      "type": "cat",
//      "name": "kitty",
//      "age": 1,
//      "breed": "Ragdoll"
//    }
//  `,
//	}
//
//	for _, data := range datas {
//		var m map[string]interface{}
//		err := jsonTest.Unmarshal([]byte(data), &m)
//		if err != nil {
//			log.Fatal(err)
//		}
//
//		switch m["type"].(string) {
//		case "person":
//			var p Person
//			mapstructure.Decode(m, &p)
//			fmt.Println("person", p)
//
//		case "cat":
//			var cat Cat
//			mapstructure.Decode(m, &cat)
//			fmt.Println("cat", cat)
//		}
//	}
//}

//func main()  {
//	var data string
//	data = "zhangsan"
//
//	res := duanyan.Duanyan(data)
//	fmt.Println(res)
//}

type Mysql struct {
	Port int `json:"port" mapstructure:"port"`
	Host string `json:"host" mapstructure:"host"`
	IsSsl bool `json:"is_ssl" mapstructure:"is_ssl"`
}

type Server struct {
	Mysql Mysql
}

type Person struct {
	Name string
	Age int
	Address string `mapstructure:"omitempty"` //这个意思是当对象中这个字段的值是默认值，就不出现在map中
}

type Friend1 struct {
	Person
}

type Friend2 struct {
	Person `mapstructure:",squash"` //squash 提到父结构体中，remain 是结构体中没有定义这个字段，这个只有当定义的字段是map[string]interface{} 或者map[interface{}]interface{}才能使用
}

func main() {

	testConfigYaml()
	os.Exit(1)
	datas := []string{`
    { 
      "type": "friend1",
      "person": {
        "name":"dj"
      }
    }
  `,
		`
    {
      "type": "friend2",
      "name": "dj2"
    }
  `,
	}

	for _, data := range datas {
		var m map[string]interface{}
		err := json.Unmarshal([]byte(data), &m)
		if err != nil {
			log.Fatal(err)
		}

		switch m["type"].(string) {
		case "friend1":
			var f1 Friend1
			mapstructure.Decode(m, &f1)
			fmt.Println("friend1", f1)

		case "friend2":
			var f2 Friend2
			mapstructure.Decode(m, &f2)
			fmt.Println("friend2", f2)
		}
	}
}

func test()  {
	var p Person
	p.Name = "zhangsan"
	p.Age = 23

	var m map[string]interface{}
	mapstructure.WeakDecode(p,&m) //要转化成谁，谁就在后面
	fmt.Println(m)


	data,_ := json.Marshal(m)
	fmt.Println(string(data))



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


~~~

参考文献
* https://juejin.cn/post/6855300813707804686