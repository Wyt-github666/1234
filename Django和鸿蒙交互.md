### **Django和鸿蒙交互**

#### 第四课

##### 前端代码理解

```json
// 每一个httpRequest对应一个HTTP请求任务，不可复用
let httpRequest = http.createHttp();
// 用于订阅HTTP响应头，此接口会比request请求先返回。可以根据业务需要订阅此消息
httpRequest.on('headersReceive', (header) => {
    console.info('header: ' + JSON.stringify(header));
});
httpRequest.request(
    // 填写HTTP请求的URL地址，可以带参数也可以不带参数。URL地址需要开发者自定义。请求的参数可以在extraData中指定
    "EXAMPLE_URL",    //发送请求的网站
    {
        method: http.RequestMethod.POST, // 可选，默认为http.RequestMethod.GET
        // 开发者根据自身业务需要添加header字段
        header: {
            'Content-Type': 'application/json'    //非表单数据，可以用request.body  request.data来获取数据
        },
        // 当使用POST请求时此字段用于传递内容
        extraData: {
            "data": "data to send",       //向指定url传递的数据
        },
        expectDataType: http.HttpDataType.STRING, // 可选，指定返回数据的类型
        usingCache: true, // 可选，默认为true
        priority: 1, // 可选，默认为1
        connectTimeout: 60000, // 可选，默认为60000ms
        readTimeout: 60000, // 可选，默认为60000ms
        usingProtocol: http.HttpProtocol.HTTP1_1, // 可选，协议类型默认值由系统自动指定
    }, (err, data) => {                 // err后端传递的状态 data 后端传递的数据 
        if (!err) {
            // data.result为HTTP响应内容，可根据业务需要进行解析
            console.info('Result:' + JSON.stringify(data.result));         // 对后端传递来的数据进行转化
            console.info('code:' + JSON.stringify(data.responseCode));       
            // data.header为HTTP响应头，可根据业务需要进行解析
            console.info('header:' + JSON.stringify(data.header));
            console.info('cookies:' + JSON.stringify(data.cookies)); 
        } else {
            console.info('error:' + JSON.stringify(err));
            // 取消订阅HTTP响应头事件
            httpRequest.off('headersReceive');
            // 当该请求使用完毕时，调用destroy方法主动销毁
            httpRequest.destroy();
        }
    }
);
```

##### 案例：简单的传递数据

```python
##后端
class Send(APIView):
    def post(self, request):
        data = request.body			# 接受前端传递来的数据
        data_1 = request.data      # 不能用request.POST来接受前端，因为其只能接受表单类型的数据
        print(data)
        print(data_1)
        json_data = {
            'animal':[              # 以json格式传递给前端
                {
                   'name':'小狗',
                   'num':2
                },
                {
                   'name': '小猫',
                   'num': 2
                }
            ]
        }
        return JsonResponse(json_data)        # 传递数据
    
"""表单数据：a=request.POST.get('a') a=request.POST.getlist('a')   Content-Type(请求头)为application/x-www-form-urlencoded(form表单默认格式)

非表单数据： str = request.body.decode()  Content-Type(请求头)为application/json(json格式)，multipart/form-data(文件)等 或  data = json.loads(request.body)（将json数据转成python对象） """
```

```json
Get(){
    let httpRequest = http.createHttp();
    httpRequest.on('headersReceive', (header) => {
      console.info('header: ' + JSON.stringify(header));
    });
    httpRequest.request(
      "http://127.0.0.1:8000/send/",                   // 前端传递数据的Url地址
      {
        method: http.RequestMethod.POST,				//  post方法 
        header: {
          'Content-Type': 'application/json'         // 开发者根据自身业务需要添加header字段.此时不能用request.POST
        },
        extraData: {
          "id":1,
          "name":"xiaoming",
          "flag":true,
          "Curriculum":
          [
            {
              "id":1,
              "name":"chinese"					// 前端向后端传递的数据
            },
            {
              "id":2,
              "name":"math"
            }
          ],
          "grades":null
        },
        expectDataType: http.HttpDataType.STRING, // 可选，指定返回数据的类型
        usingCache: true, // 可选，默认为true
        priority: 1, // 可选，默认为1
        connectTimeout: 60000, // 可选，默认为60000ms
        readTimeout: 60000, // 可选，默认为60000ms
        usingProtocol: http.HttpProtocol.HTTP1_1, // 可选，协议类型默认值由系统自动指定
      }, (err, data) => {
      if (!err) {
        let jsondata
        jsondata = data.result
        jsondata = JSON.parse(jsondata)                  // 后端传来的数据，进行格式化
        this.name1=jsondata.animal[0].name
        this.name2=jsondata.animal[1].name
        this.num1=jsondata.animal[0].num
        this.num2=jsondata.animal[1].num
        httpRequest.off('headersReceive');
        httpRequest.destroy();
      } else {
        console.info('error:' + JSON.stringify(err));
        httpRequest.off('headersReceive');
        httpRequest.destroy();
      }
    }
    );
  }
  build() {
    Row() {
      Column({ space: 20 }) {
        Text("动物1")
          .fontSize(50)
          .fontWeight(FontWeight.Bold)
        Text("动物名：" + this.name1)
          .fontSize(30)
          .fontWeight(FontWeight.Bold)
        Text("只数:" + this.num1)
          .fontSize(30)
          .fontWeight(FontWeight.Bold)
        Text("动物2")
          .fontSize(50)
          .fontWeight(FontWeight.Bold)
        Text("动物名：" + this.name2)
          .fontSize(30)
          .fontWeight(FontWeight.Bold)
        Text("只数:" + this.num2)
          .fontSize(30)
          .fontWeight(FontWeight.Bold)
        Button("获取数据")
          .width(120)
          .height(65)
          .backgroundColor(0x317aff)
      }.onClick(()=>{
        this.Get()
      })
      .width('100%')

    }
    .height('100%')
  }
}
```

#### 第五课

##### 案例：登录注册

```python
# 后端代码
class register(APIView):
    def post(self, request):
        username = request.data.get("username")
        password = request.data.get("password")                     # 获取前端传来的账号密码
        if user.objects.filter(username=username).first():          # 在数据库进行比对是否有重复账号
            json_data={
                'message':'用户已存在'
            }
            return JsonResponse(json_data)
        else:
            user.objects.create(username=username,password=password)     # 如果没有创建一个新的用户
            json_data = {
                'message': "注册成功"
            }
            return JsonResponse(json_data)

class login(APIView):
    def get(self,request):
        try:
            info = self.request.query_params
            data = user.objects.filter(username=info['username'],password=info['password'])
            print(data)
            return HttpResponse('ok')
        except Exception as e:
            return HttpResponse('error')


    def post(self,request):
        print(request.data)
        username = request.data.get("username")
        password = request.data.get("password")
        user_1 = user.objects.filter(username=username).first()
        if user_1:
            if user_1.password == password:
                data = {
                    'message':'登录成功'
                }
                return JsonResponse(data)
            else:
                data = {
                    'message':'账户或密码错误'
                }
                return JsonResponse(data)
```

#### 第六课

##### 案例：数据库数据传入前端

```python
# 后代代码
class Send(APIView):
    def post(self, request):
        data = request.data
        aa = userinfo.objects.filter()[1]
        print(data)
        json_data = {
            'animal':{
                'account':aa.account,
                'password':aa.password,
                'name':aa.name,
                'age':aa.age,
                'gender':aa.gender,
                'career':aa.career,
                'date':aa.date
            }
        }
        return JsonResponse(json_data)
```