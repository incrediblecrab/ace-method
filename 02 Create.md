# CREATE Phase: Systematic Implementation

Build with precision, ship with confidence through structured development practices.

> Part of the [ACE Method](./ACE-Method.md) framework. For CLI-specific implementation, see [CLI-ACE-Method.md](./CLI-ACE-Method.md).
> Prerequisites: Complete [01 Analyze.md](./01 Analyze.md) phase first.

## Phase Overview

The CREATE phase transforms analysis into working software. This phase emphasizes clean code, robust architecture, and production-ready implementation from day one.

### Key Deliverables
1. Production-ready code
2. Comprehensive test suite
3. Security implementation
4. Performance optimizations
5. Deployment pipeline

## 1. Development Environment Setup

### 1.1 Project Structure

**Project Organization:**

**Source Code (src/)**
- api: API endpoints
- services: Business logic
- models: Data models
- utils: Shared utilities
- config: Configuration

**Tests**
- unit: Unit tests
- integration: Integration tests
- e2e: End-to-end tests
- performance: Performance tests

**Documentation (docs/)**
- api: API documentation
- architecture: Architecture docs
- deployment: Deployment guides

**Supporting Files**
- scripts: Build and deploy scripts
- infrastructure: Infrastructure as Code definitions
- monitoring: Monitoring configurations

### 1.2 Development Standards

```markdown
## Coding Standards

### Naming Conventions
- **Variables**: camelCase (JavaScript), snake_case (Python)
- **Constants**: UPPER_SNAKE_CASE
- **Classes**: PascalCase
- **Files**: kebab-case.js, snake_case.py

### Code Style
- **Line Length**: Keep lines readable and maintainable
- **Indentation**: Follow language-specific conventions
- **Comments**: Document public APIs with appropriate format

### Git Conventions
- **Branch Naming**: feature/ACE-123-description
- **Commit Format**: type(scope): description
  - feat: New feature
  - fix: Bug fix
  - docs: Documentation
  - refactor: Code refactoring
  - test: Test updates
  - perf: Performance improvements
```

### 1.3 Development Tooling

```yaml
# Development Environment Configuration

# Language-specific tools
languages:
  javascript:
    runtime: Node.js 18+
    package_manager: npm/yarn/pnpm
    linter: ESLint
    formatter: Prettier
    type_checker: TypeScript
    
  python:
    runtime: Python 3.9+
    package_manager: pip/poetry
    linter: pylint/flake8
    formatter: black
    type_checker: mypy
    
  go:
    runtime: Go 1.19+
    linter: golangci-lint
    formatter: gofmt

# Common tools
tools:
  version_control: Git
  ide: VSCode/IntelliJ
  api_testing: Postman/Insomnia
  database_client: DBeaver/TablePlus
  container: Docker
  orchestration: Kubernetes
```

## 2. Implementation Patterns

### 2.1 API Development

```python
# RESTful API Pattern Example

from flask import Flask, request, jsonify
from datetime import datetime
import logging

class APIResponse:
    """Standardized API response format"""
    
    @staticmethod
    def success(data=None, message="Success", code=200):
        return jsonify({
            "status": "success",
            "message": message,
            "data": data,
            "timestamp": datetime.utcnow().isoformat()
        }), code
    
    @staticmethod
    def error(message, code=400, errors=None):
        return jsonify({
            "status": "error",
            "message": message,
            "errors": errors,
            "timestamp": datetime.utcnow().isoformat()
        }), code

# Middleware for request validation
def validate_request(schema):
    def decorator(f):
        def decorated_function(*args, **kwargs):
            try:
                # Validate request against schema
                data = schema.validate(request.get_json())
                request.validated_data = data
            except ValidationError as e:
                return APIResponse.error("Validation failed", 400, e.messages)
            return f(*args, **kwargs)
        return decorated_function
    return decorator

# Rate limiting decorator
def rate_limit(max_calls=100, window=3600):
    def decorator(f):
        def decorated_function(*args, **kwargs):
            # Implement rate limiting logic
            client_id = get_client_id(request)
            if exceeds_rate_limit(client_id, max_calls, window):
                return APIResponse.error("Rate limit exceeded", 429)
            return f(*args, **kwargs)
        return decorated_function
    return decorator
```

### 2.2 Database Patterns

```python
# Repository Pattern Example

from abc import ABC, abstractmethod
from typing import List, Optional
import logging

class BaseRepository(ABC):
    """Abstract base repository"""
    
    def __init__(self, db_session):
        self.db = db_session
        self.logger = logging.getLogger(self.__class__.__name__)
    
    @abstractmethod
    def get_by_id(self, id: int):
        pass
    
    @abstractmethod
    def get_all(self, limit: int = 100, offset: int = 0):
        pass
    
    @abstractmethod
    def create(self, entity):
        pass
    
    @abstractmethod
    def update(self, id: int, entity):
        pass
    
    @abstractmethod
    def delete(self, id: int):
        pass

class UserRepository(BaseRepository):
    """User-specific repository implementation"""
    
    def get_by_id(self, id: int) -> Optional[User]:
        try:
            return self.db.query(User).filter(User.id == id).first()
        except Exception as e:
            self.logger.error(f"Error fetching user {id}: {e}")
            raise
    
    def create(self, user: User) -> User:
        try:
            self.db.add(user)
            self.db.commit()
            self.db.refresh(user)
            return user
        except Exception as e:
            self.db.rollback()
            self.logger.error(f"Error creating user: {e}")
            raise

# Database migration pattern
"""
# migrations/001_create_users_table.sql
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at);
"""
```

### 2.3 Service Layer Pattern

```typescript
// Service Layer Example in TypeScript

export interface IUserService {
  getUser(id: string): Promise<User>;
  createUser(data: CreateUserDto): Promise<User>;
  updateUser(id: string, data: UpdateUserDto): Promise<User>;
  deleteUser(id: string): Promise<void>;
}

export class UserService implements IUserService {
  constructor(
    private userRepository: IUserRepository,
    private emailService: IEmailService,
    private cacheService: ICacheService,
    private logger: ILogger
  ) {}

  async getUser(id: string): Promise<User> {
    // Check cache first
    const cached = await this.cacheService.get(`user:${id}`);
    if (cached) {
      this.logger.info(`Cache hit for user ${id}`);
      return cached;
    }

    // Fetch from database
    const user = await this.userRepository.findById(id);
    if (!user) {
      throw new NotFoundException(`User ${id} not found`);
    }

    // Update cache
    await this.cacheService.set(`user:${id}`, user, 3600);
    
    return user;
  }

  async createUser(data: CreateUserDto): Promise<User> {
    // Validate business rules
    await this.validateUserData(data);
    
    // Create user
    const user = await this.userRepository.create({
      ...data,
      passwordHash: await this.hashPassword(data.password)
    });
    
    // Send welcome email
    await this.emailService.sendWelcomeEmail(user.email);
    
    // Emit event for other services
    await this.eventBus.publish('user.created', { userId: user.id });
    
    return user;
  }

  private async validateUserData(data: CreateUserDto): Promise<void> {
    // Check if email already exists
    const existing = await this.userRepository.findByEmail(data.email);
    if (existing) {
      throw new ConflictException('Email already registered');
    }
    
    // Validate password strength
    if (!this.isPasswordStrong(data.password)) {
      throw new BadRequestException('Password does not meet requirements');
    }
  }
}
```

## 3. Security Implementation

### 3.1 Authentication & Authorization

```python
# JWT Authentication Implementation

import jwt
from datetime import datetime, timedelta
from functools import wraps
import bcrypt

class AuthService:
    def __init__(self, secret_key, algorithm='HS256'):
        self.secret_key = secret_key
        self.algorithm = algorithm
    
    def hash_password(self, password: str) -> str:
        """Hash password using bcrypt"""
        salt = bcrypt.gensalt()
        return bcrypt.hashpw(password.encode('utf-8'), salt).decode('utf-8')
    
    def verify_password(self, password: str, hash: str) -> bool:
        """Verify password against hash"""
        return bcrypt.checkpw(password.encode('utf-8'), hash.encode('utf-8'))
    
    def generate_token(self, user_id: int, email: str, roles: List[str]) -> str:
        """Generate JWT token"""
        payload = {
            'user_id': user_id,
            'email': email,
            'roles': roles,
            'exp': datetime.utcnow() + timedelta(hours=24),
            'iat': datetime.utcnow(),
            'iss': 'ace-method-api'
        }
        return jwt.encode(payload, self.secret_key, algorithm=self.algorithm)
    
    def verify_token(self, token: str) -> dict:
        """Verify and decode JWT token"""
        try:
            payload = jwt.decode(token, self.secret_key, algorithms=[self.algorithm])
            return payload
        except jwt.ExpiredSignatureError:
            raise AuthenticationError("Token has expired")
        except jwt.InvalidTokenError:
            raise AuthenticationError("Invalid token")

# Authorization decorator
def require_auth(required_roles=None):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            token = request.headers.get('Authorization', '').replace('Bearer ', '')
            
            if not token:
                return APIResponse.error("No authorization token provided", 401)
            
            try:
                payload = auth_service.verify_token(token)
                request.current_user = payload
                
                # Check roles if required
                if required_roles:
                    user_roles = set(payload.get('roles', []))
                    if not user_roles.intersection(set(required_roles)):
                        return APIResponse.error("Insufficient permissions", 403)
                
                return f(*args, **kwargs)
            except AuthenticationError as e:
                return APIResponse.error(str(e), 401)
        
        return decorated_function
    return decorator
```

### 3.2 Input Validation & Sanitization

```python
# Input Validation Framework

from marshmallow import Schema, fields, validate, ValidationError
import bleach
import re

class SanitizedString(fields.String):
    """Custom field that sanitizes HTML input"""
    
    def _deserialize(self, value, attr, data, **kwargs):
        value = super()._deserialize(value, attr, data, **kwargs)
        # Remove potentially dangerous HTML
        return bleach.clean(value, tags=[], strip=True)

class EmailField(fields.Email):
    """Enhanced email field with additional validation"""
    
    def _deserialize(self, value, attr, data, **kwargs):
        value = super()._deserialize(value, attr, data, **kwargs)
        # Additional validation for common typos
        if value.endswith('.con') or value.endswith('.cm'):
            raise ValidationError("Invalid email domain")
        return value.lower()

class UserSchema(Schema):
    """User input validation schema"""
    
    email = EmailField(required=True)
    password = fields.String(
        required=True,
        validate=[
            validate.Length(min=8, max=128),
            validate.Regexp(
                r'^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]',
                error="Password must contain uppercase, lowercase, number and special character"
            )
        ]
    )
    name = SanitizedString(
        required=True,
        validate=validate.Length(min=2, max=100)
    )
    age = fields.Integer(
        validate=validate.Range(min=13, max=120)
    )
    
    class Meta:
        # Strip whitespace from all string inputs
        strip_whitespace = True

# SQL Injection Prevention
class SafeQueryBuilder:
    """Build safe SQL queries"""
    
    @staticmethod
    def build_search_query(table: str, conditions: dict) -> tuple:
        """Build parameterized query"""
        allowed_tables = ['users', 'products', 'orders']
        if table not in allowed_tables:
            raise ValueError(f"Invalid table name: {table}")
        
        where_clauses = []
        params = []
        
        for column, value in conditions.items():
            # Whitelist column names
            if not re.match(r'^[a-zA-Z_][a-zA-Z0-9_]*$', column):
                raise ValueError(f"Invalid column name: {column}")
            
            where_clauses.append(f"{column} = %s")
            params.append(value)
        
        query = f"SELECT * FROM {table}"
        if where_clauses:
            query += " WHERE " + " AND ".join(where_clauses)
        
        return query, params
```

### 3.3 Security Headers & CORS

```python
# Security Middleware Implementation

from flask import Flask, request, g
import time

def setup_security_headers(app: Flask):
    """Configure security headers for all responses"""
    
    @app.after_request
    def add_security_headers(response):
        # Prevent XSS attacks
        response.headers['X-Content-Type-Options'] = 'nosniff'
        response.headers['X-Frame-Options'] = 'DENY'
        response.headers['X-XSS-Protection'] = '1; mode=block'
        
        # Content Security Policy
        response.headers['Content-Security-Policy'] = (
            "default-src 'self'; "
            "script-src 'self' 'unsafe-inline' https://cdn.trusted.com; "
            "style-src 'self' 'unsafe-inline'; "
            "img-src 'self' data: https:; "
            "font-src 'self' https://fonts.gstatic.com; "
            "connect-src 'self' https://api.trusted.com"
        )
        
        # HTTPS enforcement
        response.headers['Strict-Transport-Security'] = 'max-age=31536000; includeSubDomains'
        
        # Referrer Policy
        response.headers['Referrer-Policy'] = 'strict-origin-when-cross-origin'
        
        # Permissions Policy
        response.headers['Permissions-Policy'] = (
            'geolocation=(), microphone=(), camera=()'
        )
        
        return response

def setup_cors(app: Flask, allowed_origins: List[str]):
    """Configure CORS with security in mind"""
    
    @app.after_request
    def add_cors_headers(response):
        origin = request.headers.get('Origin')
        
        # Only allow whitelisted origins
        if origin in allowed_origins:
            response.headers['Access-Control-Allow-Origin'] = origin
            response.headers['Access-Control-Allow-Methods'] = 'GET, POST, PUT, DELETE, OPTIONS'
            response.headers['Access-Control-Allow-Headers'] = 'Content-Type, Authorization'
            response.headers['Access-Control-Max-Age'] = '86400'
            response.headers['Access-Control-Allow-Credentials'] = 'true'
        
        return response
```

## 4. Performance Optimization

### 4.1 Caching Strategy

```python
# Multi-layer Caching Implementation

import redis
from functools import lru_cache, wraps
import pickle
import hashlib

class CacheService:
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client
        self.local_cache = {}
    
    def cache_key(self, prefix: str, *args, **kwargs) -> str:
        """Generate consistent cache key"""
        key_data = f"{prefix}:{args}:{sorted(kwargs.items())}"
        return hashlib.md5(key_data.encode()).hexdigest()
    
    def get(self, key: str, default=None):
        """Get from cache with local cache fallback"""
        # Check local cache first
        if key in self.local_cache:
            return self.local_cache[key]
        
        # Check Redis
        try:
            value = self.redis.get(key)
            if value:
                deserialized = pickle.loads(value)
                # Update local cache
                self.local_cache[key] = deserialized
                return deserialized
        except Exception as e:
            logger.error(f"Cache get error: {e}")
        
        return default
    
    def set(self, key: str, value, ttl: int = 3600):
        """Set cache value with TTL"""
        try:
            # Update local cache
            self.local_cache[key] = value
            
            # Update Redis
            serialized = pickle.dumps(value)
            self.redis.setex(key, ttl, serialized)
        except Exception as e:
            logger.error(f"Cache set error: {e}")
    
    def invalidate(self, pattern: str):
        """Invalidate cache entries matching pattern"""
        # Clear local cache
        keys_to_remove = [k for k in self.local_cache.keys() if pattern in k]
        for key in keys_to_remove:
            del self.local_cache[key]
        
        # Clear Redis cache
        for key in self.redis.scan_iter(match=f"*{pattern}*"):
            self.redis.delete(key)

# Decorator for method caching
def cached(prefix: str, ttl: int = 3600):
    def decorator(func):
        @wraps(func)
        def wrapper(self, *args, **kwargs):
            cache_key = cache_service.cache_key(prefix, *args, **kwargs)
            
            # Try to get from cache
            cached_result = cache_service.get(cache_key)
            if cached_result is not None:
                return cached_result
            
            # Execute function and cache result
            result = func(self, *args, **kwargs)
            cache_service.set(cache_key, result, ttl)
            
            return result
        return wrapper
    return decorator
```

### 4.2 Database Query Optimization

```python
# Query Optimization Patterns

from sqlalchemy import select, and_, or_, func
from sqlalchemy.orm import joinedload, selectinload

class OptimizedRepository:
    """Repository with query optimization techniques"""
    
    def get_users_with_orders(self, limit: int = 100) -> List[User]:
        """Fetch users with their orders using eager loading"""
        return self.db.query(User)\
            .options(selectinload(User.orders))\
            .limit(limit)\
            .all()
    
    def get_order_statistics(self, user_id: int) -> dict:
        """Get aggregated statistics using database functions"""
        result = self.db.query(
            func.count(Order.id).label('total_orders'),
            func.sum(Order.amount).label('total_amount'),
            func.avg(Order.amount).label('average_amount'),
            func.max(Order.amount).label('max_amount'),
            func.min(Order.amount).label('min_amount')
        ).filter(Order.user_id == user_id).first()
        
        return {
            'total_orders': result.total_orders or 0,
            'total_amount': float(result.total_amount or 0),
            'average_amount': float(result.average_amount or 0),
            'max_amount': float(result.max_amount or 0),
            'min_amount': float(result.min_amount or 0)
        }
    
    def search_products(self, query: str, filters: dict) -> List[Product]:
        """Optimized product search with full-text search"""
        search_query = self.db.query(Product)
        
        # Use full-text search if available
        if query:
            search_query = search_query.filter(
                func.to_tsvector('english', Product.name + ' ' + Product.description)\
                .match(query)
            )
        
        # Apply filters efficiently
        if filters.get('min_price'):
            search_query = search_query.filter(Product.price >= filters['min_price'])
        
        if filters.get('max_price'):
            search_query = search_query.filter(Product.price <= filters['max_price'])
        
        if filters.get('categories'):
            search_query = search_query.filter(Product.category_id.in_(filters['categories']))
        
        # Use index hints for better performance
        return search_query\
            .order_by(Product.created_at.desc())\
            .limit(100)\
            .all()
```

### 4.3 Asynchronous Processing

```python
# Asynchronous Task Processing

import asyncio
from celery import Celery
from concurrent.futures import ThreadPoolExecutor
import aiohttp

# Celery configuration for background tasks
celery_app = Celery('ace_tasks', broker='redis://localhost:6379')

@celery_app.task(bind=True, max_retries=3)
def process_heavy_task(self, data: dict):
    """Process heavy computation in background"""
    try:
        # Simulate heavy processing
        result = perform_complex_calculation(data)
        
        # Store result
        cache_service.set(f"task_result:{self.request.id}", result, ttl=86400)
        
        # Notify completion
        send_notification(data['user_id'], f"Task {self.request.id} completed")
        
        return result
    except Exception as e:
        # Retry with exponential backoff
        raise self.retry(exc=e, countdown=2 ** self.request.retries)

# Async HTTP requests
class AsyncHttpClient:
    """Efficient async HTTP client"""
    
    def __init__(self, max_connections: int = 100):
        self.connector = aiohttp.TCPConnector(limit=max_connections)
        self.session = None
    
    async def __aenter__(self):
        self.session = aiohttp.ClientSession(connector=self.connector)
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await self.session.close()
    
    async def fetch_multiple(self, urls: List[str]) -> List[dict]:
        """Fetch multiple URLs concurrently"""
        tasks = [self._fetch_one(url) for url in urls]
        return await asyncio.gather(*tasks, return_exceptions=True)
    
    async def _fetch_one(self, url: str) -> dict:
        """Fetch single URL with timeout and retry"""
        for attempt in range(3):
            try:
                async with self.session.get(url, timeout=10) as response:
                    return await response.json()
            except asyncio.TimeoutError:
                if attempt == 2:
                    raise
                await asyncio.sleep(2 ** attempt)
```

## 5. Testing Strategy

### 5.1 Unit Testing

```python
# Comprehensive Unit Testing

import pytest
from unittest.mock import Mock, patch
from freezegun import freeze_time

class TestUserService:
    """Unit tests for UserService"""
    
    @pytest.fixture
    def user_service(self):
        """Create service with mocked dependencies"""
        mock_repo = Mock()
        mock_email = Mock()
        mock_cache = Mock()
        mock_logger = Mock()
        
        return UserService(mock_repo, mock_email, mock_cache, mock_logger)
    
    def test_create_user_success(self, user_service):
        """Test successful user creation"""
        # Arrange
        user_data = {
            'email': 'test@example.com',
            'password': 'StrongP@ss1',
            'name': 'Test User'
        }
        
        user_service.user_repository.findByEmail.return_value = None
        user_service.user_repository.create.return_value = User(
            id=1,
            email=user_data['email'],
            name=user_data['name']
        )
        
        # Act
        result = user_service.create_user(user_data)
        
        # Assert
        assert result.id == 1
        assert result.email == user_data['email']
        user_service.email_service.send_welcome_email.assert_called_once_with(user_data['email'])
    
    def test_create_user_duplicate_email(self, user_service):
        """Test user creation with duplicate email"""
        # Arrange
        user_service.user_repository.findByEmail.return_value = User(id=1)
        
        # Act & Assert
        with pytest.raises(ConflictException):
            user_service.create_user({'email': 'existing@example.com'})
    
    @freeze_time("2023-01-01 12:00:00")
    def test_user_cache_expiry(self, user_service):
        """Test cache expiry behavior"""
        # Arrange
        user_id = "123"
        cached_user = User(id=user_id, name="Cached User")
        
        user_service.cache_service.get.return_value = None
        user_service.user_repository.findById.return_value = cached_user
        
        # Act
        result = user_service.get_user(user_id)
        
        # Assert
        user_service.cache_service.set.assert_called_with(
            f"user:{user_id}",
            cached_user,
            3600
        )

# Parameterized testing
class TestValidation:
    """Test input validation"""
    
    @pytest.mark.parametrize("password,expected", [
        ("short", False),  # Too short
        ("alllowercase123", False),  # No uppercase
        ("ALLUPPERCASE123", False),  # No lowercase
        ("NoNumbers@", False),  # No digits
        ("NoSpecial123", False),  # No special chars
        ("ValidP@ss123", True),  # Valid password
    ])
    def test_password_validation(self, password, expected):
        """Test password strength validation"""
        assert is_password_strong(password) == expected
```

### 5.2 Integration Testing

```python
# Integration Testing with Real Dependencies

import pytest
from testcontainers.postgres import PostgresContainer
from testcontainers.redis import RedisContainer

class TestIntegration:
    """Integration tests with real services"""
    
    @pytest.fixture(scope="class")
    def postgres_container(self):
        """Spin up PostgreSQL container"""
        with PostgresContainer("postgres:14") as postgres:
            yield postgres
    
    @pytest.fixture(scope="class")
    def redis_container(self):
        """Spin up Redis container"""
        with RedisContainer("redis:7") as redis:
            yield redis
    
    @pytest.fixture
    def test_db(self, postgres_container):
        """Create test database"""
        engine = create_engine(postgres_container.get_connection_url())
        Base.metadata.create_all(engine)
        
        session = sessionmaker(bind=engine)()
        yield session
        
        session.close()
        Base.metadata.drop_all(engine)
    
    def test_user_repository_integration(self, test_db):
        """Test repository with real database"""
        # Arrange
        repo = UserRepository(test_db)
        user = User(email="test@example.com", name="Test User")
        
        # Act
        created_user = repo.create(user)
        fetched_user = repo.get_by_id(created_user.id)
        
        # Assert
        assert fetched_user is not None
        assert fetched_user.email == "test@example.com"
        assert fetched_user.id == created_user.id
    
    def test_cache_service_integration(self, redis_container):
        """Test cache service with real Redis"""
        # Arrange
        redis_client = redis.Redis(
            host=redis_container.get_container_host_ip(),
            port=redis_container.get_exposed_port(6379)
        )
        cache = CacheService(redis_client)
        
        # Act
        cache.set("test_key", {"data": "test_value"}, ttl=60)
        result = cache.get("test_key")
        
        # Assert
        assert result == {"data": "test_value"}
```

### 5.3 End-to-End Testing

```python
# E2E Testing with Selenium

from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

class TestE2E:
    """End-to-end UI tests"""
    
    @pytest.fixture
    def driver(self):
        """Setup Chrome driver"""
        options = webdriver.ChromeOptions()
        options.add_argument('--headless')
        options.add_argument('--no-sandbox')
        
        driver = webdriver.Chrome(options=options)
        yield driver
        driver.quit()
    
    def test_user_registration_flow(self, driver, test_server_url):
        """Test complete user registration flow"""
        # Navigate to registration page
        driver.get(f"{test_server_url}/register")
        
        # Fill registration form
        driver.find_element(By.ID, "email").send_keys("newuser@example.com")
        driver.find_element(By.ID, "password").send_keys("StrongP@ss123")
        driver.find_element(By.ID, "confirm_password").send_keys("StrongP@ss123")
        driver.find_element(By.ID, "name").send_keys("New User")
        
        # Submit form
        driver.find_element(By.ID, "submit_button").click()
        
        # Wait for success message
        success_element = WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.CLASS_NAME, "success-message"))
        )
        
        assert "Registration successful" in success_element.text
        
        # Verify redirect to dashboard
        assert "/dashboard" in driver.current_url
```

## 6. CI/CD Pipeline

### 6.1 GitHub Actions Configuration

```yaml
# .github/workflows/ci-cd.yml

name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  PYTHON_VERSION: '3.9'
  NODE_VERSION: '18'

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}
      
      - name: Install dependencies
        run: |
          pip install pylint black mypy
          pip install -r requirements.txt
      
      - name: Run linting
        run: |
          black --check .
          pylint src/
          mypy src/

  test:
    runs-on: ubuntu-latest
    needs: lint
    
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: testpass
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      
      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install -r requirements-test.txt
      
      - name: Run tests with coverage
        env:
          DATABASE_URL: postgresql://postgres:testpass@localhost/testdb
          REDIS_URL: redis://localhost
        run: |
          pytest tests/ -v --cov=src --cov-report=xml
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml

  security-scan:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v3
      
      - name: Run Snyk vulnerability scan
        uses: snyk/actions/python@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      
      - name: Run SAST with Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/python

  build:
    runs-on: ubuntu-latest
    needs: [test, security-scan]
    steps:
      - uses: actions/checkout@v3
      
      - name: Build Docker image
        run: |
          docker build -t ace-app:${{ github.sha }} .
          docker tag ace-app:${{ github.sha }} ace-app:latest
      
      - name: Push to registry
        if: github.ref == 'refs/heads/main'
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker push ace-app:${{ github.sha }}
          docker push ace-app:latest

  deploy:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    
    steps:
      - name: Deploy to staging
        uses: appleboy/ssh-action@v0.1.5
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.STAGING_USER }}
          key: ${{ secrets.STAGING_SSH_KEY }}
          script: |
            docker pull ace-app:${{ github.sha }}
            docker-compose -f docker-compose.staging.yml up -d
            
      - name: Run smoke tests
        run: |
          npm install newman
          npx newman run tests/postman/smoke-tests.json \
            --env-var "base_url=${{ secrets.STAGING_URL }}"
```

### 6.2 Docker Configuration

```dockerfile
# Multi-stage Dockerfile

# Build stage
FROM python:3.9-slim AS builder

WORKDIR /app

# Install build dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    make \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements first for better caching
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Copy application code
COPY src/ ./src/

# Production stage
FROM python:3.9-slim

WORKDIR /app

# Install runtime dependencies
RUN apt-get update && apt-get install -y \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Copy Python packages from builder
COPY --from=builder /root/.local /root/.local

# Copy application code
COPY --from=builder /app/src ./src

# Make sure scripts in .local are usable
ENV PATH=/root/.local/bin:$PATH

# Add non-root user
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

# Run application
CMD ["gunicorn", "-w", "4", "-b", "0.0.0.0:8000", "src.main:app"]
```

### 6.3 Kubernetes Deployment

```yaml
# kubernetes/deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: ace-app
  labels:
    app: ace-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ace-app
  template:
    metadata:
      labels:
        app: ace-app
    spec:
      containers:
      - name: ace-app
        image: ace-app:latest
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: ace-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: ace-secrets
              key: redis-url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: ace-app-service
spec:
  selector:
    app: ace-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
  type: LoadBalancer

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ace-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ace-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

## 7. Code Quality Standards

### 7.1 Code Review Checklist

```markdown
## Code Review Checklist

### Architecture & Design
- [ ] Follows established patterns
- [ ] No unnecessary complexity
- [ ] Proper separation of concerns
- [ ] DRY principle applied appropriately

### Code Quality
- [ ] Clear, descriptive naming
- [ ] Functions are focused and small
- [ ] No commented-out code
- [ ] No TODO comments without tickets

### Testing
- [ ] Adequate test coverage (>80%)
- [ ] Tests are meaningful, not just coverage
- [ ] Edge cases covered
- [ ] Mocks used appropriately

### Security
- [ ] Input validation present
- [ ] No hardcoded secrets
- [ ] SQL injection prevention
- [ ] XSS prevention measures

### Performance
- [ ] No N+1 queries
- [ ] Appropriate caching used
- [ ] Async where beneficial
- [ ] Resource cleanup handled

### Documentation
- [ ] Public APIs documented
- [ ] Complex logic explained
- [ ] README updated if needed
- [ ] Changelog entry added
```

### 7.2 Documentation Standards

```python
"""
Module: user_service.py
Purpose: Handle user-related business logic
Author: ACE Team

"""

class UserService:
    """
    Service layer for user management.
    
    This service handles all user-related operations including
    authentication, profile management, and user preferences.
    
    Attributes:
        user_repository: Repository for user data access
        email_service: Service for sending emails
        cache_service: Service for caching user data
        
    Example:
        service = UserService(repo, email, cache)
        user = service.create_user({"email": "test@example.com"})
    """
    
    def create_user(self, user_data: dict) -> User:
        """
        Create a new user account.
        
        Args:
            user_data: Dictionary containing user information
                - email (str): User's email address
                - password (str): Plain text password
                - name (str): User's full name
                
        Returns:
            User: The created user object
            
        Raises:
            ValidationError: If user data is invalid
            ConflictException: If email already exists
            
        Example:
            >>> user = service.create_user({
            ...     "email": "john@example.com",
            ...     "password": "SecureP@ss123",
            ...     "name": "John Doe"
            ... })
            >>> print(user.id)
            123
        """
        # Implementation...
```

## 8. Monitoring & Observability

### 8.1 Structured Logging

```python
# Structured Logging Implementation

import json
import logging
from datetime import datetime
from contextlib import contextmanager
import uuid

class StructuredLogger:
    """Enhanced logger with structured output"""
    
    def __init__(self, name: str):
        self.logger = logging.getLogger(name)
        self.context = {}
    
    @contextmanager
    def request_context(self, request_id: str = None):
        """Add request context to all logs"""
        if not request_id:
            request_id = str(uuid.uuid4())
        
        old_context = self.context.copy()
        self.context.update({
            'request_id': request_id,
            'timestamp': datetime.utcnow().isoformat()
        })
        
        try:
            yield
        finally:
            self.context = old_context
    
    def log(self, level: str, message: str, **kwargs):
        """Log with structured data"""
        log_data = {
            'level': level,
            'message': message,
            'timestamp': datetime.utcnow().isoformat(),
            **self.context,
            **kwargs
        }
        
        getattr(self.logger, level.lower())(json.dumps(log_data))
    
    def info(self, message: str, **kwargs):
        self.log('INFO', message, **kwargs)
    
    def error(self, message: str, error: Exception = None, **kwargs):
        if error:
            kwargs['error_type'] = type(error).__name__
            kwargs['error_message'] = str(error)
            kwargs['error_trace'] = traceback.format_exc()
        
        self.log('ERROR', message, **kwargs)

# Usage example
logger = StructuredLogger(__name__)

@app.route('/api/users/<user_id>')
def get_user(user_id: str):
    with logger.request_context() as ctx:
        logger.info('Fetching user', user_id=user_id)
        
        try:
            user = user_service.get_user(user_id)
            logger.info('User fetched successfully', 
                       user_id=user_id,
                       cache_hit=user.from_cache)
            return jsonify(user)
        except Exception as e:
            logger.error('Failed to fetch user', 
                        error=e,
                        user_id=user_id)
            raise
```

### 8.2 Metrics Collection

```python
# Prometheus Metrics Implementation

from prometheus_client import Counter, Histogram, Gauge, generate_latest
import time
from functools import wraps

# Define metrics
request_count = Counter('http_requests_total', 
                       'Total HTTP requests', 
                       ['method', 'endpoint', 'status'])

request_duration = Histogram('http_request_duration_seconds',
                           'HTTP request latency',
                           ['method', 'endpoint'])

active_users = Gauge('active_users_total',
                    'Number of active users')

db_connections = Gauge('database_connections_active',
                      'Active database connections')

# Metrics middleware
def track_metrics(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        
        try:
            result = func(*args, **kwargs)
            status = result[1] if isinstance(result, tuple) else 200
            return result
        except Exception as e:
            status = 500
            raise
        finally:
            duration = time.time() - start_time
            
            # Update metrics
            request_count.labels(
                method=request.method,
                endpoint=request.endpoint,
                status=status
            ).inc()
            
            request_duration.labels(
                method=request.method,
                endpoint=request.endpoint
            ).observe(duration)
    
    return wrapper

# Metrics endpoint
@app.route('/metrics')
def metrics():
    return generate_latest()

# Application metrics
class MetricsCollector:
    """Collect application-specific metrics"""
    
    @staticmethod
    def record_user_action(action: str, user_id: str):
        """Record user actions for analytics"""
        user_action_count.labels(action=action).inc()
        
        # Update active users
        cache_service.sadd('active_users', user_id)
        active_users.set(cache_service.scard('active_users'))
    
    @staticmethod
    def record_business_metric(metric_name: str, value: float):
        """Record custom business metrics"""
        business_metrics.labels(name=metric_name).set(value)
```

## 9. CREATE Phase Checklist

Before proceeding to EVALUATE phase, ensure:

### Code Quality
- [ ] All code follows established patterns
- [ ] No hardcoded values or magic numbers
- [ ] Error handling comprehensive
- [ ] Logging implemented throughout

### Security
- [ ] Authentication implemented
- [ ] Authorization checks in place
- [ ] Input validation complete
- [ ] Security headers configured

### Testing
- [ ] Unit test coverage > 80%
- [ ] Integration tests passing
- [ ] E2E tests for critical paths
- [ ] Performance tests completed

### Documentation
- [ ] API documentation complete
- [ ] Code comments where needed
- [ ] README updated
- [ ] Deployment guide created

### Infrastructure
- [ ] CI/CD pipeline configured
- [ ] Docker containers built
- [ ] Kubernetes manifests ready
- [ ] Monitoring configured

## 10. Transition to EVALUATE Phase

### Handoff Package
1. Deployed application URL
2. API documentation link
3. Test coverage report
4. Performance baseline metrics
5. Security scan results

### Key Metrics to Track
- Response time percentiles
- Error rates by endpoint
- Resource utilization
- User engagement metrics
- Business KPIs

### Next Steps
Proceed to [03 Evaluate.md](./03 Evaluate.md) for validation and optimization guidance.

## Quick Reference Card - CREATE Phase

### Key Activities Checklist
- [ ] **Code Architecture**
  - [ ] Repository structure set up
  - [ ] Core modules implemented
  - [ ] Design patterns applied
- [ ] **API Development**
  - [ ] Endpoints designed (RESTful/GraphQL)
  - [ ] Request/response validation
  - [ ] Error handling implemented
- [ ] **Data Layer**
  - [ ] Database schema created
  - [ ] Migrations written
  - [ ] Query optimization done
- [ ] **Security Implementation**
  - [ ] Authentication/authorization
  - [ ] Input validation
  - [ ] Security headers configured
- [ ] **Testing Foundation**
  - [ ] Unit tests written
  - [ ] Integration tests created
  - [ ] Test coverage measured

### Key Deliverables
1. **Working Codebase** - Functional implementation
2. **API Documentation** - Complete endpoint reference
3. **Database Schema** - Optimized data models
4. **Test Suite** - Comprehensive test coverage
5. **Deployment Configuration** - Infrastructure as code

### Best Practices Applied
- ✅ SOLID principles followed
- ✅ DRY (Don't Repeat Yourself)
- ✅ KISS (Keep It Simple, Stupid)
- ✅ YAGNI (You Aren't Gonna Need It)
- ✅ Clean code conventions

### Common Pitfalls to Avoid
- ❌ Over-engineering solutions
- ❌ Ignoring error handling
- ❌ Hardcoding configurations
- ❌ Skipping input validation
- ❌ Writing code without tests

### Performance Considerations
- Database query optimization
- Caching strategy implementation
- Async operations where needed
- Resource pooling configured
- Response time monitoring

## Additional Resources
- [Resources & References](./ACE-Method.md#resources--references)
- [Implementation Guide for LLMs](./ACE-Method.md#implementation-guide-for-llms)

---

**Navigation**:
- Previous: [01 Analyze.md](./01 Analyze.md)
- Current: **02 Create.md**
- Next: [03 Evaluate.md](./03 Evaluate.md)

**Remember**: Quality code is not just about functionality. It's about maintainability, security, performance, and the ability to evolve with changing requirements.