# TOMATO - Food Ordering Website

This repository hosts the source code for TOMATO, a dynamic food ordering website built with the MERN Stack. It offers a user-friendly platform for seamless online food ordering.

## Demo

- User Panel: [https://food-delivery-frontend-s2l9.onrender.com/](https://food-delivery-frontend-s2l9.onrender.com/)
- Admin Panel: [https://food-delivery-admin-wrme.onrender.com/](https://food-delivery-admin-wrme.onrender.com/)

## Features

- User Panel
- Admin Panel
- JWT Authentication
- Password Hashing with Bcrypt
- Stripe Payment Integration
- Login/Signup
- Logout
- Add to Cart
- Place Order
- Order Management
- Products Management
- Filter Food Products
- Login/Signup
- Authenticated APIs
- REST APIs
- Role-Based Identification
- Beautiful Alerts

## Screenshots

![Hero](https://i.ibb.co/59cwY75/food-hero.png)
- Hero Section

![Products](https://i.ibb.co/JnNQPyQ/food-products.png)
- Products Section

![Cart](https://i.ibb.co/t2LrQ8p/food-cart.png)
- Cart Page

![Login](https://i.ibb.co/s6PgwkZ/food-login.png)
- Login Popup

## Run Locally

Clone the project

```bash
    git clone https://github.com/Mshandev/Food-Delivery
```
Go to the project directory

```bash
    cd Food-Delivery
```
Install dependencies (frontend)

```bash
    cd frontend
    npm install
```
Install dependencies (admin)

```bash
    cd admin
    npm install
```
Install dependencies (backend)

```bash
    cd backend
    npm install
```
Setup Environment Vaiables

```Make .env file in "backend" folder and store environment Variables
  JWT_SECRET=YOUR_SECRET_TEXT
  SALT=YOUR_SALT_VALUE
  MONGO_URL=YOUR_DATABASE_URL
  STRIPE_SECRET_KEY=YOUR_KEY
 ```

Setup the Frontend and Backend URL
   - App.jsx in Admin folder
      const url = YOUR_BACKEND_URL
     
  - StoreContext.js in Frontend folder
      const url = YOUR_BACKEND_URL

  - orderController in Backend folder
      const frontend_url = YOUR_FRONTEND_URL 

Start the Backend server

```bash
    nodemon server.js
```

Start the Frontend server

```bash
    npm start
```

Start the Backend server

```bash
    npm start
```
## Tech Stack
* [React](https://reactjs.org/)
* [Node.js](https://nodejs.org/en)
* [Express.js](https://expressjs.com/)
* [Mongodb](https://www.mongodb.com/)
* [Stripe](https://stripe.com/)
* [JWT-Authentication](https://jwt.io/introduction)
* [Multer](https://www.npmjs.com/package/multer)

## Deployment

The application is deployed on Render.

## Contributing

Contributions are always welcome!
Just raise an issue, and we will discuss it.

## Feedback

If you have any feedback, please reach out to me [here](https://www.linkedin.com/in/muhammad-shan-full-stack-developer/)

## Run In Containers On Another Server

1. Copy backend env:
   - `cp backend/.env.example backend/.env`
   - Update at least: `MONGO_URL`, `JWT_SECRET`, `STRIPE_SECRET_KEY`
   - Set `FRONTEND_URL` to your public frontend domain (example `https://food.trungdevops.vn`)
   - Set `CORS_ORIGINS` to both frontend and admin domains (example `https://food.trungdevops.vn,https://admin.food.trungdevops.vn`)

2. Build and run with a public API URL:

```bash
# Linux/macOS
export VITE_API_URL=https://food.trungdevops.vn
docker compose up -d --build
```

```powershell
# Windows PowerShell
$env:VITE_API_URL="https://food.trungdevops.vn"
docker compose up -d --build
```

`frontend` and `admin` are static builds, so `VITE_API_URL` must be correct when building images.

## Kubernetes Deployment (backend + frontend + admin)

Kubernetes manifests are in `k8s/`:
- `00-namespace.yaml`
- `01-backend-configmap.yaml`
- `02-backend-secret.yaml`
- `03-backend.yaml`
- `04-frontend.yaml`
- `05-admin.yaml`
- `06-ingress.yaml`

### 1) Build and push images

```bash
docker build -t your-dockerhub-user/food-backend:latest -f Dockerfile .
docker build -t your-dockerhub-user/food-frontend:latest --build-arg VITE_API_URL=https://food.trungdevops.vn ./frontend
docker build -t your-dockerhub-user/food-admin:latest --build-arg VITE_API_URL=https://food.trungdevops.vn ./admin

docker push your-dockerhub-user/food-backend:latest
docker push your-dockerhub-user/food-frontend:latest
docker push your-dockerhub-user/food-admin:latest
```

### 2) Update manifests

- Replace `your-dockerhub-user/...` image names in:
  - `k8s/03-backend.yaml`
  - `k8s/04-frontend.yaml`
  - `k8s/05-admin.yaml`
- Update domains in:
  - `k8s/01-backend-configmap.yaml`
  - `k8s/06-ingress.yaml`
- Update secrets in:
  - `k8s/02-backend-secret.yaml`

### 3) Apply to cluster

```bash
kubectl apply -f k8s/
```

### 4) DNS mapping

Point these DNS records to your Ingress public IP:
- `food.trungdevops.vn` -> frontend + backend routes (`/api`, `/images`)
- `admin.food.trungdevops.vn` -> admin

With this setup:
- frontend calls backend via `https://food.trungdevops.vn`
- admin calls backend via `https://food.trungdevops.vn`
- backend allows CORS for both frontend/admin domains
- Stripe redirect returns to `FRONTEND_URL`
