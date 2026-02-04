---
name: backend-python
description: Python backend development with FastAPI, Django, Flask, async programming, SQLAlchemy ORM, and API best practices
tags: [python, fastapi, django, flask, api, sqlalchemy]
author: Antigravity Team
version: 1.0.0
---

# Python Backend Development Skill

Modern Python backend development patterns.

## FastAPI (Recommended)

### Project Structure
```
app/
├── api/
│   ├── v1/
│   │   ├── endpoints/
│   │   └── router.py
│   └── deps.py
├── core/
│   ├── config.py
│   └── security.py
├── models/
├── schemas/
├── services/
├── db/
└── main.py
```

### Basic Setup
```python
from fastapi import FastAPI, Depends, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, EmailStr
from typing import Optional
import uvicorn

app = FastAPI(title="My API", version="1.0.0")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

# Pydantic schemas
class UserCreate(BaseModel):
    email: EmailStr
    name: str
    password: str

class UserResponse(BaseModel):
    id: int
    email: str
    name: str
    
    class Config:
        from_attributes = True

# Endpoints
@app.post("/users", response_model=UserResponse)
async def create_user(user: UserCreate, db: Session = Depends(get_db)):
    db_user = await crud.create_user(db, user)
    return db_user

@app.get("/users/{user_id}", response_model=UserResponse)
async def get_user(user_id: int, db: Session = Depends(get_db)):
    user = await crud.get_user(db, user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

### Dependency Injection
```python
from fastapi import Depends, Security
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Security(security),
    db: Session = Depends(get_db)
):
    token = credentials.credentials
    payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
    user = await crud.get_user(db, payload["sub"])
    if not user:
        raise HTTPException(status_code=401)
    return user

@app.get("/me")
async def get_me(user = Depends(get_current_user)):
    return user
```

## SQLAlchemy + Async

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import declarative_base, sessionmaker, relationship
from sqlalchemy import Column, Integer, String, ForeignKey

DATABASE_URL = "postgresql+asyncpg://user:pass@localhost/db"

engine = create_async_engine(DATABASE_URL, echo=True)
AsyncSessionLocal = sessionmaker(engine, class_=AsyncSession)
Base = declarative_base()

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True)
    email = Column(String, unique=True, index=True)
    name = Column(String)
    posts = relationship("Post", back_populates="author")

async def get_db():
    async with AsyncSessionLocal() as session:
        yield session

# CRUD operations
async def get_users(db: AsyncSession, skip: int = 0, limit: int = 100):
    result = await db.execute(
        select(User).offset(skip).limit(limit)
    )
    return result.scalars().all()
```

## Django REST Framework

```python
# serializers.py
from rest_framework import serializers

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'email', 'name', 'created_at']
        read_only_fields = ['id', 'created_at']

# views.py
from rest_framework import viewsets, permissions

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = [permissions.IsAuthenticated]
    
    def get_queryset(self):
        if self.request.user.is_staff:
            return User.objects.all()
        return User.objects.filter(id=self.request.user.id)

# urls.py
from rest_framework.routers import DefaultRouter

router = DefaultRouter()
router.register('users', UserViewSet)
urlpatterns = router.urls
```

## Best Practices

1. **Type Hints**: Always use type annotations
2. **Async/Await**: Use async for I/O-bound operations
3. **Pydantic**: Validate all input/output with schemas
4. **Testing**: Use pytest with async support
5. **Logging**: Use structlog for structured logs
6. **Background Tasks**: Use Celery or FastAPI BackgroundTasks
