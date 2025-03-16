---
title: HyperledgerFabric资产案例链码实例
toc: true
date: 2022-04-29 00:19:32
tags: 区块链 分布式账本
categories: Fabric
---

​​点击阅读更多查看文章内容<!--more-->


# 案例分析
- 功能
  - 用户开户和销户
  - 资产登记，资产上链，与具体的用户绑定
  - 资产转让，资产所有权变更
  - 查询功能，用户查询，资产查询，资产变更的历史查询 
- 业务实体
  - 用户
     - 名字
     - 身份证（标识）
     - 资产列表
  - 资产
    - 名字
    - 标识
    - 特殊属性列表（车：排量、品牌、座位）
 - 资产变更记录
    - 资产标识
    - 资产的原始拥有者
    - 资产变更后的拥有者  
- 交互方法
   - 用户开户
     - 参数：名字、标识
   - 用户销户
      - 参数：标识
   - 资产登记
      - 参数：名字、标识、特殊属性列表、拥有者
   - 资产转让
      - 参数：拥有者、资产标识、受让者
   - 用户查询
      - 参数：用户标识；返回值：用户实体 
    - 资产查询
      - 参数：资产标识；返回值：资产实体 
    - 资产的变更历史查询
      - 参数：资产标识；返回值：资产的变更记录列表
# 案例测试
- 创建通道的创世交易（channel名字不能有大写）
`configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./config/assetschannel.tx -channelID assetschannel`
- 启动网络
`docker-compose up -d`
- 交互执行
`docker exec -it cli bash`
- 创建通道
`peer channel create -o orderer.example.com:7050 -c assetschannel -f /etc/hyperledger/config/assetschannel.tx`
- 加入通道(一个peer可以加入多个通道)
`peer channel join -b assetschannel.block`
- 安装链码
`peer chaincode install -n assets -v 1.0.0 -l golang -p github.com/chaincode/assetsExchange`
- 实例化链码
`peer chaincode instantiate -o orderer.example.com:7050 -C assetschannel -n assets -v 1.0.0 -l golang -c'{"Args":["init"]}'`

---
**测试方法**
- 用户开户
`peer chaincode invoke -C assetschannel -n assets -c '{"Args":["userRegister","user1","user1"]}'`
- 用户查询
`peer chaincode query -C assetschannel -n assets -c '{"Args":["queryUser","user1"]}'`
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/31eaf4d89bff0e34ded5b8bdaf0b9e4f_1740930613908.png)
- 资产登记
`peer chaincode invoke -C assetschannel -n assets -c '{"Args":["assetEnroll","asset1","asset1","metadate","user1"]}'`
- 资产查询
`peer chaincode query -C assetschannel -n assets -c '{"Args":["queryAsset","asset1"]}'`
![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/87d052a2975d304448160f6cf1837c05_1740930613908.png)再查询用户，可以看到用户资产已经改变
`peer chaincode query -C assetschannel -n assets -c '{"Args":["queryUser","user1"]}'`- ![在这里插入图片描述](https://cdn.jsdelivr.net/gh/shnpd/blog-pic@main/csdn/842c10ded0ce019b580041dc0cfadd94_1740930613908.png)
- 资产转让
新建用户user2
`peer chaincode invoke -C assetschannel -n assets -c '{"Args":["userRegister","user2","user2"]}'`
资产转让，user1的资产asset1转给user2
`peer chaincode invoke -C assetschannel -n assets -c '{"Args":["assetExchange","user1","asset1","user2"]}'`
- 查询资产变更历史
`peer chaincode query -C assetschannel -n assets -c '{"Args":["queryAssetHistory","asset1"]}'`
- 删除用户，用户的资产会被同时删除
`peer chaincode invoke -C assetschannel -n assets -c '{"Args":["userDestroy","user2"]}'`

# 案例代码

```go
package main

import (
	"encoding/json"
	"fmt"
	"github.com/hyperledger/fabric/core/chaincode/shim"
	pb "github.com/hyperledger/fabric/protos/peer"
)

//定义链码
type AssetsExchangeCC struct {
}

//资产默认的原始拥有者
const (
	originOwner = "originOwnerPlaceholder"
)

//资产
type Asset struct {
	Name     string `json:"name"`
	Id       string `json:"id"`
	Metadata string `json:"metadata"`
}

//用户
type User struct {
	Name   string   `json:"name"`
	Id     string   `json:"id"`
	Assets []string `json:"assets"`
}

//资产变更记录
type AssetHistory struct {
	//资产标识
	AssetId string `json:"asset_id"`
	//资产的原始拥有者
	OriginOwnerId string `json:"origin_owner_id"`
	//变更后的拥有者
	CurrentOwnerId string `json:"current_owner_id"`
}

//链码的初始化
func (c *AssetsExchangeCC) Init(stub shim.ChaincodeStubInterface) pb.Response {
	return shim.Success(nil)
}

//链码交互
func (c *AssetsExchangeCC) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
	//得到方法名
	funcName, args := stub.GetFunctionAndParameters()
	//根据不同方法去判断
	switch funcName {
	case "userRegister":
		//用户开户
		return userRegister(stub, args)
	case "userDestroy":
		//用户销户
		return userDestroy(stub, args)
	case "assetEnroll":
		//资产登记
		return assetEnroll(stub, args)
	case "assetExchange":
		//资产转让
		return assetExchange(stub, args)
	case "queryUser":
		//用户查询
		return queryUser(stub, args)
	case "queryAsset":
		//资产查询
		return queryAsset(stub, args)
	case "queryAssetHistory":
		//资产变更历史查询
		return queryAssetHistory(stub, args)
	default:
		return shim.Error(fmt.Sprintf("不支持的方法：%s", funcName))
	}
}

//用户开户
func userRegister(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	//判断个数必须为2个
	if len(args) != 2 {
		return shim.Error("参数个数错误")
	}
	//判断传过来的参数是a否为空
	name := args[0]
	id := args[1]
	if name == "" || id == "" {
		return shim.Error("无效的参数")
	}
	//判断用户是否存在，若存在，则报错
	if userBytes, err := stub.GetState(constructUserKey(id)); err == nil && len(userBytes) != 0 {
		return shim.Error("用户已存在")
	}

	//写入世界状态，传过来的是用户的名字和id，绑定User结构体 make([]string,0)
	user := &User{
		Name:   name,
		Id:     id,
		Assets: make([]string, 0),
	}
	//序列化
	userBytes, err := json.Marshal(user)
	if err != nil {
		return shim.Error(fmt.Sprintf("序列化与用户失败 %s", err))
	}

	//将对象状态写入数据库
	if err := stub.PutState(constructUserKey(id), userBytes); err != nil {
		return shim.Error(fmt.Sprintf("存入用户失败 %s", err))
	}
	//返回成功
	return shim.Success(nil)
}
func constructUserKey(userId string) string {
	return fmt.Sprintf("user_%s", userId)
}

//用户销户
func userDestroy(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	//参数个数1个
	if len(args) != 1 {
		return shim.Error("参数个数不对")
	}
	//校验参数正确性
	id := args[0]
	if id == "" {
		return shim.Error("无效的参数")
	}
	//判断用户是否存在
	userBytes, err := stub.GetState(constructUserKey(id))
	if err != nil && len(userBytes) == 0 {
		return shim.Error("找不到用户")
	}

	//写入状态
	if err := stub.DelState(constructUserKey(id)); err != nil {
		return shim.Error(fmt.Sprintf("删除用户失败 %s", err))
	}

	//删除用户名下的资产
	user := new(User)
	if err := json.Unmarshal(userBytes, user); err != nil {
		return shim.Error(fmt.Sprintf("反序列化失败 %s", err))
	}
	for _, assetid := range user.Assets {
		if err := stub.DelState(constructAssetKey(assetid)); err != nil {
			return shim.Error(fmt.Sprintf("删除资产失败 %s", err))
		}
	}
	return shim.Success(nil)
}

//使用组合键来区分
//所有的资产，用asset开头
func constructAssetKey(assetId string) string {
	return fmt.Sprintf("asset_%s", assetId)
}

//资产登记
func assetEnroll(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	if len(args) != 4 {
		return shim.Error("参数个数不对")
	}
	//验证正确性
	assetName := args[0]
	assetId := args[1]
	metadata := args[2]
	ownerId := args[3]
	//metadata可以为空
	if assetName == "" || assetId == "" || ownerId == "" {
		return shim.Error("无效的参数")
	}
	//验证拥有者是否存在,拥有者必须存在
	userBytes, err := stub.GetState(constructUserKey(ownerId))
	if err != nil || len(userBytes) == 0 {
		return shim.Error("找不到用户")
	}
	//验证资产是否存在,资产必须不存在
	if assetBytes, err := stub.GetState(constructAssetKey(assetId)); err == nil && len(assetBytes) != 0 {
		return shim.Error("资产已经存在")
	}

	//写入状态
	asset := &Asset{
		Name:     assetName,
		Id:       assetId,
		Metadata: metadata,
	}
	//序列化
	assetBytes, err := json.Marshal(asset)
	if err != nil {
		return shim.Error(fmt.Sprintf("序列化失败 %s", err))
	}
	//保存资产
	if err := stub.PutState(constructAssetKey(assetId), assetBytes); err != nil {
		return shim.Error(fmt.Sprintf("保存失败 %s", err))
	}

	//拥有者
	user := new(User)
	if err := json.Unmarshal(userBytes, user); err != nil {
		return shim.Error(fmt.Sprintf("反序列化失败 %s", err))
	}

	user.Assets = append(user.Assets, assetId)
	if userBytes, err = json.Marshal(user); err != nil {
		return shim.Error(fmt.Sprintf("序列化用户失败 %s", err))
	}
	//存储用户状态
	if err := stub.PutState(constructUserKey(user.Id), userBytes); err != nil {
		return shim.Error(fmt.Sprintf("保存用户失败 %s", err))
	}

	//资产历史变更
	history := &AssetHistory{
		AssetId:        assetId,
		OriginOwnerId:  originOwner,
		CurrentOwnerId: ownerId,
	}
	historyBytes, err := json.Marshal(history)
	if err != nil {
		return shim.Error(fmt.Sprintf("序列化失败 %s", err))
	}

	//使用fabric内置的组合键机制
	historyKey, err := stub.CreateCompositeKey("history", []string{
		assetId,
		originOwner,
		ownerId,
	})
	if err != nil {
		return shim.Error(fmt.Sprintf("创建key失败 %s", err))
	}

	//资产变更存储
	if err := stub.PutState(historyKey, historyBytes); err != nil {
		return shim.Error(fmt.Sprintf("保存变更历史失败 %s", err))
	}

	return shim.Success(nil)
}

//资产转让
func assetExchange(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	//参数个数为3个
	if len(args) != 3 {
		return shim.Error("参数个数不对")
	}
	//参数校验
	ownerId := args[0]
	assetId := args[1]
	currentOwnerId := args[2]
	if ownerId == "" || assetId == "" || currentOwnerId == "" {
		return shim.Error("无效的参数")
	}
	//验证当前和受让后的用户是否存在
	originOwnerBytes, err := stub.GetState(constructUserKey(ownerId))
	if err != nil || len(originOwnerBytes) == 0 {
		return shim.Error("用户找不到")
	}
	currentOwnerBytes, err := stub.GetState(constructUserKey(currentOwnerId))
	if err != nil || len(currentOwnerBytes) == 0 {
		return shim.Error("用户找不到")
	}
	//验证资产存在
	assetBytes, err := stub.GetState(constructAssetKey(assetId))
	if err != nil || len(assetBytes) == 0 {
		return shim.Error("资产找不到")
	}

	//校验原始拥有者确实拥有当前变更的资产
	originOwner := new(User)
	if err := json.Unmarshal(originOwnerBytes, originOwner); err != nil {
		return shim.Error(fmt.Sprintf("反序列化失败 %s", err))
	}

	//定义标记，标识资产是否存在
	aidexist := false
	for _, aid := range originOwner.Assets {
		if aid == assetId {
			//若找到该资产，则变更状态，结束循环
			aidexist = true
			break
		}
	}

	if !aidexist {
		return shim.Error("资产所有者不匹配")
	}

	//写入状态
	//1.将资产的原始拥有者资产id删除
	//2.新拥有者写入资产id,资产绑定
	//3.资产变更记录

	assetIds := make([]string, 0)
	for _, aid := range originOwner.Assets {
		if aid == assetId {
			//遍历到了要转让的资产
			continue
		}
		assetIds = append(assetIds, aid)
	}

	originOwner.Assets = assetIds
	//序列化
	originOwnerBytes, err = json.Marshal(originOwner)
	if err != nil {
		return shim.Error(fmt.Sprintf("序列化失败 %s", err))
	}

	//存储原始拥有者
	if err := stub.PutState(constructUserKey(ownerId), originOwnerBytes); err != nil {
		return shim.Error(fmt.Sprintf("存储用户失败 %s", err))
	}

	//当前拥有者
	currentOwner := new(User)
	if err := json.Unmarshal(currentOwnerBytes, currentOwner); err != nil {
		return shim.Error(fmt.Sprintf("反序列化失败 %s", err))
	}

	//绑定资产
	currentOwner.Assets = append(currentOwner.Assets, assetId)
	currentOwnerBytes, err = json.Marshal(currentOwner)
	if err != nil {
		return shim.Error(fmt.Sprintf("序列化失败 %s", err))
	}
	//存储
	if err := stub.PutState(constructUserKey(currentOwnerId), currentOwnerBytes); err != nil {
		return shim.Error("保存用户失败")
	}

	//插入资产变更记录
	history := &AssetHistory{
		AssetId:        assetId,
		OriginOwnerId:  ownerId,
		CurrentOwnerId: currentOwnerId,
	}

	historyBytes, err := json.Marshal(history)
	if err != nil {
		return shim.Error(fmt.Sprintf("序列化失败 %s", err))
	}
	historyKey, err := stub.CreateCompositeKey("history", []string{
		assetId,
		ownerId,
		currentOwnerId,
	})
	if err != nil {
		return shim.Error(fmt.Sprintf("创建key失败 %s", err))
	}
	//存储历史变更记录
	if err := stub.PutState(historyKey, historyBytes); err != nil {
		return shim.Error(fmt.Sprintf("保存失败 %s", err))
	}
	return shim.Success(nil)
}

//用户查询
func queryUser(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	//参数个数1个
	if len(args) != 1 {
		return shim.Error("参数个数不对")
	}

	//校验正确性
	ownerId := args[0]
	if ownerId == "" {
		return shim.Error("无效的参数")
	}

	userBytes, err := stub.GetState(constructUserKey(ownerId))
	if err != nil || len(userBytes) == 0 {
		return shim.Error("找不到用户")
	}
	return shim.Success(userBytes)
}

//资产查询
func queryAsset(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	//参数个数1个
	if len(args) != 1 {
		return shim.Error("参数个数不对")
	}

	//校验正确性
	assetId := args[0]
	if assetId == "" {
		return shim.Error("无效的参数")
	}

	assetBytes, err := stub.GetState(constructAssetKey(assetId))
	if err != nil || len(assetBytes) == 0 {
		return shim.Error("找不到资产")
	}
	return shim.Success(assetBytes)
}

//资产历史变更查询
func queryAssetHistory(stub shim.ChaincodeStubInterface, args []string) pb.Response {
	//参数个数1个
	if len(args) != 1 && len(args) != 2 {
		return shim.Error("参数个数不对")
	}

	//校验参数的正确性
	assetId := args[0]
	if assetId == "" {
		return shim.Error("无效的参数")
	}

	//queryType：all
	//默认为all
	queryType := "all"
	if len(args) == 2 {
		//变为用户传的值
		queryType = args[1]
	}

	//参数校验
	if queryType != "all" && queryType != "exchange" && queryType != "enroll" {
		return shim.Error(fmt.Sprintf("未知的查询类型 %s", queryType))
	}

	//校验资产是否存在
	assetBytes, err := stub.GetState(constructAssetKey(assetId))
	if err != nil || len(assetBytes) == 0 {
		return shim.Error("资产找不到")
	}

	keys := make([]string, 0)
	keys = append(keys, assetId)
	switch queryType {
	case "enroll":
		//资产登记
		keys = append(keys, originOwner)
	case "exchange", "all":
	default:
		return shim.Error(fmt.Sprintf("不支持的类型 %s", queryType))
	}

	//组合键
	//得到迭代器
	result, err := stub.GetStateByPartialCompositeKey("history", keys)
	if err != nil {
		return shim.Error(fmt.Sprintf("查询历史错误 %s", err))
	}

	//关闭
	defer result.Close()

	histories := make([]*AssetHistory, 0)
	for result.HasNext() {
		historyVal, err := result.Next()
		if err != nil {
			return shim.Error(fmt.Sprintf("查询错误 %s", err))
		}

		history := new(AssetHistory)
		if err := json.Unmarshal(historyVal.GetValue(), history); err != nil {
			return shim.Error(fmt.Sprintf("反序列化失败 %s", err))
		}

		//过滤，不是资产转让的记录过滤
		if queryType == "exchange" && history.OriginOwnerId == originOwner {
			continue
		}
		histories = append(histories, history)
	}

	historiesBytes, err := json.Marshal(histories)
	if err != nil {
		return shim.Error(fmt.Sprintf("序列化失败 %s", err))
	}
	return shim.Success(historiesBytes)
}

func main() {
	err := shim.Start(new(AssetsExchangeCC))
	if err != nil {
		fmt.Println("启动链码失败")
	}
}

```

