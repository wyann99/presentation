# CRM 2.0 Infrastructure
Dong Bin
---
## Why 2.0
- The existing CRM has lots of efficiency and design issues
- Each business department has built its own system, which is a waste
- The code and infrastructure was not design well for maintain and extensibility
---
## Structure
![Structure](crm_infrastructure/assets/structure.png)
---
## Console
![Console](annual-report/assets/console.png)
---
## Car Recommendation
![Recommendation](annual-report/assets/recommendation.png)
---
## Task List
![Clue List](annual-report/assets/cluelist.png)
---
## New tools and infrastructure
- Docker cloud
- Protubuf and grpc
- New frontend and Grpcweb
- ETCD
- BI Log pipeline
- Async Job
- Some useful tools

---
## Docker cloud
- Easy for deploy and CI |
- Environment consistency |
- Scale and cost efficiency |
- Demo |
---
## Protobuf and gRPC
- It is a painful and wasteful dead cost to debug interface |
- Strict and robust protocol between processes, language and teams |
- Code generation to avoid human mistake |
- Document automation and consistency |
- Performance for bindwidth and marshal |
---
![GRPC](crm_infrastructure/assets/grpc.jpg)
---
## Example
```protobuf
/**
 * 任务信息服务
 */
service AssignmentInfo {
    /// 获取任务详情接口
    rpc Get(AssignmentIDRequest) returns (AssignmentInfoResponse) {}
}
/**
 * 请求：任务ID
 */
message AssignmentIDRequest {
    int32 id = 1; /// 任务号
}

/**
 * 响应：任务详细信息
 */
message AssignmentInfoResponse {
    int32 id = 1;                                       /// 任务号
    BusinessLine business_line = 2;                     /// 业务线
    Category category = 3;                              /// 任务类别: 1:新线索, 2:回访 3:失败工单
    Status  status = 4;    
...
```

Document: http://call-center-proto.guazi-cloud.com/#assignment.AssignmentInfo
---
## Service code example
```go
// InfoService 任务信息服务
type InfoService struct{}

// Get 获取任务信息接口
func (s *InfoService) Get(ctx context.Context, assignmentIDRequest *assignment.AssignmentIDRequest) (*assignment.AssignmentInfoResponse, error) {
	bl, err := buslogic.GetInstanceWithContext(ctx)
	if err != nil {
		return nil, status.Error(codes.Internal, "服务器内部错误[BusLogic]")
	}
	assignmentInfo, err := bl.GetAssignmentInfo(int(assignmentIDRequest.Id))
	if err != nil {
		return nil, status.Error(codes.Internal, err.Error())
	}
	return assignmentInfo, nil
}
```
---
## New web framework and grpcweb
- Element: Reused components across team |
- grpcweb: leverage grpc and js generation |
- Typescript? |
- [Demo](http://crm.guazi-cloud.com/)

---
## Flexible and Dynamic Configuration by ETCD
- Dynamic config without restart or reload |
- Centralize config for multiple projects |
- Web UI for review and update |
- Awesome HA and performance |
- [ETCD UI Demo](http://e3w-staging.guazi-cloud.com/#/kv/?_k=9pmfzd) |
---
## BI Log Pipeline
### Background
- Why it is stupid and evil to store log in Mysql
- What is Avro
- Pipeline automation
---
## Pipeline Architect

![Schema](crm_infrastructure/assets/schema-registry-usage.png)

---

![Kafka Connect](crm_infrastructure/assets/kafka-connect.png)
---
## How to Define Avro
```avdl
@namespace("com.guazi.avro.call")
protocol GuaziCall {
    import idl "../common/common.avdl";

    enum ProviderType {
        ZT, //zhong tong
        RL,
        TR
    }
    record FactCallRecord {
        /** 业务线 */
        int business_line;
        /** 任务处理ID */
        string bundle_id;
```
---
## How to Send Avro Log
```go
gzavro.SaveAvro( &call.FactCallRecord{
   Business_line: 2,
   Bundle_id:     "pxhtest2",
   Call_type:     call.CallType_CALLIN.Enum(), //枚举赋值
   Contact_phone: "123423423******",
   Call_tag_id:   "abc132444564",
   Provider_type: call.ProviderType_TR.Enum(),
   //嵌套结构赋值
   Operator_header: &call.OperatorHeader{
      Timestamp:   time.Now().UnixNano() / 1e6,
      Http_host:   "test.pxh",
      Request_id:  "sweffsewffewf",
      Operator_id: int32(23453), // union int 类赋值
   },
})
```
---
## Async Job
- Batch job is not HA and scalable |
- Hard for failure tolerant |
- Hard to test
---
Start worker
```go
worker.NewServer(mainConfig.Redis.Address, configs.DelayTaskQueue)
```
Enqueue a job
```go
recycleUnassignedTimeout := configs.GetRecycleUnassignedTimeout()
eta := time.Now().Add(recycleUnassignedTimeout)
signature := &tasks.Signature{
  UUID:       fmt.Sprintf("%s_%s", configs.TaskNameAfterSaleRecycleUnassigned, eta.Format(time.RFC3339)),
  Name:       configs.TaskNameAfterSaleRecycleUnassigned,
  ETA:        &eta,
  RetryCount: configs.DelayTaskRetryCount,
}
//Send task
worker.SendTask(signature)
```
---
## New SQL API
- New go-orm and squirrel
```go
 t.Exec(`UPDATE sale_clue_queue SET assignment_status = ?, result_type = ? WHERE id in (??)`, status, resultType, assignmentIDs)
sqlObj := squirrel.Select("operator,count(1) as count").From("assignment").NotEq("operator", 0).Condition()
query, args, err := sqlObj.GroupBy("operator").ToSql()
```
---
## Kafka Stream
- Realtime dashboard and data analysis
- Monitor
- Session and Window aggregation
---
## Useful tools
- gip
- Code checking
- Http test fixture
