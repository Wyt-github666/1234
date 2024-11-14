## Flask

### 1,flask快速使用

安装

```python
pip install flask
```

#### 1.1 依赖wsgi werkzeug

```python
from werkzeug import run_simple


def func(environ, start_response):
    print("ok")
    return 'hello world'


if __name__ == '__main__':
    run_simple('127.0.0.1', 5000, func)
```

```python
from werkzeug import run_simple

class Flask(object):

    def __call__(self):
        return 'ok'

    def run(self):
        run_simple('localhost', 5000, self)

app = Flask()

if __name__ == '__main__':
    app.run()
```

#### 1.2 快速使用flask

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def func():
    return 'hello world'


if __name__ == '__main__':
    app.run()
```

总结：

flask是基于werkzeug的wsgi实现的，flask自己没有wsgi

用户请求一旦到来，就会与之对应的 app.__ call __方法

#### 1.3 用户登录&&用户管理

```python
from flask import Flask,render_template,jsonify,request,redirect,url_for

app = Flask(__name__)

DATA_DICT ={
    '1':{'name':'王钰滔','age':18},
    '2':{'name':'王秋童','age':18},
}
@app.route('/',methods=['GET','POST'])
def login():  # put application's code here
    # return '登录'
    if request.method == 'GET':
        return render_template('login.html')
    # return jsonify({'status': 'ok'})
    user = request.form.get('user')
    pwd = request.form.get('pwd')
    if user == 'wangyutao' and pwd == '123456':
        return redirect('/index')
    error = '用户名或者密码错误'
    return render_template('login.html',error=error)


@app.route('/index',methods=['GET','POST'],endpoint='idx')
def index():
    data_list = DATA_DICT
    return render_template('index.html',data_dict=data_list)


@app.route('/edit',methods=['GET','POST'])
def edit():
    nid = request.args.get('nid')
    if request.method == 'GET':
        info = DATA_DICT[nid]
        return render_template('edit.html',info=info)
    user = request.form.get('user')
    age = request.form.get('age')
    DATA_DICT[nid]['age'] = age
    DATA_DICT[nid]['name'] = user
    return redirect(url_for('idx'))

@app.route('/delete/<nid>',methods=['GET','POST'])
def delete(nid):
    del DATA_DICT[nid]
    return redirect(url_for('idx'))

if __name__ == '__main__':
    app.run()
```

##### 总结

###### 1，flask 路由用的装饰器 路由的参数

```python
@app.route('/index',methods=['GET','POST'],endpoint='idx') # 默认情况endpoint==index
def index():
    data_list = DATA_DICT
    return render_template('index.html',data_dict=data_list)
# endpoint不能重名
```

###### 2，动态路由

```python
@app.route('/delete/<int:nid>',methods=['GET','POST'])
def delete(nid):
    del DATA_DICT[nid]
    return redirect(url_for('idx'))
```

###### 3，获取提交的数据

```python
request.args  #get形式传递的参数
request.form  #post形式传递的参数
```

###### 4，返回数据

```python
return render_template("模版文件")
return jsonify()
return redirect('/index') #返回url 或者url的别名 redirct(url_for("别名"))
return '**'
```

###### 5，模版处理

```html
{{*}}
{%  for key,value in data_dict.items() %}
  <tr>
     <td>{{ key }}</td>
     <td>{{ value.name }}</td>
     <td>{{ value.age }}</td>
 {% endfor %}
```

#### 1.4 保存用户的会话信息

```python
from flask import Flask, render_template, jsonify, request, redirect, session, url_for
import functools

app = Flask(__name__)

app.secret_key = 'wangyutao12345678'

DATA_DICT ={
    '1':{'name':'王钰滔','age':18},
    '2':{'name':'王秋童','age':18},
}

def auth(func):
    @functools.wraps(func)
    def inner(*args, **kwargs):
        username = session.get('user')
        if not username:
            return redirect('/')
        return func(*args, **kwargs)
    return inner                           #装饰器 判断用户是否登陆过


@app.route('/',methods=['GET','POST'])
def login():  # put application's code here
    # return '登录'
    if request.method == 'GET':
        return render_template('login.html')
    # return jsonify({'status': 'ok'})
    user = request.form.get('user')
    pwd = request.form.get('pwd')
    if user == 'wangyutao' and pwd == '123456':
        session['user'] = user
        return redirect('/index')
    error = '用户名或者密码错误'
    return render_template('login.html',error=error)


@app.route('/index',methods=['GET','POST'],endpoint='idx')
@auth
def index():
    data_list = DATA_DICT
    return render_template('index.html',data_dict=data_list)


@app.route('/edit',methods=['GET','POST'])
@auth
def edit():
    nid = request.args.get('nid')
    if request.method == 'GET':
        info = DATA_DICT[nid]
        return render_template('edit.html',info=info)
    user = request.form.get('user')
    age = request.form.get('age')
    DATA_DICT[nid]['age'] = age
    DATA_DICT[nid]['name'] = user
    return redirect(url_for('idx'))

@app.route('/delete/<nid>',methods=['GET','POST'])
@auth
def delete(nid):
    del DATA_DICT[nid]
    return redirect(url_for('idx'))


if __name__ == '__main__':
    app.run()
```

### 2，蓝图（blue print）

构建目录结构，