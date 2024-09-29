---
title: FastApi-tortoise-jwt-mysql
date: 2024-09-29 17:28:08
tags: python
categories: [python,fastapi]
---
### 抽了半天时间学了一下fastapi,为了方便，代码没分结构。
```
import sys
import jwt
from pydantic import BaseModel
import uvicorn,asyncio,signal,os
from fastapi import FastAPI, HTTPException, Depends
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from tortoise import fields
from tortoise.models import Model
from tortoise.contrib.pydantic import pydantic_model_creator
from tortoise.contrib.fastapi import register_tortoise
from fastapi import FastAPI, Request
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()
async def startup_event():
    print("Application has started.")

async def shutdown_event():
     print("Application is shutting down.")

app.add_event_handler("startup", startup_event)
app.add_event_handler("shutdown", shutdown_event)

db_config = {
  'connections': {
    'default': {
      'engine': 'tortoise.backends.mysql',
      'credentials': {
        'host': 'localhost',
        'port': '3306',
        'user': 'root',
        'password': 'root',
        'database': 'fiber'
      }
    },
  },
  'apps': {
    'models': {
      'models': ['main'],  # model所在得包位置，
      'default_connection': 'default',  # 更新为数据库连接名称
    }
  }
}

register_tortoise(
  app,
  config=db_config,
  generate_schemas=False
)

# JWT验证中间件
async def jwt_middleware(request: Request, call_next):
    # 检查是否是登录接口
    if request.url.path == "/token":
        response = await call_next(request)
        return response
    http_bearer = HTTPBearer()
    credentials: HTTPAuthorizationCredentials = await http_bearer(request)
    token = credentials.credentials

    # 检查令牌是否存在
    if token is None:
        raise HTTPException(status_code=401, detail="Token missing")
    try:
        payload = jwt.decode(token, JWT_SECRET, algorithms=['HS256'])
        user = await User.get(id=payload.get('id'))
    except Exception:
        raise HTTPException(status_code=401, detail="Token invalid")
    # 将解析后的payload存储在请求的状态中
    request.state.payload = payload
    response = await call_next(request)
    return response

# 添加JWT验证中间件到应用程序
app.middleware("http")(jwt_middleware)
# 全局中间件，用于处理程序中的异常
@app.exception_handler(Exception)
async def handle_exceptions(request: Request, exc: Exception):
    # 在这里可以根据需要进行相应的异常处理逻辑
    # 例如记录日志、返回自定义的错误响应等
    return {
        "code": 500,
        "message": "Internal Server Error",
    }

# 添加CORS中间件
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

class User(Model):
    id = fields.IntField(pk=True,generated=True)
    username = fields.CharField(max_length=50, unique=True)
    password = fields.CharField(max_length=128)
    def __str__(self):
        return self.username
    class Meta:
        table = 'users'
    @classmethod
    def get_user(cls, username):
        return cls.get(username=username)

    # 这里没有真正校验pass
    def verify_password(self, password):
        return password == self.password

UserPydantic = pydantic_model_creator(User, name='User')
UserInPydantic = pydantic_model_creator(User, name='UserIn',exclude_readonly=True)
# 其实下面的jsonbody体更常用
#定义读取类
class UserInPydantic2(BaseModel):
    username: str
    password: str

oauth2_sceheme = OAuth2PasswordBearer(tokenUrl="token")
JWT_SECRET = 'myeasypeasyjwtsecret'
# async def get_current_user(token: str = Depends(oauth2_sceheme)):
#     try:
#         payload = jwt.decode(token, JWT_SECRET, algorithms=['HS256'])
#         user = await User.get(id=payload.get('id'))
#     except:
#         raise HTTPException(
#             status_code=401,
#             detail='Invalid username or password'
#         )
#     return await UserPydantic.from_tortoise_orm(user)
async def authenticate_user(username: str, password: str):
    user = await User.get(username=username)
    if not user:
        return False
    if not user.verify_password(password):
        return False
    return user

# @app.post('/token')
# async def generate_token(form_data: OAuth2PasswordRequestForm = Depends()):
#     user = await authenticate_user(form_data.username, form_data.password)
#     if not user:
#         return {"Error": "Invalid username or password"}
#     user_obj = await UserPydantic.from_tortoise_orm(user)
#     token = jwt.encode(user_obj.dict(), JWT_SECRET)
#     return {
#         "access_token": token,
#         "token_type": "Bearer",
#     }

@app.post('/token')
async def generate_token(form_data: UserInPydantic):
    user = await authenticate_user(form_data.username, form_data.password)
    if not user:
        return {"Error": "Invalid username or password"}
    user_obj = await UserPydantic.from_tortoise_orm(user)
    token = jwt.encode(user_obj.dict(), JWT_SECRET)
    return {
        "access_token": token,
        "token_type": "Bearer",
    }

@app.post('/users')
async def create_user(user: UserInPydantic):
    try:
        user_obj = User(username=user.username, password=user.password) # 使用哈希密码
        await user_obj.save()
        return await UserPydantic.from_tortoise_orm(user_obj)
    except Exception as e:
        print(f"An error occurred while saving the user: {e}")
        raise HTTPException(status_code=500, detail="Internal server error")

# @app.get('/users/me')
# async def get_user(current_user: UserPydantic = Depends(get_current_user)):
#     return current_user
@app.get('/users/me')
async def get_user(request: Request):
    return request.state.payload


if __name__ == "__main__":
    try:
        # 程序主逻辑
        uvicorn.run('main:app', host="127.0.0.1", port=8000, reload=True)
    except KeyboardInterrupt:
        sys.exit(0)
        # 执行必要的清理工作
    else:
        # 程序正常退出
        print("Program exited normally")


```
### 如果要把路由提出来,常见得方法,新建文件夹api，并添加user.py
```
from fastapi import APIRouter
api_user = APIRouter()

@api_user.get('/get')
async def get_user(request: Request):
    return {"route":"get_user"}
```
### app.py引用
```
....省略
from api.user import api_user
app = FastAPI()
app.include_router(api_user, prefix="/user", tags=["用户接口"])
```
