# 🚀 12-Month Roadmap: 10+ LPA Software Engineering Job
### Java-First | Full-Stack | Backend Engineering | Product Companies

> **Profile:** BCA Graduate | Full-Stack Developer | React.js, React Native, Node.js, NestJS, MongoDB, PostgreSQL, Docker, AWS  
> **Goal:** 10+ LPA at Amazon, PhonePe, Razorpay, Swiggy, Zomato, Walmart, Atlassian within 12 months  
> **Strategy:** Java + Spring Boot for backend, DSA in Java, System Design, and strong portfolio

---

## 📊 TL;DR — What Actually Matters for 10+ LPA

| Priority | Skill | Why |
|----------|-------|-----|
| 🔴 CRITICAL | DSA in Java (LeetCode) | Every product company screens here |
| 🔴 CRITICAL | Java + Spring Boot | Industry standard backend at target companies |
| 🔴 CRITICAL | System Design | Asked at every 10+ LPA interview |
| 🟠 HIGH | PostgreSQL + SQL | Backend engineering core |
| 🟠 HIGH | Redis | Used heavily at PhonePe, Razorpay, Swiggy |
| 🟠 HIGH | Your existing React.js skills | Keep sharp — full-stack edge |
| 🟡 MEDIUM | Docker + Kubernetes | Deployment and scalability questions |
| 🟡 MEDIUM | AWS (EC2, S3, RDS, SQS) | Cloud-native product companies |
| 🟢 OPTIONAL | Open Source Contributions | Bonus, not mandatory |
| ⬛ IGNORE | Learning another JS framework | You already know enough JS |
| ⬛ IGNORE | Kotlin, Golang, Rust | No ROI right now |
| ⬛ IGNORE | ML/AI unless interviewing for ML roles | Off-target |

**LeetCode Target:** 300–400 problems (150 Easy, 120 Medium, 30–50 Hard)  
**Projects to Build:** 3–4 strong Java/Spring Boot projects  
**Job Applications:** 10–15 per week in months 9–12  

---

## 🗓️ 12-Month Overview

```
Month 1–2   → Java Fundamentals + Core DSA
Month 3–4   → Spring Boot + Intermediate DSA
Month 5–6   → System Design + PostgreSQL + Redis
Month 7–8   → Docker, K8s, AWS + Advanced DSA
Month 9–10  → Projects + Open Source + Resume Polish
Month 11–12 → Full Interview Mode + Job Applications
```

---

## 📅 PHASE 1 — Foundation (Month 1–2)

### 🔵 Java Roadmap (Start Here)

#### Month 1 — Core Java

**Week 1–2: Java Basics**
- Install JDK 17 (LTS) + IntelliJ IDEA (Community)
- Variables, Data Types, Operators, Type Casting
- Control Flow: if/else, switch, for, while, do-while
- Arrays (1D, 2D), Strings and String methods
- Methods, Recursion basics
- Input/Output: Scanner, BufferedReader
- Practice: Solve 20 easy array/string problems on LeetCode in Java

**Week 3–4: Object-Oriented Programming (OOP)**
- Classes and Objects
- Constructors, `this` keyword
- Inheritance (`extends`), Method Overriding
- Polymorphism (compile-time and runtime)
- Encapsulation: private fields, getters/setters
- Abstraction: abstract classes, interfaces
- `static` keyword, `final` keyword
- `super` keyword
- Practice: Build a simple Bank Account system in Java

#### Month 2 — Java Intermediate + Collections

**Week 5–6: Java Collections Framework**

```
java.util package:
├── List Interface
│   ├── ArrayList        ← most used, know O(n) ops
│   └── LinkedList       ← know when to use vs ArrayList
├── Set Interface
│   ├── HashSet          ← O(1) average, unordered
│   ├── LinkedHashSet    ← insertion order
│   └── TreeSet          ← sorted, O(log n)
├── Map Interface
│   ├── HashMap          ← O(1) average, must know internals
│   ├── LinkedHashMap    ← ordered by insertion
│   └── TreeMap          ← sorted keys
├── Queue Interface
│   ├── PriorityQueue    ← min-heap by default
│   └── ArrayDeque       ← use instead of Stack
└── Stack (use Deque)
```

Key concepts to master:
- `Iterator`, `for-each` loop
- `Comparable` vs `Comparator`
- `Collections.sort()`, custom sorting
- `Arrays.sort()`, `Arrays.fill()`, `Arrays.copyOf()`

**Week 7–8: Java Advanced Concepts**
- Generics: `<T>`, bounded wildcards `<? extends T>`
- Exception Handling: try/catch/finally, custom exceptions, checked vs unchecked
- File I/O: FileReader, BufferedWriter, NIO basics
- Multithreading basics: Thread, Runnable, synchronized
- Java 8+ Features (CRITICAL for interviews):
  - Lambda expressions: `(a, b) -> a + b`
  - Functional Interfaces: Predicate, Function, Consumer, Supplier
  - Stream API: filter, map, reduce, collect, sorted
  - Optional class
  - `forEach`, method references

```java
// Java 8 Stream Example — must know cold
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5);
int sumOfEvenSquares = nums.stream()
    .filter(n -> n % 2 == 0)
    .map(n -> n * n)
    .reduce(0, Integer::sum);
```

---

### 🔵 DSA Roadmap — Java (Month 1–2 Focus)

#### DSA Phase 1: Arrays, Strings, Math (Month 1)

**Must-know patterns:**
- Two Pointers
- Sliding Window
- Prefix Sum
- Kadane's Algorithm (Max Subarray)
- Binary Search basics

**LeetCode problems to solve (target: 60 problems this month):**

```
Arrays (Easy–Medium):
- Two Sum (#1)
- Best Time to Buy and Sell Stock (#121)
- Maximum Subarray (#53)
- Contains Duplicate (#217)
- Product of Array Except Self (#238)
- Maximum Product Subarray (#152)
- Find Minimum in Rotated Sorted Array (#153)
- Search in Rotated Sorted Array (#33)
- Container With Most Water (#11)
- 3Sum (#15)

Strings:
- Valid Anagram (#242)
- Group Anagrams (#49)
- Longest Substring Without Repeating Characters (#3)
- Longest Repeating Character Replacement (#424)
- Minimum Window Substring (#76)
- Valid Parentheses (#20)
- Longest Palindromic Substring (#5)
- Palindromic Substrings (#647)
```

#### DSA Phase 2: Linked Lists, Stacks, Queues (Month 2)

```java
// Build your own LinkedList — interviewers love this
class ListNode {
    int val;
    ListNode next;
    ListNode(int val) { this.val = val; }
}
```

**LeetCode problems (target: 50 problems this month):**
```
Linked Lists:
- Reverse Linked List (#206)
- Merge Two Sorted Lists (#21)
- Linked List Cycle (#141)
- Find the Duplicate Number (#287)
- LRU Cache (#146) ← very important
- Merge K Sorted Lists (#23)
- Remove Nth Node From End (#19)

Stacks & Queues:
- Valid Parentheses (#20)
- Min Stack (#155)
- Evaluate Reverse Polish Notation (#150)
- Daily Temperatures (#739)
- Car Fleet (#853)
- Largest Rectangle in Histogram (#84)
```

**Running total: ~110 LeetCode problems by end of Month 2**

---

## 📅 PHASE 2 — Core Backend (Month 3–4)

### 🔵 Spring Boot Roadmap

#### Month 3 — Spring Boot Fundamentals

**Week 9–10: Spring Core Concepts**

```
Spring Ecosystem:
├── Spring Core (IoC, DI)
├── Spring Boot (auto-configuration)
├── Spring MVC (REST APIs)
├── Spring Data JPA (ORM)
├── Spring Security (auth)
└── Spring Boot Test (testing)
```

Topics:
- Spring IoC Container: ApplicationContext
- Dependency Injection: `@Autowired`, `@Component`, `@Service`, `@Repository`
- Bean lifecycle, `@Bean`, `@Configuration`
- Spring Boot auto-configuration: how it works
- `application.properties` vs `application.yml`
- Profiles: `@Profile`, `spring.profiles.active`
- `@Value`, `@ConfigurationProperties`

**Week 11–12: REST API with Spring Boot**

```java
// Standard Spring Boot REST Controller
@RestController
@RequestMapping("/api/v1/users")
public class UserController {
    
    private final UserService userService;
    
    @Autowired  // constructor injection preferred
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(userService.findById(id));
    }
    
    @PostMapping
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserRequest request) {
        UserDTO created = userService.create(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<UserDTO> updateUser(@PathVariable Long id, 
                                               @Valid @RequestBody UpdateUserRequest request) {
        return ResponseEntity.ok(userService.update(id, request));
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

Topics:
- `@RestController`, `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`
- `@PathVariable`, `@RequestParam`, `@RequestBody`
- `ResponseEntity<T>` — always return this in real apps
- `@Valid`, `@NotNull`, `@NotBlank`, `@Size` (Bean Validation)
- Global Exception Handling: `@ControllerAdvice`, `@ExceptionHandler`
- Custom Error Response structure
- `@RequestMapping` versioning: `/api/v1/`, `/api/v2/`

**Build:** A basic CRUD REST API for User management with proper validation and error handling

#### Month 4 — Spring Boot Advanced

**Week 13–14: Spring Data JPA + Database**

```java
// JPA Entity — know all annotations
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_email", columnList = "email", unique = true)
})
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    private String name;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Order> orders;
    
    @CreationTimestamp
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    private LocalDateTime updatedAt;
}

// Repository — extending JpaRepository gives you 20+ methods free
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    List<User> findByNameContainingIgnoreCase(String name);
    
    @Query("SELECT u FROM User u WHERE u.createdAt > :date")
    List<User> findRecentUsers(@Param("date") LocalDateTime date);
    
    // Native query when JPQL isn't enough
    @Query(value = "SELECT * FROM users WHERE email LIKE %:domain", nativeQuery = true)
    List<User> findByEmailDomain(@Param("domain") String domain);
}
```

Topics:
- JPA relationships: `@OneToMany`, `@ManyToOne`, `@ManyToMany`, `@OneToOne`
- Fetch types: `EAGER` vs `LAZY` (always use LAZY by default)
- N+1 problem and how to fix it (`@EntityGraph`, `JOIN FETCH`)
- Transactions: `@Transactional`, propagation, isolation levels
- Database migrations: Flyway (preferred) or Liquibase
- DTO pattern: never expose entities directly in API responses

**Week 15–16: Spring Security + Advanced Topics**

Topics:
- Spring Security filter chain
- JWT Authentication (implement from scratch)
- Role-based authorization: `@PreAuthorize("hasRole('ADMIN')")`
- Password encoding: BCryptPasswordEncoder
- CORS configuration
- OAuth2 basics (optional but good to know)
- Caching: `@Cacheable`, `@CacheEvict` with Redis
- Async processing: `@Async`, CompletableFuture
- Scheduling: `@Scheduled`
- Spring Boot Actuator: health checks, metrics
- Testing: `@SpringBootTest`, `@WebMvcTest`, `@DataJpaTest`, Mockito

**Build:** Full authentication service with JWT, role-based access, and Flyway migrations

---

### 🔵 DSA Month 3–4: Trees, Graphs, Recursion, DP Intro

#### Trees (Month 3)

```java
class TreeNode {
    int val;
    TreeNode left, right;
    TreeNode(int val) { this.val = val; }
}
```

**LeetCode problems (target: 60 problems):**
```
Binary Trees:
- Invert Binary Tree (#226)
- Maximum Depth of Binary Tree (#104)
- Diameter of Binary Tree (#543)
- Balanced Binary Tree (#110)
- Same Tree (#100)
- Subtree of Another Tree (#572)
- Lowest Common Ancestor of BST (#235)
- Binary Tree Level Order Traversal (#102)
- Binary Tree Right Side View (#199)
- Count Good Nodes in Binary Tree (#1448)
- Validate Binary Search Tree (#98)
- Kth Smallest Element in BST (#230)
- Construct Binary Tree from Preorder and Inorder (#105)

Heap/Priority Queue:
- Kth Largest Element (#215)
- Task Scheduler (#621)
- Design Twitter (#355)
- Find Median from Data Stream (#295)
```

#### Graphs (Month 4)

**LeetCode problems (target: 50 problems):**
```
Graph BFS/DFS:
- Number of Islands (#200)
- Clone Graph (#133)
- Max Area of Island (#695)
- Pacific Atlantic Water Flow (#417)
- Surrounded Regions (#130)
- Rotting Oranges (#994)
- Walls and Gates (#286)
- Course Schedule (#207)   ← Topological sort
- Course Schedule II (#210)
- Redundant Connection (#684)
- Number of Connected Components (#323)
- Graph Valid Tree (#261)
- Word Ladder (#127)
```

**Running total: ~220 LeetCode problems by end of Month 4**

---

## 📅 PHASE 3 — System Design + Databases (Month 5–6)

### 🔵 PostgreSQL Roadmap

#### Month 5 — PostgreSQL Deep Dive

**Week 17–18: Advanced SQL**

```sql
-- Must-know SQL for interviews

-- Window Functions (asked frequently)
SELECT 
    employee_id,
    salary,
    department,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as rank,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rank_with_gaps,
    LAG(salary) OVER (PARTITION BY department ORDER BY salary) as prev_salary,
    SUM(salary) OVER (PARTITION BY department) as dept_total
FROM employees;

-- CTEs (Common Table Expressions)
WITH dept_avg AS (
    SELECT department, AVG(salary) as avg_salary
    FROM employees
    GROUP BY department
)
SELECT e.*, d.avg_salary
FROM employees e
JOIN dept_avg d ON e.department = d.department
WHERE e.salary > d.avg_salary;

-- Recursive CTE (org hierarchy)
WITH RECURSIVE org_hierarchy AS (
    SELECT id, name, manager_id, 0 as level
    FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.id, e.name, e.manager_id, oh.level + 1
    FROM employees e
    JOIN org_hierarchy oh ON e.manager_id = oh.id
)
SELECT * FROM org_hierarchy;

-- EXPLAIN ANALYZE — always use this for optimization
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 123;
```

**Topics:**
- ACID properties (Atomicity, Consistency, Isolation, Durability) — memorize this
- Indexing: B-Tree (default), Hash, GIN, GiST
- When to add indexes, when not to
- Index types: single, composite, partial, covering
- Query optimization: EXPLAIN, EXPLAIN ANALYZE
- Joins: INNER, LEFT, RIGHT, FULL OUTER, CROSS, SELF JOIN
- Subqueries vs CTEs vs Joins — when to use which
- Transactions and isolation levels: READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE
- Deadlocks: what they are, how to prevent
- Connection pooling: PgBouncer, HikariCP (Spring Boot's default)
- PostgreSQL-specific: JSONB, Arrays, UUID, Sequences
- Partitioning: range, list, hash
- Vacuum and Autovacuum
- Replication: primary-replica setup

**Week 19–20: Database Design**
- Normalization: 1NF, 2NF, 3NF (know by heart)
- When to denormalize
- Schema design for: e-commerce, social media, payment systems
- Foreign keys, cascades
- Soft deletes: `deleted_at TIMESTAMP`
- Audit tables pattern
- UUID vs BIGINT for primary keys (pros/cons)

### 🔵 Redis Roadmap

**Week 19–20: Redis**

```
Redis Data Structures:
├── String      → caching, counters, session tokens
├── List        → message queues, activity feeds
├── Set         → unique tags, friend lists
├── Sorted Set  → leaderboards, rate limiting
├── Hash        → user sessions, object caching
├── Pub/Sub     → real-time notifications
└── Streams     → event sourcing (advanced)
```

**Key concepts:**
- Cache patterns: Cache-Aside, Write-Through, Write-Behind
- Cache invalidation strategies
- TTL (Time-To-Live) — always set it
- Redis as a session store
- Distributed locking with SETNX / Redlock
- Rate limiting with Redis (sliding window, token bucket)
- Redis persistence: RDB vs AOF
- Redis Cluster vs Redis Sentinel
- Spring Boot + Redis: `@Cacheable`, `@CacheEvict`, `RedisTemplate`

```java
// Redis in Spring Boot
@Service
public class ProductService {
    
    @Cacheable(value = "products", key = "#id", unless = "#result == null")
    public Product getProduct(Long id) {
        return productRepository.findById(id).orElse(null);
    }
    
    @CacheEvict(value = "products", key = "#id")
    public void updateProduct(Long id, UpdateProductRequest req) {
        // cache cleared on update
    }
}
```

### 🔵 System Design Roadmap

#### Month 6 — System Design

**Core concepts to master (in this order):**

**1. Building blocks — learn each in depth:**
```
├── Load Balancers (Round Robin, Least Connections, IP Hash)
├── CDN (Content Delivery Network)
├── DNS
├── Web Servers vs Application Servers
├── Databases (SQL vs NoSQL — when to choose what)
├── Caching (Redis, Memcached)
├── Message Queues (Kafka, RabbitMQ, SQS)
├── API Gateway
├── Service Discovery
└── Reverse Proxy (Nginx — you already know this!)
```

**2. CAP Theorem — must know cold:**
- Consistency, Availability, Partition Tolerance — pick 2
- CP systems: ZooKeeper, HBase
- AP systems: Cassandra, DynamoDB
- CA systems: RDBMS (only in single-node)

**3. Scalability concepts:**
- Horizontal vs Vertical scaling
- Database sharding: range, hash, directory-based
- Database replication: read replicas
- Stateless services
- Microservices vs Monolith

**4. System Design problems to practice (in order):**
```
Beginner (Month 5):
1. Design a URL Shortener (Bit.ly)
2. Design a Pastebin
3. Design a Rate Limiter

Intermediate (Month 6):
4. Design Instagram/Twitter Feed
5. Design WhatsApp
6. Design Uber
7. Design Netflix

Advanced (practice during interview prep):
8. Design a Payment System (Razorpay-style)
9. Design a Food Delivery System (Swiggy-style)
10. Design Amazon's Order System
```

**Resources:**
- Book: "System Design Interview" by Alex Xu (Vol 1 + 2)
- GitHub: donnemartin/system-design-primer
- YouTube: Gaurav Sen, Tech Dummies

**Framework for answering System Design questions:**
```
1. Clarify requirements (2–3 min)
   - Functional: what does it do?
   - Non-functional: scale, availability, latency

2. Capacity estimation (2–3 min)
   - DAU, QPS (queries per second), Storage

3. High-level design (10 min)
   - Draw boxes: Client → API Gateway → Services → DB

4. Deep dive (15–20 min)
   - Database schema
   - API design
   - Bottlenecks and solutions

5. Wrap up (2–3 min)
   - Trade-offs you made
   - What you'd improve
```

### 🔵 DSA Month 5–6: Dynamic Programming + Advanced

**Dynamic Programming Patterns:**

```
DP Patterns:
├── 1D DP (Fibonacci, Climbing Stairs)
├── 2D DP (Grid problems)
├── Knapsack (0/1, Unbounded)
├── LCS/LIS (String DP)
├── Interval DP
└── Tree DP
```

**LeetCode problems (target: 70 problems):**
```
1D DP:
- Climbing Stairs (#70)
- House Robber (#198)
- House Robber II (#213)
- Longest Palindromic Substring (#5)
- Palindromic Substrings (#647)
- Decode Ways (#91)
- Coin Change (#322)
- Maximum Product Subarray (#152)
- Word Break (#139)
- Combination Sum IV (#377)

2D DP:
- Unique Paths (#62)
- Longest Common Subsequence (#1143)
- Best Time to Buy and Sell Stock with Cooldown (#309)
- Coin Change 2 (#518)
- Target Sum (#494)
- Interleaving String (#97)
- Longest Increasing Subsequence (#300)
- Partition Equal Subset Sum (#416)
- Edit Distance (#72)
```

**Running total: ~290 LeetCode problems by end of Month 6**

---

## 📅 PHASE 4 — DevOps + Cloud (Month 7–8)

### 🔵 Docker + Kubernetes Roadmap

#### Month 7 — Docker (Deep Dive)

**Docker concepts (you already know basics):**

```dockerfile
# Production-grade Spring Boot Dockerfile
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/*.jar app.jar

# Non-root user for security
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s CMD wget -q localhost:8080/actuator/health -O - || exit 1
ENTRYPOINT ["java", "-jar", "app.jar"]
```

```yaml
# docker-compose.yml for local development
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/mydb
      - SPRING_REDIS_HOST=redis
    depends_on:
      - postgres
      - redis
    
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

**Advanced Docker topics:**
- Multi-stage builds (save image size)
- Docker networking: bridge, host, overlay
- Docker volumes vs bind mounts
- Docker secrets for production
- .dockerignore optimization
- Container health checks
- Resource limits: `--memory`, `--cpus`

#### Month 7 — Kubernetes Basics

```yaml
# Kubernetes Deployment for Spring Boot
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: spring-app
  template:
    metadata:
      labels:
        app: spring-app
    spec:
      containers:
      - name: spring-app
        image: myrepo/spring-app:1.0.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        readinessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 30
        livenaryProbe:
          httpGet:
            path: /actuator/health
            port: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: spring-app-service
spec:
  selector:
    app: spring-app
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
```

**Kubernetes concepts to learn:**
- Pod, Deployment, ReplicaSet, Service
- ConfigMap and Secrets
- Ingress and Ingress Controller (Nginx Ingress)
- Horizontal Pod Autoscaler (HPA)
- Namespaces
- PersistentVolume, PersistentVolumeClaim
- Rolling updates and rollbacks
- kubectl commands (must know 20+ commands)
- Helm: what it is and basic usage

**Local practice:** Minikube or Kind (Kubernetes in Docker)

### 🔵 AWS Roadmap

#### Month 8 — AWS

**Services to learn (in priority order for product companies):**

```
Tier 1 — Must Know:
├── EC2: instances, AMIs, security groups, key pairs
├── S3: buckets, policies, presigned URLs, lifecycle rules
├── RDS: PostgreSQL on AWS, read replicas, Multi-AZ
├── ElastiCache: Redis on AWS
├── VPC: subnets, route tables, NAT gateway, security groups
├── IAM: users, roles, policies (least privilege)
└── CloudWatch: logs, metrics, alarms

Tier 2 — Know Conceptually:
├── SQS: message queuing (vs Kafka)
├── SNS: pub/sub notifications
├── Lambda: serverless functions
├── API Gateway: REST/HTTP APIs
├── ECS/EKS: container orchestration
├── Route 53: DNS management
└── CloudFront: CDN

Tier 3 — Awareness Only:
├── DynamoDB
├── Kinesis
└── Secrets Manager
```

**Practical AWS tasks:**
- Deploy your Spring Boot app on EC2
- Set up RDS PostgreSQL and connect from app
- Store files in S3, generate presigned URLs
- Set up Application Load Balancer
- Configure auto-scaling group
- Set up CloudWatch alarms for high CPU
- Use IAM roles (never hardcode AWS credentials)

**CI/CD with AWS:**
- GitHub Actions + ECR + ECS (most common pipeline)
- Or: Jenkins + Docker + EC2

```yaml
# GitHub Actions CI/CD Pipeline
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'
      
      - name: Build with Maven
        run: mvn clean package -DskipTests
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-south-1
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1
      
      - name: Build, tag, push Docker image
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
```

### 🔵 DSA Month 7–8: Advanced + Revision

**Advanced topics:**
- Backtracking (Subsets, Permutations, N-Queens)
- Tries (Prefix trees)
- Segment Trees (range queries)
- Union-Find / Disjoint Set Union
- Dijkstra's, Bellman-Ford, Floyd-Warshall (Graph shortest paths)
- Topological Sort (Kahn's algorithm, DFS approach)

**LeetCode problems (target: 60–80 more):**
```
Backtracking:
- Subsets (#78)
- Combination Sum (#39)
- Permutations (#46)
- Word Search (#79)
- N-Queens (#51)
- Palindrome Partitioning (#131)

Tries:
- Implement Trie (#208)
- Design Add and Search Words (#211)
- Word Search II (#212)

Advanced Graphs:
- Cheapest Flights Within K Stops (#787)
- Network Delay Time (#743)
- Swim in Rising Water (#778)
- Alien Dictionary (#269)
- Sequence Reconstruction (#444)
```

**Running total: ~370 LeetCode problems by end of Month 8**

---

## 📅 PHASE 5 — Projects + Open Source (Month 9–10)

### 🔵 Project Ideas for Product Companies

Build **3 strong projects** (not 10 mediocre ones). Quality > quantity.

---

#### Project 1 — E-Commerce Microservices Platform
**Tech Stack:** Java 17, Spring Boot, PostgreSQL, Redis, Kafka, Docker, Kubernetes, AWS

**Services:**
```
├── user-service          (auth, JWT, user management)
├── product-service       (catalog, search, inventory)
├── order-service         (order lifecycle, saga pattern)
├── payment-service       (payment processing, idempotency)
├── notification-service  (email, SMS via Kafka events)
└── api-gateway           (Spring Cloud Gateway)
```

**Key features that impress interviewers:**
- Distributed transactions using Saga pattern
- Idempotency keys for payment APIs
- Rate limiting with Redis
- Full-text search with PostgreSQL
- Event-driven architecture with Kafka
- Docker Compose for local, Kubernetes manifests for prod
- CI/CD pipeline with GitHub Actions

**What to emphasize in interviews:** "I designed it to handle 10,000 orders/hour and implemented saga pattern to ensure eventual consistency across microservices."

---

#### Project 2 — Real-Time Chat Application
**Tech Stack:** Spring Boot, WebSockets (STOMP), Redis Pub/Sub, PostgreSQL, React.js

**Features:**
- 1-to-1 and group messaging
- Online status using Redis
- Message persistence in PostgreSQL
- Read receipts
- File sharing via S3
- JWT authentication

**Why this impresses:** Shows you can build real-time systems, understand WebSockets vs polling, and handle state at scale.

---

#### Project 3 — URL Shortener at Scale (System Design Project)
**Tech Stack:** Spring Boot, Redis, PostgreSQL, Docker, AWS

**Features:**
- Custom short URLs
- Analytics dashboard (click counts, locations)
- Rate limiting per API key
- Redirect with 301/302
- URL expiry
- REST API with OpenAPI docs (Swagger)

**Why this impresses:** Classic system design problem made real. Shows you know when to use Redis vs DB, understand URL hashing, and think about scale.

---

#### Project 4 (Optional, high impact) — Razorpay-style Payment Gateway
**Tech Stack:** Spring Boot, PostgreSQL, Redis, Kafka, Webhook system

**Features:**
- Payment initiation API
- Idempotency key handling
- Webhook delivery with retry logic
- Transaction history with pagination
- Refund processing
- Merchant dashboard

**Why this impresses:** Directly relevant to PhonePe, Razorpay, Paytm interviews.

---

### 🔵 Open Source Contribution Roadmap

**Month 9: Start contributing**

**Step 1: Find good repos**
- Search GitHub: `label:good-first-issue language:java`
- Spring Boot ecosystem: spring-boot, spring-data, spring-security
- Apache projects: Apache Commons, Apache Kafka (Java)
- Awesome Java list repos

**Step 2: Start small**
- Fix documentation, typos
- Write missing unit tests
- Fix "good first issue" bugs
- Add missing validation

**Step 3: Level up**
- Implement a small feature from issue tracker
- Review other PRs (gets you noticed)
- Start discussing in GitHub issues

**Month 10: Aim for 3–5 merged PRs**
- Even 1–2 PRs to well-known projects is impressive
- Write about it: "Contributed to Apache Commons / Spring Security"

---

## 📅 PHASE 6 — Interview Preparation + Job Hunt (Month 11–12)

### 🔵 Interview Preparation Plan

#### DSA Interview Prep (Month 11)
- Revise all 370+ problems categorized by pattern
- Do daily 2–3 LeetCode problems (don't stop)
- Focus on: Medium problems with time constraints (45 min)
- Practice explaining your approach BEFORE coding

**Coding Interview Framework:**
```
1. Read the problem fully (2 min)
2. Ask clarifying questions:
   - Input constraints? Sorted? Null safe?
   - Expected time/space complexity?
3. Think out loud — say your approach
4. Start with brute force, then optimize
5. Write clean code with good variable names
6. Test with edge cases: empty, single, duplicates
7. Analyze complexity: O(n) time, O(1) space
```

#### Behavioral Interview Prep (STAR Method)
**Questions to prepare for (Amazon Leadership Principles):**

```
1. Tell me about yourself (90-second pitch)
2. Why do you want to join [Company]?
3. Tell me about a challenging technical problem you solved
4. Tell me about a time you disagreed with your team
5. Tell me about your most impactful project
6. Tell me about a time you failed (and what you learned)
7. Tell me about a time you had to make a decision with incomplete info
8. How do you handle tight deadlines?
9. Tell me about a time you went beyond your job description
10. Where do you see yourself in 3 years?
```

**STAR Format:**
```
Situation: Set the context (2 sentences)
Task: What was your responsibility?
Action: What did YOU specifically do? (most important)
Result: Quantify the outcome — numbers matter
```

#### Technical Interview Topics to Revise:
```
Java:
- How does HashMap work internally (hashCode, equals, buckets)
- Difference between String, StringBuilder, StringBuffer
- Java memory model: Stack vs Heap
- Garbage Collection (G1GC, ZGC)
- Concurrency: synchronized, volatile, ReentrantLock
- Thread pool: ExecutorService, ThreadPoolExecutor
- Java 8: Stream API, Optional, CompletableFuture

Spring Boot:
- How Spring Boot auto-configuration works
- Bean scopes: singleton, prototype, request, session
- Spring Security filter chain
- Transaction management: @Transactional propagation
- N+1 problem and solutions
- JPA vs Hibernate vs JDBC

Database:
- ACID vs BASE
- Index types and when to use
- Query optimization
- CAP theorem
- Isolation levels and when to use each

System Design:
- 3 complete system designs: URL shortener, Twitter, Uber
- Trade-off discussions
- Scaling strategies
```

---

### 🔵 Resume Improvements

**One-page resume structure:**

```
[Full Name]
[Phone] | [Email] | [LinkedIn] | [GitHub] | [Portfolio]

SUMMARY (3 lines)
Full-Stack Developer with X years experience building scalable Java/Spring Boot
and React applications. Strong in distributed systems, REST API design, and
cloud deployment. Actively targeting backend/full-stack roles at product companies.

TECHNICAL SKILLS
Languages:    Java 17, JavaScript, SQL
Frameworks:   Spring Boot, React.js, React Native, NestJS
Databases:    PostgreSQL, MongoDB, Redis
Cloud/DevOps: AWS (EC2, S3, RDS), Docker, Kubernetes, CI/CD, Nginx
Tools:        Git, Maven, Kafka, WebSockets, Spring Security, JPA/Hibernate

EXPERIENCE
Company Name | Full-Stack Developer | Month Year – Present
• Built [feature] that [result] for [X users/clients]
• Reduced [metric] by X% by [what you did]
• Led [initiative] that delivered [outcome]
• Technologies: Spring Boot, React.js, PostgreSQL, AWS

PROJECTS (only your 3 best)
E-Commerce Microservices Platform | GitHub Link | Live Demo
• Architected 5-service system handling 10K orders/hour using Spring Boot + Kafka
• Implemented Saga pattern for distributed transactions across payment and order services
• Deployed on Kubernetes with auto-scaling, CI/CD via GitHub Actions

EDUCATION
Bachelor of Computer Applications | [University] | Year
```

**Resume rules:**
- Quantify EVERYTHING: "Improved performance by 40%", "Serving 50K+ users"
- Use action verbs: Built, Designed, Optimized, Led, Reduced, Implemented
- Never write "Worked on" or "Helped with"
- One page ONLY (you don't have 10 years of experience)
- ATS-friendly: no tables, no columns, no images
- Use these keywords: Spring Boot, Microservices, REST API, PostgreSQL, Redis, AWS, Docker, Kubernetes, System Design

---

### 🔵 GitHub Portfolio Improvements

**GitHub profile must-haves:**

**Profile README** (`username/username/README.md`):
```markdown
# Hi, I'm [Name] 👋

Full-Stack Developer | Java + Spring Boot | React.js | Open to SDE roles at product companies

## 🛠️ Tech Stack
Java · Spring Boot · React.js · PostgreSQL · Redis · Docker · AWS

## 🚀 Projects
- **E-Commerce Microservices** — Spring Boot + Kafka + Kubernetes
- **Real-Time Chat** — WebSockets + Redis Pub/Sub
- **URL Shortener at Scale** — Redis + PostgreSQL + AWS

## 📈 GitHub Stats
<!-- Add GitHub stats widget -->
```

**For each project repository:**
- Comprehensive README with: what it does, architecture diagram, setup instructions, API docs
- Architecture diagram (draw.io or Excalidraw — save as PNG)
- Demo GIF or screenshots
- `.env.example` file (never commit real credentials)
- Tests (unit + integration)
- CI/CD badge (GitHub Actions)
- OpenAPI/Swagger documentation

**Target:** 3–4 high-quality repos, not 20 empty ones

---

### 🔵 Job Application Strategy

**Month 9–10: Preparation**
- Polish resume and GitHub
- Set up LinkedIn profile (make it 100% complete)
- Connect with engineers at target companies
- Start applying to 2–3 "practice" companies per week

**Month 11–12: Full Application Mode**

**Target: 10–15 applications per week**

```
Application channels (in priority order):
1. Employee referrals         ← 60% of hires at big companies
2. LinkedIn (Easy Apply + manual)
3. Company career pages directly
4. Instahyre, Cutshort, AngelList (for Indian startups)
5. Naukri.com (for broader reach)
```

**Referral strategy:**
- LinkedIn: search "[Company Name] Software Engineer India"
- Message template: "Hi [Name], I'm a Full-Stack Developer applying to [Company]. I've been using your product and I'm particularly excited about [specific feature/team]. I'd love a referral or just 15 min to learn about your experience there. Here's my resume: [link]. No worries if not!"
- Send 5–10 referral requests per week

**Application tracking:**
Use a Google Sheet:
```
| Company | Role | Applied | Referral | Status | Next Step | Date |
```

**Weekly cadence:**
- Monday: Apply to 5 new roles, follow up on pending
- Tuesday: 2 hours LeetCode + 1 hour system design
- Wednesday: Apply to 5 more roles, mock interview
- Thursday: Review rejections, update resume if needed
- Friday: Reach out to 5 people for referrals
- Weekend: Project work, open source, learning

---

## 📆 Weekly Study Plan (All 12 Months)

**Weekdays (Mon–Fri):**
```
6:00–7:30 AM  → DSA (2 LeetCode problems)
7:30–9:00 AM  → [Topic of the month: Java / Spring Boot / System Design]
8:00 PM–9:30 PM → Review notes, read articles, watch 1 YouTube video
```

**Weekends:**
```
Saturday:
9:00 AM–12:00 PM → Project work / open source
2:00–5:00 PM     → System Design practice
5:00–6:00 PM     → Behavioral interview prep

Sunday:
10:00 AM–12:00 PM → Weekly review, plan next week
2:00–4:00 PM      → Solve 5+ LeetCode problems
4:00–5:00 PM      → Rest/reading
```

**Total weekly hours: ~20–25 hours (realistic and sustainable)**

---

## 📅 Daily Study Plan

**On workdays (you're employed, so be realistic):**

```
Morning (1.5 hours):
5:30 AM – Wake up
6:00–7:30 AM – DSA (2 problems) OR Java study

Evening (1.5 hours):
8:00–9:30 PM – Spring Boot / System Design / Projects

Before sleep (15 min):
Read 1 article: ByteByteGo newsletter, Java blog, engineering blog
```

**On weekends (4–5 hours/day):**
```
Morning session: 3 hours deep work (projects / hard topics)
Evening session: 2 hours DSA / system design
```

---

## 📅 Monthly Milestones

| Month | DSA Target | Java / Spring | Other |
|-------|-----------|---------------|-------|
| 1 | 60 problems (Easy focus) | Core Java, OOP, Collections | Setup dev environment |
| 2 | 110 problems (Linked Lists, Stacks) | Java 8 features, streams | LeetCode profile active |
| 3 | 170 problems (Trees) | Spring Boot REST, JPA | First CRUD API deployed |
| 4 | 220 problems (Graphs) | Spring Security, JWT | Auth service live on EC2 |
| 5 | 260 problems (DP intro) | PostgreSQL deep dive | Database schema designs |
| 6 | 290 problems (DP advanced) | Redis, System Design | 2 system designs mastered |
| 7 | 330 problems (Backtracking) | Docker advanced, K8s basics | Project 1 started |
| 8 | 370 problems (Advanced) | AWS deployment | Project 1 deployed |
| 9 | 390 problems (revision) | — | Project 2 complete |
| 10 | 400 problems (mock exams) | — | 3 projects live, resume ready |
| 11 | +20 (company-specific) | Full revision | 10–15 apps/week |
| 12 | Maintain streak | Interview mode | Target: 2–3 offers |

---

## ❌ Common Mistakes to Avoid

**DSA mistakes:**
- Grinding 700+ LeetCode problems without understanding patterns — quality > quantity
- Skipping Hard problems entirely — do 30–50 Hards, especially graph/DP ones
- Not practicing on a whiteboard or timer
- Not explaining your thought process while coding
- Memorizing solutions instead of patterns

**Java/Spring Boot mistakes:**
- Using field injection (`@Autowired` on fields) — use constructor injection
- Exposing JPA entities directly in API responses — always use DTOs
- Not handling exceptions globally — always use `@ControllerAdvice`
- Using `EAGER` fetch type — always `LAZY`
- Not writing tests — write at least unit tests for service layer
- Ignoring `EXPLAIN ANALYZE` for slow queries

**Job search mistakes:**
- Applying to 100 companies cold without referrals
- Generic cover letter/message (personalize every message)
- Applying before your resume/GitHub is polished
- Not negotiating the salary offer
- Accepting the first offer without interviewing elsewhere
- Not asking for feedback after rejections

**Learning mistakes:**
- Switching between too many languages (stick to Java for backend)
- Watching tutorials without coding along
- Not building real projects
- Learning Kubernetes before mastering Docker
- Trying to learn everything at once

---

## 💰 Expected Salary Progression

| Timeline | Role | Expected CTC |
|----------|------|-------------|
| Now (current) | Full-Stack Developer | 4–7 LPA |
| Month 6 | Stronger profile, better projects | Ready for 8–10 LPA interviews |
| Month 9 | Projects complete, 300+ LC | Start applying: 10–15 LPA |
| Month 12 | Full prep + 1–2 offers | 10–18 LPA target |
| Year 2 | 1 year at product company | 18–28 LPA |
| Year 3–4 | Senior Engineer | 25–45 LPA |

**Realistic targets by company (fresh from non-IIT background):**
```
Zomato / Swiggy      → 12–18 LPA
PhonePe / Razorpay   → 14–22 LPA
Walmart Global Tech  → 15–22 LPA
Atlassian            → 18–28 LPA
Amazon               → 20–35 LPA (SDE-II after 2 years)
```

**Negotiation tips:**
- Never give a number first — say "I'm open to competitive offers"
- Always have 2+ competing offers before negotiating
- Ask about joining bonus, stock options (ESOPs), variable pay
- At 10+ LPA, ESOP can be more valuable than base

---

## 🎯 Quick Reference: What Matters Most

**For getting the interview:**
- Resume with quantified impact
- GitHub with 3 good projects
- LinkedIn profile at 100%
- Referrals (highest conversion rate)

**For clearing the interview:**
- 300–400 LeetCode problems (pattern recognition, not memorization)
- System Design: 5 designs practiced end-to-end
- Java + Spring Boot fundamentals solid
- STAR stories for 10 behavioral questions

**Non-negotiable skills for 10+ LPA:**
1. DSA proficiency (LeetCode Medium ~75% success rate)
2. Java + Spring Boot
3. System Design basics
4. PostgreSQL + Redis
5. REST API best practices

**Optional (adds value but won't make or break):**
- Kubernetes
- Open source contributions
- React.js (you already have this)
- AWS certifications

**Completely ignore for now:**
- Golang, Rust, Kotlin
- Machine learning
- Mobile development (you already have React Native)
- Another JS framework
- Blockchain

---

## 📚 Resources Summary

| Category | Resource |
|----------|----------|
| DSA | LeetCode, NeetCode.io (free), "Cracking the Coding Interview" |
| Java | "Effective Java" by Joshua Bloch (must read), Baeldung.com |
| Spring Boot | Official docs, Amigoscode (YouTube), Java Brains (YouTube) |
| System Design | "System Design Interview" by Alex Xu, Gaurav Sen (YouTube), ByteByteGo |
| PostgreSQL | Official docs, "Learning PostgreSQL" book, pgexercises.com |
| Redis | Official docs, Redis University (free) |
| Docker/K8s | official docs, TechWorld with Nana (YouTube) |
| AWS | AWS free tier + hands-on, Adrian Cantrill course |
| Behavioral | "Grokking the Behavioral Interview" |

---

*Built for: BCA Graduate, Full-Stack Developer → 10+ LPA at Product Company in 12 months*  
*Last updated: 2025 | All salaries in INR CTC*
