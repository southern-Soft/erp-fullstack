# Quick Start Guide - Southern Apparels ERP

## 🚀 Getting Started in 5 Minutes

### Step 1: Start the System

```bash
# Navigate to project root
cd erp_southern_final/final/erp_southern_final/erp_southern_final/shad-theme-dashboard

# Start all services (Database, Backend, Frontend)
docker-compose up -d

# Wait 30 seconds for services to initialize
```

### Step 2: Access the Application

- **Frontend**: http://localhost:3000
- **Backend API Docs**: http://localhost:8000/docs
- **Login Page**: http://localhost:3000/dashboard/login

### Step 3: Login

- **Username**: `admin`
- **Password**: `admin`

---

## 📁 Where to Find Things

### I want to...

#### Add a new field to Buyers table
1. Edit: `backend/modules/clients/models/client.py` (add field to `Buyer` class)
2. Edit: `backend/modules/clients/schemas/buyer.py` (add to schema)
3. Edit: Frontend form in `frontend/app/dashboard/(authenticated)/erp/clients/buyers/page.tsx`
4. Restart backend: `docker-compose restart backend`

#### Create a new page (e.g., "Products")
1. Create: `frontend/app/dashboard/(authenticated)/erp/products/page.tsx`
2. Add API: `frontend/lib/api.ts` (add `products` section)
3. Add to sidebar: `frontend/components/layout/sidebar/nav-main.tsx`
4. Create backend: Follow pattern in `backend/modules/clients/`

#### Change database connection
1. Edit: `backend/core/config.py` (update `DATABASE_URL`)
2. Or set environment variables in `docker-compose.yml`

#### Add authentication to an endpoint
```python
from fastapi import Depends, Header
from core.security import decode_token

def get_current_user(authorization: str = Header(None)):
    token = authorization.replace("Bearer ", "")
    payload = decode_token(token)
    if not payload:
        raise HTTPException(401, "Not authenticated")
    return payload

@router.get("/protected")
def protected_route(user = Depends(get_current_user)):
    return {"user": user}
```

#### Change UI styling
- Global styles: `frontend/app/globals.css`
- Component styles: Use Tailwind classes in component files
- Theme: `frontend/lib/themes.ts`

#### View database
```bash
# Connect to PostgreSQL
docker exec -it erp_postgres psql -U postgres -d rmg_erp

# List tables
\dt

# View data
SELECT * FROM buyers;
```

---

## 🔍 Understanding the Flow

### How a page works:

1. **User visits** `/dashboard/erp/clients/buyers`
2. **Page loads** → `frontend/app/dashboard/(authenticated)/erp/clients/buyers/page.tsx`
3. **Component calls** → `api.buyers.getAll()` from `frontend/lib/api.ts`
4. **Request goes to** → `/api/v1/buyers` (Next.js API route)
5. **Next.js proxies to** → `http://backend:8000/api/v1/buyers`
6. **Backend route** → `backend/modules/clients/routes/buyers.py` → `get_buyers()`
7. **Database query** → `db.query(Buyer).all()`
8. **Response flows back** → Frontend → Component updates → UI renders

### File relationships:

```
Frontend Page
  ↓ uses
API Client (lib/api.ts)
  ↓ calls
Next.js API Route (app/api/v1/[...path]/route.ts)
  ↓ proxies to
Backend Route (modules/*/routes/*.py)
  ↓ uses
Database Model (modules/*/models/*.py)
  ↓ queries
PostgreSQL Database
```

---

## 🛠️ Common Commands

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f backend
docker-compose logs -f frontend

# Restart a service
docker-compose restart backend
docker-compose restart frontend

# Rebuild after code changes
docker-compose up -d --build

# Access database
docker exec -it erp_postgres psql -U postgres -d rmg_erp

# Access backend container
docker exec -it erp_backend bash

# Access frontend container
docker exec -it erp_frontend sh
```

---

## 📚 Key Concepts

### Backend (FastAPI)
- **Models** = Database tables (SQLAlchemy)
- **Schemas** = Data validation (Pydantic)
- **Routes** = API endpoints
- **Dependencies** = Reusable functions (like `get_db()`)

### Frontend (Next.js)
- **Pages** = Routes (file-based routing)
- **Components** = Reusable UI pieces
- **API Client** = Functions to call backend
- **Context** = Global state (AuthContext)

### Database
- **Tables** = Created from SQLAlchemy models
- **Relationships** = Foreign keys between tables
- **Auto-created** = On backend startup

---

## 🐛 Troubleshooting

| Problem | Solution |
|---------|----------|
| Can't connect to backend | Check `docker-compose ps` - all services running? |
| Database errors | Check PostgreSQL is running: `docker-compose logs db` |
| Frontend shows errors | Check browser console (F12) |
| Changes not appearing | Restart service: `docker-compose restart [service]` |
| Login not working | Clear cookies, check token in DevTools |

---

## 📖 Full Documentation

See `DOCUMENTATION.md` for complete details on:
- Architecture deep dive
- Database structure
- All modules explained
- Advanced features
- Development best practices

