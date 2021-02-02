# set 数据类型

## set的数据结构特点
1、在整个集合中是只能存在一份数据，因为他是无序的，所以只能存储一份数据。类似的会把之前的替换掉

## 常用方法和时间复杂度
### 1、添加元素
```redshift
    sadd key value
    
    sadd user:favorite:info:123 124 //123关注的
```

### 2、获取所有的元素
```redshift
    members key

    members user:info:123 //获取123用户关注的所有人员id
```

### 3、获取元素数量
```redshift
    scard key
    
    scard user:info:123 //获取123用户关注人数量
```

### 4、判断一个元素是否在一个集合中
```redshift
    sismember key member

    sismember user:info:123 124
```
### 5、移除并返回集合中一个随机元素
这个用户：？

插入顺序：101，102，103，104 
实验结果：先除掉了104，再去掉了102，所以这个是随机的。不是一定顺序的。
```redshift
    spop key 
    spop user:favorite:100
```
### 6、移除集合中一个或多个随机数
```redshift
    srem KEY 
```

## 用go实现所有方法
```golang
package initialize

import (
	"fmt"
	"github.com/go-redis/redis/v7"
)

var RedisClient *redis.Client

func redisOptions() *redis.Options {
	//可以使用配置文件

	operations := redis.Options{
		Addr: "localhost:16379",
		Password: "",
		DB: 0,
	}

	return &operations
}

func init()  {
	redisConnectOperations := redisOptions()
	RedisClient = redis.NewClient(redisConnectOperations)

	_,pingErr := RedisClient.Ping().Result()
	if pingErr != nil {
		fmt.Println("connect failed",pingErr)
	}
	fmt.Println("connect success")
	//defer RedisClient.Close()
}


package user

import (
	"fmt"
	"github.com/go-redis/redis/v7"
	"strconv"
	"warehouse/initialize"
)

type User struct {
	Id uint64 `json:"id"`
	Name string `json:"name"`
	Favorite []uint64 `json:"favorite"`
}

func (oldUser User)AddFavorite(user User) *redis.IntCmd {
	cacheKey := "user:info:" + strconv.FormatInt(int64(oldUser.Id),10) //后面那个表示进制
	newUserId := strconv.FormatInt(int64(user.Id),10)
	res := initialize.RedisClient.SAdd(cacheKey,newUserId)
	return res
}

func (user User) GetAllFavoriteMembers() *redis.StringSliceCmd {
	cacheKey := "user:info:" + strconv.FormatInt(int64(user.Id),10) //后面那个表示进制
	allFavorite := initialize.RedisClient.SMembers(cacheKey)
	return allFavorite
}

func (user User) GetFavoriteCount() *redis.IntCmd {
	cacheKey := "user:info:" + strconv.FormatInt(int64(user.Id),10) //后面那个表示进制
	result := initialize.RedisClient.SCard(cacheKey)
	return result
}

func (user User) IsFavoriteUser(u User) bool  {
	cacheKey := "user:info:" + strconv.FormatInt(int64(user.Id),10) //后面那个表示进制
	result := initialize.RedisClient.SIsMember(cacheKey,u.Id)
	return result.Val()
}

/**
	差集，获取user喜欢的，u不喜欢的
 */
func (user User)DiffFavorite(u User) []string {
	cacheKey := "user:info:" + strconv.FormatInt(int64(user.Id),10) //后面那个表示进制
	newCacheKey := "user:info:" + strconv.FormatInt(int64(u.Id),10) //后面那个表示进制
	result := initialize.RedisClient.SDiff(cacheKey,newCacheKey)
	diffResult := result.Val()
	return diffResult
}

/**
	他们两个都喜欢的
 */
func (user User) InterFavorite(u User) []string {
	cacheKey := "user:info:" + strconv.FormatInt(int64(user.Id),10) //后面那个表示进制
	newCacheKey := "user:info:" + strconv.FormatInt(int64(u.Id),10) //后面那个表示进制
	result := initialize.RedisClient.SInter(cacheKey,newCacheKey)
	diffResult := result.Val()
	return diffResult
}

func (user User) MoveFavorite()  {
	
}

/**
	随机移除一个元素并且返回
 */
func (user User) MoveUserFavoriteOfRand() string {
	cacheKey := "user:info:" + strconv.FormatInt(int64(user.Id),10) //后面那个表示进制
	popResult := initialize.RedisClient.SPop(cacheKey)
	return popResult.Val()
}

func (user User) RandFavorite(u User)  {
	
}


/**
这个用户已经不存在了，删除他的记录
*/
func (user User) DeleteUserFavorite() int64 {
	cacheKey := "user:info:" + strconv.FormatInt(int64(user.Id),10) //后面那个表示进制
	popResult := initialize.RedisClient.Del(cacheKey)
	return popResult.Val()
}

func (user User) CancelFavorite(u ...User) int64 {
	cacheKey := "user:info:" + strconv.FormatInt(int64(user.Id),10) //后面那个表示进制
	var uidArray []uint64
	for _, u2 := range u {
		uidArray = append(uidArray,u2.Id)
	}
	fmt.Println(uidArray)

	removeResult := initialize.RedisClient.SRem(cacheKey,uidArray)
	return removeResult.Val()
}
```


## 源码分析