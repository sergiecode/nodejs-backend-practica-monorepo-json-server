# JSON Server Micro-Backend for Angular + Nx Monorepo

A lightweight mock REST API backend using JSON Server, designed to work seamlessly with Angular frontend applications in an Nx monorepo architecture.

## Overview

This micro-backend provides a RESTful API for managing courses, students, and enrollments. It's perfect for prototyping, development, and testing your Angular applications without the need for a complex backend setup.

## Features

- 🚀 **Quick Setup**: Ready-to-use JSON database with sample data
- 📚 **Educational Content**: Pre-loaded with courses and student data
- 🔄 **Full CRUD Operations**: Create, Read, Update, Delete for all entities
- 🌐 **RESTful API**: Standard HTTP methods and endpoints
- 🔧 **Development Ready**: Perfect for Angular development and testing

## API Endpoints

### Courses
- `GET /courses` - Get all courses
- `GET /courses/:id` - Get a specific course
- `POST /courses` - Create a new course
- `PUT /courses/:id` - Update a course
- `DELETE /courses/:id` - Delete a course

### Students
- `GET /students` - Get all students
- `GET /students/:id` - Get a specific student
- `POST /students` - Create a new student
- `PUT /students/:id` - Update a student
- `DELETE /students/:id` - Delete a student

### Enrollments
- `GET /enrollments` - Get all enrollments
- `GET /enrollments/:id` - Get a specific enrollment
- `POST /enrollments` - Create a new enrollment
- `PUT /enrollments/:id` - Update an enrollment
- `DELETE /enrollments/:id` - Delete an enrollment

## Quick Start

### Prerequisites
- Node.js (v14 or higher)
- npm or yarn

### Installation & Running

1. **Clone or download this backend**
   ```bash
   git clone <repository-url>
   cd node_json_server_db
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Start the server**
   ```bash
   npm start
   ```

The server will start on `http://localhost:3000` by default.

## Integration with Angular + Nx Monorepo

### Step 1: Setup Your Nx Monorepo

If you don't have an Nx workspace yet:

```bash
# Create a new Nx workspace
npx create-nx-workspace@latest my-workspace --preset=angular-monorepo

# Navigate to your workspace
cd my-workspace
```

### Step 2: Create Your Angular Application

```bash
# Generate a new Angular app
nx generate @nx/angular:application my-app

# Or generate a library for shared services
nx generate @nx/angular:library shared-services
```

### Step 3: Create Services to Connect to the Backend

Create a service to interact with your JSON Server backend:

```bash
# Generate a service
nx generate @nx/angular:service services/courses --project=my-app
```

Example service implementation:

```typescript
// apps/my-app/src/app/services/courses.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

export interface Course {
  id: string;
  title: string;
  description: string;
  teacher: string;
}

@Injectable({
  providedIn: 'root'
})
export class CoursesService {
  private apiUrl = 'http://localhost:3000/courses';

  constructor(private http: HttpClient) {}

  getCourses(): Observable<Course[]> {
    return this.http.get<Course[]>(this.apiUrl);
  }

  getCourse(id: string): Observable<Course> {
    return this.http.get<Course>(`${this.apiUrl}/${id}`);
  }

  createCourse(course: Omit<Course, 'id'>): Observable<Course> {
    return this.http.post<Course>(this.apiUrl, course);
  }

  updateCourse(id: string, course: Course): Observable<Course> {
    return this.http.put<Course>(`${this.apiUrl}/${id}`, course);
  }

  deleteCourse(id: string): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

### Step 4: Configure HTTP Client

Make sure to import `HttpClientModule` in your app module:

```typescript
// apps/my-app/src/app/app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HttpClientModule } from '@angular/common/http';

import { AppComponent } from './app.component';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, HttpClientModule],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

### Step 5: Handle CORS (if needed)

If you encounter CORS issues, you can start the JSON Server with CORS enabled:

```bash
# Add this to package.json scripts
"start:cors": "json-server --watch db.json --middlewares ./cors-middleware.js"
```

Create a `cors-middleware.js` file:

```javascript
module.exports = (req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept, Authorization');
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
  
  if (req.method === 'OPTIONS') {
    res.sendStatus(200);
  } else {
    next();
  }
};
```

### Step 6: Nx Development Workflow

```bash
# Start your backend
npm start

# In another terminal, start your Angular app
nx serve my-app

# Run tests
nx test my-app

# Build for production
nx build my-app

# Run linting
nx lint my-app
```

## Development Workflow

### Concurrent Development
You can run both the backend and frontend simultaneously:

```bash
# Terminal 1: Start JSON Server
cd node_json_server_db
npm start

# Terminal 2: Start Angular app
cd my-nx-workspace
nx serve my-app
```

### Environment Configuration

Create environment files for different stages:

```typescript
// apps/my-app/src/environments/environment.ts
export const environment = {
  production: false,
  apiUrl: 'http://localhost:3000'
};

// apps/my-app/src/environments/environment.prod.ts
export const environment = {
  production: true,
  apiUrl: 'https://your-production-api.com'
};
```

Update your service to use the environment:

```typescript
import { environment } from '../environments/environment';

@Injectable({
  providedIn: 'root'
})
export class CoursesService {
  private apiUrl = `${environment.apiUrl}/courses`;
  // ... rest of the service
}
```

## Sample Data Structure

The `db.json` file contains the following sample data:

### Courses
```json
{
  "id": "1",
  "title": "Introducción a Angular",
  "description": "Curso básico para empezar con Angular",
  "teacher": "Sergie Code"
}
```

### Students
```json
{
  "id": "1",
  "name": "Carlos López",
  "email": "carlos@example.com"
}
```

### Enrollments
```json
{
  "id": "1",
  "studentId": 1,
  "courseId": 1,
  "date": "2025-09-04"
}
```

## Advanced Features

### Custom Routes
You can add custom routes by creating a `routes.json` file:

```json
{
  "/api/*": "/$1",
  "/courses/:id/students": "/enrollments?courseId=:id"
}
```

Then start the server with:
```bash
json-server --watch db.json --routes routes.json
```

### Middleware
Add custom middleware for authentication, logging, etc.:

```javascript
// middleware.js
module.exports = (req, res, next) => {
  console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
  next();
};
```

Start with middleware:
```bash
json-server --watch db.json --middlewares middleware.js
```

## Deployment

### Backend Deployment
You can deploy this JSON Server to platforms like:
- Heroku
- Vercel
- Railway
- Render

### Frontend Deployment
Your Nx Angular app can be deployed to:
- Netlify
- Vercel
- AWS S3 + CloudFront
- Firebase Hosting

## Nx Commands Reference

```bash
# Generate new app
nx generate @nx/angular:application my-app

# Generate new library
nx generate @nx/angular:library my-lib

# Generate component
nx generate @nx/angular:component my-component --project=my-app

# Generate service
nx generate @nx/angular:service services/my-service --project=my-app

# Serve application
nx serve my-app

# Build application
nx build my-app

# Test application
nx test my-app

# Lint application
nx lint my-app

# Show dependency graph
nx graph
```

## Troubleshooting

### Common Issues

1. **Port Already in Use**
   ```bash
   # Use a different port
   json-server --watch db.json --port 3001
   ```

2. **CORS Issues**
   - Use the CORS middleware shown above
   - Or use Angular proxy configuration

3. **Data Not Persisting**
   - Check file permissions on `db.json`
   - Ensure the file is not read-only

### Angular Proxy Configuration

Create `proxy.conf.json` in your Angular app root:

```json
{
  "/api/*": {
    "target": "http://localhost:3000",
    "secure": true,
    "changeOrigin": true,
    "logLevel": "debug"
  }
}
```

Update `angular.json`:
```json
"serve": {
  "builder": "@angular-devkit/build-angular:dev-server",
  "options": {
    "proxyConfig": "proxy.conf.json"
  }
}
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

ISC License - see package.json for details

## Author

**Sergie Code** - Backend setup and configuration

---

Happy coding! 🚀 This micro-backend is perfect for rapid Angular development in your Nx monorepo setup.
