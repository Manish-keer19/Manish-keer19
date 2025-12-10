# 📚 Complete JavaScript Array Methods Guide
## With React & Node.js Real-World Examples

---

## 🎯 **1. map()** - Transform Each Item

### **What it does:**
Creates a new array by transforming each element.

### **Basic Example:**
```javascript
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6, 8, 10]
```

### **React Example - Rendering Lists:**
```tsx
// Display list of users
function UserList() {
    const users = [
        { id: 1, name: 'John', email: 'john@example.com' },
        { id: 2, name: 'Jane', email: 'jane@example.com' },
        { id: 3, name: 'Bob', email: 'bob@example.com' }
    ];

    return (
        <div>
            {users.map(user => (
                <div key={user.id} className="user-card">
                    <h3>{user.name}</h3>
                    <p>{user.email}</p>
                </div>
            ))}
        </div>
    );
}
```

### **Node.js Example - API Response Formatting:**
```typescript
// Format database results for API response
app.get('/api/products', async (req, res) => {
    const products = await prisma.product.findMany();
    
    // Transform to include only necessary fields
    const formattedProducts = products.map(product => ({
        id: product.id,
        name: product.name,
        price: `$${product.price.toFixed(2)}`,
        inStock: product.quantity > 0
    }));
    
    res.json(formattedProducts);
});
```

---

## 🎯 **2. filter()** - Select Items

### **What it does:**
Creates a new array with items that pass a test.

### **Basic Example:**
```javascript
const numbers = [1, 2, 3, 4, 5, 6];
const evens = numbers.filter(num => num % 2 === 0);
console.log(evens); // [2, 4, 6]
```

### **React Example - Search/Filter Functionality:**
```tsx
function ProductList() {
    const [searchTerm, setSearchTerm] = useState('');
    const [products] = useState([
        { id: 1, name: 'Laptop', category: 'Electronics', price: 999 },
        { id: 2, name: 'Shirt', category: 'Clothing', price: 29 },
        { id: 3, name: 'Phone', category: 'Electronics', price: 699 }
    ]);

    // Filter products based on search term
    const filteredProducts = products.filter(product =>
        product.name.toLowerCase().includes(searchTerm.toLowerCase())
    );

    return (
        <div>
            <input
                type="text"
                placeholder="Search products..."
                value={searchTerm}
                onChange={(e) => setSearchTerm(e.target.value)}
            />
            {filteredProducts.map(product => (
                <div key={product.id}>{product.name} - ${product.price}</div>
            ))}
        </div>
    );
}
```

### **Node.js Example - Filter Query Results:**
```typescript
// Filter users by role and active status
app.get('/api/users', async (req, res) => {
    const { role, activeOnly } = req.query;
    
    let users = await prisma.user.findMany();
    
    // Filter by role if provided
    if (role) {
        users = users.filter(user => user.role === role);
    }
    
    // Filter active users only
    if (activeOnly === 'true') {
        users = users.filter(user => user.isActive);
    }
    
    res.json(users);
});
```

---

## 🎯 **3. reduce()** - Accumulate/Combine

### **What it does:**
Reduces array to a single value by accumulating.

### **Basic Example:**
```javascript
const numbers = [1, 2, 3, 4, 5];
const sum = numbers.reduce((total, num) => total + num, 0);
console.log(sum); // 15
```

### **React Example - Calculate Cart Total:**
```tsx
function ShoppingCart() {
    const [cartItems] = useState([
        { id: 1, name: 'Laptop', price: 999, quantity: 1 },
        { id: 2, name: 'Mouse', price: 29, quantity: 2 },
        { id: 3, name: 'Keyboard', price: 79, quantity: 1 }
    ]);

    // Calculate total price
    const totalPrice = cartItems.reduce((total, item) => {
        return total + (item.price * item.quantity);
    }, 0);

    // Group items by category
    const itemsByCategory = cartItems.reduce((acc, item) => {
        const category = item.category || 'Other';
        if (!acc[category]) acc[category] = [];
        acc[category].push(item);
        return acc;
    }, {});

    return (
        <div>
            <h2>Cart Total: ${totalPrice.toFixed(2)}</h2>
            {cartItems.map(item => (
                <div key={item.id}>
                    {item.name} x {item.quantity} = ${item.price * item.quantity}
                </div>
            ))}
        </div>
    );
}
```

### **Node.js Example - Aggregate Statistics:**
```typescript
// Calculate statistics from data
app.get('/api/stats', async (req, res) => {
    const orders = await prisma.order.findMany({
        include: { items: true }
    });

    const stats = orders.reduce((acc, order) => {
        // Total revenue
        acc.totalRevenue += order.total;
        
        // Count by status
        acc.ordersByStatus[order.status] = (acc.ordersByStatus[order.status] || 0) + 1;
        
        // Average order value
        acc.orderCount++;
        
        return acc;
    }, {
        totalRevenue: 0,
        ordersByStatus: {},
        orderCount: 0
    });

    stats.averageOrderValue = stats.totalRevenue / stats.orderCount;

    res.json(stats);
});
```

---

## 🎯 **4. find()** - Find First Match

### **What it does:**
Returns the first element that matches the condition.

### **Basic Example:**
```javascript
const users = [
    { id: 1, name: 'John' },
    { id: 2, name: 'Jane' },
    { id: 3, name: 'Bob' }
];
const user = users.find(u => u.id === 2);
console.log(user); // { id: 2, name: 'Jane' }
```

### **React Example - Find and Display User:**
```tsx
function UserProfile({ userId }) {
    const [users] = useState([
        { id: 1, name: 'John', email: 'john@example.com', role: 'admin' },
        { id: 2, name: 'Jane', email: 'jane@example.com', role: 'user' },
        { id: 3, name: 'Bob', email: 'bob@example.com', role: 'user' }
    ]);

    // Find specific user
    const currentUser = users.find(user => user.id === userId);

    if (!currentUser) {
        return <div>User not found</div>;
    }

    return (
        <div>
            <h2>{currentUser.name}</h2>
            <p>Email: {currentUser.email}</p>
            <p>Role: {currentUser.role}</p>
        </div>
    );
}
```

### **Node.js Example - Find User by Email:**
```typescript
// Login endpoint
app.post('/api/login', async (req, res) => {
    const { email, password } = req.body;
    
    const users = await prisma.user.findMany();
    
    // Find user by email
    const user = users.find(u => u.email === email);
    
    if (!user) {
        return res.status(404).json({ error: 'User not found' });
    }
    
    // Verify password
    const isValid = await bcrypt.compare(password, user.password);
    
    if (!isValid) {
        return res.status(401).json({ error: 'Invalid password' });
    }
    
    res.json({ token: generateToken(user) });
});
```

---

## 🎯 **5. forEach()** - Loop Through Each Item

### **What it does:**
Executes a function for each element (no return value).

### **Basic Example:**
```javascript
const numbers = [1, 2, 3];
numbers.forEach(num => console.log(num * 2));
// Output: 2, 4, 6
```

### **React Example - Side Effects:**
```tsx
function NotificationManager() {
    const [notifications] = useState([
        { id: 1, message: 'New message', read: false },
        { id: 2, message: 'Update available', read: false }
    ]);

    useEffect(() => {
        // Mark all as read after 3 seconds
        const timer = setTimeout(() => {
            notifications.forEach(notification => {
                markAsRead(notification.id);
            });
        }, 3000);

        return () => clearTimeout(timer);
    }, []);

    return (
        <div>
            {notifications.map(notif => (
                <div key={notif.id}>{notif.message}</div>
            ))}
        </div>
    );
}
```

### **Node.js Example - Batch Operations:**
```typescript
// Send emails to all users
app.post('/api/send-newsletter', async (req, res) => {
    const { subject, content } = req.body;
    const users = await prisma.user.findMany({
        where: { subscribed: true }
    });

    const results = [];

    // Send email to each user
    users.forEach(async (user) => {
        try {
            await sendEmail({
                to: user.email,
                subject: subject,
                html: content
            });
            results.push({ email: user.email, status: 'sent' });
        } catch (error) {
            results.push({ email: user.email, status: 'failed' });
        }
    });

    res.json({ message: 'Emails sent', results });
});
```

---

## 🎯 **6. some()** - Test If Any Match

### **What it does:**
Returns true if at least one element passes the test.

### **Basic Example:**
```javascript
const numbers = [1, 2, 3, 4, 5];
const hasEven = numbers.some(num => num % 2 === 0);
console.log(hasEven); // true
```

### **React Example - Form Validation:**
```tsx
function RegistrationForm() {
    const [formData, setFormData] = useState({
        username: '',
        email: '',
        password: ''
    });

    const [errors, setErrors] = useState([]);

    const validateForm = () => {
        const newErrors = [];

        if (formData.username.length < 3) {
            newErrors.push('Username too short');
        }
        if (!formData.email.includes('@')) {
            newErrors.push('Invalid email');
        }
        if (formData.password.length < 8) {
            newErrors.push('Password too short');
        }

        setErrors(newErrors);

        // Check if there are any errors
        const hasErrors = newErrors.some(error => error.length > 0);
        return !hasErrors;
    };

    const handleSubmit = (e) => {
        e.preventDefault();
        if (validateForm()) {
            // Submit form
            console.log('Form is valid!');
        }
    };

    return (
        <form onSubmit={handleSubmit}>
            {/* Form fields */}
            {errors.length > 0 && (
                <div className="errors">
                    {errors.map((error, i) => <p key={i}>{error}</p>)}
                </div>
            )}
        </form>
    );
}
```

### **Node.js Example - Permission Check:**
```typescript
// Check if user has any admin permission
app.get('/api/admin/dashboard', async (req, res) => {
    const userId = req.user.id;
    const user = await prisma.user.findUnique({
        where: { id: userId },
        include: { permissions: true }
    });

    const adminPermissions = ['admin', 'super_admin', 'moderator'];
    
    // Check if user has any admin permission
    const isAdmin = user.permissions.some(perm =>
        adminPermissions.includes(perm.name)
    );

    if (!isAdmin) {
        return res.status(403).json({ error: 'Access denied' });
    }

    res.json({ message: 'Welcome to admin dashboard' });
});
```

---

## 🎯 **7. every()** - Test If All Match

### **What it does:**
Returns true if ALL elements pass the test.

### **Basic Example:**
```javascript
const numbers = [2, 4, 6, 8];
const allEven = numbers.every(num => num % 2 === 0);
console.log(allEven); // true
```

### **React Example - Check All Items Selected:**
```tsx
function TodoList() {
    const [todos, setTodos] = useState([
        { id: 1, text: 'Buy groceries', completed: false },
        { id: 2, text: 'Clean house', completed: false },
        { id: 3, text: 'Pay bills', completed: false }
    ]);

    // Check if all todos are completed
    const allCompleted = todos.every(todo => todo.completed);

    const toggleAll = () => {
        setTodos(todos.map(todo => ({
            ...todo,
            completed: !allCompleted
        })));
    };

    return (
        <div>
            <button onClick={toggleAll}>
                {allCompleted ? 'Uncheck All' : 'Check All'}
            </button>
            {allCompleted && <p>🎉 All tasks completed!</p>}
            {todos.map(todo => (
                <div key={todo.id}>
                    <input
                        type="checkbox"
                        checked={todo.completed}
                        onChange={() => toggleTodo(todo.id)}
                    />
                    {todo.text}
                </div>
            ))}
        </div>
    );
}
```

### **Node.js Example - Validate All Required Fields:**
```typescript
// Check if all required fields are present
app.post('/api/create-order', async (req, res) => {
    const requiredFields = ['customerId', 'items', 'shippingAddress', 'paymentMethod'];
    
    // Check if all required fields exist
    const allFieldsPresent = requiredFields.every(field =>
        req.body[field] !== undefined && req.body[field] !== null
    );

    if (!allFieldsPresent) {
        return res.status(400).json({
            error: 'Missing required fields',
            required: requiredFields
        });
    }

    // Check if all items have valid quantities
    const allItemsValid = req.body.items.every(item =>
        item.quantity > 0 && item.productId
    );

    if (!allItemsValid) {
        return res.status(400).json({ error: 'Invalid items in order' });
    }

    const order = await prisma.order.create({ data: req.body });
    res.json(order);
});
```

---

## 🎯 **8. sort()** - Sort Array

### **What it does:**
Sorts array in place.

### **Basic Example:**
```javascript
const numbers = [3, 1, 4, 1, 5, 9];
numbers.sort((a, b) => a - b); // Ascending
console.log(numbers); // [1, 1, 3, 4, 5, 9]
```

### **React Example - Sortable Table:**
```tsx
function ProductTable() {
    const [products, setProducts] = useState([
        { id: 1, name: 'Laptop', price: 999, rating: 4.5 },
        { id: 2, name: 'Mouse', price: 29, rating: 4.8 },
        { id: 3, name: 'Keyboard', price: 79, rating: 4.2 }
    ]);
    const [sortBy, setSortBy] = useState('name');
    const [sortOrder, setSortOrder] = useState('asc');

    const sortedProducts = [...products].sort((a, b) => {
        const aValue = a[sortBy];
        const bValue = b[sortBy];

        if (sortOrder === 'asc') {
            return aValue > bValue ? 1 : -1;
        } else {
            return aValue < bValue ? 1 : -1;
        }
    });

    return (
        <div>
            <button onClick={() => setSortBy('name')}>Sort by Name</button>
            <button onClick={() => setSortBy('price')}>Sort by Price</button>
            <button onClick={() => setSortBy('rating')}>Sort by Rating</button>
            <button onClick={() => setSortOrder(sortOrder === 'asc' ? 'desc' : 'asc')}>
                {sortOrder === 'asc' ? '↑' : '↓'}
            </button>

            <table>
                <tbody>
                    {sortedProducts.map(product => (
                        <tr key={product.id}>
                            <td>{product.name}</td>
                            <td>${product.price}</td>
                            <td>⭐ {product.rating}</td>
                        </tr>
                    ))}
                </tbody>
            </table>
        </div>
    );
}
```

### **Node.js Example - Sort Query Results:**
```typescript
// Get products sorted by various criteria
app.get('/api/products', async (req, res) => {
    const { sortBy = 'createdAt', order = 'desc' } = req.query;
    
    let products = await prisma.product.findMany();

    // Sort based on query parameters
    products.sort((a, b) => {
        let aValue = a[sortBy];
        let bValue = b[sortBy];

        // Handle string comparison
        if (typeof aValue === 'string') {
            aValue = aValue.toLowerCase();
            bValue = bValue.toLowerCase();
        }

        if (order === 'asc') {
            return aValue > bValue ? 1 : -1;
        } else {
            return aValue < bValue ? 1 : -1;
        }
    });

    res.json(products);
});
```

---

## 🎯 **9. slice()** - Extract Portion

### **What it does:**
Returns a shallow copy of a portion of array.

### **Basic Example:**
```javascript
const numbers = [1, 2, 3, 4, 5];
const sliced = numbers.slice(1, 4);
console.log(sliced); // [2, 3, 4]
```

### **React Example - Pagination:**
```tsx
function PaginatedList() {
    const [currentPage, setCurrentPage] = useState(1);
    const itemsPerPage = 5;

    const allItems = [
        { id: 1, name: 'Item 1' },
        { id: 2, name: 'Item 2' },
        // ... 20 items total
    ];

    // Calculate slice indices
    const startIndex = (currentPage - 1) * itemsPerPage;
    const endIndex = startIndex + itemsPerPage;

    // Get items for current page
    const currentItems = allItems.slice(startIndex, endIndex);

    const totalPages = Math.ceil(allItems.length / itemsPerPage);

    return (
        <div>
            {currentItems.map(item => (
                <div key={item.id}>{item.name}</div>
            ))}

            <div className="pagination">
                <button
                    disabled={currentPage === 1}
                    onClick={() => setCurrentPage(currentPage - 1)}
                >
                    Previous
                </button>
                <span>Page {currentPage} of {totalPages}</span>
                <button
                    disabled={currentPage === totalPages}
                    onClick={() => setCurrentPage(currentPage + 1)}
                >
                    Next
                </button>
            </div>
        </div>
    );
}
```

### **Node.js Example - Paginated API:**
```typescript
// Paginated endpoint
app.get('/api/posts', async (req, res) => {
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 10;

    const allPosts = await prisma.post.findMany({
        orderBy: { createdAt: 'desc' }
    });

    // Calculate pagination
    const startIndex = (page - 1) * limit;
    const endIndex = startIndex + limit;

    // Slice the results
    const paginatedPosts = allPosts.slice(startIndex, endIndex);

    res.json({
        page,
        limit,
        total: allPosts.length,
        totalPages: Math.ceil(allPosts.length / limit),
        data: paginatedPosts
    });
});
```

---

## 🎯 **10. concat()** - Merge Arrays

### **What it does:**
Combines arrays into a new array.

### **Basic Example:**
```javascript
const arr1 = [1, 2];
const arr2 = [3, 4];
const combined = arr1.concat(arr2);
console.log(combined); // [1, 2, 3, 4]
```

### **React Example - Combine Multiple Data Sources:**
```tsx
function CombinedFeed() {
    const [posts, setPosts] = useState([
        { id: 1, type: 'post', content: 'Hello', timestamp: '2024-01-01' }
    ]);
    const [comments, setComments] = useState([
        { id: 2, type: 'comment', content: 'Nice!', timestamp: '2024-01-02' }
    ]);
    const [likes, setLikes] = useState([
        { id: 3, type: 'like', content: 'User liked', timestamp: '2024-01-03' }
    ]);

    // Combine all activities
    const allActivities = posts.concat(comments, likes);

    // Sort by timestamp
    const sortedActivities = allActivities.sort((a, b) =>
        new Date(b.timestamp) - new Date(a.timestamp)
    );

    return (
        <div>
            <h2>Activity Feed</h2>
            {sortedActivities.map(activity => (
                <div key={activity.id} className={`activity-${activity.type}`}>
                    {activity.content}
                </div>
            ))}
        </div>
    );
}
```

### **Node.js Example - Merge Multiple Query Results:**
```typescript
// Get combined results from multiple sources
app.get('/api/search', async (req, res) => {
    const { query } = req.query;

    // Search in multiple tables
    const users = await prisma.user.findMany({
        where: { name: { contains: query } }
    });

    const products = await prisma.product.findMany({
        where: { name: { contains: query } }
    });

    const posts = await prisma.post.findMany({
        where: { title: { contains: query } }
    });

    // Add type to each result
    const usersWithType = users.map(u => ({ ...u, type: 'user' }));
    const productsWithType = products.map(p => ({ ...p, type: 'product' }));
    const postsWithType = posts.map(p => ({ ...p, type: 'post' }));

    // Combine all results
    const allResults = usersWithType.concat(productsWithType, postsWithType);

    res.json({
        query,
        totalResults: allResults.length,
        results: allResults
    });
});
```

---

## 📊 Quick Reference - When to Use Each Method

| Method | React Use Case | Node.js Use Case |
|--------|---------------|------------------|
| **map()** | Render lists, transform data for display | Format API responses, transform DB results |
| **filter()** | Search/filter UI, conditional rendering | Filter query results, validate data |
| **reduce()** | Calculate totals, group data | Aggregate statistics, combine data |
| **find()** | Find specific item to display | Find user/record by ID |
| **forEach()** | Side effects, logging | Batch operations, send emails |
| **some()** | Check if any condition met | Permission checks, validation |
| **every()** | Check if all conditions met | Validate all fields, check completion |
| **sort()** | Sortable tables/lists | Sort query results |
| **slice()** | Pagination, "Load More" | Paginated APIs |
| **concat()** | Combine multiple data sources | Merge query results |

---

## 🎯 Most Common Patterns

### **1. Map + Filter (React):**
```tsx
// Show only active users
{users
    .filter(user => user.isActive)
    .map(user => <UserCard key={user.id} user={user} />)
}
```

### **2. Filter + Reduce (Node.js):**
```typescript
// Calculate total revenue from completed orders
const totalRevenue = orders
    .filter(order => order.status === 'completed')
    .reduce((sum, order) => sum + order.total, 0);
```

### **3. Sort + Slice (React Pagination):**
```tsx
// Show top 10 products by rating
const topProducts = products
    .sort((a, b) => b.rating - a.rating)
    .slice(0, 10);
```

---

## 🚀 Performance Tips

1. **Avoid chaining too many methods** - Each creates a new array
2. **Use `for` loop for large datasets** if performance is critical
3. **Don't use `map()` if you don't need the result** - Use `forEach()` instead
4. **Cache sorted/filtered results** in React using `useMemo()`

```tsx
const sortedData = useMemo(() => {
    return data.sort((a, b) => a.value - b.value);
}, [data]);
```

---

## ✅ Summary

**Master these 5 for 90% of your needs:**
1. **map()** - Transform/render data
2. **filter()** - Select/search data
3. **reduce()** - Aggregate/combine data
4. **find()** - Find specific items
5. **sort()** - Order data

**You're now ready to handle any array manipulation in React and Node.js!** 🎉
