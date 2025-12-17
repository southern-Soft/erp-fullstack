# Southern Apparels ERP System - Complete Documentation

## Table of Contents
1. [System Overview](#system-overview)
2. [Architecture Overview](#architecture-overview)
3. [Technology Stack](#technology-stack)
4. [Project Structure](#project-structure)
5. [How Everything Connects](#how-everything-connects)
6. [Database Structure](#database-structure)
7. [Backend (FastAPI) Deep Dive](#backend-fastapi-deep-dive)
8. [Frontend (Next.js) Deep Dive](#frontend-nextjs-deep-dive)
9. [Authentication & Authorization](#authentication--authorization)
10. [Where to Edit/Add Features](#where-to-editadd-features)
11. [Development Setup](#development-setup)
12. [Common Tasks Guide](#common-tasks-guide)

---

## System Overview

This is a **Ready-Made Garment (RMG) ERP System** built for Southern Apparels and Holdings. It manages:
- **Client Information** (Buyers, Suppliers, Contacts, Shipping, Banking)
- **Sample Department** (Styles, Variants, Materials, Samples, Operations, SMV)
- **Order Management** (Orders, Production Planning, Inventory, Reports)
- **User Management** (Authentication, Permissions, Department Access)

### Key Features
- ✅ User authentication with JWT tokens
- ✅ Role-based access control (department permissions)
- ✅ RESTful API backend
- ✅ Modern React frontend with Next.js
- ✅ PostgreSQL database
- ✅ Docker containerization
- ✅ Redis caching for performance
- ✅ Optimized for 200-250+ concurrent users

---

## Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    User's Browser                       │
│              (Next.js Frontend - Port 3000)             │
└────────────────────┬────────────────────────────────────┘
                     │
                     │ HTTP Requests
                     │ (via API Proxy)
                     ▼
┌─────────────────────────────────────────────────────────┐
│              Next.js API Routes (Proxy)                   │
│         /app/api/v1/[...path]/route.ts                   │
└────────────────────┬────────────────────────────────────┘
                     │
                     │ Proxies to Backend
                     │
                     ▼
┌─────────────────────────────────────────────────────────┐
│            FastAPI Backend (Port 8000)                   │
│              /api/v1/* endpoints                         │
└──────────────┬──────────────────────┬───────────────────┘
               │                      │
               ▼                      ▼
    ┌──────────────────┐    ┌──────────────────┐
    │  PostgreSQL DB   │    │   Redis Cache    │
    │   (Port 5432)    │    │   (Port 6379)    │
    └──────────────────┘    └──────────────────┘
```

### How Data Flows

1. **User Action** → User clicks a button in the frontend
2. **Frontend API Call** → Frontend calls `api.buyers.getAll()` from `lib/api.ts`
3. **Next.js Proxy** → Request goes to `/api/v1/buyers` which proxies to backend
4. **Backend Route** → FastAPI route handler processes the request
5. **Database Query** → SQLAlchemy queries PostgreSQL
6. **Response** → Data flows back through the same path
7. **UI Update** → React components re-render with new data

---

## Technology Stack

### Backend
- **FastAPI** - Modern Python web framework (like Express.js for Node.js)
- **SQLAlchemy** - Database ORM (Object-Relational Mapping)
- **PostgreSQL** - Relational database
- **Redis** - Caching layer
- **JWT** - Authentication tokens
- **Pydantic** - Data validation

### Frontend
- **Next.js 15** - React framework with server-side rendering
- **React 19** - UI library
- **TypeScript** - Type-safe JavaScript
- **Tailwind CSS** - Styling
- **shadcn/ui** - UI component library
- **Zustand** - State management (if needed)

### Infrastructure
- **Docker** - Containerization
- **Docker Compose** - Multi-container orchestration

---

## Project Structure

```
erp_southern_final/
├── backend/                    # FastAPI Backend
│   ├── core/                   # Core utilities
│   │   ├── config.py          # Settings & configuration
│   │   ├── database.py        # Database connection
│   │   ├── security.py        # Password hashing, JWT tokens
│   │   ├── logging.py          # Logging setup
│   │   └── cache.py           # Redis caching
│   ├── modules/                # Feature modules
│   │   ├── auth/              # Authentication
│   │   │   └── routes/
│   │   │       └── auth.py    # Login, register, /me endpoints
│   │   ├── clients/           # Client management
│   │   │   ├── models/
│   │   │   │   └── client.py  # Buyer, Supplier, Contact models
│   │   │   └── routes/
│   │   │       ├── buyers.py  # Buyer CRUD endpoints
│   │   │       ├── suppliers.py
│   │   │       └── contacts.py
│   │   ├── samples/           # Sample department
│   │   │   ├── models/
│   │   │   │   └── sample.py  # Sample, Style, Variant models
│   │   │   ├── routes/
│   │   │   │   └── samples.py # Sample endpoints
│   │   │   └── schemas/
│   │   │       └── sample.py  # Data validation schemas
│   │   ├── orders/            # Order management
│   │   ├── materials/        # Material management
│   │   └── users/            # User management
│   ├── main.py               # FastAPI app entry point
│   └── init_data.py          # Initial admin user creation
│
├── frontend/                  # Next.js Frontend
│   ├── app/                   # Next.js App Router
│   │   ├── api/               # API proxy routes
│   │   │   └── v1/
│   │   │       └── [...path]/
│   │   │           └── route.ts # Proxies all /api/v1/* to backend
│   │   ├── dashboard/         # Main application
│   │   │   ├── (authenticated)/ # Protected routes
│   │   │   │   ├── layout.tsx  # Sidebar + Header layout
│   │   │   │   └── erp/        # ERP modules
│   │   │   │       ├── clients/
│   │   │   │       │   ├── buyers/page.tsx
│   │   │   │   │       ├── suppliers/page.tsx
│   │   │   │   │       └── contacts/page.tsx
│   │   │   │       ├── samples/
│   │   │   │       │   └── [various sample pages]
│   │   │   │       └── orders/page.tsx
│   │   │   └── (public)/      # Public routes
│   │   │       └── login/
│   │   │           └── page.tsx
│   │   └── layout.tsx         # Root layout
│   ├── components/            # Reusable React components
│   │   ├── ui/               # shadcn/ui components
│   │   ├── layout/           # Layout components
│   │   │   ├── sidebar/
│   │   │   │   └── app-sidebar.tsx
│   │   │   └── header/
│   │   └── shared/           # Shared utilities
│   ├── lib/                  # Utility functions
│   │   ├── api.ts            # API client functions
│   │   ├── auth-context.tsx  # Authentication context
│   │   └── utils.ts          # Helper functions
│   └── middleware.ts         # Route protection
│
└── docker-compose.yml        # Docker setup
```

---

## How Everything Connects

### 1. Database Connection

**Location**: `backend/core/database.py`

```python
# Creates database engine
engine = create_engine(
    settings.DATABASE_URL,  # postgresql://user:pass@host:port/dbname
    pool_size=100,          # Connection pool
    max_overflow=100
)

# Creates session factory
SessionLocal = sessionmaker(bind=engine)

# Dependency for routes
def get_db():
    db = SessionLocal()
    try:
        yield db  # Provides database session
    finally:
        db.close()  # Closes after request
```

**How it works**:
- On startup, `main.py` calls `init_db()` which creates all tables
- Each API route uses `db: Session = Depends(get_db)` to get a database session
- SQLAlchemy models (like `User`, `Buyer`) map to database tables

### 2. Frontend-Backend Connection

**Step 1: Frontend makes API call**
```typescript
// frontend/lib/api.ts
export const api = {
  buyers: {
    getAll: async () => {
      const response = await fetch('/api/v1/buyers');
      return response.json();
    }
  }
}
```

**Step 2: Next.js API route proxies to backend**
```typescript
// frontend/app/api/v1/[...path]/route.ts
const BACKEND_URL = 'http://backend:8000';  // Docker service name

export async function GET(request, { params }) {
  const targetUrl = `${BACKEND_URL}/api/v1/${path.join('/')}`;
  const response = await fetch(targetUrl);
  return response;  // Forward response to frontend
}
```

**Step 3: Backend processes request**
```python
# backend/modules/clients/routes/buyers.py
@router.get("/buyers")
def get_buyers(db: Session = Depends(get_db)):
    buyers = db.query(Buyer).all()
    return buyers
```

**Step 4: Response flows back**
- Backend → Next.js proxy → Frontend → React component updates

### 3. Authentication Flow

1. **User logs in** → `frontend/lib/auth-context.tsx` → `login()` function
2. **API call** → `POST /api/v1/auth/login` with username/password
3. **Backend validates** → `backend/modules/auth/routes/auth.py` → `login()` function
4. **JWT token created** → `backend/core/security.py` → `create_access_token()`
5. **Token stored** → localStorage + cookie (for middleware)
6. **Protected routes** → `frontend/middleware.ts` checks cookie
7. **API requests** → Include `Authorization: Bearer <token>` header

---

## Database Structure

### Core Tables

#### Users Table
```sql
users
├── id (Primary Key)
├── username (Unique)
├── email (Unique)
├── hashed_password
├── full_name
├── department (e.g., "Sample", "IE", "Planning")
├── designation
├── is_active
├── is_superuser
├── department_access (JSON array)  -- ["client_info", "sample_department"]
├── created_at
└── updated_at
```

#### Clients Tables

**Buyers**
```sql
buyers
├── id
├── buyer_name
├── company_name
├── brand_name
├── email, phone, website
├── head_office_country
├── tax_id
├── rating
└── status
```

**Suppliers**
```sql
suppliers
├── id
├── supplier_name
├── company_name
├── supplier_type (Fabric, Trims, Accessories, etc.)
├── contact_person
├── email, phone, country
└── ...
```

**Contact Persons**
```sql
contact_persons
├── id
├── contact_person_name
├── company, department, designation
├── phone_number, corporate_mail
├── buyer_id (Foreign Key → buyers.id)
└── supplier_id (Foreign Key → suppliers.id)
```

**Shipping Info**
```sql
shipping_info
├── id
├── buyer_id (Foreign Key)
├── destination_country, destination_port
├── address, incoterm
└── ...
```

**Banking Info**
```sql
banking_info
├── id
├── client_name
├── bank_name, account_number
├── swift_code, sort_code
└── currency
```

#### Sample Department Tables

**Style Summaries**
```sql
style_summaries
├── id
├── buyer_id (Foreign Key)
├── style_name
├── style_id (Unique)
├── product_category, product_type
├── type_of_construction, gauge
├── is_set (Boolean)
└── set_piece_count
```

**Style Variants**
```sql
style_variants
├── id
├── style_summary_id (Foreign Key)
├── style_name, style_id
├── colour_name, colour_code, colour_ref
├── is_multicolor (Boolean)
├── sizes (JSON array)
└── piece_name (for sets)
```

**Variant Color Parts** (for multi-color variants)
```sql
style_variant_colors
├── id
├── style_variant_id (Foreign Key)
├── part_name (e.g., "Body", "Collar")
├── colour_name, colour_code, colour_ref
└── sort_order
```

**Required Materials**
```sql
required_materials
├── id
├── style_variant_id (Foreign Key)
├── material (material name)
├── uom (Unit of Measurement)
├── consumption_per_piece
├── converted_uom, converted_consumption
└── remarks
```

**Samples**
```sql
samples
├── id
├── sample_id (Unique)
├── buyer_id (Foreign Key)
├── style_id (Foreign Key → style_summaries.id)
├── sample_type (Proto, Fit, PP, etc.)
├── worksheet_rcv_date, yarn_rcv_date, required_date
├── color, assigned_designer
├── required_sample_quantity, round
├── submit_status
└── notes
```

**Sample Operations**
```sql
sample_operations
├── id
├── sample_id (Foreign Key → samples.id)
├── operation_type (Knitting, Linking, etc.)
├── name_of_operation
├── number_of_operation
├── size
├── duration (minutes)
└── total_duration
```

#### Order Management

**Orders**
```sql
order_management
├── id
├── order_no (Unique)
├── buyer_id (Foreign Key)
├── style_id (Foreign Key)
├── style_name, season, order_category
├── sales_contract, scl_po, fob
├── order_quantity, unit_price, total_value
├── order_date, delivery_date, shipment_date
└── order_status
```

### Relationships (Foreign Keys)

```
Buyer
  ├──→ ContactPerson (one-to-many)
  ├──→ ShippingInfo (one-to-many)
  ├──→ StyleSummary (one-to-many)
  ├──→ Sample (one-to-many)
  └──→ OrderManagement (one-to-many)

StyleSummary
  ├──→ StyleVariant (one-to-many)
  ├──→ Sample (one-to-many)
  └──→ OrderManagement (one-to-many)

StyleVariant
  ├──→ RequiredMaterial (one-to-many)
  └──→ VariantColorPart (one-to-many)

Sample
  └──→ SampleOperation (one-to-many)
```

---

## Backend (FastAPI) Deep Dive

### Entry Point: `backend/main.py`

This is where the FastAPI application starts:

```python
app = FastAPI(title="RMG ERP System", version="1.0.0")

# CORS middleware (allows frontend to call backend)
app.add_middleware(CORSMiddleware, allow_origins=["*"])

# Register all routers
app.include_router(auth_router, prefix="/api/v1/auth")
app.include_router(buyers_router, prefix="/api/v1/buyers")
# ... etc
```

**On Startup**:
1. Database tables are created (`init_db()`)
2. Admin user is created if it doesn't exist (`init_sample_data()`)

### Core Modules

#### 1. Configuration (`backend/core/config.py`)

Stores all settings:
- Database connection string
- JWT secret key
- Redis settings
- CORS origins

**To change settings**: Edit this file or set environment variables.

#### 2. Database (`backend/core/database.py`)

- Creates SQLAlchemy engine
- Provides `get_db()` dependency for routes
- Connection pooling (100 base + 100 overflow)

#### 3. Security (`backend/core/security.py`)

Functions:
- `get_password_hash()` - Hashes passwords with bcrypt
- `verify_password()` - Checks password against hash
- `create_access_token()` - Creates JWT token
- `decode_token()` - Validates and decodes JWT

#### 4. Caching (`backend/core/cache.py`)

Redis caching decorator:
```python
@cache_response(key_prefix="buyers", ttl=300)
def get_buyers():
    # This response will be cached for 5 minutes
    return db.query(Buyer).all()
```

### Module Structure

Each module follows this pattern:

```
modules/
└── clients/
    ├── __init__.py          # Exports router
    ├── models/
    │   └── client.py        # SQLAlchemy models (Buyer, Supplier)
    ├── routes/
    │   ├── buyers.py        # API endpoints
    │   └── suppliers.py
    └── schemas/
        └── buyer.py         # Pydantic schemas (validation)
```

### Creating a New Module

**Step 1: Create model**
```python
# backend/modules/products/models/product.py
from core.database import Base
from sqlalchemy import Column, Integer, String

class Product(Base):
    __tablename__ = "products"
    id = Column(Integer, primary_key=True)
    name = Column(String, nullable=False)
```

**Step 2: Create schema**
```python
# backend/modules/products/schemas/product.py
from pydantic import BaseModel

class ProductCreate(BaseModel):
    name: str

class ProductResponse(BaseModel):
    id: int
    name: str
```

**Step 3: Create routes**
```python
# backend/modules/products/routes/products.py
from fastapi import APIRouter, Depends
from core import get_db
from modules.products.models.product import Product
from modules.products.schemas.product import ProductCreate, ProductResponse

router = APIRouter()

@router.get("/products", response_model=List[ProductResponse])
def get_products(db: Session = Depends(get_db)):
    return db.query(Product).all()

@router.post("/products", response_model=ProductResponse)
def create_product(product: ProductCreate, db: Session = Depends(get_db)):
    new_product = Product(**product.model_dump())
    db.add(new_product)
    db.commit()
    db.refresh(new_product)
    return new_product
```

**Step 4: Register in main.py**
```python
from modules.products import products_router
app.include_router(products_router, prefix="/api/v1/products")
```

### API Endpoint Structure

All endpoints follow RESTful conventions:

- `GET /api/v1/buyers` - List all buyers
- `GET /api/v1/buyers/{id}` - Get one buyer
- `POST /api/v1/buyers` - Create buyer
- `PUT /api/v1/buyers/{id}` - Update buyer
- `DELETE /api/v1/buyers/{id}` - Delete buyer

### Request/Response Flow

1. **Request comes in** → FastAPI receives HTTP request
2. **Route handler** → Matches URL to function
3. **Dependencies** → `get_db()` provides database session
4. **Validation** → Pydantic schema validates request body
5. **Database query** → SQLAlchemy queries PostgreSQL
6. **Response** → Pydantic schema serializes response
7. **JSON returned** → FastAPI converts to JSON

---

## Frontend (Next.js) Deep Dive

### Next.js App Router Structure

Next.js 13+ uses the **App Router** (not Pages Router):

```
app/
├── layout.tsx          # Root layout (wraps everything)
├── page.tsx            # Home page (/)
├── dashboard/
│   ├── layout.tsx      # Dashboard layout (sidebar + header)
│   └── erp/
│       └── buyers/
│           └── page.tsx # Buyers page (/dashboard/erp/buyers)
```

**File-based routing**: The folder structure = URL structure.

### Key Files

#### 1. Root Layout (`app/layout.tsx`)

Wraps entire application:
- Theme provider (dark/light mode)
- Fonts
- Global styles
- Client providers (AuthProvider)

#### 2. API Proxy (`app/api/v1/[...path]/route.ts`)

**What it does**: Proxies all `/api/v1/*` requests to the FastAPI backend.

**Why**: 
- Frontend runs on `localhost:3000`
- Backend runs on `localhost:8000` (or `backend:8000` in Docker)
- Browser can't directly call `backend:8000` (different domain)
- Next.js API route acts as a proxy

**How it works**:
```typescript
// User calls: /api/v1/buyers
// Next.js proxy forwards to: http://backend:8000/api/v1/buyers
// Returns response to frontend
```

#### 3. Middleware (`middleware.ts`)

**Purpose**: Protects routes before they load.

**What it does**:
- Checks for `auth_token` cookie
- If no token and accessing protected route → redirect to login
- If has token and accessing login → redirect to dashboard

**Protected routes**: All routes under `/dashboard` except `/dashboard/login`

#### 4. API Client (`lib/api.ts`)

**Purpose**: Centralized API functions.

**Usage**:
```typescript
import { api } from '@/lib/api';

// Get all buyers
const buyers = await api.buyers.getAll();

// Create buyer
const newBuyer = await api.buyers.create({
  buyer_name: "ABC Corp",
  company_name: "ABC Corporation"
});
```

**Why centralized**: 
- Single place to change API URLs
- Consistent error handling
- Type safety

#### 5. Auth Context (`lib/auth-context.tsx`)

**Purpose**: Manages authentication state globally.

**Provides**:
- `user` - Current user object
- `token` - JWT token
- `login()` - Login function
- `logout()` - Logout function
- `isLoading` - Loading state

**Usage in components**:
```typescript
import { useAuth } from '@/lib/auth-context';

function MyComponent() {
  const { user, logout } = useAuth();
  
  return (
    <div>
      <p>Hello, {user?.full_name}</p>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

### Component Structure

#### Page Components

Located in `app/dashboard/(authenticated)/erp/*/page.tsx`

**Example**: `app/dashboard/(authenticated)/erp/clients/buyers/page.tsx`

```typescript
"use client";  // Client component (can use hooks)

export default function BuyersPage() {
  const [buyers, setBuyers] = useState([]);
  
  useEffect(() => {
    // Fetch buyers on mount
    api.buyers.getAll().then(setBuyers);
  }, []);
  
  return (
    <div>
      {/* UI */}
    </div>
  );
}
```

#### Reusable Components

Located in `components/`

- `components/ui/` - shadcn/ui components (Button, Table, Dialog, etc.)
- `components/layout/` - Layout components (Sidebar, Header)
- `components/shared/` - Shared utilities

### Adding a New Page

**Step 1: Create page file**
```typescript
// app/dashboard/(authenticated)/erp/products/page.tsx
"use client";

import { api } from '@/lib/api';
import { useState, useEffect } from 'react';

export default function ProductsPage() {
  const [products, setProducts] = useState([]);
  
  useEffect(() => {
    api.products.getAll().then(setProducts);
  }, []);
  
  return (
    <div>
      <h1>Products</h1>
      {/* Your UI */}
    </div>
  );
}
```

**Step 2: Add to sidebar navigation**
```typescript
// components/layout/sidebar/nav-main.tsx
{
  title: "Products",
  href: "/dashboard/erp/products",
  icon: PackageIcon
}
```

**Step 3: Add API functions** (if needed)
```typescript
// lib/api.ts
export const api = {
  products: {
    getAll: async () => {
      const response = await fetch('/api/v1/products');
      return response.json();
    }
  }
}
```

### State Management

**Local State**: Use `useState` for component-specific state
```typescript
const [buyers, setBuyers] = useState([]);
```

**Global State**: Use Context API (AuthContext) or Zustand
```typescript
const { user } = useAuth();  // From AuthContext
```

**Server State**: Fetch on mount with `useEffect`
```typescript
useEffect(() => {
  api.buyers.getAll().then(setBuyers);
}, []);
```

---

## Authentication & Authorization

### Authentication Flow

1. **User enters credentials** → Login page
2. **Frontend calls** → `POST /api/v1/auth/login`
3. **Backend validates** → Checks username/password
4. **JWT token created** → Contains user ID and username
5. **Token returned** → Frontend receives `{ access_token: "..." }`
6. **Token stored**:
   - `localStorage.setItem("token", token)` - For API calls
   - `setCookie("auth_token", token)` - For middleware
   - `setUser(userData)` - User info in context
7. **Protected routes** → Middleware checks cookie
8. **API calls** → Include `Authorization: Bearer <token>` header

### JWT Token Structure

```json
{
  "sub": "username",
  "user_id": 1,
  "exp": 1234567890  // Expiration timestamp
}
```

**Token expires**: 7 days (configurable in `backend/core/config.py`)

### Authorization (Permissions)

**User Model Fields**:
- `is_superuser` - Full access to everything
- `department_access` - JSON array of allowed departments

**Department IDs**:
- `"client_info"` - Access to Client Info section
- `"sample_department"` - Access to Sample Department
- `"orders"` - Access to Order Info

**How it works**:
1. User object has `department_access: ["client_info", "sample_department"]`
2. Sidebar navigation (`nav-main.tsx`) filters menu items based on access
3. Backend can also check permissions (not currently implemented)

**Example**:
```typescript
// components/layout/sidebar/nav-main.tsx
const hasAccess = (departmentId: string): boolean => {
  if (user.is_superuser) return true;
  return user.department_access?.includes(departmentId) || false;
};
```

### Protecting Routes

**Frontend**: Middleware automatically redirects unauthenticated users

**Backend**: Add dependency to check token:
```python
from fastapi import Depends, HTTPException
from core.security import decode_token

def get_current_user(authorization: str = Header(None)):
    if not authorization or not authorization.startswith("Bearer "):
        raise HTTPException(401, "Not authenticated")
    
    token = authorization.replace("Bearer ", "")
    payload = decode_token(token)
    if not payload:
        raise HTTPException(401, "Invalid token")
    
    return payload

@router.get("/protected")
def protected_route(user = Depends(get_current_user)):
    return {"message": "You are authenticated"}
```

---

## Where to Edit/Add Features

### Adding a New Database Table

1. **Create model** → `backend/modules/[module]/models/[model].py`
   ```python
   class NewTable(Base):
       __tablename__ = "new_table"
       id = Column(Integer, primary_key=True)
       name = Column(String)
   ```

2. **Import in database init** → Models are auto-discovered if imported
   ```python
   # In main.py or __init__.py
   from modules.[module].models.[model] import NewTable
   ```

3. **Tables auto-created** → On startup, `init_db()` creates all tables

### Adding a New API Endpoint

1. **Add route function** → `backend/modules/[module]/routes/[routes].py`
   ```python
   @router.get("/new-endpoint")
   def get_new_data(db: Session = Depends(get_db)):
       return {"data": "value"}
   ```

2. **Router already registered** → If module router is in `main.py`, endpoint is available

### Adding a New Frontend Page

1. **Create page** → `frontend/app/dashboard/(authenticated)/erp/[section]/page.tsx`
2. **Add to sidebar** → `frontend/components/layout/sidebar/nav-main.tsx`
3. **Add API functions** → `frontend/lib/api.ts` (if needed)

### Adding a New Field to Existing Table

1. **Update model** → `backend/modules/[module]/models/[model].py`
   ```python
   new_field = Column(String, nullable=True)
   ```

2. **Update schema** → `backend/modules/[module]/schemas/[schema].py`
   ```python
   new_field: Optional[str] = None
   ```

3. **Database migration** → Tables auto-update on restart (SQLAlchemy)
   - **Note**: For production, use Alembic migrations (not currently set up)

### Changing UI/Stlying

1. **Page components** → `frontend/app/dashboard/(authenticated)/erp/*/page.tsx`
2. **Reusable components** → `frontend/components/`
3. **Global styles** → `frontend/app/globals.css`
4. **Tailwind classes** → Use Tailwind utility classes

### Changing Business Logic

1. **Backend logic** → `backend/modules/[module]/routes/[routes].py`
2. **Frontend logic** → `frontend/app/dashboard/(authenticated)/erp/*/page.tsx`

### Adding Authentication to Endpoint

```python
from fastapi import Depends, Header
from core.security import decode_token

def get_current_user(authorization: str = Header(None)):
    # ... validation logic
    return user_data

@router.get("/protected")
def protected_endpoint(user = Depends(get_current_user)):
    return {"user": user}
```

---

## Development Setup

### Prerequisites

- Docker & Docker Compose
- Node.js 18+ (for local frontend dev)
- Python 3.11+ (for local backend dev)

### Running with Docker (Recommended)

1. **Start all services**:
   ```bash
   docker-compose up -d
   ```

2. **Services available**:
   - Frontend: http://localhost:3000
   - Backend API: http://localhost:8000
   - Backend Docs: http://localhost:8000/docs
   - PostgreSQL: localhost:5432

3. **View logs**:
   ```bash
   docker-compose logs -f backend
   docker-compose logs -f frontend
   ```

4. **Stop services**:
   ```bash
   docker-compose down
   ```

### Running Locally (Without Docker)

#### Backend

1. **Install dependencies**:
   ```bash
   cd backend
   pip install -r requirements.txt
   ```

2. **Set up PostgreSQL**:
   - Install PostgreSQL
   - Create database: `rmg_erp`
   - Update `backend/core/config.py` with connection details

3. **Run backend**:
   ```bash
   cd backend
   uvicorn main:app --reload --host 0.0.0.0 --port 8000
   ```

#### Frontend

1. **Install dependencies**:
   ```bash
   cd frontend
   npm install
   ```

2. **Set environment variable**:
   ```bash
   # Create .env.local
   BACKEND_URL=http://localhost:8000
   ```

3. **Run frontend**:
   ```bash
   cd frontend
   npm run dev
   ```

### Default Login Credentials

- **Username**: `admin`
- **Password**: `admin`

**Location**: Created by `backend/init_data.py` on first startup

### Database Access

**With Docker**:
```bash
docker exec -it erp_postgres psql -U postgres -d rmg_erp
```

**SQL Queries**:
```sql
-- List all tables
\dt

-- View users
SELECT * FROM users;

-- View buyers
SELECT * FROM buyers;
```

### Environment Variables

**Backend** (in `docker-compose.yml` or `.env`):
- `POSTGRES_USER` - Database user
- `POSTGRES_PASSWORD` - Database password
- `POSTGRES_HOST` - Database host
- `POSTGRES_DB` - Database name
- `SECRET_KEY` - JWT secret key (change in production!)

**Frontend** (in `.env.local`):
- `BACKEND_URL` - Backend API URL

---

## Common Tasks Guide

### Task 1: Add a New Buyer Field

**Step 1**: Update database model
```python
# backend/modules/clients/models/client.py
class Buyer(Base):
    # ... existing fields
    new_field = Column(String, nullable=True)  # Add this
```

**Step 2**: Update schema
```python
# backend/modules/clients/schemas/buyer.py
class BuyerCreate(BaseModel):
    # ... existing fields
    new_field: Optional[str] = None  # Add this
```

**Step 3**: Update frontend form
```typescript
// In buyers page component
<Input
  label="New Field"
  value={formData.new_field}
  onChange={(e) => setFormData({...formData, new_field: e.target.value})}
/>
```

**Step 4**: Restart backend (tables auto-update)

### Task 2: Create a New Module (e.g., "Products")

**Backend**:

1. Create folder structure:
   ```
   backend/modules/products/
   ├── __init__.py
   ├── models/
   │   └── product.py
   ├── routes/
   │   └── products.py
   └── schemas/
       └── product.py
   ```

2. Create model:
   ```python
   # models/product.py
   from core.database import Base
   from sqlalchemy import Column, Integer, String
   
   class Product(Base):
       __tablename__ = "products"
       id = Column(Integer, primary_key=True)
       name = Column(String, nullable=False)
   ```

3. Create schema:
   ```python
   # schemas/product.py
   from pydantic import BaseModel
   
   class ProductCreate(BaseModel):
       name: str
   
   class ProductResponse(BaseModel):
       id: int
       name: str
   ```

4. Create routes:
   ```python
   # routes/products.py
   from fastapi import APIRouter, Depends
   from core import get_db
   from modules.products.models.product import Product
   from modules.products.schemas.product import ProductCreate, ProductResponse
   
   router = APIRouter()
   
   @router.get("/products", response_model=List[ProductResponse])
   def get_products(db: Session = Depends(get_db)):
       return db.query(Product).all()
   ```

5. Export router:
   ```python
   # __init__.py
   from .routes.products import router as products_router
   ```

6. Register in main.py:
   ```python
   from modules.products import products_router
   app.include_router(products_router, prefix="/api/v1", tags=["products"])
   ```

**Frontend**:

1. Add API functions:
   ```typescript
   // lib/api.ts
   export const api = {
     products: {
       getAll: async () => {
         const response = await fetch('/api/v1/products');
         return response.json();
       }
     }
   }
   ```

2. Create page:
   ```typescript
   // app/dashboard/(authenticated)/erp/products/page.tsx
   "use client";
   import { api } from '@/lib/api';
   
   export default function ProductsPage() {
     const [products, setProducts] = useState([]);
     
     useEffect(() => {
       api.products.getAll().then(setProducts);
     }, []);
     
     return <div>{/* UI */}</div>;
   }
   ```

3. Add to sidebar:
   ```typescript
   // components/layout/sidebar/nav-main.tsx
   {
     title: "Products",
     href: "/dashboard/erp/products",
     icon: PackageIcon
   }
   ```

### Task 3: Add Validation to API Endpoint

```python
from pydantic import BaseModel, validator

class BuyerCreate(BaseModel):
    buyer_name: str
    email: str
    
    @validator('email')
    def validate_email(cls, v):
        if '@' not in v:
            raise ValueError('Invalid email')
        return v
```

### Task 4: Add Caching to Endpoint

```python
from core.cache import cache_response, CacheTTL

@router.get("/buyers")
@cache_response(key_prefix="buyers", ttl=CacheTTL.LOOKUP_DATA)
def get_buyers(db: Session = Depends(get_db)):
    return db.query(Buyer).all()
```

### Task 5: Add Search/Filter Functionality

**Backend**:
```python
@router.get("/buyers")
def get_buyers(
    search: str = Query(None),
    status: str = Query(None),
    db: Session = Depends(get_db)
):
    query = db.query(Buyer)
    
    if search:
        query = query.filter(Buyer.buyer_name.ilike(f"%{search}%"))
    if status:
        query = query.filter(Buyer.status == status)
    
    return query.all()
```

**Frontend**:
```typescript
const [search, setSearch] = useState("");

useEffect(() => {
  const url = `/api/v1/buyers${search ? `?search=${search}` : ''}`;
  fetch(url).then(res => res.json()).then(setBuyers);
}, [search]);
```

### Task 6: Add Export to Excel

```typescript
import * as XLSX from 'xlsx';

function exportToExcel(data: any[], filename: string) {
  const ws = XLSX.utils.json_to_sheet(data);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, "Sheet1");
  XLSX.writeFile(wb, `${filename}.xlsx`);
}

// Usage
<Button onClick={() => exportToExcel(buyers, "buyers")}>
  Export Excel
</Button>
```

---

## Quick Reference

### Backend File Locations

| What | Where |
|------|-------|
| Database models | `backend/modules/*/models/*.py` |
| API routes | `backend/modules/*/routes/*.py` |
| Data schemas | `backend/modules/*/schemas/*.py` |
| Database config | `backend/core/database.py` |
| Auth logic | `backend/core/security.py` |
| Settings | `backend/core/config.py` |
| Main app | `backend/main.py` |

### Frontend File Locations

| What | Where |
|------|-------|
| Pages | `frontend/app/dashboard/(authenticated)/erp/*/page.tsx` |
| API client | `frontend/lib/api.ts` |
| Auth context | `frontend/lib/auth-context.tsx` |
| Components | `frontend/components/` |
| Sidebar nav | `frontend/components/layout/sidebar/nav-main.tsx` |
| Middleware | `frontend/middleware.ts` |
| API proxy | `frontend/app/api/v1/[...path]/route.ts` |

### Common URLs

| Service | URL |
|---------|-----|
| Frontend | http://localhost:3000 |
| Backend API | http://localhost:8000 |
| API Docs | http://localhost:8000/docs |
| Login | http://localhost:3000/dashboard/login |
| Dashboard | http://localhost:3000/dashboard/erp |

### Database Tables

| Table | Purpose |
|-------|---------|
| `users` | User accounts |
| `buyers` | Buyer information |
| `suppliers` | Supplier information |
| `contact_persons` | Contact details |
| `shipping_info` | Shipping addresses |
| `banking_info` | Banking details |
| `style_summaries` | Garment styles |
| `style_variants` | Style color variants |
| `required_materials` | Material requirements |
| `samples` | Sample records |
| `sample_operations` | Sample operations |
| `order_management` | Orders |

---

## Troubleshooting

### Backend won't start
- Check PostgreSQL is running
- Check database credentials in `config.py`
- Check logs: `docker-compose logs backend`

### Frontend can't connect to backend
- Check `BACKEND_URL` environment variable
- Check backend is running on port 8000
- Check CORS settings in `main.py`

### Database connection errors
- Verify PostgreSQL is running
- Check connection string in `config.py`
- Check network in Docker: `docker network ls`

### Authentication not working
- Check token in browser DevTools → Application → Cookies
- Check `SECRET_KEY` matches in backend
- Clear cookies and login again

### Tables not created
- Check `init_db()` is called in `main.py`
- Check models are imported
- Check database user has CREATE TABLE permission

---

## Next Steps

1. **Read the code**: Start with `backend/main.py` and `frontend/app/layout.tsx`
2. **Explore a module**: Pick one (e.g., buyers) and trace the flow
3. **Make a small change**: Add a field or modify UI
4. **Check API docs**: Visit http://localhost:8000/docs
5. **Read FastAPI docs**: https://fastapi.tiangolo.com
6. **Read Next.js docs**: https://nextjs.org/docs

---

## Support

For questions or issues:
1. Check this documentation
2. Review code comments
3. Check FastAPI/Next.js official documentation
4. Review error logs in Docker: `docker-compose logs`

---

**Last Updated**: 2024
**Version**: 1.0.0

