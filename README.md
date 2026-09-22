# TravelMemory – Cloud Deployment with Custom Domain, Cloudflare & AWS Load Balancer

## Project Overview

TravelMemory is a full-stack MERN application that allows users to store and view their travel experiences.

The application consists of:

- React.js frontend
- Node.js / Express.js backend
- MongoDB Atlas database
- Nginx web server / reverse proxy
- PM2 process manager
- AWS EC2 instances
- AWS Application Load Balancer
- AWS Certificate Manager (ACM)
- Cloudflare DNS
- HTTPS / SSL

The application was deployed on AWS using multiple EC2 instances to demonstrate a scalable architecture with separate frontend and backend servers.

---

# Live Application

Frontend:

https://yuveshkb.site

Backend API:

https://api.yuveshkb.site/trip

The `/trip` endpoint is used to retrieve the travel experiences stored in MongoDB.

---

# Architecture

The final deployment architecture consists of:

- 2 Frontend EC2 instances
- 2 Backend EC2 instances
- 1 Application Load Balancer
- 2 Target Groups
- MongoDB Atlas
- Cloudflare DNS
- AWS Certificate Manager
- Nginx
- PM2

Architecture:

                         Internet
                            |
                            |
                     Cloudflare DNS
                            |
                            |
                 Application Load Balancer
                            |
                 +----------+----------+
                 |                     |
              HTTPS 443             HTTPS 443
                 |                     |
          Host: yuveshkb.site   Host: api.yuveshkb.site
                 |                     |
                 |                     |
        Frontend Target Group    Backend Target Group
                 |                     |
          +------+-------+       +-----+------+
          |              |       |            |
      Frontend-1     Frontend-2 Backend-1  Backend-2
          |              |       |            |
        Nginx          Nginx    Nginx        Nginx
          |              |       |            |
        React          React    Node.js      Node.js
                               Port 3001     Port 3001
                                    \          /
                                     \        /
                                      MongoDB
                                        Atlas

---

# Technology Stack

## Frontend

- React.js
- JavaScript
- HTML
- CSS
- Axios

## Backend

- Node.js
- Express.js
- Mongoose
- CORS
- dotenv

## Database

- MongoDB Atlas

## Server

- Ubuntu EC2
- Nginx
- PM2

## AWS Services

- Amazon EC2
- Application Load Balancer
- Target Groups
- Security Groups
- AWS Certificate Manager

## DNS / SSL

- Cloudflare
- HTTPS
- ACM SSL Certificate

---

# 1. Source Code

The original TravelMemory project was cloned from GitHub.

Repository:

https://github.com/UnpredictablePrashant/TravelMemory

The repository contains two main directories:

    frontend/
    backend/

The backend uses port `3001`.

The frontend gets its backend URL from:

    REACT_APP_BACKEND_URL

The original project documentation also specifies `MONGO_URI` and `PORT=3001` for the backend.

---

# 2. MongoDB Atlas Setup

MongoDB Atlas was used as the cloud database.

The backend requires a MongoDB connection string.

The backend `.env` file contains:

    MONGO_URI=<MongoDB Atlas connection string>
    PORT=3001

The MongoDB connection string was kept inside `.env` and was not committed to GitHub.

MongoDB Atlas was also configured to allow connections from the EC2 backend servers.

The database stores the travel experiences submitted through the application.

---

# 3. Backend EC2 Instances

Two EC2 instances were created for the backend:

    Backend-1
    Backend-2

Both servers run the same backend application.

The backend directory is:

    /var/www/TravelMemory/TravelMemory/backend

The backend was installed using:

    cd /var/www/TravelMemory/TravelMemory/backend
    npm install

---

# 4. Backend Environment Configuration

A `.env` file was created inside the backend directory.

Example:

    MONGO_URI=<MongoDB Atlas connection string>
    PORT=3001

The actual MongoDB credentials should never be committed to GitHub.

The backend reads the environment variables using dotenv.

---

# 5. Starting the Backend

The backend was initially tested using:

    node index.js

The application starts on:

    http://localhost:3001

The backend provides the `/trip` API.

For example:

    GET /trip

returns the stored travel experiences.

---

# 6. PM2 Process Manager

PM2 was used to keep the Node.js backend running continuously.

The backend was started using:

    pm2 start index.js --name travelmemory-backend

The process was saved:

    pm2 save

PM2 status can be checked using:

    pm2 status

Logs can be viewed using:

    pm2 logs travelmemory-backend

PM2 allows the backend process to continue running after the SSH session is closed.

---

# 7. Nginx on Backend Servers

Nginx was installed on both backend EC2 instances.

Nginx listens on HTTP port 80.

Node.js runs internally on port 3001.

Therefore the architecture is:

    Internet / ALB
           |
         Port 80
           |
         Nginx
           |
      Port 3001
           |
       Node.js

The Nginx configuration forwards requests to:

    http://127.0.0.1:3001

Example configuration:

    server {
        listen 80;
        listen [::]:80;

        server_name _;

        location / {
            proxy_pass http://127.0.0.1:3001;
            proxy_http_version 1.1;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

After configuration:

    sudo nginx -t

was used to test the configuration.

Then Nginx was restarted:

    sudo systemctl restart nginx

---

# 8. Backend API Testing

The backend was tested using:

    curl http://localhost:3001/trip

and:

    curl http://localhost/trip

The API returned:

    []

when there were no travel records.

After a successful trip submission, travel data was stored in MongoDB and returned by the API.

The production API is:

    https://api.yuveshkb.site/trip

---

# 9. Frontend EC2 Instances

Two EC2 instances were created for the frontend:

    Frontend-1
    Frontend-2

Both frontend servers contain the React application.

The frontend directory is:

    /var/www/TravelMemory/frontend

Dependencies were installed using:

    npm install

---

# 10. Frontend Environment Variable

The frontend needs to know the backend API address.

The `.env` file contains:

    REACT_APP_BACKEND_URL=https://api.yuveshkb.site

This is important because the React application uses this value while creating the production build.

The application source contains a backend URL configuration similar to:

    export const baseUrl =
      process.env.REACT_APP_BACKEND_URL ||
      "http://localhost:3001";

---

# 11. React Production Build

After changing the frontend `.env`, the React application was rebuilt.

Command:

    npm run build

This creates the production files inside:

    frontend/build

The build must be regenerated whenever the backend URL changes because React environment variables are embedded into the production build.

---

# 12. Nginx on Frontend Servers

Nginx was configured to serve the React production build.

Example configuration:

    server {
        listen 80;
        listen [::]:80;

        server_name _;

        root /var/www/TravelMemory/frontend/build;
        index index.html;

        location / {
            try_files $uri $uri/ /index.html;
        }
    }

The configuration was tested:

    sudo nginx -t

Then Nginx was restarted:

    sudo systemctl restart nginx

The frontend could then be accessed through port 80.

---

# 13. Frontend Target Group

An AWS Target Group was created:

    travelmemory-frontend-tg

Configuration:

    Target type: Instances
    Protocol: HTTP
    Port: 80

The following instances were registered:

    Frontend-1
    Frontend-2

Health check:

    Protocol: HTTP
    Path: /
    Port: traffic-port
    Success code: 200

The ALB checks both frontend servers to make sure they are available.

---

# 14. Backend Target Group

Another target group was created:

    travelmemory-backend-tg

Configuration:

    Target type: Instances
    Protocol: HTTP
    Port: 80

The following instances were registered:

    Backend-1
    Backend-2

Health check:

    Protocol: HTTP
    Path: /trip
    Port: traffic-port
    Success code: 200

The backend target group uses port 80 because Nginx receives the request on port 80 and forwards it internally to Node.js on port 3001.

---

# 15. Why Port 3001 Is Not Directly Exposed

Node.js runs on:

    3001

However, port 3001 does not need to be publicly exposed.

The request flow is:

    ALB
      |
      | HTTP :80
      v
    Nginx
      |
      | localhost:3001
      v
    Node.js

This provides an additional layer between the public load balancer and the Node.js application.

---

# 16. Application Load Balancer

An Application Load Balancer named:

    travelmemory-alb

was created.

The ALB is internet-facing.

It connects users to the frontend and backend target groups.

The ALB uses HTTP/HTTPS listeners.

---

# 17. HTTP Listener

An HTTP listener was created:

    Port: 80
    Protocol: HTTP

The default action forwards requests to:

    travelmemory-frontend-tg

Therefore normal website requests go to the frontend servers.

---

# 18. Backend Routing Rule

A host-based routing rule was added for:

    api.yuveshkb.site

The rule forwards requests to:

    travelmemory-backend-tg

Therefore:

    yuveshkb.site
            |
            v
    Frontend Target Group

while:

    api.yuveshkb.site
            |
            v
    Backend Target Group

This allows one ALB to handle both frontend and backend traffic.

---

# 19. Cloudflare DNS

Cloudflare was used for DNS management.

The domain used for the deployment is:

    yuveshkb.site

The API subdomain is:

    api.yuveshkb.site

DNS records were configured to point toward the AWS Application Load Balancer.

The routing is:

    yuveshkb.site
          |
          v
    AWS ALB
          |
          v
    Frontend Target Group

and:

    api.yuveshkb.site
          |
          v
    AWS ALB
          |
          v
    Backend Target Group

Cloudflare nameservers were configured at the domain registrar.

---

# 20. SSL Certificate

AWS Certificate Manager was used to obtain an SSL/TLS certificate.

The certificate covers:

    yuveshkb.site

and:

    *.yuveshkb.site

DNS validation was completed through Cloudflare.

The ACM validation record was kept as DNS-only during certificate validation.

After successful validation, the certificate status became:

    Issued

---

# 21. HTTPS Listener

An HTTPS listener was added to the ALB:

    Protocol: HTTPS
    Port: 443

The ACM certificate was attached to this listener.

The default HTTPS action forwards traffic to:

    travelmemory-frontend-tg

A host-based rule sends:

    api.yuveshkb.site

to:

    travelmemory-backend-tg

---

# 22. Final HTTPS Architecture

The final request flow is:

    User Browser
          |
          | HTTPS
          v
    Cloudflare DNS
          |
          v
    AWS Application Load Balancer
          |
          +----------------------------+
          |                            |
          | yuveshkb.site              | api.yuveshkb.site
          |                            |
          v                            v
    Frontend Target Group       Backend Target Group
          |                            |
      +---+---+                    +---+---+
      |       |                    |       |
      v       v                    v       v
    FE-1    FE-2                  BE-1    BE-2
      |       |                    |       |
     Nginx   Nginx                Nginx   Nginx
      |       |                    |       |
     React   React               Node.js Node.js
                                    |
                                    |
                               MongoDB Atlas

---

# 23. Security Groups

Security groups were configured to control traffic between the components.

## ALB Security Group

Allowed:

    HTTP  :80
    HTTPS :443

from the required internet sources.

---

## Frontend Security Group

Allowed:

    HTTP :80

from the ALB security group.

SSH:

    TCP :22

was allowed from the administrator's required IP address.

---

## Backend Security Group

Allowed:

    HTTP :80

from the ALB security group.

SSH:

    TCP :22

was allowed from the administrator's required IP address.

Node.js port `3001` was kept internal and was not required as a public ALB-facing port.

---

# 24. MongoDB Security

MongoDB Atlas was configured to allow the backend EC2 servers to connect.

The backend uses:

    MONGO_URI

for the MongoDB connection.

The database is independent of the EC2 filesystem.

Therefore restarting an EC2 instance does not delete MongoDB Atlas data.

---

# 25. Testing the Deployment

The following tests were performed.

## Frontend Test

Open:

    https://yuveshkb.site

The TravelMemory frontend loads successfully.

---

## Backend Test

Open:

    https://api.yuveshkb.site/trip

The API responds with JSON.

Example when no trips exist:

    []

---

## Add Experience Test

A new travel experience was submitted through:

    https://yuveshkb.site

The browser sent the request to:

    https://api.yuveshkb.site/trip

The browser first sent a CORS preflight:

    OPTIONS /trip

which returned:

    204 No Content

The actual POST request returned:

    200 OK

The backend received the submitted travel information and stored it in MongoDB.

---

# 26. CORS

The backend uses the Express CORS middleware:

    const cors = require('cors');

    app.use(cors());

This allows the React frontend to communicate with the backend API.

The browser may send an OPTIONS request before the actual POST request.

A response such as:

    204 No Content

for OPTIONS is a normal CORS preflight response.

The actual POST request returning:

    200 OK

confirms that the request reached the backend successfully.

---

# 27. Problem Encountered During Deployment

During testing, the frontend initially appeared empty.

The API returned:

    []

This meant the frontend was successfully communicating with the backend, but there were no travel records available through the API.

After submitting a new experience, the backend logs showed the submitted object but only displayed:

    ERROR

The backend controller contained:

    catch(error){
        console.log('ERROR')
        res.send('SOMETHING WENT WRONG')
    }

Therefore the actual error was hidden.

The catch block was changed temporarily to:

    catch(error){
        console.log('ERROR:', error)
        res.send('SOMETHING WENT WRONG')
    }

This allowed the actual error to be identified and corrected.

After fixing the issue, trip submission and retrieval worked correctly.

---

# 28. Backend Controller

The backend trip controller creates a MongoDB document using the submitted request body.

The controller receives fields such as:

    tripName
    startDateOfJourney
    endDateOfJourney
    nameOfHotels
    placesVisited
    totalCost
    tripType
    experience
    image
    shortDescription
    featured

The document is then saved using:

    await tripDetail.save()

After a successful save, the backend responds:

    Trip added Successfully

---

# 29. Final API Flow

When the user submits an experience:

    React Form
        |
        v
    Axios / HTTP Request
        |
        v
    https://api.yuveshkb.site/trip
        |
        v
    AWS ALB
        |
        v
    Backend Target Group
        |
        v
    Backend EC2
        |
        v
    Nginx :80
        |
        v
    Node.js :3001
        |
        v
    Express Route
        |
        v
    Trip Controller
        |
        v
    Mongoose
        |
        v
    MongoDB Atlas

---

# 30. Final Application Flow

When a user opens the website:

    Browser
       |
       v
    https://yuveshkb.site
       |
       v
    Cloudflare
       |
       v
    AWS ALB
       |
       v
    Frontend Target Group
       |
       +----------+
       |          |
       v          v
    Frontend-1  Frontend-2
       |          |
       v          v
      Nginx      Nginx
       |          |
       v          v
     React      React

When React needs travel data:

    React
      |
      v
    https://api.yuveshkb.site/trip
      |
      v
    AWS ALB
      |
      v
    Backend Target Group
      |
      +----------+
      |          |
      v          v
    Backend-1  Backend-2
      |          |
     Nginx      Nginx
      |          |
    Node.js    Node.js
      |          |
      +-----+----+
            |
            v
       MongoDB Atlas

---

# 31. High Availability

Two frontend instances and two backend instances were used.

Frontend:

    Frontend-1
    Frontend-2

Backend:

    Backend-1
    Backend-2

The ALB distributes traffic between healthy instances.

If one frontend instance becomes unhealthy, the target group health check can detect it and the ALB can stop sending traffic to that unhealthy target.

Similarly, backend instances are monitored using the `/trip` health check.

---

# 32. Health Checks

Frontend health check:

    GET /

Expected response:

    HTTP 200

Backend health check:

    GET /trip

Expected response:

    HTTP 200

For the backend, `/trip` was used as the health-check endpoint because it is an existing API endpoint that responds successfully when the backend and its database connection are functioning.

---

# 33. Useful Commands

## Check PM2

    pm2 status

## View logs

    pm2 logs travelmemory-backend

## Restart backend

    pm2 restart travelmemory-backend

## Save PM2 configuration

    pm2 save

## Check Nginx

    sudo systemctl status nginx

## Restart Nginx

    sudo systemctl restart nginx

## Test Nginx configuration

    sudo nginx -t

## Test backend directly

    curl http://localhost:3001/trip

## Test backend through Nginx

    curl http://localhost/trip

## Check listening ports

    sudo ss -tulpn

---

# 34. Deployment Steps Summary

The complete deployment process was:

1. Created AWS EC2 instances.
2. Created two frontend instances.
3. Created two backend instances.
4. Installed Node.js and required dependencies.
5. Cloned the TravelMemory GitHub repository.
6. Installed backend dependencies.
7. Created backend `.env`.
8. Connected backend to MongoDB Atlas.
9. Started backend using PM2.
10. Configured Nginx as a reverse proxy.
11. Tested backend API.
12. Installed frontend dependencies.
13. Configured `REACT_APP_BACKEND_URL`.
14. Created React production build.
15. Configured Nginx to serve the React build.
16. Created frontend target group.
17. Registered Frontend-1 and Frontend-2.
18. Created backend target group.
19. Registered Backend-1 and Backend-2.
20. Created Application Load Balancer.
21. Added HTTP listener.
22. Added host-based routing for the backend API.
23. Configured Cloudflare DNS.
24. Connected the custom domain to the ALB.
25. Requested an ACM certificate.
26. Completed DNS validation.
27. Added HTTPS listener on port 443.
28. Attached the ACM certificate.
29. Updated frontend API URL to HTTPS.
30. Rebuilt both frontend instances.
31. Tested the frontend.
32. Tested the backend API.
33. Tested CORS.
34. Tested adding a new travel experience.
35. Verified MongoDB data insertion.
36. Verified the final HTTPS deployment.

---

# 35. Final URLs

Application:

    https://yuveshkb.site

Backend API:

    https://api.yuveshkb.site/trip

Source Repository:

    https://github.com/UnpredictablePrashant/TravelMemory

---

# 36. Final Result

The TravelMemory MERN application was successfully deployed on AWS using a multi-server architecture.

The final system contains:

    2 Frontend EC2 Instances
    2 Backend EC2 Instances
    1 Application Load Balancer
    2 Target Groups
    MongoDB Atlas
    Nginx
    PM2
    Cloudflare DNS
    AWS ACM SSL Certificate
    HTTPS

The frontend is available through the custom domain:

    https://yuveshkb.site

The backend API is available through:

    https://api.yuveshkb.site/trip

The application successfully supports:

    Frontend access
    Backend API access
    HTTPS communication
    Adding travel experiences
    Retrieving travel experiences
    MongoDB persistence
    Load balancing
    Multiple frontend servers
    Multiple backend servers
    Health checks
    DNS-based routing
    SSL/TLS encryption
