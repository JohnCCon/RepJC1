# Examples and Tutorials

This document provides practical examples and tutorials for using RepJC1 APIs, functions, and components.

## Table of Contents

- [Getting Started](#getting-started)
- [Basic Examples](#basic-examples)
- [Advanced Examples](#advanced-examples)
- [Real-World Scenarios](#real-world-scenarios)
- [Best Practices](#best-practices)
- [Common Patterns](#common-patterns)
- [Troubleshooting](#troubleshooting)

## Getting Started

### Quick Start Example

Here's a complete example of setting up a basic RepJC1 application:

```jsx
// App.tsx
import React from 'react';
import { 
  Button, 
  Container, 
  Grid, 
  Card,
  Navbar 
} from '@/components';
import { useLocalStorage } from '@/hooks';
import { formatDate } from '@/utils';

function App() {
  const [user, setUser] = useLocalStorage('currentUser', null);
  const [theme, setTheme] = useLocalStorage('theme', 'light');

  const handleLogin = () => {
    setUser({ 
      id: 1, 
      name: 'John Doe', 
      email: 'john@example.com',
      loginTime: new Date()
    });
  };

  const handleLogout = () => {
    setUser(null);
  };

  const toggleTheme = () => {
    setTheme(theme === 'light' ? 'dark' : 'light');
  };

  return (
    <div className={`app theme-${theme}`}>
      <Navbar
        brand={<h1>RepJC1 Demo</h1>}
        actions={
          <div>
            <Button onClick={toggleTheme}>
              {theme === 'light' ? '🌙' : '☀️'}
            </Button>
            {user ? (
              <Button onClick={handleLogout}>
                Logout ({user.name})
              </Button>
            ) : (
              <Button variant="primary" onClick={handleLogin}>
                Login
              </Button>
            )}
          </div>
        }
      />

      <Container maxWidth="lg">
        <Grid container spacing={3}>
          <Grid item xs={12}>
            <Card title="Welcome to RepJC1">
              {user ? (
                <div>
                  <p>Hello, {user.name}!</p>
                  <p>Last login: {formatDate(user.loginTime, 'MMMM Do, YYYY [at] HH:mm')}</p>
                </div>
              ) : (
                <p>Please log in to get started.</p>
              )}
            </Card>
          </Grid>
        </Grid>
      </Container>
    </div>
  );
}

export default App;
```

### Project Setup

```bash
# Install dependencies
npm install

# Start development server
npm start

# Run tests
npm test

# Build for production
npm run build
```

## Basic Examples

### 1. User Management with API

```jsx
// UserManager.tsx
import React, { useState, useEffect } from 'react';
import { 
  Table, 
  Button, 
  Modal, 
  Form, 
  Input, 
  Loading,
  Card 
} from '@/components';
import { makeRequest } from '@/api';
import { debounce } from '@/utils';

interface User {
  id: number;
  name: string;
  email: string;
  status: 'active' | 'inactive';
  createdAt: string;
}

export function UserManager() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(false);
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [editingUser, setEditingUser] = useState<User | null>(null);
  const [searchQuery, setSearchQuery] = useState('');

  // Debounced search function
  const debouncedSearch = debounce(async (query: string) => {
    setLoading(true);
    try {
      const searchUsers = await makeRequest<User[]>('/api/users', {
        method: 'GET',
        data: { search: query }
      });
      setUsers(searchUsers);
    } catch (error) {
      console.error('Search failed:', error);
    } finally {
      setLoading(false);
    }
  }, 300);

  // Load users on component mount
  useEffect(() => {
    loadUsers();
  }, []);

  // Search when query changes
  useEffect(() => {
    if (searchQuery) {
      debouncedSearch(searchQuery);
    } else {
      loadUsers();
    }
  }, [searchQuery]);

  const loadUsers = async () => {
    setLoading(true);
    try {
      const userData = await makeRequest<User[]>('/api/users');
      setUsers(userData);
    } catch (error) {
      console.error('Failed to load users:', error);
    } finally {
      setLoading(false);
    }
  };

  const handleCreateUser = () => {
    setEditingUser(null);
    setIsModalOpen(true);
  };

  const handleEditUser = (user: User) => {
    setEditingUser(user);
    setIsModalOpen(true);
  };

  const handleDeleteUser = async (userId: number) => {
    if (confirm('Are you sure you want to delete this user?')) {
      try {
        await makeRequest(`/api/users/${userId}`, { method: 'DELETE' });
        await loadUsers(); // Refresh the list
      } catch (error) {
        console.error('Failed to delete user:', error);
      }
    }
  };

  const handleSaveUser = async (userData: Partial<User>) => {
    try {
      if (editingUser) {
        // Update existing user
        await makeRequest(`/api/users/${editingUser.id}`, {
          method: 'PUT',
          data: userData
        });
      } else {
        // Create new user
        await makeRequest('/api/users', {
          method: 'POST',
          data: userData
        });
      }
      
      setIsModalOpen(false);
      await loadUsers(); // Refresh the list
    } catch (error) {
      console.error('Failed to save user:', error);
    }
  };

  const columns = [
    { key: 'name', title: 'Name', sortable: true },
    { key: 'email', title: 'Email', filterable: true },
    { 
      key: 'status', 
      title: 'Status',
      render: (status: string) => (
        <span className={`status-badge status-${status}`}>
          {status}
        </span>
      )
    },
    {
      key: 'actions',
      title: 'Actions',
      render: (_, user: User) => (
        <div className="action-buttons">
          <Button 
            size="small" 
            onClick={() => handleEditUser(user)}
          >
            Edit
          </Button>
          <Button 
            size="small" 
            variant="danger"
            onClick={() => handleDeleteUser(user.id)}
          >
            Delete
          </Button>
        </div>
      )
    }
  ];

  return (
    <Card title="User Management">
      <div className="user-manager">
        <div className="toolbar">
          <Input
            placeholder="Search users..."
            value={searchQuery}
            onChange={(e) => setSearchQuery(e.target.value)}
            icon={<SearchIcon />}
          />
          <Button variant="primary" onClick={handleCreateUser}>
            Add User
          </Button>
        </div>

        {loading ? (
          <Loading text="Loading users..." />
        ) : (
          <Table
            data={users}
            columns={columns}
            onRowClick={(user) => handleEditUser(user)}
          />
        )}

        <Modal
          isOpen={isModalOpen}
          onClose={() => setIsModalOpen(false)}
          title={editingUser ? 'Edit User' : 'Create User'}
        >
          <UserForm
            user={editingUser}
            onSave={handleSaveUser}
            onCancel={() => setIsModalOpen(false)}
          />
        </Modal>
      </div>
    </Card>
  );
}

// UserForm component
function UserForm({ 
  user, 
  onSave, 
  onCancel 
}: {
  user: User | null;
  onSave: (userData: Partial<User>) => void;
  onCancel: () => void;
}) {
  const handleSubmit = (data: any) => {
    onSave(data);
  };

  return (
    <Form
      onSubmit={handleSubmit}
      initialValues={{
        name: user?.name || '',
        email: user?.email || '',
        status: user?.status || 'active'
      }}
    >
      <Input name="name" label="Full Name" required />
      <Input name="email" label="Email Address" type="email" required />
      <Select
        name="status"
        label="Status"
        options={[
          { value: 'active', label: 'Active' },
          { value: 'inactive', label: 'Inactive' }
        ]}
      />
      
      <div className="form-actions">
        <Button type="button" onClick={onCancel}>
          Cancel
        </Button>
        <Button type="submit" variant="primary">
          {user ? 'Update' : 'Create'} User
        </Button>
      </div>
    </Form>
  );
}
```

### 2. Data Visualization Dashboard

```jsx
// Dashboard.tsx
import React, { useState, useEffect } from 'react';
import { 
  Grid, 
  Card, 
  Loading,
  Select,
  Container 
} from '@/components';
import { 
  formatCurrency, 
  formatDate, 
  groupBy,
  sortBy 
} from '@/utils';
import { makeRequest } from '@/api';

interface DashboardData {
  totalRevenue: number;
  totalOrders: number;
  averageOrderValue: number;
  topProducts: Product[];
  recentOrders: Order[];
  salesByCategory: CategorySales[];
}

interface Product {
  id: number;
  name: string;
  sales: number;
  revenue: number;
}

interface Order {
  id: number;
  customerName: string;
  amount: number;
  status: string;
  createdAt: string;
}

interface CategorySales {
  category: string;
  sales: number;
  revenue: number;
}

export function Dashboard() {
  const [data, setData] = useState<DashboardData | null>(null);
  const [loading, setLoading] = useState(true);
  const [timeRange, setTimeRange] = useState('7d');
  const [refreshInterval, setRefreshInterval] = useState<NodeJS.Timeout | null>(null);

  useEffect(() => {
    loadDashboardData();
    
    // Set up auto-refresh every 5 minutes
    const interval = setInterval(loadDashboardData, 5 * 60 * 1000);
    setRefreshInterval(interval);

    return () => {
      if (refreshInterval) {
        clearInterval(refreshInterval);
      }
    };
  }, [timeRange]);

  const loadDashboardData = async () => {
    setLoading(true);
    try {
      const dashboardData = await makeRequest<DashboardData>('/api/dashboard', {
        method: 'GET',
        data: { timeRange }
      });
      setData(dashboardData);
    } catch (error) {
      console.error('Failed to load dashboard data:', error);
    } finally {
      setLoading(false);
    }
  };

  const handleTimeRangeChange = (newTimeRange: string) => {
    setTimeRange(newTimeRange);
  };

  if (loading && !data) {
    return <Loading text="Loading dashboard..." />;
  }

  return (
    <Container maxWidth="xl">
      <div className="dashboard">
        <div className="dashboard-header">
          <h1>Sales Dashboard</h1>
          <Select
            options={[
              { value: '1d', label: 'Last 24 Hours' },
              { value: '7d', label: 'Last 7 Days' },
              { value: '30d', label: 'Last 30 Days' },
              { value: '90d', label: 'Last 90 Days' }
            ]}
            value={timeRange}
            onChange={handleTimeRangeChange}
          />
        </div>

        {loading && (
          <div className="loading-overlay">
            <Loading text="Updating..." />
          </div>
        )}

        <Grid container spacing={3}>
          {/* Key Metrics */}
          <Grid item xs={12} md={4}>
            <MetricCard
              title="Total Revenue"
              value={formatCurrency(data?.totalRevenue || 0)}
              trend="+12.5%"
              positive
            />
          </Grid>
          <Grid item xs={12} md={4}>
            <MetricCard
              title="Total Orders"
              value={data?.totalOrders?.toLocaleString() || '0'}
              trend="+8.2%"
              positive
            />
          </Grid>
          <Grid item xs={12} md={4}>
            <MetricCard
              title="Avg Order Value"
              value={formatCurrency(data?.averageOrderValue || 0)}
              trend="-2.1%"
              positive={false}
            />
          </Grid>

          {/* Top Products */}
          <Grid item xs={12} md={6}>
            <Card title="Top Products">
              <ProductList products={data?.topProducts || []} />
            </Card>
          </Grid>

          {/* Sales by Category */}
          <Grid item xs={12} md={6}>
            <Card title="Sales by Category">
              <CategoryChart data={data?.salesByCategory || []} />
            </Card>
          </Grid>

          {/* Recent Orders */}
          <Grid item xs={12}>
            <Card title="Recent Orders">
              <OrderList orders={data?.recentOrders || []} />
            </Card>
          </Grid>
        </Grid>
      </div>
    </Container>
  );
}

// Supporting components
function MetricCard({ 
  title, 
  value, 
  trend, 
  positive 
}: {
  title: string;
  value: string;
  trend: string;
  positive: boolean;
}) {
  return (
    <Card padding="large">
      <div className="metric-card">
        <h3 className="metric-title">{title}</h3>
        <div className="metric-value">{value}</div>
        <div className={`metric-trend ${positive ? 'positive' : 'negative'}`}>
          {trend}
        </div>
      </div>
    </Card>
  );
}

function ProductList({ products }: { products: Product[] }) {
  const sortedProducts = sortBy(products, 'revenue', 'desc');

  return (
    <div className="product-list">
      {sortedProducts.map((product) => (
        <div key={product.id} className="product-item">
          <div className="product-info">
            <div className="product-name">{product.name}</div>
            <div className="product-sales">{product.sales} units sold</div>
          </div>
          <div className="product-revenue">
            {formatCurrency(product.revenue)}
          </div>
        </div>
      ))}
    </div>
  );
}

function CategoryChart({ data }: { data: CategorySales[] }) {
  const maxRevenue = Math.max(...data.map(item => item.revenue));

  return (
    <div className="category-chart">
      {data.map((item) => (
        <div key={item.category} className="category-bar">
          <div className="category-label">{item.category}</div>
          <div className="category-bar-container">
            <div 
              className="category-bar-fill"
              style={{ width: `${(item.revenue / maxRevenue) * 100}%` }}
            />
          </div>
          <div className="category-value">
            {formatCurrency(item.revenue)}
          </div>
        </div>
      ))}
    </div>
  );
}

function OrderList({ orders }: { orders: Order[] }) {
  const recentOrders = sortBy(orders, 'createdAt', 'desc').slice(0, 10);

  return (
    <div className="order-list">
      {recentOrders.map((order) => (
        <div key={order.id} className="order-item">
          <div className="order-info">
            <div className="order-id">#{order.id}</div>
            <div className="order-customer">{order.customerName}</div>
            <div className="order-date">
              {formatDate(order.createdAt, 'MMM DD, YYYY [at] HH:mm')}
            </div>
          </div>
          <div className="order-details">
            <div className="order-amount">
              {formatCurrency(order.amount)}
            </div>
            <div className={`order-status status-${order.status}`}>
              {order.status}
            </div>
          </div>
        </div>
      ))}
    </div>
  );
}
```

### 3. File Upload with Progress

```jsx
// FileUploader.tsx
import React, { useState, useCallback } from 'react';
import { 
  Card, 
  Button, 
  Loading,
  Modal 
} from '@/components';
import { 
  validateFileType, 
  readFileAsText,
  downloadFile 
} from '@/utils';
import { makeRequest } from '@/api';

interface UploadedFile {
  id: string;
  name: string;
  size: number;
  type: string;
  url: string;
  uploadProgress: number;
  status: 'uploading' | 'completed' | 'error';
}

export function FileUploader() {
  const [files, setFiles] = useState<UploadedFile[]>([]);
  const [dragActive, setDragActive] = useState(false);
  const [previewFile, setPreviewFile] = useState<UploadedFile | null>(null);

  const allowedTypes = [
    'image/jpeg',
    'image/png',
    'image/gif',
    'application/pdf',
    'text/csv',
    'application/json'
  ];

  const handleFileSelect = useCallback((selectedFiles: FileList) => {
    Array.from(selectedFiles).forEach(file => {
      if (!validateFileType(file, allowedTypes)) {
        alert(`File type ${file.type} is not allowed`);
        return;
      }

      const uploadFile: UploadedFile = {
        id: `${Date.now()}-${Math.random()}`,
        name: file.name,
        size: file.size,
        type: file.type,
        url: '',
        uploadProgress: 0,
        status: 'uploading'
      };

      setFiles(prev => [...prev, uploadFile]);
      uploadFileToServer(file, uploadFile.id);
    });
  }, []);

  const uploadFileToServer = async (file: File, fileId: string) => {
    const formData = new FormData();
    formData.append('file', file);

    try {
      const response = await makeRequest('/api/upload', {
        method: 'POST',
        data: formData,
        // Progress tracking would be implemented here
        onProgress: (progress: number) => {
          setFiles(prev => prev.map(f => 
            f.id === fileId 
              ? { ...f, uploadProgress: progress }
              : f
          ));
        }
      });

      // Update file with server response
      setFiles(prev => prev.map(f => 
        f.id === fileId 
          ? { 
              ...f, 
              url: response.url,
              uploadProgress: 100,
              status: 'completed'
            }
          : f
      ));
    } catch (error) {
      console.error('Upload failed:', error);
      setFiles(prev => prev.map(f => 
        f.id === fileId 
          ? { ...f, status: 'error' }
          : f
      ));
    }
  };

  const handleDrag = useCallback((e: React.DragEvent) => {
    e.preventDefault();
    e.stopPropagation();
  }, []);

  const handleDragIn = useCallback((e: React.DragEvent) => {
    e.preventDefault();
    e.stopPropagation();
    setDragActive(true);
  }, []);

  const handleDragOut = useCallback((e: React.DragEvent) => {
    e.preventDefault();
    e.stopPropagation();
    setDragActive(false);
  }, []);

  const handleDrop = useCallback((e: React.DragEvent) => {
    e.preventDefault();
    e.stopPropagation();
    setDragActive(false);

    const droppedFiles = e.dataTransfer.files;
    if (droppedFiles.length > 0) {
      handleFileSelect(droppedFiles);
    }
  }, [handleFileSelect]);

  const handleFileInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    if (e.target.files) {
      handleFileSelect(e.target.files);
    }
  };

  const handlePreview = async (file: UploadedFile) => {
    if (file.type.startsWith('text/') || file.type === 'application/json') {
      try {
        // For text files, fetch and show content
        const response = await fetch(file.url);
        const blob = await response.blob();
        const content = await readFileAsText(blob as File);
        setPreviewFile({ ...file, content });
      } catch (error) {
        console.error('Failed to read file:', error);
      }
    } else {
      setPreviewFile(file);
    }
  };

  const handleDownload = async (file: UploadedFile) => {
    try {
      await downloadFile(file.url, file.name);
    } catch (error) {
      console.error('Download failed:', error);
    }
  };

  const handleRemove = (fileId: string) => {
    setFiles(prev => prev.filter(f => f.id !== fileId));
  };

  const formatFileSize = (bytes: number) => {
    if (bytes === 0) return '0 Bytes';
    const k = 1024;
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
  };

  return (
    <Card title="File Upload">
      <div className="file-uploader">
        {/* Drop Zone */}
        <div
          className={`drop-zone ${dragActive ? 'active' : ''}`}
          onDragEnter={handleDragIn}
          onDragLeave={handleDragOut}
          onDragOver={handleDrag}
          onDrop={handleDrop}
        >
          <div className="drop-zone-content">
            <p>Drag and drop files here, or</p>
            <Button
              onClick={() => document.getElementById('file-input')?.click()}
            >
              Browse Files
            </Button>
            <input
              id="file-input"
              type="file"
              multiple
              style={{ display: 'none' }}
              onChange={handleFileInputChange}
              accept={allowedTypes.join(',')}
            />
          </div>
        </div>

        {/* File List */}
        {files.length > 0 && (
          <div className="file-list">
            <h3>Uploaded Files</h3>
            {files.map(file => (
              <div key={file.id} className="file-item">
                <div className="file-info">
                  <div className="file-name">{file.name}</div>
                  <div className="file-details">
                    {formatFileSize(file.size)} • {file.type}
                  </div>
                  
                  {file.status === 'uploading' && (
                    <div className="upload-progress">
                      <div className="progress-bar">
                        <div 
                          className="progress-fill"
                          style={{ width: `${file.uploadProgress}%` }}
                        />
                      </div>
                      <span>{file.uploadProgress}%</span>
                    </div>
                  )}
                </div>

                <div className="file-actions">
                  {file.status === 'completed' && (
                    <>
                      <Button 
                        size="small"
                        onClick={() => handlePreview(file)}
                      >
                        Preview
                      </Button>
                      <Button 
                        size="small"
                        onClick={() => handleDownload(file)}
                      >
                        Download
                      </Button>
                    </>
                  )}
                  
                  {file.status === 'error' && (
                    <span className="error-status">Upload failed</span>
                  )}

                  <Button 
                    size="small"
                    variant="danger"
                    onClick={() => handleRemove(file.id)}
                  >
                    Remove
                  </Button>
                </div>
              </div>
            ))}
          </div>
        )}

        {/* Preview Modal */}
        <Modal
          isOpen={previewFile !== null}
          onClose={() => setPreviewFile(null)}
          title={`Preview: ${previewFile?.name}`}
          size="large"
        >
          {previewFile && (
            <div className="file-preview">
              {previewFile.type.startsWith('image/') ? (
                <img 
                  src={previewFile.url} 
                  alt={previewFile.name}
                  style={{ maxWidth: '100%', height: 'auto' }}
                />
              ) : previewFile.type === 'application/pdf' ? (
                <iframe
                  src={previewFile.url}
                  width="100%"
                  height="600px"
                  title={previewFile.name}
                />
              ) : (previewFile as any).content ? (
                <pre className="text-preview">
                  {(previewFile as any).content}
                </pre>
              ) : (
                <p>Preview not available for this file type.</p>
              )}
            </div>
          )}
        </Modal>
      </div>
    </Card>
  );
}
```

## Advanced Examples

### 4. Real-time Chat Application

```jsx
// ChatApp.tsx
import React, { useState, useEffect, useRef } from 'react';
import { 
  Card, 
  Input, 
  Button,
  Loading,
  Grid 
} from '@/components';
import { 
  formatDate,
  debounce,
  generateId 
} from '@/utils';
import { useLocalStorage } from '@/hooks';

interface Message {
  id: string;
  text: string;
  userId: string;
  userName: string;
  timestamp: Date;
  type: 'user' | 'system';
}

interface User {
  id: string;
  name: string;
  status: 'online' | 'offline' | 'typing';
  lastSeen: Date;
}

interface ChatRoom {
  id: string;
  name: string;
  users: User[];
  messages: Message[];
}

export function ChatApp() {
  const [currentUser] = useLocalStorage('chatUser', {
    id: generateId(8, 'user_'),
    name: 'Anonymous User'
  });
  
  const [rooms, setRooms] = useState<ChatRoom[]>([]);
  const [currentRoom, setCurrentRoom] = useState<string>('general');
  const [message, setMessage] = useState('');
  const [isTyping, setIsTyping] = useState(false);
  const [onlineUsers, setOnlineUsers] = useState<User[]>([]);
  const [connecting, setConnecting] = useState(true);
  
  const messagesEndRef = useRef<HTMLDivElement>(null);
  const wsRef = useRef<WebSocket | null>(null);

  // WebSocket connection
  useEffect(() => {
    connectWebSocket();
    return () => {
      if (wsRef.current) {
        wsRef.current.close();
      }
    };
  }, []);

  // Auto-scroll to bottom when new messages arrive
  useEffect(() => {
    scrollToBottom();
  }, [rooms]);

  const connectWebSocket = () => {
    const ws = new WebSocket('ws://localhost:8080/chat');
    wsRef.current = ws;

    ws.onopen = () => {
      setConnecting(false);
      // Join the chat
      ws.send(JSON.stringify({
        type: 'join',
        userId: currentUser.id,
        userName: currentUser.name,
        room: currentRoom
      }));
    };

    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      handleWebSocketMessage(data);
    };

    ws.onclose = () => {
      setConnecting(true);
      // Attempt to reconnect after 3 seconds
      setTimeout(connectWebSocket, 3000);
    };

    ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
  };

  const handleWebSocketMessage = (data: any) => {
    switch (data.type) {
      case 'message':
        addMessage(data.message);
        break;
      case 'userJoined':
        addSystemMessage(`${data.userName} joined the chat`);
        updateOnlineUsers(data.users);
        break;
      case 'userLeft':
        addSystemMessage(`${data.userName} left the chat`);
        updateOnlineUsers(data.users);
        break;
      case 'userTyping':
        updateUserTypingStatus(data.userId, data.isTyping);
        break;
      case 'roomUpdate':
        setRooms(data.rooms);
        break;
      default:
        console.log('Unknown message type:', data.type);
    }
  };

  const addMessage = (message: Message) => {
    setRooms(prev => prev.map(room => 
      room.id === currentRoom 
        ? {
            ...room,
            messages: [...room.messages, message]
          }
        : room
    ));
  };

  const addSystemMessage = (text: string) => {
    const systemMessage: Message = {
      id: generateId(),
      text,
      userId: 'system',
      userName: 'System',
      timestamp: new Date(),
      type: 'system'
    };
    addMessage(systemMessage);
  };

  const updateOnlineUsers = (users: User[]) => {
    setOnlineUsers(users);
  };

  const updateUserTypingStatus = (userId: string, typing: boolean) => {
    setOnlineUsers(prev => prev.map(user => 
      user.id === userId 
        ? { ...user, status: typing ? 'typing' : 'online' }
        : user
    ));
  };

  // Debounced typing indicator
  const debouncedStopTyping = debounce(() => {
    setIsTyping(false);
    if (wsRef.current?.readyState === WebSocket.OPEN) {
      wsRef.current.send(JSON.stringify({
        type: 'typing',
        userId: currentUser.id,
        isTyping: false
      }));
    }
  }, 1000);

  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setMessage(e.target.value);
    
    if (!isTyping && e.target.value.length > 0) {
      setIsTyping(true);
      if (wsRef.current?.readyState === WebSocket.OPEN) {
        wsRef.current.send(JSON.stringify({
          type: 'typing',
          userId: currentUser.id,
          isTyping: true
        }));
      }
    }
    
    debouncedStopTyping();
  };

  const handleSendMessage = (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!message.trim() || wsRef.current?.readyState !== WebSocket.OPEN) {
      return;
    }

    const newMessage: Message = {
      id: generateId(),
      text: message.trim(),
      userId: currentUser.id,
      userName: currentUser.name,
      timestamp: new Date(),
      type: 'user'
    };

    wsRef.current.send(JSON.stringify({
      type: 'message',
      message: newMessage,
      room: currentRoom
    }));

    setMessage('');
    setIsTyping(false);
  };

  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  };

  const getCurrentRoom = () => {
    return rooms.find(room => room.id === currentRoom);
  };

  const currentRoomData = getCurrentRoom();
  const typingUsers = onlineUsers.filter(user => 
    user.status === 'typing' && user.id !== currentUser.id
  );

  if (connecting) {
    return (
      <Card title="Chat">
        <div className="chat-connecting">
          <Loading text="Connecting to chat..." />
        </div>
      </Card>
    );
  }

  return (
    <div className="chat-app">
      <Grid container spacing={2}>
        {/* Chat Rooms */}
        <Grid item xs={12} md={3}>
          <Card title="Rooms">
            <div className="room-list">
              {rooms.map(room => (
                <div
                  key={room.id}
                  className={`room-item ${room.id === currentRoom ? 'active' : ''}`}
                  onClick={() => setCurrentRoom(room.id)}
                >
                  {room.name}
                  <span className="user-count">({room.users.length})</span>
                </div>
              ))}
            </div>
          </Card>

          <Card title="Online Users" style={{ marginTop: '1rem' }}>
            <div className="user-list">
              {onlineUsers.map(user => (
                <div key={user.id} className="user-item">
                  <div className={`user-status ${user.status}`} />
                  <span className="user-name">{user.name}</span>
                  {user.status === 'typing' && (
                    <span className="typing-indicator">typing...</span>
                  )}
                </div>
              ))}
            </div>
          </Card>
        </Grid>

        {/* Chat Messages */}
        <Grid item xs={12} md={9}>
          <Card title={`#${currentRoomData?.name || currentRoom}`}>
            <div className="chat-container">
              <div className="messages-container">
                {currentRoomData?.messages.map(msg => (
                  <div
                    key={msg.id}
                    className={`message ${msg.type} ${
                      msg.userId === currentUser.id ? 'own' : ''
                    }`}
                  >
                    {msg.type === 'user' && msg.userId !== currentUser.id && (
                      <div className="message-author">{msg.userName}</div>
                    )}
                    <div className="message-content">{msg.text}</div>
                    <div className="message-time">
                      {formatDate(msg.timestamp, 'HH:mm')}
                    </div>
                  </div>
                ))}
                
                {typingUsers.length > 0 && (
                  <div className="typing-indicator-container">
                    {typingUsers.map(user => user.name).join(', ')} 
                    {typingUsers.length === 1 ? ' is' : ' are'} typing...
                  </div>
                )}
                
                <div ref={messagesEndRef} />
              </div>

              <form onSubmit={handleSendMessage} className="message-input-form">
                <Input
                  value={message}
                  onChange={handleInputChange}
                  placeholder="Type a message..."
                  disabled={connecting}
                />
                <Button
                  type="submit"
                  variant="primary"
                  disabled={!message.trim() || connecting}
                >
                  Send
                </Button>
              </form>
            </div>
          </Card>
        </Grid>
      </Grid>
    </div>
  );
}
```

## Real-World Scenarios

### 5. E-commerce Product Catalog

```jsx
// ProductCatalog.tsx
import React, { useState, useEffect, useMemo } from 'react';
import { 
  Grid, 
  Card, 
  Input, 
  Select, 
  Button,
  Loading,
  Modal 
} from '@/components';
import { 
  formatCurrency,
  debounce,
  groupBy,
  sortBy 
} from '@/utils';
import { makeRequest } from '@/api';
import { useLocalStorage } from '@/hooks';

interface Product {
  id: number;
  name: string;
  description: string;
  price: number;
  originalPrice?: number;
  category: string;
  brand: string;
  rating: number;
  reviewCount: number;
  images: string[];
  inStock: boolean;
  tags: string[];
}

interface CartItem {
  product: Product;
  quantity: number;
}

interface Filters {
  category: string;
  priceRange: [number, number];
  brand: string;
  rating: number;
  inStock: boolean;
}

export function ProductCatalog() {
  const [products, setProducts] = useState<Product[]>([]);
  const [filteredProducts, setFilteredProducts] = useState<Product[]>([]);
  const [loading, setLoading] = useState(true);
  const [searchQuery, setSearchQuery] = useState('');
  const [sortBy, setSortBy] = useState('name');
  const [sortDirection, setSortDirection] = useState<'asc' | 'desc'>('asc');
  const [selectedProduct, setSelectedProduct] = useState<Product | null>(null);
  const [cart, setCart] = useLocalStorage<CartItem[]>('shopping_cart', []);
  
  const [filters, setFilters] = useState<Filters>({
    category: '',
    priceRange: [0, 1000],
    brand: '',
    rating: 0,
    inStock: false
  });

  // Load products
  useEffect(() => {
    loadProducts();
  }, []);

  // Filter and sort products
  useEffect(() => {
    let filtered = [...products];

    // Apply search filter
    if (searchQuery) {
      filtered = filtered.filter(product =>
        product.name.toLowerCase().includes(searchQuery.toLowerCase()) ||
        product.description.toLowerCase().includes(searchQuery.toLowerCase()) ||
        product.tags.some(tag => tag.toLowerCase().includes(searchQuery.toLowerCase()))
      );
    }

    // Apply category filter
    if (filters.category) {
      filtered = filtered.filter(product => product.category === filters.category);
    }

    // Apply brand filter
    if (filters.brand) {
      filtered = filtered.filter(product => product.brand === filters.brand);
    }

    // Apply price range filter
    filtered = filtered.filter(product => 
      product.price >= filters.priceRange[0] && 
      product.price <= filters.priceRange[1]
    );

    // Apply rating filter
    if (filters.rating > 0) {
      filtered = filtered.filter(product => product.rating >= filters.rating);
    }

    // Apply stock filter
    if (filters.inStock) {
      filtered = filtered.filter(product => product.inStock);
    }

    // Sort products
    filtered = sortByField(filtered, sortBy, sortDirection);

    setFilteredProducts(filtered);
  }, [products, searchQuery, filters, sortBy, sortDirection]);

  const loadProducts = async () => {
    setLoading(true);
    try {
      const productData = await makeRequest<Product[]>('/api/products');
      setProducts(productData);
    } catch (error) {
      console.error('Failed to load products:', error);
    } finally {
      setLoading(false);
    }
  };

  const sortByField = (items: Product[], field: string, direction: 'asc' | 'desc') => {
    return sortBy(items, field as keyof Product, direction);
  };

  // Debounced search
  const debouncedSearch = useMemo(
    () => debounce((query: string) => {
      setSearchQuery(query);
    }, 300),
    []
  );

  // Get unique values for filter options
  const categories = useMemo(() => {
    return [...new Set(products.map(p => p.category))].sort();
  }, [products]);

  const brands = useMemo(() => {
    return [...new Set(products.map(p => p.brand))].sort();
  }, [products]);

  const priceRange = useMemo(() => {
    const prices = products.map(p => p.price);
    return [Math.min(...prices), Math.max(...prices)];
  }, [products]);

  const handleAddToCart = (product: Product) => {
    setCart(prevCart => {
      const existingItem = prevCart.find(item => item.product.id === product.id);
      
      if (existingItem) {
        return prevCart.map(item =>
          item.product.id === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      } else {
        return [...prevCart, { product, quantity: 1 }];
      }
    });
  };

  const cartItemCount = cart.reduce((total, item) => total + item.quantity, 0);
  const cartTotal = cart.reduce((total, item) => 
    total + (item.product.price * item.quantity), 0
  );

  return (
    <div className="product-catalog">
      {/* Header with cart */}
      <div className="catalog-header">
        <h1>Product Catalog</h1>
        <div className="cart-summary">
          <Button onClick={() => setSelectedProduct({ cart: true } as any)}>
            Cart ({cartItemCount}) - {formatCurrency(cartTotal)}
          </Button>
        </div>
      </div>

      <Grid container spacing={3}>
        {/* Filters Sidebar */}
        <Grid item xs={12} md={3}>
          <Card title="Filters">
            <div className="filters">
              {/* Search */}
              <Input
                placeholder="Search products..."
                onChange={(e) => debouncedSearch(e.target.value)}
              />

              {/* Category Filter */}
              <Select
                label="Category"
                options={[
                  { value: '', label: 'All Categories' },
                  ...categories.map(cat => ({ value: cat, label: cat }))
                ]}
                value={filters.category}
                onChange={(value) => 
                  setFilters(prev => ({ ...prev, category: value as string }))
                }
              />

              {/* Brand Filter */}
              <Select
                label="Brand"
                options={[
                  { value: '', label: 'All Brands' },
                  ...brands.map(brand => ({ value: brand, label: brand }))
                ]}
                value={filters.brand}
                onChange={(value) => 
                  setFilters(prev => ({ ...prev, brand: value as string }))
                }
              />

              {/* Price Range */}
              <div className="price-range-filter">
                <label>Price Range</label>
                <div className="price-inputs">
                  <Input
                    type="number"
                    placeholder="Min"
                    value={filters.priceRange[0]}
                    onChange={(e) => 
                      setFilters(prev => ({ 
                        ...prev, 
                        priceRange: [Number(e.target.value), prev.priceRange[1]]
                      }))
                    }
                  />
                  <Input
                    type="number"
                    placeholder="Max"
                    value={filters.priceRange[1]}
                    onChange={(e) => 
                      setFilters(prev => ({ 
                        ...prev, 
                        priceRange: [prev.priceRange[0], Number(e.target.value)]
                      }))
                    }
                  />
                </div>
              </div>

              {/* Rating Filter */}
              <Select
                label="Minimum Rating"
                options={[
                  { value: '0', label: 'Any Rating' },
                  { value: '4', label: '4+ Stars' },
                  { value: '3', label: '3+ Stars' },
                  { value: '2', label: '2+ Stars' },
                  { value: '1', label: '1+ Stars' }
                ]}
                value={filters.rating.toString()}
                onChange={(value) => 
                  setFilters(prev => ({ ...prev, rating: Number(value) }))
                }
              />

              {/* In Stock Filter */}
              <label className="checkbox-label">
                <input
                  type="checkbox"
                  checked={filters.inStock}
                  onChange={(e) => 
                    setFilters(prev => ({ ...prev, inStock: e.target.checked }))
                  }
                />
                In Stock Only
              </label>
            </div>
          </Card>
        </Grid>

        {/* Products Grid */}
        <Grid item xs={12} md={9}>
          <div className="products-header">
            <div className="results-count">
              {filteredProducts.length} products found
            </div>
            
            <div className="sort-controls">
              <Select
                options={[
                  { value: 'name', label: 'Name' },
                  { value: 'price', label: 'Price' },
                  { value: 'rating', label: 'Rating' },
                  { value: 'reviewCount', label: 'Reviews' }
                ]}
                value={sortBy}
                onChange={(value) => setSortBy(value as string)}
              />
              <Button
                onClick={() => setSortDirection(prev => prev === 'asc' ? 'desc' : 'asc')}
              >
                {sortDirection === 'asc' ? '↑' : '↓'}
              </Button>
            </div>
          </div>

          {loading ? (
            <Loading text="Loading products..." />
          ) : (
            <Grid container spacing={2}>
              {filteredProducts.map(product => (
                <Grid key={product.id} item xs={12} sm={6} md={4}>
                  <ProductCard
                    product={product}
                    onViewDetails={() => setSelectedProduct(product)}
                    onAddToCart={() => handleAddToCart(product)}
                  />
                </Grid>
              ))}
            </Grid>
          )}

          {!loading && filteredProducts.length === 0 && (
            <div className="no-products">
              <p>No products found matching your criteria.</p>
              <Button onClick={() => {
                setFilters({
                  category: '',
                  priceRange: priceRange,
                  brand: '',
                  rating: 0,
                  inStock: false
                });
                setSearchQuery('');
              }}>
                Clear Filters
              </Button>
            </div>
          )}
        </Grid>
      </Grid>

      {/* Product Details Modal */}
      <Modal
        isOpen={selectedProduct !== null}
        onClose={() => setSelectedProduct(null)}
        title={selectedProduct?.name || ''}
        size="large"
      >
        {selectedProduct && (
          <ProductDetails
            product={selectedProduct}
            onAddToCart={() => handleAddToCart(selectedProduct)}
            onClose={() => setSelectedProduct(null)}
          />
        )}
      </Modal>
    </div>
  );
}

// Supporting components
function ProductCard({ 
  product, 
  onViewDetails, 
  onAddToCart 
}: {
  product: Product;
  onViewDetails: () => void;
  onAddToCart: () => void;
}) {
  const discount = product.originalPrice 
    ? Math.round((1 - product.price / product.originalPrice) * 100)
    : 0;

  return (
    <Card className="product-card" clickable onClick={onViewDetails}>
      <div className="product-image">
        <img src={product.images[0]} alt={product.name} />
        {discount > 0 && (
          <div className="discount-badge">-{discount}%</div>
        )}
        {!product.inStock && (
          <div className="out-of-stock-overlay">Out of Stock</div>
        )}
      </div>
      
      <div className="product-info">
        <h3 className="product-name">{product.name}</h3>
        <p className="product-description">
          {product.description.substring(0, 100)}...
        </p>
        
        <div className="product-rating">
          <StarRating rating={product.rating} />
          <span className="review-count">({product.reviewCount})</span>
        </div>
        
        <div className="product-price">
          <span className="current-price">
            {formatCurrency(product.price)}
          </span>
          {product.originalPrice && (
            <span className="original-price">
              {formatCurrency(product.originalPrice)}
            </span>
          )}
        </div>
        
        <Button
          variant="primary"
          disabled={!product.inStock}
          onClick={(e) => {
            e.stopPropagation();
            onAddToCart();
          }}
        >
          {product.inStock ? 'Add to Cart' : 'Out of Stock'}
        </Button>
      </div>
    </Card>
  );
}

function ProductDetails({ 
  product, 
  onAddToCart, 
  onClose 
}: {
  product: Product;
  onAddToCart: () => void;
  onClose: () => void;
}) {
  const [selectedImage, setSelectedImage] = useState(0);

  return (
    <div className="product-details">
      <Grid container spacing={3}>
        <Grid item xs={12} md={6}>
          <div className="product-images">
            <img 
              src={product.images[selectedImage]} 
              alt={product.name}
              className="main-image"
            />
            <div className="image-thumbnails">
              {product.images.map((image, index) => (
                <img
                  key={index}
                  src={image}
                  alt={`${product.name} ${index + 1}`}
                  className={`thumbnail ${index === selectedImage ? 'active' : ''}`}
                  onClick={() => setSelectedImage(index)}
                />
              ))}
            </div>
          </div>
        </Grid>
        
        <Grid item xs={12} md={6}>
          <div className="product-details-info">
            <h1>{product.name}</h1>
            <div className="product-meta">
              <span className="brand">{product.brand}</span>
              <span className="category">{product.category}</span>
            </div>
            
            <div className="product-rating">
              <StarRating rating={product.rating} />
              <span className="review-count">({product.reviewCount} reviews)</span>
            </div>
            
            <p className="product-description">{product.description}</p>
            
            <div className="product-tags">
              {product.tags.map(tag => (
                <span key={tag} className="tag">{tag}</span>
              ))}
            </div>
            
            <div className="product-price">
              <span className="current-price">
                {formatCurrency(product.price)}
              </span>
              {product.originalPrice && (
                <span className="original-price">
                  {formatCurrency(product.originalPrice)}
                </span>
              )}
            </div>
            
            <div className="product-actions">
              <Button
                variant="primary"
                size="large"
                disabled={!product.inStock}
                onClick={onAddToCart}
              >
                {product.inStock ? 'Add to Cart' : 'Out of Stock'}
              </Button>
              <Button onClick={onClose}>
                Continue Shopping
              </Button>
            </div>
          </div>
        </Grid>
      </Grid>
    </div>
  );
}

function StarRating({ rating }: { rating: number }) {
  return (
    <div className="star-rating">
      {[1, 2, 3, 4, 5].map(star => (
        <span
          key={star}
          className={`star ${star <= rating ? 'filled' : ''}`}
        >
          ★
        </span>
      ))}
    </div>
  );
}
```

## Best Practices

### Performance Optimization

```jsx
// Example: Optimized List Component
import React, { memo, useMemo, useCallback } from 'react';
import { debounce } from '@/utils';

const OptimizedList = memo(({ 
  items, 
  onItemSelect, 
  searchQuery 
}: {
  items: any[];
  onItemSelect: (item: any) => void;
  searchQuery: string;
}) => {
  // Memoize filtered items to avoid recalculation
  const filteredItems = useMemo(() => {
    if (!searchQuery) return items;
    return items.filter(item => 
      item.name.toLowerCase().includes(searchQuery.toLowerCase())
    );
  }, [items, searchQuery]);

  // Memoize callback to prevent child re-renders
  const handleItemClick = useCallback((item: any) => {
    onItemSelect(item);
  }, [onItemSelect]);

  return (
    <div className="optimized-list">
      {filteredItems.map(item => (
        <OptimizedListItem
          key={item.id}
          item={item}
          onClick={handleItemClick}
        />
      ))}
    </div>
  );
});

const OptimizedListItem = memo(({ 
  item, 
  onClick 
}: {
  item: any;
  onClick: (item: any) => void;
}) => {
  const handleClick = useCallback(() => {
    onClick(item);
  }, [item, onClick]);

  return (
    <div className="list-item" onClick={handleClick}>
      {item.name}
    </div>
  );
});
```

### Error Handling Patterns

```jsx
// Example: Comprehensive Error Handling
import React from 'react';
import { ErrorBoundary } from '@/components';
import { makeRequest } from '@/api';

function DataComponent() {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(false);

  const loadData = async () => {
    setLoading(true);
    setError(null);
    
    try {
      const result = await makeRequest('/api/data', {
        retries: 3,
        retryDelay: 1000
      });
      setData(result);
    } catch (err) {
      setError(err.message);
      // Log error for monitoring
      console.error('Data loading failed:', err);
    } finally {
      setLoading(false);
    }
  };

  if (error) {
    return (
      <div className="error-state">
        <h3>Something went wrong</h3>
        <p>{error}</p>
        <Button onClick={loadData}>Try Again</Button>
      </div>
    );
  }

  return (
    <ErrorBoundary
      fallback={({ error, resetError }) => (
        <div className="error-boundary">
          <h3>Component Error</h3>
          <p>{error.message}</p>
          <Button onClick={resetError}>Reset</Button>
        </div>
      )}
    >
      {/* Component content */}
    </ErrorBoundary>
  );
}
```

## Troubleshooting

### Common Issues and Solutions

1. **Component not re-rendering**
   - Check if props are changing reference
   - Use React DevTools to inspect state changes
   - Ensure useState/useEffect dependencies are correct

2. **API calls failing**
   - Check network connectivity
   - Verify API endpoint URLs
   - Check authentication tokens
   - Review CORS settings

3. **Performance issues**
   - Use React.memo for pure components
   - Implement proper key props for lists
   - Debounce expensive operations
   - Use useMemo for computed values

4. **Memory leaks**
   - Clean up event listeners in useEffect
   - Cancel pending API requests
   - Clear intervals and timeouts

5. **Styling issues**
   - Check CSS specificity
   - Verify class names are correct
   - Use browser developer tools
   - Check for conflicting styles

For more detailed troubleshooting, refer to the browser console and React Developer Tools.