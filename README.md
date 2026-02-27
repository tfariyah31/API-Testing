# API Testing - Postman


[![API Tests](https://github.com/tfariyah31/Postman-API-Testing-Demo/actions/workflows/main.yml/badge.svg)](https://github.com/tfariyah31/Postman-API-Testing-Demo/actions/workflows/main.yml)

This project demonstrates API testing using **Postman**, **Newman**, and **GitHub Actions**, tested against a locally hosted Node.js backend application.

## Project Structure
```
Postman-API-Testing-Demo/
├── backend/
│   └── Node.js backend app
├── postman-tests/
│   ├── SimpleWebApp_API.json
│   ├── IterationSimpleWebApp_API.json
│   ├── QA_environment.json
│   └── README.md 
├── newman/
│   └── SimpleWebApp API.html
│   └── Iteration SimpleWebApp.html
├── .github/
│   └── workflows/
│       └── api-tests.yml 
├── README.md

```


## Getting the Backend Up and Running

The backend is a simple Node.js + Express app that supports user registration, login and a product list.

### Tech Stack
`Node.js` · ` Express.js` · `MongoDB` · `Mongoose` · `JWT` 

### Prerequisites

- [Node.js](https://nodejs.org/) v16 or higher
- [MongoDB](https://www.mongodb.com/try/download/community) (local) or a [MongoDB Atlas](https://www.mongodb.com/atlas) cloud URI
- [Git](https://git-scm.com/)


### Setup Instructions
##### 1. Clone the Repository

```bash
git clone https://github.com/tfariyah31/SimpleWebApp.git
cd SimpleWebApp
```

#### 2. Set Up the Backend

```bash
cd backend
npm install
```

Create a `.env` file in the `backend` folder:

```env
PORT=5001
MONGO_URI=mongodb://localhost:27017/mywebapp
JWT_SECRET=your_secret_key_here
```

> Replace `MONGO_URI` with your MongoDB Atlas connection string if using a cloud database. Use a strong, unique value for `JWT_SECRET` — never commit it to version control.

Start the backend server:

```bash
node server.js
```

The backend will be available at `http://localhost:5001`.

---

## Running API Tests Locally

### Tools Used
`Postman` · `Newman + newman-reporter-htmlextra` · `GitHub Actions`

### What’s Included
- Postman test scripts for a simple API app (backend)
- Newman HTML reports
- CI workflow to run tests on every push

### ✅ Prerequisites
Install Newman globally if you haven’t:

```bash
npm install -g newman newman-reporter-htmlextra
```
### Run the Postman collection:
Main API test suite:
```bash
newman run postman-tests/SimpleWebApp_API.json \
  -e postman-tests/QA_environment.json \
  -r cli,htmlextra
```
Rate limit / iteration tests (runs the collection 3 times):

```bash
newman run postman-tests/IterationSimpleWebApp_API.json \
  -e postman-tests/QA_environment.json \
  -n 3 \
  -r cli,htmlextra
```

HTMLExtra reports are saved to the ```newman/``` folder by default. To specify a custom output path, append ```--reporter-htmlextra-export newman/my-report.html``` to either command.

## Endpoints Under Test

| Method | Endpoint | Auth Required | Description |
|--------|----------|:---:|-------------|
| `POST` | `/auth/register` | ❌ | Create a new user account |
| `POST` | `/auth/login` | ❌ | Authenticate and receive JWT tokens |
| `POST` | `/auth/refresh` | ❌ | Exchange refresh token for new access token |
| `POST` | `/auth/logout` | ✅ | Invalidate session |
| `GET` | `/products` | ✅ | Fetch all products |
| `POST` | `/products` | ✅ | Create a new product |
| `GET` | `/products/:id` | ✅ | Fetch single product by ID |
| `PUT` | `/products/:id` | ✅ | Update product information |
| `DELETE` | `/products/:id` | ✅ | Delete a product |

---

### Test Coverage
[Test Coverage Details](./postman/README.md)

### 🔄 Continuous Integration with GitHub Actions
This project includes a GitHub Actions workflow that:

- Installs Node.js and Newman
- Runs the Postman test suite on every push or pull request
- Uploads the Newman HTML report as an artifact

You can find the workflow file here:
.github/workflows/api-tests.yml

---

### 📊 Newman Report Preview

<img src="Newman_Report.png" width="600" height="350">



### 👨‍💻 Author
👤 Tasnim Fariyah
🔗 [Github](https://github.com/tfariyah31)
🔗 [LinkedIn](https://www.linkedin.com/in/tasnim-fariyah/)

---

