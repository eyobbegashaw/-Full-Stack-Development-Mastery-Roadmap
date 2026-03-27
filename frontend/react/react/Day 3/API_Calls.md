# **📡 React Data Fetching & API Calls - Complete Guide**

## **🎯 Understanding Data Fetching in React**

### **Concept (70%)**
Data fetching is how React applications **communicate with backend services** to get, create, update, or delete data. The landscape has evolved from simple `fetch()` to sophisticated libraries that handle **caching, synchronization, and real-time updates**.

**The Data Fetching Evolution:**
```
fetch() → Axios → SWR → React Query → RTK Query → GraphQL Clients
Simple   →   Enhanced   →   Smart   →   Advanced   →   Specialized
```




**Key Challenges in Data Fetching:**
1. **Loading states** - Show spinners while data loads
2. **Error handling** - Graceful error states
3. **Caching** - Avoid redundant network requests
4. **Pagination** - Load data in chunks
5. **Optimistic updates** - Update UI before server response
6. **Real-time synchronization** - Keep data fresh
7. **Background refetching** - Update stale data

---

## **1️⃣ REST APIs with Fetch & Axios**

### **Concept (70%)**
REST (Representational State Transfer) is the **traditional API architecture** using HTTP methods (GET, POST, PUT, DELETE) to manipulate resources.

**REST Principles:**
- **Resources** identified by URLs (`/users`, `/posts`)
- **HTTP methods** define operations (GET, POST, etc.)
- **Stateless** - Each request contains all needed info
- **JSON** as primary data format

**Fetch vs Axios:**
- `fetch()`: Native browser API, lightweight, needs polyfills
- `Axios`: Third-party library, more features, better error handling

### **Code (30%)**
```jsx
// 1. Basic Fetch API
function fetchWithFetch() {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const fetchData = async () => {
    setLoading(true);
    try {
      const response = await fetch('https://api.example.com/users');
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      const jsonData = await response.json();
      setData(jsonData);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  useEffect(() => {
    fetchData();
  }, []);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  return <div>{JSON.stringify(data)}</div>;
}

// 2. Axios - More features, better error handling
import axios from 'axios';

const api = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 5000,
  headers: {
    'Content-Type': 'application/json',
    Authorization: `Bearer ${token}`
  }
});

// Request/Response interceptors
api.interceptors.request.use(
  config => {
    // Add auth token to every request
    config.headers.Authorization = `Bearer ${localStorage.getItem('token')}`;
    return config;
  },
  error => Promise.reject(error)
);

api.interceptors.response.use(
  response => response,
  error => {
    if (error.response?.status === 401) {
      // Handle unauthorized
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

// Using Axios
function UserList() {
  const [users, setUsers] = useState([]);
  
  const fetchUsers = async () => {
    try {
      const response = await api.get('/users', {
        params: { page: 1, limit: 10 }
      });
      setUsers(response.data);
    } catch (error) {
      console.error('Failed to fetch users:', error);
    }
  };
  
  const createUser = async (userData) => {
    try {
      const response = await api.post('/users', userData);
      return response.data;
    } catch (error) {
      if (error.response) {
        // Server responded with error
        console.error('Server error:', error.response.data);
      } else if (error.request) {
        // No response received
        console.error('Network error:', error.request);
      } else {
        console.error('Error:', error.message);
      }
    }
  };
  
  return <div>{/* Render users */}</div>;
}

// 3. Custom Hook for REST API
function useFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  const fetchData = useCallback(async () => {
    try {
      const response = await fetch(url, options);
      if (!response.ok) throw new Error('Network response was not ok');
      const json = await response.json();
      setData(json);
    } catch (err) {
      setError(err);
    } finally {
      setLoading(false);
    }
  }, [url, options]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  const refetch = () => {
    setLoading(true);
    fetchData();
  };
  
  return { data, loading, error, refetch };
}
```

---

## **2️⃣ React Query (TanStack Query) - The Modern Standard**

### **Concept (70%)**
React Query is **not a data fetching library** but a **server state management library**. It handles caching, synchronization, updates, and garbage collection of server data.

**Core Concepts:**
- 🎯 **Queries** - For fetching data (GET operations)
- 🔄 **Mutations** - For changing data (POST, PUT, DELETE)
- 💾 **Cache** - Automatic caching with stale-while-revalidate
- 🔍 **Devtools** - Visualize cache state
- ⚡ **Background updates** - Refetch data when needed
- 🎭 **Optimistic updates** - Update UI before server response

**Why React Query?**
1. **No more loading/error state boilerplate**
2. **Automatic caching & deduplication**
3. **Background refetching & synchronization**
4. **Pagination & infinite query support**
5. **Excellent TypeScript support**

### **Code (30%)**
```jsx
import { 
  QueryClient, 
  QueryClientProvider, 
  useQuery, 
  useMutation,
  useQueryClient,
  useInfiniteQuery 
} from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

// 1. Setup
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 10 * 60 * 1000, // 10 minutes
      retry: 2,
      refetchOnWindowFocus: true,
    },
  },
});

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <UserDashboard />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}

// 2. Basic Query
function UserProfile({ userId }) {
  const { data, isLoading, error, refetch } = useQuery({
    queryKey: ['user', userId], // Unique cache key
    queryFn: () => 
      fetch(`/api/users/${userId}`).then(res => res.json()),
    enabled: !!userId, // Only run if userId exists
  });
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <div>
      <h1>{data.name}</h1>
      <button onClick={() => refetch()}>Refresh</button>
    </div>
  );
}

// 3. Parallel & Dependent Queries
function UserDashboard({ userId }) {
  // Parallel queries
  const userQuery = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });
  
  const postsQuery = useQuery({
    queryKey: ['posts', userId],
    queryFn: () => fetchUserPosts(userId),
    enabled: !!userQuery.data, // Run only after user loads
  });
  
  // Combined loading state
  const isLoading = userQuery.isLoading || postsQuery.isLoading;
  
  if (isLoading) return <div>Loading dashboard...</div>;
  
  return (
    <div>
      <h1>{userQuery.data.name}'s Dashboard</h1>
      <PostsList posts={postsQuery.data} />
    </div>
  );
}

// 4. Mutations (Create/Update/Delete)
function CreateUserForm() {
  const queryClient = useQueryClient();
  
  const createUserMutation = useMutation({
    mutationFn: (newUser) => 
      fetch('/api/users', {
        method: 'POST',
        body: JSON.stringify(newUser),
      }).then(res => res.json()),
    
    // Optimistic update
    onMutate: async (newUser) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries(['users']);
      
      // Snapshot previous value
      const previousUsers = queryClient.getQueryData(['users']);
      
      // Optimistically update cache
      queryClient.setQueryData(['users'], old => [...old, newUser]);
      
      return { previousUsers };
    },
    
    // Error handling
    onError: (err, newUser, context) => {
      // Rollback on error
      queryClient.setQueryData(['users'], context.previousUsers);
    },
    
    // Success handling
    onSuccess: (data) => {
      // Invalidate and refetch
      queryClient.invalidateQueries(['users']);
      console.log('User created:', data);
    },
    
    // Always runs
    onSettled: () => {
      console.log('Mutation settled');
    },
  });
  
  const handleSubmit = (userData) => {
    createUserMutation.mutate(userData, {
      onSuccess: () => {
        // Additional success logic
      },
    });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input name="name" />
      <button 
        type="submit"
        disabled={createUserMutation.isLoading}
      >
        {createUserMutation.isLoading ? 'Creating...' : 'Create User'}
      </button>
    </form>
  );
}

// 5. Infinite Scroll / Pagination
function PostFeed() {
  const {
    data,
    fetchNextPage,
    hasNextPage,
    isFetchingNextPage,
    status,
  } = useInfiniteQuery({
    queryKey: ['posts'],
    queryFn: ({ pageParam = 1 }) => 
      fetchPosts({ page: pageParam, limit: 10 }),
    getNextPageParam: (lastPage, allPages) => {
      return lastPage.hasNextPage ? allPages.length + 1 : undefined;
    },
  });
  
  return (
    <div>
      {status === 'loading' ? (
        <p>Loading...</p>
      ) : status === 'error' ? (
        <p>Error loading posts</p>
      ) : (
        <>
          {data.pages.map((page, i) => (
            <div key={i}>
              {page.posts.map(post => (
                <Post key={post.id} post={post} />
              ))}
            </div>
          ))}
          
          <button
            onClick={() => fetchNextPage()}
            disabled={!hasNextPage || isFetchingNextPage}
          >
            {isFetchingNextPage 
              ? 'Loading more...' 
              : hasNextPage 
                ? 'Load More' 
                : 'Nothing more to load'
            }
          </button>
        </>
      )}
    </div>
  );
}

// 6. Prefetching & Cache Management
function UserList() {
  const queryClient = useQueryClient();
  
  // Prefetch user data on hover
  const prefetchUser = (userId) => {
    queryClient.prefetchQuery({
      queryKey: ['user', userId],
      queryFn: () => fetchUser(userId),
      staleTime: 5 * 60 * 1000,
    });
  };
  
  return (
    <div>
      {users.map(user => (
        <div 
          key={user.id}
          onMouseEnter={() => prefetchUser(user.id)}
          onClick={() => navigate(`/user/${user.id}`)}
        >
          {user.name}
        </div>
      ))}
    </div>
  );
}
```

---

## **3️⃣ SWR (Stale-While-Revalidate) - The Lightweight Alternative**

### **Concept (70%)**
SWR is a **lightweight data fetching library** from Vercel that implements the **stale-while-revalidate** caching strategy. It's simpler than React Query but still powerful.

**SWR Philosophy:**
- ⚡ **Lightweight** - Smaller bundle than React Query
- 🎯 **Hook-based** - Simple `useSWR` hook
- 🔄 **Stale-while-revalidate** - Show cached data while fetching fresh
- 🔧 **Flexible** - Works with any data fetching method
- 📱 **Realtime** - Built-in revalidation strategies

**SWR vs React Query:**
- SWR: **Simpler**, **lighter**, great for simpler apps
- React Query: **More features**, **enterprise-grade**, better TypeScript

### **Code (30%)**
```jsx
import useSWR, { useSWRConfig } from 'swr';

// 1. Fetcher function
const fetcher = (url) => fetch(url).then(res => res.json());

// 2. Basic usage
function UserProfile({ userId }) {
  const { data, error, isLoading, isValidating, mutate } = useSWR(
    `/api/users/${userId}`,
    fetcher,
    {
      revalidateOnFocus: true, // Refetch on window focus
      refreshInterval: 30000, // Refetch every 30 seconds
      dedupingInterval: 2000, // Deduplicate requests within 2 seconds
      shouldRetryOnError: true,
      errorRetryCount: 3,
    }
  );
  
  if (error) return <div>Failed to load</div>;
  if (isLoading) return <div>Loading...</div>;
  
  return (
    <div>
      <h1>{data.name}</h1>
      {isValidating && <div>Updating...</div>}
      <button onClick={() => mutate()}>Refresh</button>
    </div>
  );
}

// 3. Dependent fetching
function UserPosts({ userId }) {
  // Fetch user first
  const { data: user } = useSWR(`/api/users/${userId}`, fetcher);
  
  // Then fetch posts (enabled when user is loaded)
  const { data: posts } = useSWR(
    user ? `/api/users/${userId}/posts` : null,
    fetcher
  );
  
  if (!posts) return <div>Loading posts...</div>;
  return <PostsList posts={posts} />;
}

// 4. Mutations (Optimistic updates)
function LikeButton({ postId }) {
  const { mutate } = useSWRConfig();
  
  const likePost = async () => {
    // Optimistic update
    mutate(
      `/api/posts/${postId}`,
      async (currentData) => {
        return { ...currentData, likes: currentData.likes + 1 };
      },
      false // Don't revalidate yet
    );
    
    // Send request
    await fetch(`/api/posts/${postId}/like`, { method: 'POST' });
    
    // Revalidate to get fresh data
    mutate(`/api/posts/${postId}`);
  };
  
  return <button onClick={likePost}>Like</button>;
}

// 5. Cache management
function CacheManager() {
  const { cache, mutate } = useSWRConfig();
  
  const clearCache = () => {
    // Clear all cache
    cache.clear();
  };
  
  const updateUserCache = (userId, newData) => {
    // Update specific cache key
    mutate(`/api/users/${userId}`, newData, false);
  };
  
  return (
    <div>
      <button onClick={clearCache}>Clear All Cache</button>
    </div>
  );
}

// 6. Global configuration with provider
import { SWRConfig } from 'swr';

function App() {
  return (
    <SWRConfig 
      value={{
        fetcher: (url) => fetch(url).then(res => res.json()),
        refreshInterval: 3000,
        onError: (error, key) => {
          if (error.status !== 403 && error.status !== 404) {
            // Log to error reporting service
            console.error('SWR Error:', error);
          }
        },
      }}
    >
      <UserDashboard />
    </SWRConfig>
  );
}
```

---

## **4️⃣ RTK Query - Redux Toolkit's Solution**

### **Concept (70%)**
RTK Query is a **powerful data fetching and caching solution** built into Redux Toolkit. It's designed for **Redux users** who want integrated data fetching.

**RTK Query Features:**
- 🔌 **Built into Redux** - Seamless Redux integration
- 🎯 **Auto-generated hooks** - From API definitions
- 🔄 **Cache management** - Built on Redux state
- 📝 **TypeScript** - Excellent type inference
- 🔧 **Code splitting** - Auto-generated slices

**When to use RTK Query:**
- Already using Redux in your app
- Need tight integration with Redux state
- Want auto-generated API code

### **Code (30%)**
```jsx
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';
import { configureStore } from '@reduxjs/toolkit';
import { Provider } from 'react-redux';

// 1. Define API service
const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ 
    baseUrl: 'https://api.example.com',
    prepareHeaders: (headers) => {
      const token = localStorage.getItem('token');
      if (token) {
        headers.set('Authorization', `Bearer ${token}`);
      }
      return headers;
    },
  }),
  tagTypes: ['User', 'Post'], // For cache invalidation
  endpoints: (builder) => ({
    // Query endpoints (GET)
    getUsers: builder.query({
      query: ({ page = 1, limit = 10 }) => 
        `users?page=${page}&limit=${limit}`,
      providesTags: ['User'], // Invalidate when User tag is invalidated
    }),
    
    getUser: builder.query({
      query: (id) => `users/${id}`,
      providesTags: (result, error, id) => 
        [{ type: 'User', id }],
    }),
    
    // Mutation endpoints (POST, PUT, DELETE)
    createUser: builder.mutation({
      query: (userData) => ({
        url: 'users',
        method: 'POST',
        body: userData,
      }),
      invalidatesTags: ['User'], // Invalidate User cache
    }),
    
    updateUser: builder.mutation({
      query: ({ id, ...userData }) => ({
        url: `users/${id}`,
        method: 'PUT',
        body: userData,
      }),
      invalidatesTags: (result, error, { id }) => 
        [{ type: 'User', id }],
      
      // Optimistic update
      async onQueryStarted({ id, ...patch }, { dispatch, queryFulfilled }) {
        const patchResult = dispatch(
          api.util.updateQueryData('getUser', id, (draft) => {
            Object.assign(draft, patch);
          })
        );
        
        try {
          await queryFulfilled();
        } catch {
          patchResult.undo();
        }
      },
    }),
  }),
});

// 2. Configure store
const store = configureStore({
  reducer: {
    [api.reducerPath]: api.reducer,
    // other reducers...
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(api.middleware),
});

// 3. Provider setup
function App() {
  return (
    <Provider store={store}>
      <UserDashboard />
    </Provider>
  );
}

// 4. Using generated hooks in components
function UserList() {
  // Auto-generated hook
  const { data: users, isLoading, error, refetch } = api.useGetUsersQuery({
    page: 1,
    limit: 10,
  });
  
  const [createUser] = api.useCreateUserMutation();
  
  const handleCreate = async (userData) => {
    try {
      await createUser(userData).unwrap();
      // Success - cache automatically invalidated
    } catch (error) {
      console.error('Failed to create user:', error);
    }
  };
  
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <div>
      {users.map(user => (
        <div key={user.id}>{user.name}</div>
      ))}
      <button onClick={() => refetch()}>Refresh</button>
    </div>
  );
}

// 5. Pagination with RTK Query
function PaginatedUserList() {
  const [page, setPage] = useState(1);
  const { data, isLoading, isFetching } = api.useGetUsersQuery(
    { page, limit: 10 },
    { skip: page < 1 } // Skip query if page < 1
  );
  
  return (
    <div>
      {data?.users.map(user => (
        <div key={user.id}>{user.name}</div>
      ))}
      
      <div>
        <button 
          onClick={() => setPage(p => p - 1)}
          disabled={page === 1 || isFetching}
        >
          Previous
        </button>
        <span>Page {page}</span>
        <button 
          onClick={() => setPage(p => p + 1)}
          disabled={!data?.hasMore || isFetching}
        >
          Next
        </button>
      </div>
    </div>
  );
}
```

---

## **5️⃣ GraphQL Clients (Apollo & Relay)**

### **Concept (70%)**
GraphQL is a **query language for APIs** that allows clients to request exactly the data they need. Apollo Client and Relay are the two main GraphQL clients for React.

**GraphQL vs REST:**
- **REST**: Multiple endpoints, fixed responses
- **GraphQL**: Single endpoint, flexible queries

**Apollo vs Relay:**
- **Apollo**: More flexible, easier to start
- **Relay**: More opinionated, better performance

### **Apollo Client Code (30%)**
```jsx
import { 
  ApolloClient, 
  InMemoryCache, 
  ApolloProvider, 
  gql, 
  useQuery, 
  useMutation 
} from '@apollo/client';

// 1. Setup Apollo Client
const client = new ApolloClient({
  uri: 'https://api.example.com/graphql',
  cache: new InMemoryCache(),
  defaultOptions: {
    watchQuery: {
      fetchPolicy: 'cache-and-network',
    },
  },
});

function App() {
  return (
    <ApolloProvider client={client}>
      <UserDashboard />
    </ApolloProvider>
  );
}

// 2. Define GraphQL queries
const GET_USERS = gql`
  query GetUsers($limit: Int, $offset: Int) {
    users(limit: $limit, offset: $offset) {
      id
      name
      email
      posts {
        id
        title
      }
    }
  }
`;

const CREATE_USER = gql`
  mutation CreateUser($input: CreateUserInput!) {
    createUser(input: $input) {
      id
      name
      email
    }
  }
`;

// 3. Using queries
function UserList() {
  const { data, loading, error, fetchMore } = useQuery(GET_USERS, {
    variables: { limit: 10, offset: 0 },
  });
  
  const [createUser] = useMutation(CREATE_USER, {
    update(cache, { data: { createUser } }) {
      // Update cache after mutation
      cache.modify({
        fields: {
          users(existingUsers = []) {
            const newUserRef = cache.writeFragment({
              data: createUser,
              fragment: gql`
                fragment NewUser on User {
                  id
                  name
                  email
                }
              `,
            });
            return [...existingUsers, newUserRef];
          },
        },
      });
    },
  });
  
  const handleCreate = async (userData) => {
    await createUser({
      variables: { input: userData },
      optimisticResponse: {
        __typename: 'Mutation',
        createUser: {
          __typename: 'User',
          id: 'temp-id',
          ...userData,
        },
      },
    });
  };
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <div>
      {data.users.map(user => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}

// 4. Real-time subscriptions
const USER_CREATED = gql`
  subscription OnUserCreated {
    userCreated {
      id
      name
      email
    }
  }
`;

function UserFeed() {
  const { data, loading } = useSubscription(USER_CREATED);
  
  useEffect(() => {
    if (data) {
      console.log('New user created:', data.userCreated);
    }
  }, [data]);
  
  return <div>{/* Display real-time feed */}</div>;
}
```

---

## **📊 Data Fetching Library Comparison**

| **Library** | **Size** | **Learning Curve** | **Best For** | **GraphQL** | **Cache** |
|------------|----------|-------------------|-------------|------------|----------|
| **Fetch/Axios** | 0-10KB | Easy | Simple apps | ❌ | Manual |
| **SWR** | 10KB | Easy | Lightweight apps | ❌ | Auto |
| **React Query** | 40KB | Medium | Most React apps | ❌ | Excellent |
| **RTK Query** | 30KB+ | Medium | Redux apps | ❌ | Excellent |
| **Apollo Client** | 100KB+ | High | GraphQL apps | ✅ | Excellent |
| **Relay** | 200KB+ | Very High | Large GraphQL apps | ✅ | Excellent |

---

## **🎯 Choosing the Right Solution**

### **Decision Tree:**
```
Do you use GraphQL?
├── Yes → Large app? → Yes → Relay
│               └── No → Apollo Client
│
└── No → Do you use Redux?
         ├── Yes → RTK Query
         │
         └── No → Need advanced features?
                  ├── Yes → React Query
                  │
                  └── No → Simple app? → Yes → SWR
                                          └── No → React Query
```

### **Modern Stack Recommendations:**

**1. Standard React App:**
```jsx
// React Query + Axios
// - Most versatile
// - Best developer experience
// - Great for REST APIs
```

**2. Next.js App:**
```jsx
// SWR or React Query
// - Both work great with Next.js
// - SWR from Vercel (Next.js creators)
// - React Query for more features
```

**3. Redux App:**
```jsx
// RTK Query
// - Built into Redux Toolkit
// - No extra dependencies
// - Tight Redux integration
```

**4. GraphQL App:**
```jsx
// Apollo Client (most)
// Relay (Facebook-scale apps)
```

---

## **🚀 Advanced Patterns**

### **1. Request Cancellation & Debouncing**
```jsx
// With React Query
function SearchInput() {
  const [search, setSearch] = useState('');
  
  const { data } = useQuery({
    queryKey: ['search', search],
    queryFn: async ({ signal }) => {
      const response = await fetch(`/api/search?q=${search}`, { signal });
      return response.json();
    },
    enabled: search.length > 2, // Only search after 3 chars
  });
  
  // Debounced setSearch
  const debouncedSetSearch = useDebouncedCallback(setSearch, 300);
  
  return (
    <input 
      onChange={(e) => debouncedSetSearch(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

### **2. Offline Support & Background Sync**
```jsx
// React Query with offline support
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retryDelay: (attemptIndex) => 
        Math.min(1000 * 2 ** attemptIndex, 30000),
      networkMode: 'offlineFirst',
    },
  },
});

// Background sync manager
function SyncManager() {
  const queryClient = useQueryClient();
  
  useSyncExternalStore(
    subscribeToOnlineStatus,
    getOnlineStatus,
    () => true
  );
  
  const isOnline = useNetworkState();
  
  useEffect(() => {
    if (isOnline) {
      // Refetch all queries when coming online
      queryClient.refetchQueries();
    }
  }, [isOnline, queryClient]);
}
```

### **3. Request/Response Transformation**
```jsx
// Axios interceptors for transformation
api.interceptors.response.use(
  response => {
    // Transform response data
    if (response.data?.data) {
      response.data = response.data.data;
    }
    return response;
  },
  error => {
    // Transform error format
    if (error.response?.data?.message) {
      error.message = error.response.data.message;
    }
    return Promise.reject(error);
  }
);
```

---

## **📋 Best Practices**

### **DO:**
1. ✅ **Use a library** for anything beyond simple fetches
2. ✅ **Handle loading & error states** consistently
3. ✅ **Cache aggressively** when data doesn't change often
4. ✅ **Use TypeScript** for API responses
5. ✅ **Implement request cancellation** for search/typeahead
6. ✅ **Consider offline support** for better UX

### **DON'T:**
1. ❌ **Fetch in loops** - Batch requests instead
2. ❌ **Ignore error handling** - Always handle errors
3. ❌ **Overfetch data** - Request only what you need
4. ❌ **Cache sensitive data** without encryption
5. ❌ **Forget cleanup** - Cancel requests on unmount

---

## **🎓 Key Takeaways:**

1. **Start simple**: Fetch/Axios for basic needs
2. **Upgrade when needed**: Add React Query/SWR for caching
3. **Choose based on stack**: RTK Query for Redux, Apollo for GraphQL
4. **Always handle**: Loading, error, and empty states
5. **Cache smartly**: Stale-while-revalidate is your friend

**Modern Recommendation for 2024:**
```jsx
// For most React apps:
// React Query (TanStack Query) + Axios
// Why?
// ✅ Most features
// ✅ Best TypeScript support  
// ✅ Excellent caching
// ✅ Active development
// ✅ Great community

// For Next.js apps:
// SWR (simpler) or React Query (more features)
```

**Remember**: Good data fetching is invisible to users. They should see **fast, consistent, and reliable** data without thinking about network requests! 🚀
