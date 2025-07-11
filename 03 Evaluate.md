# EVALUATE Phase: Continuous Validation

Measure twice, cut once, then measure again through comprehensive validation and optimization.

> Part of the [ACE Method](./ACE-Method.md) framework. For CLI-specific evaluation, see [CLI-ACE-Method.md](./CLI-ACE-Method.md).
> Prerequisites: Complete [02 Create.md](./02 Create.md) phase first.

## Phase Overview

The EVALUATE phase ensures your solution meets all requirements, performs at scale, and maintains security standards. This phase transforms a working solution into a production-ready system.

### Key Deliverables
1. Comprehensive test results
2. Performance benchmarks
3. Security audit report
4. Monitoring dashboard
5. Production deployment

## 1. Functional Validation

### 1.1 Acceptance Testing Framework

```markdown
## Acceptance Criteria Validation

### User Story: [Feature Name]
**As a** [user type]
**I want** [goal/desire]
**So that** [benefit]

### Acceptance Tests
**Acceptance Tests**

**Happy Path**
- Steps: Navigate to page, Enter valid data, Submit form
- Expected Result: Success message displayed
- Status: Pass

**Edge Case**
- Steps: Navigate to page, Enter boundary values, Submit form
- Expected Result: Appropriate handling
- Status: Pass

**Error Case**
- Steps: Navigate to page, Enter invalid data, Submit form
- Expected Result: Validation error shown
- Status: Pass

### Test Evidence
- Screenshots: /tests/evidence/feature-x/
- Test logs: /tests/logs/acceptance/
- Video recordings: /tests/recordings/
```

### 1.2 User Journey Testing

```python
# User Journey Test Implementation

import pytest
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import allure

@allure.feature('User Registration')
@allure.story('Complete Registration Flow')
class TestUserJourney:
    """Test complete user journeys through the application"""
    
    @pytest.fixture
    def authenticated_driver(self, driver):
        """Return driver with authenticated user"""
        driver.get(f"{BASE_URL}/login")
        driver.find_element(By.ID, "email").send_keys("test@example.com")
        driver.find_element(By.ID, "password").send_keys("TestPass123!")
        driver.find_element(By.ID, "submit").click()
        
        # Wait for dashboard
        WebDriverWait(driver, 10).until(
            EC.presence_of_element_located((By.CLASS_NAME, "dashboard"))
        )
        return driver
    
    @allure.title("New User Complete Journey")
    def test_new_user_journey(self, driver):
        """Test journey from registration to first action"""
        with allure.step("Navigate to registration"):
            driver.get(f"{BASE_URL}/register")
            assert "Register" in driver.title
        
        with allure.step("Complete registration form"):
            driver.find_element(By.ID, "email").send_keys("newuser@test.com")
            driver.find_element(By.ID, "password").send_keys("SecurePass123!")
            driver.find_element(By.ID, "name").send_keys("Test User")
            driver.find_element(By.ID, "submit").click()
        
        with allure.step("Verify email"):
            # Simulate email verification
            verification_link = get_verification_link("newuser@test.com")
            driver.get(verification_link)
        
        with allure.step("Complete onboarding"):
            # Fill profile
            driver.find_element(By.ID, "bio").send_keys("Test bio")
            driver.find_element(By.ID, "save_profile").click()
        
        with allure.step("Perform first action"):
            driver.find_element(By.ID, "create_project").click()
            driver.find_element(By.ID, "project_name").send_keys("My First Project")
            driver.find_element(By.ID, "create").click()
            
            # Verify success
            success_msg = WebDriverWait(driver, 10).until(
                EC.presence_of_element_located((By.CLASS_NAME, "success"))
            )
            assert "Project created" in success_msg.text
    
    @allure.title("Power User Workflow")
    def test_power_user_workflow(self, authenticated_driver):
        """Test advanced user workflows"""
        driver = authenticated_driver
        
        with allure.step("Bulk operations"):
            driver.get(f"{BASE_URL}/projects")
            
            # Select multiple items
            checkboxes = driver.find_elements(By.CLASS_NAME, "select-item")
            for checkbox in checkboxes[:5]:
                checkbox.click()
            
            # Perform bulk action
            driver.find_element(By.ID, "bulk_action").click()
            driver.find_element(By.ID, "bulk_archive").click()
            
            # Verify
            alert = WebDriverWait(driver, 10).until(
                EC.presence_of_element_located((By.CLASS_NAME, "alert"))
            )
            assert "5 items archived" in alert.text
```

### 1.3 Regression Testing

```python
# Regression Test Suite

import pytest
from tests.fixtures import test_data
import hashlib

class TestRegression:
    """Ensure existing functionality remains intact"""
    
    @pytest.fixture(autouse=True)
    def capture_state(self, db_session):
        """Capture system state before and after tests"""
        # Before test
        initial_state = self._capture_db_state(db_session)
        
        yield
        
        # After test
        final_state = self._capture_db_state(db_session)
        
        # Verify no unexpected changes
        self._verify_state_integrity(initial_state, final_state)
    
    def _capture_db_state(self, session):
        """Capture current database state"""
        state = {}
        for table in ['users', 'projects', 'settings']:
            query = f"SELECT COUNT(*) as count, MAX(updated_at) as latest FROM {table}"
            result = session.execute(query).first()
            state[table] = {
                'count': result.count,
                'latest': result.latest,
                'checksum': self._calculate_table_checksum(session, table)
            }
        return state
    
    def _calculate_table_checksum(self, session, table):
        """Calculate checksum for table data"""
        query = f"SELECT * FROM {table} ORDER BY id"
        results = session.execute(query).fetchall()
        
        # Create consistent hash
        data_str = ''.join([str(row) for row in results])
        return hashlib.md5(data_str.encode()).hexdigest()
    
    @pytest.mark.regression
    def test_user_authentication_unchanged(self):
        """Verify authentication logic hasn't regressed"""
        # Test with previous version's test data
        for test_case in test_data.authentication_cases:
            response = auth_service.authenticate(
                test_case['username'],
                test_case['password']
            )
            assert response.status_code == test_case['expected_status']
            assert response.json() == test_case['expected_response']
    
    @pytest.mark.regression
    def test_api_backwards_compatibility(self):
        """Ensure API maintains backwards compatibility"""
        # Test v1 endpoints still work
        v1_endpoints = [
            '/api/v1/users',
            '/api/v1/projects',
            '/api/v1/auth/login'
        ]
        
        for endpoint in v1_endpoints:
            response = client.get(endpoint)
            assert response.status_code != 404, f"v1 endpoint {endpoint} removed"
            
            # Verify response structure
            assert 'data' in response.json()
            assert 'status' in response.json()
```

## 2. Performance Testing

### 2.1 Load Testing Strategy

```python
# Load Testing with Locust

from locust import HttpUser, task, between
import random
import json

class APIUser(HttpUser):
    """Simulate typical API usage patterns"""
    wait_time = between(1, 3)
    
    def on_start(self):
        """Login and store token"""
        response = self.client.post("/api/auth/login", json={
            "email": f"user{random.randint(1, 1000)}@test.com",
            "password": "TestPass123!"
        })
        self.token = response.json()['token']
        self.headers = {'Authorization': f'Bearer {self.token}'}
    
    @task(3)
    def view_dashboard(self):
        """Most common operation"""
        self.client.get("/api/dashboard", headers=self.headers)
    
    @task(2)
    def list_projects(self):
        """Frequent operation"""
        self.client.get("/api/projects", headers=self.headers)
    
    @task(1)
    def create_project(self):
        """Less frequent write operation"""
        self.client.post("/api/projects", 
            headers=self.headers,
            json={
                "name": f"Project {random.randint(1, 10000)}",
                "description": "Load test project"
            }
        )

class AdminUser(HttpUser):
    """Simulate admin operations"""
    wait_time = between(5, 10)
    
    def on_start(self):
        """Admin login"""
        response = self.client.post("/api/auth/login", json={
            "email": "admin@test.com",
            "password": "AdminPass123!"
        })
        self.token = response.json()['token']
        self.headers = {'Authorization': f'Bearer {self.token}'}
    
    @task
    def view_analytics(self):
        """Heavy analytics queries"""
        self.client.get("/api/admin/analytics", headers=self.headers)
    
    @task
    def export_data(self):
        """Resource-intensive export"""
        self.client.get("/api/admin/export?format=csv", headers=self.headers)

# Run with: locust -f load_test.py --host=https://api.example.com
```

### 2.2 Performance Benchmarking

```python
# Performance Benchmark Suite

import time
import statistics
import concurrent.futures
from datetime import datetime
import psutil
import matplotlib.pyplot as plt

class PerformanceBenchmark:
    """Comprehensive performance testing"""
    
    def __init__(self, base_url):
        self.base_url = base_url
        self.results = []
    
    def benchmark_endpoint(self, endpoint, method='GET', data=None, iterations=100):
        """Benchmark a single endpoint"""
        response_times = []
        errors = 0
        
        for i in range(iterations):
            start_time = time.time()
            try:
                response = requests.request(method, f"{self.base_url}{endpoint}", json=data)
                response_time = time.time() - start_time
                
                if response.status_code >= 400:
                    errors += 1
                else:
                    response_times.append(response_time * 1000)  # Convert to ms
            except Exception as e:
                errors += 1
                print(f"Error in iteration {i}: {e}")
        
        # Calculate statistics
        if response_times:
            stats = {
                'endpoint': endpoint,
                'method': method,
                'iterations': iterations,
                'errors': errors,
                'min': min(response_times),
                'max': max(response_times),
                'mean': statistics.mean(response_times),
                'median': statistics.median(response_times),
                'p95': statistics.quantiles(response_times, n=20)[18],  # 95th percentile
                'p99': statistics.quantiles(response_times, n=100)[98],  # 99th percentile
                'stdev': statistics.stdev(response_times) if len(response_times) > 1 else 0
            }
            self.results.append(stats)
            return stats
        else:
            return None
    
    def concurrent_load_test(self, endpoint, concurrent_users=50, duration=60):
        """Test with concurrent users"""
        start_time = time.time()
        results = []
        
        with concurrent.futures.ThreadPoolExecutor(max_workers=concurrent_users) as executor:
            futures = []
            
            while time.time() - start_time < duration:
                future = executor.submit(self._make_request, endpoint)
                futures.append(future)
                time.sleep(0.1)  # Stagger requests
            
            # Collect results
            for future in concurrent.futures.as_completed(futures):
                result = future.result()
                if result:
                    results.append(result)
        
        # Analyze concurrent performance
        response_times = [r['response_time'] for r in results if 'response_time' in r]
        error_count = sum(1 for r in results if r.get('error', False))
        
        return {
            'total_requests': len(results),
            'successful_requests': len(results) - error_count,
            'error_rate': error_count / len(results) if results else 0,
            'requests_per_second': len(results) / duration,
            'avg_response_time': statistics.mean(response_times) if response_times else 0,
            'p95_response_time': statistics.quantiles(response_times, n=20)[18] if len(response_times) > 20 else 0
        }
    
    def stress_test(self, endpoint, initial_load=10, increment=10, max_load=1000):
        """Find breaking point"""
        current_load = initial_load
        results = []
        
        while current_load <= max_load:
            print(f"Testing with {current_load} concurrent users...")
            
            # Monitor system resources
            cpu_before = psutil.cpu_percent(interval=1)
            memory_before = psutil.virtual_memory().percent
            
            # Run load test
            result = self.concurrent_load_test(endpoint, current_load, duration=30)
            
            # Monitor after
            cpu_after = psutil.cpu_percent(interval=1)
            memory_after = psutil.virtual_memory().percent
            
            result.update({
                'concurrent_users': current_load,
                'cpu_usage': max(cpu_before, cpu_after),
                'memory_usage': max(memory_before, memory_after)
            })
            
            results.append(result)
            
            # Check if system is struggling
            if result['error_rate'] > 0.05 or result['p95_response_time'] > 5000:
                print(f"System degradation detected at {current_load} users")
                break
            
            current_load += increment
        
        return results
    
    def generate_report(self, output_file='performance_report.html'):
        """Generate HTML performance report"""
        html_template = """
        <html>
        <head>
            <title>Performance Test Report</title>
            <style>
                body { font-family: Arial, sans-serif; margin: 20px; }
                table { border-collapse: collapse; width: 100%; margin: 20px 0; }
                th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
                th { background-color: #4CAF50; color: white; }
                .warning { color: orange; }
                .error { color: red; }
                .good { color: green; }
            </style>
        </head>
        <body>
            <h1>Performance Test Report</h1>
            <h2>Generated: {timestamp}</h2>
            
            <h3>Endpoint Performance</h3>
            <table>
                <tr>
                    <th>Endpoint</th>
                    <th>Method</th>
                    <th>Mean (ms)</th>
                    <th>P95 (ms)</th>
                    <th>P99 (ms)</th>
                    <th>Error Rate</th>
                    <th>Status</th>
                </tr>
                {endpoint_rows}
            </table>
            
            <h3>Performance Criteria</h3>
            <ul>
                <li class="good">Good: P95 < 500ms, Error Rate < 0.1%</li>
                <li class="warning">Warning: P95 < 1000ms, Error Rate < 1%</li>
                <li class="error">Critical: P95 > 1000ms or Error Rate > 1%</li>
            </ul>
        </body>
        </html>
        """
        
        endpoint_rows = ""
        for result in self.results:
            status_class = self._get_status_class(result)
            endpoint_rows += f"""
                <tr>
                    <td>{result['endpoint']}</td>
                    <td>{result['method']}</td>
                    <td>{result['mean']:.2f}</td>
                    <td>{result['p95']:.2f}</td>
                    <td>{result['p99']:.2f}</td>
                    <td>{result['errors'] / result['iterations'] * 100:.2f}%</td>
                    <td class="{status_class}">{status_class.upper()}</td>
                </tr>
            """
        
        html = html_template.format(
            timestamp=datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            endpoint_rows=endpoint_rows
        )
        
        with open(output_file, 'w') as f:
            f.write(html)
    
    def _get_status_class(self, result):
        """Determine performance status"""
        error_rate = result['errors'] / result['iterations']
        
        if result['p95'] > 1000 or error_rate > 0.01:
            return 'error'
        elif result['p95'] > 500 or error_rate > 0.001:
            return 'warning'
        else:
            return 'good'
```

### 2.3 Database Performance Optimization

```python
# Database Performance Analysis

import time
from sqlalchemy import event, create_engine
from sqlalchemy.engine import Engine
import logging

class DatabasePerformanceAnalyzer:
    """Analyze and optimize database performance"""
    
    def __init__(self, connection_string):
        self.engine = create_engine(connection_string)
        self.slow_queries = []
        self.query_stats = {}
        
        # Set up query logging
        self._setup_query_logging()
    
    def _setup_query_logging(self):
        """Log all SQL queries with timing"""
        @event.listens_for(Engine, "before_cursor_execute")
        def before_cursor_execute(conn, cursor, statement, parameters, context, executemany):
            context._query_start_time = time.time()
            
        @event.listens_for(Engine, "after_cursor_execute")
        def after_cursor_execute(conn, cursor, statement, parameters, context, executemany):
            total_time = time.time() - context._query_start_time
            
            # Log slow queries
            if total_time > 0.1:  # 100ms threshold
                self.slow_queries.append({
                    'query': statement,
                    'parameters': parameters,
                    'duration': total_time,
                    'timestamp': time.time()
                })
            
            # Track query statistics
            query_type = statement.split()[0].upper()
            if query_type not in self.query_stats:
                self.query_stats[query_type] = {
                    'count': 0,
                    'total_time': 0,
                    'max_time': 0
                }
            
            stats = self.query_stats[query_type]
            stats['count'] += 1
            stats['total_time'] += total_time
            stats['max_time'] = max(stats['max_time'], total_time)
    
    def analyze_slow_queries(self):
        """Analyze patterns in slow queries"""
        analysis = {
            'total_slow_queries': len(self.slow_queries),
            'slowest_queries': sorted(self.slow_queries, key=lambda x: x['duration'], reverse=True)[:10],
            'common_patterns': self._find_query_patterns(),
            'recommendations': []
        }
        
        # Generate recommendations
        for pattern, count in analysis['common_patterns'].items():
            if 'SELECT *' in pattern:
                analysis['recommendations'].append(
                    f"Avoid SELECT * in queries. Found {count} instances."
                )
            if 'JOIN' in pattern and count > 10:
                analysis['recommendations'].append(
                    f"Review JOIN operations. {count} slow JOIN queries detected."
                )
            if 'NOT IN' in pattern or 'NOT EXISTS' in pattern:
                analysis['recommendations'].append(
                    "Consider replacing NOT IN/NOT EXISTS with LEFT JOIN for better performance."
                )
        
        return analysis
    
    def _find_query_patterns(self):
        """Identify common patterns in slow queries"""
        patterns = {}
        
        for query_info in self.slow_queries:
            query = query_info['query']
            
            # Extract patterns
            if 'SELECT *' in query:
                patterns['SELECT *'] = patterns.get('SELECT *', 0) + 1
            if 'JOIN' in query:
                patterns['JOIN'] = patterns.get('JOIN', 0) + 1
            if 'NOT IN' in query:
                patterns['NOT IN'] = patterns.get('NOT IN', 0) + 1
            if 'LIKE \'%' in query:
                patterns['Leading wildcard'] = patterns.get('Leading wildcard', 0) + 1
        
        return patterns
    
    def check_missing_indexes(self):
        """Identify potentially missing indexes"""
        # PostgreSQL specific query
        missing_indexes_query = """
        SELECT 
            schemaname,
            tablename,
            attname,
            n_distinct,
            most_common_vals
        FROM pg_stats
        WHERE n_distinct > 100
        AND schemaname NOT IN ('pg_catalog', 'information_schema')
        ORDER BY n_distinct DESC;
        """
        
        with self.engine.connect() as conn:
            result = conn.execute(missing_indexes_query)
            potential_indexes = []
            
            for row in result:
                # Check if this column is used in WHERE clauses
                column_usage = self._check_column_usage(row.tablename, row.attname)
                
                if column_usage > 10:  # Used frequently
                    potential_indexes.append({
                        'table': row.tablename,
                        'column': row.attname,
                        'cardinality': row.n_distinct,
                        'usage_count': column_usage,
                        'recommendation': f"CREATE INDEX idx_{row.tablename}_{row.attname} ON {row.tablename}({row.attname});"
                    })
            
            return potential_indexes
    
    def generate_optimization_report(self):
        """Generate comprehensive optimization report"""
        report = {
            'summary': self.query_stats,
            'slow_queries': self.analyze_slow_queries(),
            'missing_indexes': self.check_missing_indexes(),
            'connection_pool_stats': self._get_connection_pool_stats(),
            'recommendations': self._generate_recommendations()
        }
        
        return report
```

## 3. Security Testing

### 3.1 Vulnerability Scanning

```python
# Security Testing Framework

import requests
from urllib.parse import urljoin, urlparse
import re
import json
from bs4 import BeautifulSoup

class SecurityScanner:
    """Comprehensive security vulnerability scanner"""
    
    def __init__(self, base_url, auth_token=None):
        self.base_url = base_url
        self.session = requests.Session()
        if auth_token:
            self.session.headers['Authorization'] = f'Bearer {auth_token}'
        self.vulnerabilities = []
    
    def scan_sql_injection(self, endpoints):
        """Test for SQL injection vulnerabilities"""
        sql_payloads = [
            "' OR '1'='1",
            "1' OR '1'='1' --",
            "' UNION SELECT NULL--",
            "1'; DROP TABLE users--",
            "' OR 1=1#",
            "admin'--"
        ]
        
        for endpoint in endpoints:
            for payload in sql_payloads:
                # Test GET parameters
                response = self.session.get(
                    f"{self.base_url}{endpoint}",
                    params={'id': payload, 'search': payload}
                )
                
                if self._detect_sql_error(response.text):
                    self.vulnerabilities.append({
                        'type': 'SQL Injection',
                        'severity': 'Critical',
                        'endpoint': endpoint,
                        'method': 'GET',
                        'payload': payload,
                        'evidence': self._extract_error(response.text)
                    })
                
                # Test POST body
                response = self.session.post(
                    f"{self.base_url}{endpoint}",
                    json={'username': payload, 'search': payload}
                )
                
                if self._detect_sql_error(response.text):
                    self.vulnerabilities.append({
                        'type': 'SQL Injection',
                        'severity': 'Critical',
                        'endpoint': endpoint,
                        'method': 'POST',
                        'payload': payload,
                        'evidence': self._extract_error(response.text)
                    })
    
    def scan_xss(self, endpoints):
        """Test for Cross-Site Scripting vulnerabilities"""
        xss_payloads = [
            '<script>alert("XSS")</script>',
            '<img src=x onerror=alert("XSS")>',
            '<svg onload=alert("XSS")>',
            'javascript:alert("XSS")',
            '<iframe src="javascript:alert(`XSS`)">',
            '"><script>alert(String.fromCharCode(88,83,83))</script>'
        ]
        
        for endpoint in endpoints:
            for payload in xss_payloads:
                # Test reflected XSS
                response = self.session.get(
                    f"{self.base_url}{endpoint}",
                    params={'q': payload, 'name': payload}
                )
                
                if payload in response.text and response.headers.get('Content-Type', '').startswith('text/html'):
                    self.vulnerabilities.append({
                        'type': 'Cross-Site Scripting (XSS)',
                        'severity': 'High',
                        'endpoint': endpoint,
                        'method': 'GET',
                        'payload': payload,
                        'evidence': 'Payload reflected in response without encoding'
                    })
    
    def scan_authentication(self):
        """Test authentication mechanisms"""
        tests = []
        
        # Test for weak password policy
        weak_passwords = ['password', '123456', 'admin', 'test123']
        for password in weak_passwords:
            response = self.session.post(
                f"{self.base_url}/api/auth/register",
                json={'email': 'test@example.com', 'password': password}
            )
            if response.status_code == 200:
                tests.append({
                    'type': 'Weak Password Policy',
                    'severity': 'Medium',
                    'evidence': f'Accepted weak password: {password}'
                })
        
        # Test for missing rate limiting
        for i in range(100):
            response = self.session.post(
                f"{self.base_url}/api/auth/login",
                json={'email': 'test@example.com', 'password': 'wrong'}
            )
            if i == 99 and response.status_code != 429:
                tests.append({
                    'type': 'Missing Rate Limiting',
                    'severity': 'Medium',
                    'evidence': 'No rate limiting on authentication endpoint'
                })
        
        # Test for session fixation
        old_session = self.session.cookies.get('session_id')
        self.session.post(
            f"{self.base_url}/api/auth/login",
            json={'email': 'valid@example.com', 'password': 'ValidPass123!'}
        )
        new_session = self.session.cookies.get('session_id')
        
        if old_session == new_session:
            tests.append({
                'type': 'Session Fixation',
                'severity': 'High',
                'evidence': 'Session ID not regenerated after login'
            })
        
        return tests
    
    def scan_headers(self):
        """Check security headers"""
        response = self.session.get(self.base_url)
        headers = response.headers
        
        security_headers = {
            'X-Frame-Options': {
                'expected': ['DENY', 'SAMEORIGIN'],
                'severity': 'Medium',
                'description': 'Prevents clickjacking attacks'
            },
            'X-Content-Type-Options': {
                'expected': ['nosniff'],
                'severity': 'Medium',
                'description': 'Prevents MIME type sniffing'
            },
            'Strict-Transport-Security': {
                'expected': ['max-age='],
                'severity': 'High',
                'description': 'Enforces HTTPS'
            },
            'Content-Security-Policy': {
                'expected': ["default-src"],
                'severity': 'High',
                'description': 'Prevents XSS and other attacks'
            },
            'X-XSS-Protection': {
                'expected': ['1; mode=block'],
                'severity': 'Low',
                'description': 'Legacy XSS protection'
            }
        }
        
        missing_headers = []
        for header, config in security_headers.items():
            if header not in headers:
                missing_headers.append({
                    'type': 'Missing Security Header',
                    'severity': config['severity'],
                    'header': header,
                    'description': config['description']
                })
            elif not any(expected in headers[header] for expected in config['expected']):
                missing_headers.append({
                    'type': 'Misconfigured Security Header',
                    'severity': config['severity'],
                    'header': header,
                    'current_value': headers[header],
                    'expected_values': config['expected']
                })
        
        return missing_headers
    
    def scan_api_security(self):
        """Test API-specific security issues"""
        api_tests = []
        
        # Test for API versioning
        response = self.session.get(f"{self.base_url}/api/v1/users")
        if response.status_code == 404:
            api_tests.append({
                'type': 'Missing API Versioning',
                'severity': 'Low',
                'description': 'API should support versioning for backwards compatibility'
            })
        
        # Test for excessive data exposure
        response = self.session.get(f"{self.base_url}/api/users/1")
        if response.status_code == 200:
            data = response.json()
            sensitive_fields = ['password', 'passwordHash', 'ssn', 'creditCard']
            exposed = [field for field in sensitive_fields if field in data]
            
            if exposed:
                api_tests.append({
                    'type': 'Excessive Data Exposure',
                    'severity': 'High',
                    'exposed_fields': exposed,
                    'endpoint': '/api/users/1'
                })
        
        # Test for CORS misconfiguration
        response = self.session.options(
            f"{self.base_url}/api/users",
            headers={'Origin': 'http://evil.com'}
        )
        
        if response.headers.get('Access-Control-Allow-Origin') == '*':
            api_tests.append({
                'type': 'CORS Misconfiguration',
                'severity': 'Medium',
                'description': 'API allows requests from any origin'
            })
        
        return api_tests
    
    def generate_security_report(self):
        """Generate comprehensive security report"""
        report = {
            'summary': {
                'total_vulnerabilities': len(self.vulnerabilities),
                'critical': len([v for v in self.vulnerabilities if v.get('severity') == 'Critical']),
                'high': len([v for v in self.vulnerabilities if v.get('severity') == 'High']),
                'medium': len([v for v in self.vulnerabilities if v.get('severity') == 'Medium']),
                'low': len([v for v in self.vulnerabilities if v.get('severity') == 'Low'])
            },
            'vulnerabilities': self.vulnerabilities,
            'recommendations': self._generate_recommendations()
        }
        
        return report
    
    def _generate_recommendations(self):
        """Generate security recommendations based on findings"""
        recommendations = []
        
        # Check for common vulnerability patterns
        if any(v['type'] == 'SQL Injection' for v in self.vulnerabilities):
            recommendations.append({
                'priority': 'Critical',
                'recommendation': 'Implement parameterized queries for all database operations',
                'resources': ['https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html']
            })
        
        if any(v['type'] == 'Cross-Site Scripting (XSS)' for v in self.vulnerabilities):
            recommendations.append({
                'priority': 'High',
                'recommendation': 'Implement proper output encoding and Content Security Policy',
                'resources': ['https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html']
            })
        
        return recommendations
```

### 3.2 Penetration Testing

```python
# Automated Penetration Testing

import subprocess
import json
from typing import List, Dict

class PenetrationTester:
    """Automated penetration testing framework"""
    
    def __init__(self, target_url):
        self.target_url = target_url
        self.findings = []
    
    def run_nmap_scan(self):
        """Run network scan to identify open ports and services"""
        domain = urlparse(self.target_url).netloc
        
        try:
            # Run nmap scan
            result = subprocess.run(
                ['nmap', '-sV', '-sC', '-oX', '-', domain],
                capture_output=True,
                text=True
            )
            
            # Parse results
            open_ports = []
            for line in result.stdout.split('\n'):
                if 'open' in line and 'tcp' in line:
                    parts = line.split()
                    port = parts[0].split('/')[0]
                    service = parts[2] if len(parts) > 2 else 'unknown'
                    open_ports.append({
                        'port': port,
                        'service': service,
                        'state': 'open'
                    })
            
            if open_ports:
                self.findings.append({
                    'type': 'Open Ports',
                    'severity': 'Information',
                    'details': open_ports
                })
            
        except Exception as e:
            print(f"Nmap scan failed: {e}")
    
    def run_nikto_scan(self):
        """Run web vulnerability scan"""
        try:
            result = subprocess.run(
                ['nikto', '-h', self.target_url, '-Format', 'json'],
                capture_output=True,
                text=True
            )
            
            # Parse Nikto results
            vulnerabilities = json.loads(result.stdout)
            
            for vuln in vulnerabilities.get('vulnerabilities', []):
                self.findings.append({
                    'type': 'Web Vulnerability',
                    'severity': self._map_nikto_severity(vuln.get('severity')),
                    'description': vuln.get('msg'),
                    'url': vuln.get('url'),
                    'method': vuln.get('method')
                })
                
        except Exception as e:
            print(f"Nikto scan failed: {e}")
    
    def test_ssl_tls(self):
        """Test SSL/TLS configuration"""
        domain = urlparse(self.target_url).netloc
        
        try:
            # Test with testssl.sh
            result = subprocess.run(
                ['testssl', '--json', '-', domain],
                capture_output=True,
                text=True
            )
            
            ssl_issues = []
            results = json.loads(result.stdout)
            
            # Check for weak protocols
            if results.get('protocols', {}).get('sslv2'):
                ssl_issues.append('SSLv2 enabled (deprecated)')
            if results.get('protocols', {}).get('sslv3'):
                ssl_issues.append('SSLv3 enabled (deprecated)')
            
            # Check for weak ciphers
            weak_ciphers = [c for c in results.get('ciphers', []) 
                           if c.get('strength') < 128]
            if weak_ciphers:
                ssl_issues.append(f'{len(weak_ciphers)} weak ciphers supported')
            
            # Check certificate
            cert = results.get('certificate', {})
            if cert.get('expired'):
                ssl_issues.append('Certificate expired')
            if not cert.get('match_hostname'):
                ssl_issues.append('Certificate hostname mismatch')
            
            if ssl_issues:
                self.findings.append({
                    'type': 'SSL/TLS Configuration',
                    'severity': 'High',
                    'issues': ssl_issues
                })
                
        except Exception as e:
            print(f"SSL/TLS test failed: {e}")
    
    def fuzz_inputs(self, endpoints: List[str]):
        """Fuzz test input validation"""
        fuzz_payloads = [
            # Buffer overflow attempts
            'A' * 10000,
            # Format string
            '%s%s%s%s%s',
            # Command injection
            '; cat /etc/passwd',
            '| whoami',
            '`id`',
            # Path traversal
            '../../../../etc/passwd',
            '..\\..\\..\\..\\windows\\system32\\config\\sam',
            # XML injection
            '<?xml version="1.0"?><!DOCTYPE root [<!ENTITY test SYSTEM "file:///etc/passwd">]><root>&test;</root>',
            # NoSQL injection
            '{"$ne": null}',
            '{"$gt": ""}',
            # LDAP injection
            '*)(uid=*))(|(uid=*',
            # Special characters
            '\x00\x01\x02\x03\x04',
            '\r\n\r\n',
            # Unicode
            '\u0000\uffff'
        ]
        
        for endpoint in endpoints:
            for payload in fuzz_payloads:
                try:
                    # Test various input methods
                    responses = []
                    
                    # GET parameter fuzzing
                    response = requests.get(
                        f"{self.target_url}{endpoint}",
                        params={'input': payload},
                        timeout=5
                    )
                    responses.append(('GET', response))
                    
                    # POST body fuzzing
                    response = requests.post(
                        f"{self.target_url}{endpoint}",
                        json={'input': payload},
                        timeout=5
                    )
                    responses.append(('POST', response))
                    
                    # Header fuzzing
                    response = requests.get(
                        f"{self.target_url}{endpoint}",
                        headers={'X-Custom': payload},
                        timeout=5
                    )
                    responses.append(('HEADER', response))
                    
                    # Check for crashes or errors
                    for method, response in responses:
                        if response.status_code == 500:
                            self.findings.append({
                                'type': 'Input Validation Error',
                                'severity': 'Medium',
                                'endpoint': endpoint,
                                'method': method,
                                'payload': payload[:100] + '...',
                                'response_code': response.status_code
                            })
                        
                        # Check for information disclosure
                        error_patterns = [
                            r'stack trace',
                            r'SQLException',
                            r'error in your SQL syntax',
                            r'undefined index',
                            r'fatal error'
                        ]
                        
                        for pattern in error_patterns:
                            if re.search(pattern, response.text, re.IGNORECASE):
                                self.findings.append({
                                    'type': 'Information Disclosure',
                                    'severity': 'Medium',
                                    'endpoint': endpoint,
                                    'method': method,
                                    'pattern': pattern,
                                    'evidence': response.text[:200]
                                })
                                break
                
                except requests.exceptions.Timeout:
                    # Potential DoS
                    self.findings.append({
                        'type': 'Potential DoS',
                        'severity': 'Medium',
                        'endpoint': endpoint,
                        'payload': payload[:100] + '...',
                        'description': 'Request timed out with fuzzing payload'
                    })
                except Exception as e:
                    # Other errors might indicate vulnerabilities
                    pass
```

## 4. Accessibility Testing

### 4.1 WCAG Compliance Testing

```javascript
// Accessibility Testing with Puppeteer and Axe

const puppeteer = require('puppeteer');
const { AxePuppeteer } = require('@axe-core/puppeteer');

class AccessibilityTester {
    constructor(baseUrl) {
        this.baseUrl = baseUrl;
        this.violations = [];
        this.browser = null;
    }

    async initialize() {
        this.browser = await puppeteer.launch({
            headless: true,
            args: ['--no-sandbox']
        });
    }

    async testPage(path, options = {}) {
        const page = await this.browser.newPage();
        
        // Set viewport for responsive testing
        await page.setViewport({
            width: options.width || 1280,
            height: options.height || 720
        });

        try {
            await page.goto(`${this.baseUrl}${path}`, {
                waitUntil: 'networkidle2'
            });

            // Run axe accessibility tests
            const results = await new AxePuppeteer(page)
                .withTags(['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa'])
                .analyze();

            // Process violations
            if (results.violations.length > 0) {
                this.violations.push({
                    url: `${this.baseUrl}${path}`,
                    viewport: `${options.width || 1280}x${options.height || 720}`,
                    violations: results.violations.map(v => ({
                        id: v.id,
                        impact: v.impact,
                        description: v.description,
                        help: v.help,
                        helpUrl: v.helpUrl,
                        nodes: v.nodes.length,
                        tags: v.tags
                    }))
                });
            }

            // Additional custom tests
            await this.customAccessibilityTests(page, path);

        } catch (error) {
            console.error(`Error testing ${path}:`, error);
        } finally {
            await page.close();
        }
    }

    async customAccessibilityTests(page, path) {
        const customTests = [];

        // Test keyboard navigation
        const keyboardNav = await this.testKeyboardNavigation(page);
        if (!keyboardNav.passed) {
            customTests.push({
                type: 'Keyboard Navigation',
                severity: 'High',
                issues: keyboardNav.issues
            });
        }

        // Test color contrast manually for dynamic content
        const colorContrast = await this.testColorContrast(page);
        if (!colorContrast.passed) {
            customTests.push({
                type: 'Color Contrast',
                severity: 'Medium',
                issues: colorContrast.issues
            });
        }

        // Test form labels
        const formLabels = await this.testFormLabels(page);
        if (!formLabels.passed) {
            customTests.push({
                type: 'Form Labels',
                severity: 'High',
                issues: formLabels.issues
            });
        }

        if (customTests.length > 0) {
            const existingViolation = this.violations.find(v => v.url.includes(path));
            if (existingViolation) {
                existingViolation.customTests = customTests;
            }
        }
    }

    async testKeyboardNavigation(page) {
        const issues = [];

        // Get all interactive elements
        const interactiveElements = await page.$$eval(
            'a, button, input, select, textarea, [tabindex]',
            elements => elements.map(el => ({
                tag: el.tagName,
                text: el.innerText || el.value || el.placeholder,
                tabindex: el.tabIndex,
                visible: el.offsetWidth > 0 && el.offsetHeight > 0
            }))
        );

        // Check for proper tab order
        const tabOrder = interactiveElements
            .filter(el => el.visible && el.tabindex >= 0)
            .sort((a, b) => {
                if (a.tabindex === 0 && b.tabindex === 0) return 0;
                if (a.tabindex === 0) return 1;
                if (b.tabindex === 0) return -1;
                return a.tabindex - b.tabindex;
            });

        // Check for keyboard traps
        for (let i = 0; i < tabOrder.length; i++) {
            if (tabOrder[i].tabindex > 0 && tabOrder[i].tabindex !== i + 1) {
                issues.push(`Non-sequential tabindex: ${tabOrder[i].tabindex}`);
            }
        }

        // Test skip links
        const skipLinks = await page.$('a[href^="#"]');
        if (!skipLinks) {
            issues.push('No skip navigation links found');
        }

        return {
            passed: issues.length === 0,
            issues
        };
    }

    async testColorContrast(page) {
        const issues = [];

        // Get all text elements with their computed styles
        const textElements = await page.evaluate(() => {
            const elements = document.querySelectorAll('*');
            const results = [];

            for (const el of elements) {
                if (el.innerText && el.innerText.trim()) {
                    const style = window.getComputedStyle(el);
                    const bg = style.backgroundColor;
                    const fg = style.color;

                    if (bg !== 'rgba(0, 0, 0, 0)' && fg !== 'rgba(0, 0, 0, 0)') {
                        results.push({
                            text: el.innerText.substring(0, 50),
                            background: bg,
                            foreground: fg,
                            fontSize: parseFloat(style.fontSize),
                            fontWeight: style.fontWeight
                        });
                    }
                }
            }

            return results;
        });

        // Calculate contrast ratios
        for (const element of textElements) {
            const ratio = this.calculateContrastRatio(
                element.foreground,
                element.background
            );

            const isLargeText = element.fontSize >= 18 || 
                (element.fontSize >= 14 && element.fontWeight >= 700);

            const requiredRatio = isLargeText ? 3 : 4.5;

            if (ratio < requiredRatio) {
                issues.push({
                    text: element.text,
                    ratio: ratio.toFixed(2),
                    required: requiredRatio,
                    foreground: element.foreground,
                    background: element.background
                });
            }
        }

        return {
            passed: issues.length === 0,
            issues
        };
    }

    async testFormLabels(page) {
        const issues = [];

        const formIssues = await page.evaluate(() => {
            const problems = [];
            
            // Check all form inputs
            const inputs = document.querySelectorAll('input, select, textarea');
            
            for (const input of inputs) {
                if (input.type === 'hidden' || input.type === 'submit') continue;

                const id = input.id;
                const hasLabel = id && document.querySelector(`label[for="${id}"]`);
                const hasAriaLabel = input.getAttribute('aria-label');
                const hasAriaLabelledBy = input.getAttribute('aria-labelledby');
                const hasTitle = input.getAttribute('title');

                if (!hasLabel && !hasAriaLabel && !hasAriaLabelledBy && !hasTitle) {
                    problems.push({
                        type: input.type,
                        name: input.name,
                        id: input.id,
                        issue: 'No accessible label'
                    });
                }
            }

            // Check for form validation messages
            const requiredInputs = document.querySelectorAll('[required]');
            for (const input of requiredInputs) {
                if (!input.getAttribute('aria-required')) {
                    problems.push({
                        type: input.type,
                        name: input.name,
                        issue: 'Required field missing aria-required'
                    });
                }
            }

            return problems;
        });

        return {
            passed: formIssues.length === 0,
            issues: formIssues
        };
    }

    calculateContrastRatio(fg, bg) {
        // Convert colors to RGB
        const fgRgb = this.parseColor(fg);
        const bgRgb = this.parseColor(bg);

        // Calculate relative luminance
        const fgLum = this.relativeLuminance(fgRgb);
        const bgLum = this.relativeLuminance(bgRgb);

        // Calculate contrast ratio
        const lighter = Math.max(fgLum, bgLum);
        const darker = Math.min(fgLum, bgLum);

        return (lighter + 0.05) / (darker + 0.05);
    }

    parseColor(color) {
        const match = color.match(/rgba?\((\d+),\s*(\d+),\s*(\d+)/);
        return match ? {
            r: parseInt(match[1]),
            g: parseInt(match[2]),
            b: parseInt(match[3])
        } : { r: 0, g: 0, b: 0 };
    }

    relativeLuminance(rgb) {
        const rsRGB = rgb.r / 255;
        const gsRGB = rgb.g / 255;
        const bsRGB = rgb.b / 255;

        const r = rsRGB <= 0.03928 ? rsRGB / 12.92 : Math.pow((rsRGB + 0.055) / 1.055, 2.4);
        const g = gsRGB <= 0.03928 ? gsRGB / 12.92 : Math.pow((gsRGB + 0.055) / 1.055, 2.4);
        const b = bsRGB <= 0.03928 ? bsRGB / 12.92 : Math.pow((bsRGB + 0.055) / 1.055, 2.4);

        return 0.2126 * r + 0.7152 * g + 0.0722 * b;
    }

    async generateReport() {
        const report = {
            summary: {
                totalPages: this.violations.length,
                totalViolations: this.violations.reduce((sum, page) => 
                    sum + page.violations.length, 0),
                criticalViolations: this.violations.reduce((sum, page) => 
                    sum + page.violations.filter(v => v.impact === 'critical').length, 0),
                seriousViolations: this.violations.reduce((sum, page) => 
                    sum + page.violations.filter(v => v.impact === 'serious').length, 0)
            },
            violations: this.violations,
            wcagCoverage: this.calculateWCAGCoverage()
        };

        return report;
    }

    calculateWCAGCoverage() {
        const allTags = new Set();
        const coveredTags = new Set();

        // WCAG 2.1 AA criteria
        const wcag21AA = [
            'wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa',
            'wcag111', 'wcag121', 'wcag122', 'wcag123',
            'wcag131', 'wcag141', 'wcag142', 'wcag143'
        ];

        wcag21AA.forEach(tag => allTags.add(tag));

        this.violations.forEach(page => {
            page.violations.forEach(violation => {
                violation.tags.forEach(tag => {
                    if (wcag21AA.includes(tag)) {
                        coveredTags.add(tag);
                    }
                });
            });
        });

        return {
            total: allTags.size,
            covered: coveredTags.size,
            percentage: (coveredTags.size / allTags.size * 100).toFixed(2)
        };
    }

    async cleanup() {
        if (this.browser) {
            await this.browser.close();
        }
    }
}

module.exports = AccessibilityTester;
```

## 5. Production Monitoring

### 5.1 Real-time Monitoring Setup

```python
# Production Monitoring Implementation

from datadog import initialize, statsd
import sentry_sdk
from prometheus_client import Counter, Histogram, Gauge
import logging
import time
from functools import wraps

class ProductionMonitor:
    """Comprehensive production monitoring"""
    
    def __init__(self, config):
        # Initialize monitoring services
        self._init_datadog(config.get('datadog'))
        self._init_sentry(config.get('sentry'))
        self._init_logging(config.get('logging'))
        
        # Define metrics
        self.request_counter = Counter(
            'app_requests_total',
            'Total requests',
            ['method', 'endpoint', 'status']
        )
        
        self.request_duration = Histogram(
            'app_request_duration_seconds',
            'Request duration',
            ['method', 'endpoint']
        )
        
        self.active_users = Gauge(
            'app_active_users',
            'Currently active users'
        )
        
        self.business_metrics = {}
    
    def _init_datadog(self, config):
        """Initialize Datadog monitoring"""
        if config:
            initialize(**config)
            self.datadog_enabled = True
        else:
            self.datadog_enabled = False
    
    def _init_sentry(self, config):
        """Initialize Sentry error tracking"""
        if config:
            sentry_sdk.init(
                dsn=config['dsn'],
                traces_sample_rate=config.get('traces_sample_rate', 0.1),
                environment=config.get('environment', 'production')
            )
            self.sentry_enabled = True
        else:
            self.sentry_enabled = False
    
    def monitor_request(self, func):
        """Decorator to monitor HTTP requests"""
        @wraps(func)
        def wrapper(*args, **kwargs):
            start_time = time.time()
            
            # Extract request info
            endpoint = request.endpoint or 'unknown'
            method = request.method
            
            try:
                # Execute request
                response = func(*args, **kwargs)
                status = response.status_code if hasattr(response, 'status_code') else 200
                
                # Record success metrics
                self._record_request_metrics(method, endpoint, status, start_time)
                
                return response
                
            except Exception as e:
                # Record error metrics
                self._record_request_metrics(method, endpoint, 500, start_time)
                
                # Send to Sentry
                if self.sentry_enabled:
                    sentry_sdk.capture_exception(e)
                
                # Re-raise
                raise
        
        return wrapper
    
    def _record_request_metrics(self, method, endpoint, status, start_time):
        """Record request metrics to all monitoring systems"""
        duration = time.time() - start_time
        
        # Prometheus metrics
        self.request_counter.labels(
            method=method,
            endpoint=endpoint,
            status=status
        ).inc()
        
        self.request_duration.labels(
            method=method,
            endpoint=endpoint
        ).observe(duration)
        
        # Datadog metrics
        if self.datadog_enabled:
            statsd.increment('app.requests', tags=[
                f'method:{method}',
                f'endpoint:{endpoint}',
                f'status:{status}'
            ])
            
            statsd.histogram('app.request.duration', duration * 1000, tags=[
                f'method:{method}',
                f'endpoint:{endpoint}'
            ])
    
    def monitor_business_metric(self, metric_name, value, tags=None):
        """Track business-specific metrics"""
        # Update local tracking
        if metric_name not in self.business_metrics:
            self.business_metrics[metric_name] = Gauge(
                f'app_business_{metric_name}',
                f'Business metric: {metric_name}'
            )
        
        self.business_metrics[metric_name].set(value)
        
        # Send to Datadog
        if self.datadog_enabled:
            statsd.gauge(f'app.business.{metric_name}', value, tags=tags)
    
    def create_alert(self, name, condition, threshold, message):
        """Create monitoring alert"""
        alert_config = {
            'name': name,
            'condition': condition,
            'threshold': threshold,
            'message': message,
            'channels': ['email', 'slack', 'pagerduty']
        }
        
        # Store alert configuration
        self.alerts[name] = alert_config
        
        # Create Datadog monitor
        if self.datadog_enabled:
            from datadog import api
            api.Monitor.create(
                type='metric alert',
                query=condition,
                name=name,
                message=message,
                options={
                    'thresholds': {'critical': threshold},
                    'notify_no_data': True,
                    'no_data_timeframe': 10
                }
            )
    
    def health_check(self):
        """Comprehensive health check endpoint"""
        health_status = {
            'status': 'healthy',
            'timestamp': time.time(),
            'checks': {}
        }
        
        # Database check
        try:
            db.session.execute('SELECT 1')
            health_status['checks']['database'] = {
                'status': 'healthy',
                'response_time': self._measure_db_response()
            }
        except Exception as e:
            health_status['status'] = 'unhealthy'
            health_status['checks']['database'] = {
                'status': 'unhealthy',
                'error': str(e)
            }
        
        # Redis check
        try:
            redis_client.ping()
            health_status['checks']['redis'] = {
                'status': 'healthy',
                'response_time': self._measure_redis_response()
            }
        except Exception as e:
            health_status['status'] = 'degraded'
            health_status['checks']['redis'] = {
                'status': 'unhealthy',
                'error': str(e)
            }
        
        # External service checks
        for service_name, service_url in self.external_services.items():
            try:
                response = requests.get(service_url, timeout=5)
                health_status['checks'][service_name] = {
                    'status': 'healthy' if response.status_code == 200 else 'degraded',
                    'response_time': response.elapsed.total_seconds(),
                    'status_code': response.status_code
                }
            except Exception as e:
                health_status['status'] = 'degraded'
                health_status['checks'][service_name] = {
                    'status': 'unhealthy',
                    'error': str(e)
                }
        
        return health_status
```

### 5.2 Log Aggregation

```python
# Centralized Logging System

import json
import logging
from pythonjsonlogger import jsonlogger
from elasticsearch import Elasticsearch
import threading
import queue

class CentralizedLogger:
    """Send logs to centralized logging system"""
    
    def __init__(self, app_name, environment):
        self.app_name = app_name
        self.environment = environment
        
        # Configure JSON formatter
        log_format = '%(timestamp)s %(level)s %(name)s %(message)s'
        formatter = jsonlogger.JsonFormatter(log_format)
        
        # Configure handlers
        self.handlers = []
        
        # Console handler
        console_handler = logging.StreamHandler()
        console_handler.setFormatter(formatter)
        self.handlers.append(console_handler)
        
        # Elasticsearch handler
        self.es_client = Elasticsearch(['http://elasticsearch:9200'])
        self.log_queue = queue.Queue()
        self.start_log_worker()
        
        # Configure root logger
        logging.basicConfig(
            level=logging.INFO,
            handlers=self.handlers
        )
    
    def start_log_worker(self):
        """Start background worker for log shipping"""
        def worker():
            batch = []
            while True:
                try:
                    # Collect logs for batch processing
                    log_entry = self.log_queue.get(timeout=1)
                    batch.append(log_entry)
                    
                    # Ship batch when size or time threshold reached
                    if len(batch) >= 100:
                        self._ship_logs(batch)
                        batch = []
                        
                except queue.Empty:
                    # Ship any remaining logs
                    if batch:
                        self._ship_logs(batch)
                        batch = []
        
        thread = threading.Thread(target=worker, daemon=True)
        thread.start()
    
    def _ship_logs(self, logs):
        """Ship logs to Elasticsearch"""
        try:
            # Bulk index logs
            actions = []
            for log in logs:
                actions.append({
                    'index': {
                        '_index': f'logs-{self.app_name}-{self.environment}',
                        '_type': '_doc'
                    }
                })
                actions.append(log)
            
            self.es_client.bulk(body=actions)
        except Exception as e:
            # Fallback to local logging
            logging.error(f"Failed to ship logs: {e}")
    
    def log(self, level, message, **kwargs):
        """Enhanced logging with context"""
        log_entry = {
            'timestamp': time.time(),
            'level': level,
            'app': self.app_name,
            'environment': self.environment,
            'message': message,
            **kwargs
        }
        
        # Add request context if available
        if hasattr(g, 'request_id'):
            log_entry['request_id'] = g.request_id
        if hasattr(g, 'user_id'):
            log_entry['user_id'] = g.user_id
        
        # Queue for shipping
        self.log_queue.put(log_entry)
        
        # Also log locally
        getattr(logging, level.lower())(json.dumps(log_entry))
    
    def create_dashboard_queries(self):
        """Kibana dashboard queries"""
        return {
            'error_rate': {
                'query': {
                    'bool': {
                        'must': [
                            {'term': {'level': 'ERROR'}},
                            {'range': {'timestamp': {'gte': 'now-1h'}}}
                        ]
                    }
                },
                'aggs': {
                    'errors_over_time': {
                        'date_histogram': {
                            'field': 'timestamp',
                            'interval': '1m'
                        }
                    }
                }
            },
            'slow_requests': {
                'query': {
                    'bool': {
                        'must': [
                            {'range': {'duration': {'gte': 1000}}},
                            {'range': {'timestamp': {'gte': 'now-1h'}}}
                        ]
                    }
                },
                'sort': [{'duration': 'desc'}],
                'size': 100
            },
            'user_activity': {
                'query': {
                    'range': {'timestamp': {'gte': 'now-24h'}}
                },
                'aggs': {
                    'unique_users': {
                        'cardinality': {'field': 'user_id'}
                    },
                    'top_endpoints': {
                        'terms': {
                            'field': 'endpoint',
                            'size': 10
                        }
                    }
                }
            }
        }
```

## 6. EVALUATE Phase Checklist

Before considering the phase complete:

### Functional Testing
- [ ] All acceptance criteria validated
- [ ] User journeys tested end-to-end
- [ ] Edge cases handled properly
- [ ] Regression tests passing

### Performance Testing
- [ ] Load tests meet SLAs
- [ ] Stress tests identify limits
- [ ] Database queries optimized
- [ ] Caching strategy effective

### Security Testing
- [ ] Vulnerability scan clean
- [ ] Penetration test passed
- [ ] Security headers configured
- [ ] Authentication/authorization solid

### Accessibility Testing
- [ ] WCAG 2.1 AA compliant
- [ ] Keyboard navigation works
- [ ] Screen reader compatible
- [ ] Color contrast adequate

### Production Readiness
- [ ] Monitoring configured
- [ ] Alerts defined
- [ ] Logging centralized
- [ ] Runbooks documented

## 7. Post-Deployment Validation

### 7.1 Smoke Tests

```python
# Production Smoke Tests

class SmokeTests:
    """Critical path validation in production"""
    
    def __init__(self, base_url, test_user_credentials):
        self.base_url = base_url
        self.credentials = test_user_credentials
        self.results = []
    
    def run_all_tests(self):
        """Run all smoke tests"""
        tests = [
            self.test_homepage_loads,
            self.test_api_health,
            self.test_authentication,
            self.test_critical_endpoints,
            self.test_database_connectivity,
            self.test_external_integrations
        ]
        
        for test in tests:
            try:
                result = test()
                self.results.append({
                    'test': test.__name__,
                    'status': 'PASS' if result else 'FAIL',
                    'timestamp': time.time()
                })
            except Exception as e:
                self.results.append({
                    'test': test.__name__,
                    'status': 'ERROR',
                    'error': str(e),
                    'timestamp': time.time()
                })
        
        return self.results
    
    def test_homepage_loads(self):
        """Verify homepage is accessible"""
        response = requests.get(self.base_url)
        return response.status_code == 200
    
    def test_api_health(self):
        """Verify API health endpoint"""
        response = requests.get(f"{self.base_url}/api/health")
        data = response.json()
        return data.get('status') == 'healthy'
    
    def test_authentication(self):
        """Verify authentication works"""
        response = requests.post(
            f"{self.base_url}/api/auth/login",
            json=self.credentials
        )
        return response.status_code == 200 and 'token' in response.json()
    
    def test_critical_endpoints(self):
        """Test business-critical endpoints"""
        # Get auth token
        auth_response = requests.post(
            f"{self.base_url}/api/auth/login",
            json=self.credentials
        )
        token = auth_response.json()['token']
        headers = {'Authorization': f'Bearer {token}'}
        
        critical_endpoints = [
            '/api/users/profile',
            '/api/projects',
            '/api/dashboard'
        ]
        
        for endpoint in critical_endpoints:
            response = requests.get(
                f"{self.base_url}{endpoint}",
                headers=headers
            )
            if response.status_code != 200:
                return False
        
        return True
```

### 7.2 Rollback Procedures

```bash
#!/bin/bash
# Automated Rollback Script

set -e

# Configuration
APP_NAME="ace-app"
NAMESPACE="production"
PREVIOUS_VERSION="${1:-}"

if [ -z "$PREVIOUS_VERSION" ]; then
    echo "Getting previous version..."
    PREVIOUS_VERSION=$(kubectl rollout history deployment/$APP_NAME -n $NAMESPACE | tail -2 | head -1 | awk '{print $1}')
fi

echo "Rolling back to version: $PREVIOUS_VERSION"

# Create backup of current state
kubectl get deployment $APP_NAME -n $NAMESPACE -o yaml > backup-deployment-$(date +%s).yaml

# Perform rollback
kubectl rollout undo deployment/$APP_NAME -n $NAMESPACE --to-revision=$PREVIOUS_VERSION

# Wait for rollback to complete
kubectl rollout status deployment/$APP_NAME -n $NAMESPACE

# Run smoke tests
python smoke_tests.py

# Check rollback success
if [ $? -eq 0 ]; then
    echo "Rollback successful!"
    
    # Notify team
    curl -X POST $SLACK_WEBHOOK -d '{
        "text": "Rollback completed successfully to version '$PREVIOUS_VERSION'"
    }'
else
    echo "Rollback verification failed!"
    exit 1
fi
```

## 8. Continuous Improvement

### 8.1 Metrics Analysis

```python
# Performance Metrics Analysis

class MetricsAnalyzer:
    """Analyze production metrics for optimization opportunities"""
    
    def __init__(self, metrics_client):
        self.client = metrics_client
    
    def analyze_performance_trends(self, days=7):
        """Identify performance degradation"""
        # Query response time trends
        response_times = self.client.query(
            'avg(app_request_duration_seconds) by (endpoint)',
            start=f'-{days}d',
            end='now'
        )
        
        trends = []
        for endpoint, data in response_times.items():
            # Calculate trend
            if len(data) > 2:
                start_avg = sum(data[:len(data)//3]) / (len(data)//3)
                end_avg = sum(data[-len(data)//3:]) / (len(data)//3)
                
                change_percent = ((end_avg - start_avg) / start_avg) * 100
                
                if change_percent > 10:  # 10% degradation
                    trends.append({
                        'endpoint': endpoint,
                        'degradation': f'{change_percent:.1f}%',
                        'current_avg': f'{end_avg*1000:.0f}ms',
                        'recommendation': 'Investigate performance regression'
                    })
        
        return trends
    
    def identify_error_patterns(self):
        """Find patterns in errors"""
        # Query error logs
        errors = self.client.query(
            'sum(app_requests_total{status=~"5.."}) by (endpoint, status)',
            start='-24h'
        )
        
        patterns = []
        for key, count in errors.items():
            if count > 10:  # Threshold for concern
                endpoint, status = key
                patterns.append({
                    'endpoint': endpoint,
                    'status': status,
                    'count': count,
                    'severity': 'High' if count > 100 else 'Medium'
                })
        
        return patterns
    
    def optimization_recommendations(self):
        """Generate optimization recommendations"""
        recommendations = []
        
        # Check cache hit rates
        cache_stats = self.client.query('app_cache_hit_rate')
        if cache_stats < 0.8:  # 80% hit rate target
            recommendations.append({
                'area': 'Caching',
                'issue': f'Low cache hit rate: {cache_stats:.1%}',
                'recommendation': 'Review cache keys and TTL settings'
            })
        
        # Check database connection pool
        db_connections = self.client.query('database_connections_active')
        max_connections = self.client.query('database_connections_max')
        
        if db_connections / max_connections > 0.8:
            recommendations.append({
                'area': 'Database',
                'issue': 'High connection pool utilization',
                'recommendation': 'Increase connection pool size or optimize queries'
            })
        
        return recommendations
```

## 9. OODA Loop for Rapid Debugging

When production issues arise, apply the OODA (Observe, Orient, Decide, Act) loop for systematic problem resolution:

### 9.1 Observe: Gather Data

```python
# OODA Loop - Observe Phase
class IncidentObserver:
    """Systematic data collection during incidents"""
    
    def __init__(self, monitoring_client, log_client, metrics_client):
        self.monitoring = monitoring_client
        self.logs = log_client
        self.metrics = metrics_client
        
    def observe_incident(self, incident_id, timeframe_minutes=30):
        """Gather all relevant data about the incident"""
        observations = {
            'incident_id': incident_id,
            'timestamp': datetime.utcnow(),
            'metrics': {},
            'logs': {},
            'alerts': {},
            'user_reports': []
        }
        
        # Collect metrics
        end_time = datetime.utcnow()
        start_time = end_time - timedelta(minutes=timeframe_minutes)
        
        # System metrics
        observations['metrics']['cpu'] = self.metrics.get_cpu_usage(start_time, end_time)
        observations['metrics']['memory'] = self.metrics.get_memory_usage(start_time, end_time)
        observations['metrics']['disk_io'] = self.metrics.get_disk_io(start_time, end_time)
        observations['metrics']['network'] = self.metrics.get_network_stats(start_time, end_time)
        
        # Application metrics
        observations['metrics']['response_times'] = self.metrics.get_response_times(start_time, end_time)
        observations['metrics']['error_rates'] = self.metrics.get_error_rates(start_time, end_time)
        observations['metrics']['throughput'] = self.metrics.get_throughput(start_time, end_time)
        
        # Collect relevant logs
        observations['logs']['errors'] = self.logs.search(
            query='level:error',
            start_time=start_time,
            end_time=end_time,
            limit=100
        )
        
        observations['logs']['warnings'] = self.logs.search(
            query='level:warning',
            start_time=start_time,
            end_time=end_time,
            limit=50
        )
        
        # Active alerts
        observations['alerts'] = self.monitoring.get_active_alerts()
        
        return observations
```

### 9.2 Orient: Analyze Patterns

```python
# OODA Loop - Orient Phase
class IncidentAnalyzer:
    """Pattern analysis and root cause identification"""
    
    def __init__(self, historical_data):
        self.historical = historical_data
        self.patterns = self._load_known_patterns()
        
    def orient(self, observations):
        """Analyze data to understand the situation"""
        analysis = {
            'patterns_detected': [],
            'anomalies': [],
            'correlations': [],
            'probable_causes': []
        }
        
        # Detect anomalies
        for metric_name, metric_data in observations['metrics'].items():
            anomalies = self._detect_anomalies(metric_name, metric_data)
            if anomalies:
                analysis['anomalies'].extend(anomalies)
        
        # Pattern matching
        for pattern in self.patterns:
            if self._matches_pattern(observations, pattern):
                analysis['patterns_detected'].append({
                    'pattern': pattern['name'],
                    'confidence': pattern['confidence'],
                    'typical_cause': pattern['typical_cause']
                })
        
        # Correlation analysis
        correlations = self._find_correlations(observations)
        analysis['correlations'] = correlations
        
        # Root cause analysis
        analysis['probable_causes'] = self._analyze_root_causes(
            analysis['patterns_detected'],
            analysis['anomalies'],
            correlations
        )
        
        return analysis
        
    def _detect_anomalies(self, metric_name, metric_data):
        """Detect statistical anomalies in metrics"""
        anomalies = []
        
        # Simple z-score based anomaly detection
        values = [point['value'] for point in metric_data]
        mean = np.mean(values)
        std = np.std(values)
        
        for i, point in enumerate(metric_data):
            z_score = abs((point['value'] - mean) / std) if std > 0 else 0
            if z_score > 3:  # 3 standard deviations
                anomalies.append({
                    'metric': metric_name,
                    'timestamp': point['timestamp'],
                    'value': point['value'],
                    'z_score': z_score
                })
        
        return anomalies
```

### 9.3 Decide: Formulate Response

```python
# OODA Loop - Decide Phase
class IncidentDecider:
    """Decision making based on analysis"""
    
    def __init__(self, playbook_repository):
        self.playbooks = playbook_repository
        self.decision_history = []
        
    def decide(self, analysis, severity_assessment):
        """Formulate response strategy"""
        decision = {
            'timestamp': datetime.utcnow(),
            'severity': severity_assessment,
            'actions': [],
            'escalation_needed': False,
            'estimated_impact': {}
        }
        
        # Match to playbooks
        for cause in analysis['probable_causes']:
            playbook = self.playbooks.get_playbook(cause['type'])
            if playbook:
                decision['actions'].append({
                    'playbook': playbook['name'],
                    'steps': playbook['steps'],
                    'priority': cause['confidence'] * playbook['effectiveness']
                })
        
        # Determine escalation needs
        if severity_assessment['level'] >= 3 or not decision['actions']:
            decision['escalation_needed'] = True
            decision['escalation_reason'] = 'High severity or no automated response available'
        
        # Sort actions by priority
        decision['actions'].sort(key=lambda x: x['priority'], reverse=True)
        
        # Estimate impact of actions
        for action in decision['actions'][:3]:  # Top 3 actions
            decision['estimated_impact'][action['playbook']] = {
                'resolution_probability': action['priority'] / 100,
                'estimated_time': self._estimate_resolution_time(action),
                'risk_level': self._assess_action_risk(action)
            }
        
        self.decision_history.append(decision)
        return decision
```

### 9.4 Act: Execute Response

```python
# OODA Loop - Act Phase
class IncidentResponder:
    """Execute incident response actions"""
    
    def __init__(self, automation_client, notification_client):
        self.automation = automation_client
        self.notifications = notification_client
        self.action_log = []
        
    def act(self, decision, dry_run=False):
        """Execute the decided actions"""
        execution_results = {
            'started_at': datetime.utcnow(),
            'actions_taken': [],
            'notifications_sent': [],
            'success': True
        }
        
        # Send initial notifications
        if decision['escalation_needed']:
            self._escalate_to_oncall(decision)
            execution_results['notifications_sent'].append('on-call escalation')
        
        # Execute playbook actions
        for action in decision['actions']:
            if dry_run:
                print(f"DRY RUN: Would execute {action['playbook']}")
                continue
                
            try:
                result = self._execute_playbook(action)
                execution_results['actions_taken'].append({
                    'playbook': action['playbook'],
                    'status': 'success',
                    'result': result
                })
                
                # Check if issue is resolved
                if self._verify_resolution():
                    execution_results['resolution_time'] = (
                        datetime.utcnow() - execution_results['started_at']
                    ).total_seconds()
                    break
                    
            except Exception as e:
                execution_results['actions_taken'].append({
                    'playbook': action['playbook'],
                    'status': 'failed',
                    'error': str(e)
                })
                execution_results['success'] = False
        
        return execution_results
        
    def _execute_playbook(self, action):
        """Execute a specific playbook"""
        results = []
        
        for step in action['steps']:
            if step['type'] == 'scale':
                result = self.automation.scale_service(
                    service=step['target'],
                    replicas=step['value']
                )
            elif step['type'] == 'restart':
                result = self.automation.restart_service(
                    service=step['target']
                )
            elif step['type'] == 'rollback':
                result = self.automation.rollback_deployment(
                    service=step['target'],
                    version=step['version']
                )
            elif step['type'] == 'circuit_breaker':
                result = self.automation.toggle_circuit_breaker(
                    service=step['target'],
                    state=step['state']
                )
            
            results.append({
                'step': step['type'],
                'result': result
            })
            
            # Wait for stabilization
            time.sleep(step.get('wait_seconds', 30))
            
        return results
```

### 9.5 Complete OODA Implementation

```python
# Full OODA Loop Implementation
class OODALoop:
    """Complete OODA loop for incident response"""
    
    def __init__(self, monitoring, logs, metrics, playbooks):
        self.observer = IncidentObserver(monitoring, logs, metrics)
        self.analyzer = IncidentAnalyzer(historical_data)
        self.decider = IncidentDecider(playbooks)
        self.responder = IncidentResponder(automation, notifications)
        self.loop_count = 0
        
    def execute_loop(self, incident_id, max_loops=5):
        """Execute OODA loop until resolution or max attempts"""
        
        while self.loop_count < max_loops:
            self.loop_count += 1
            print(f"OODA Loop iteration {self.loop_count}")
            
            # Observe
            observations = self.observer.observe_incident(
                incident_id, 
                timeframe_minutes=30
            )
            
            # Orient
            analysis = self.analyzer.orient(observations)
            
            # Assess severity
            severity = self._assess_severity(observations, analysis)
            
            # Decide
            decision = self.decider.decide(analysis, severity)
            
            # Act
            results = self.responder.act(decision)
            
            # Check if resolved
            if results.get('resolution_time'):
                print(f"Incident resolved in {results['resolution_time']}s")
                break
                
            # Wait before next loop
            time.sleep(60)
        
        return {
            'loops_executed': self.loop_count,
            'resolved': bool(results.get('resolution_time')),
            'final_state': self._get_current_state()
        }
```

### 9.6 OODA Loop Best Practices

```markdown
## OODA Loop Guidelines

### Speed is Key
- Minimize observation time with pre-built dashboards
- Use automated pattern matching for orientation
- Have pre-approved playbooks for common scenarios
- Automate safe actions to reduce human latency

### Continuous Learning
- Record all loops for post-incident review
- Update pattern library with new incidents
- Refine playbooks based on effectiveness
- Share learnings across teams

### Human-in-the-Loop
- Set clear automation boundaries
- Require human approval for high-risk actions
- Maintain manual override capabilities
- Document decision rationale

### Feedback Integration
- Monitor action effectiveness in real-time
- Adjust strategy based on results
- Know when to break the loop and escalate
- Capture metrics for improvement
```

## Conclusion

The EVALUATE phase transforms good code into production-ready systems. Through comprehensive testing, monitoring, and continuous improvement, we ensure our solutions not only work but excel under real-world conditions.

Remember: **"In God we trust, all others must bring data."** - W. Edwards Deming

## Quick Reference Card - EVALUATE Phase

### Key Activities Checklist
- [ ] **Functional Validation**
  - [ ] Acceptance tests passing
  - [ ] User journey tests complete
  - [ ] Edge cases covered
- [ ] **Performance Testing**
  - [ ] Load tests executed
  - [ ] Response times measured
  - [ ] Resource usage optimized
- [ ] **Security Testing**
  - [ ] Vulnerability scans clean
  - [ ] Penetration tests passed
  - [ ] Security headers verified
- [ ] **Accessibility Testing**
  - [ ] WCAG compliance checked
  - [ ] Keyboard navigation tested
  - [ ] Screen reader compatible
- [ ] **Production Readiness**
  - [ ] Monitoring configured
  - [ ] Alerts defined
  - [ ] Runbooks documented

### Key Deliverables
1. **Test Reports** - Complete validation results
2. **Performance Benchmarks** - Speed and scalability metrics
3. **Security Audit** - Vulnerability assessment
4. **Monitoring Dashboard** - Real-time system health
5. **Operations Runbook** - Incident response procedures

### Testing Pyramid Applied
- Unit Tests (70%) - Fast, isolated component tests
- Integration Tests (20%) - Component interaction tests
- E2E Tests (10%) - Full user journey validation

### OODA Loop for Issues
1. **Observe** - Gather metrics and logs
2. **Orient** - Analyze patterns and causes
3. **Decide** - Choose response strategy
4. **Act** - Execute and measure impact

### Production Checklist
- ✅ All tests passing
- ✅ Performance targets met
- ✅ Security standards satisfied
- ✅ Monitoring active
- ✅ Rollback plan ready

### Common Pitfalls to Avoid
- ❌ Testing only happy paths
- ❌ Ignoring non-functional requirements
- ❌ Skipping accessibility testing
- ❌ Inadequate monitoring setup
- ❌ Missing documentation

## Additional Resources
- [Resources & References](./ACE-Method.md#resources--references)
- [Implementation Guide for LLMs](./ACE-Method.md#implementation-guide-for-llms)

---

**Navigation**:
- Previous: [02 Create.md](./02 Create.md)
- Current: **03 Evaluate.md**
- Next: Return to [01 Analyze.md](./01 Analyze.md) for next iteration

**Framework Home**: [ACE-Method.md](./ACE-Method.md) or [CLI-ACE-Method.md](./CLI-ACE-Method.md)

**Next Steps**: With evaluation complete, the cycle continues. Return to ANALYZE with insights gained, creating an ever-improving system.