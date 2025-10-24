# Tiltfile for LibreChat Development
# This orchestrates the full development environment with live reload

# Build the development Docker image
docker_build(
    'librechat-dev',
    context='.',
    dockerfile='./Dockerfile.dev',
    dockerfile_contents=read_file('./Dockerfile.dev'),
    ignore=['.dockerignore.dev'],
    live_update=[
        # Sync source code changes into the container
        sync('./api', '/app/api'),
        sync('./client', '/app/client'),
        sync('./packages', '/app/packages'),
        sync('./config', '/app/config'),

        # Restart backend on API changes
        run('echo "Backend files changed, nodemon will auto-restart"', trigger=['./api']),

        # Vite handles frontend HMR automatically, no restart needed
        run('echo "Frontend files changed, Vite HMR active"', trigger=['./client']),
    ],
)

# Override the 'api' service from docker-compose to use our dev image
k8s_yaml(blob("""
apiVersion: v1
kind: Service
metadata:
  name: librechat-dev
spec:
  ports:
    - name: backend
      port: 3080
      targetPort: 3080
    - name: frontend
      port: 3090
      targetPort: 3090
  selector:
    app: librechat-dev
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: librechat-dev
  labels:
    app: librechat-dev
spec:
  selector:
    matchLabels:
      app: librechat-dev
  template:
    metadata:
      labels:
        app: librechat-dev
    spec:
      containers:
      - name: librechat
        image: librechat-dev
        ports:
        - containerPort: 3080
          name: backend
        - containerPort: 3090
          name: frontend
        env:
        - name: NODE_ENV
          value: "development"
        - name: HOST
          value: "0.0.0.0"
        - name: PORT
          value: "3090"
        - name: MONGO_URI
          value: "mongodb://mongodb:27017/LibreChat"
        - name: MEILI_HOST
          value: "http://meilisearch:7700"
        - name: RAG_API_URL
          value: "http://rag_api:8000"
        volumeMounts:
        - name: env
          mountPath: /app/.env
          subPath: .env
        - name: images
          mountPath: /app/client/public/images
        - name: uploads
          mountPath: /app/uploads
        - name: logs
          mountPath: /app/logs
      volumes:
      - name: env
        hostPath:
          path: ./.env
      - name: images
        hostPath:
          path: ./images
      - name: uploads
        hostPath:
          path: ./uploads
      - name: logs
        hostPath:
          path: ./logs
"""))

# Port forward for local access
k8s_resource(
    'librechat-dev',
    port_forwards=[
        '3080:3080',  # Backend API
        '3090:3090',  # Vite dev server
    ],
    labels=['librechat'],
)

# Configure existing services from docker-compose
k8s_resource('mongodb', labels=['database'])
k8s_resource('meilisearch', labels=['search'])
k8s_resource('vectordb', labels=['database'])
k8s_resource('rag_api', labels=['ai'])

# Set resource ordering
k8s_resource('librechat-dev', resource_deps=['mongodb', 'meilisearch', 'rag_api'])

# Print helpful information
print("""
╔════════════════════════════════════════════════════════════════╗
║                   LibreChat Dev Environment                    ║
╚════════════════════════════════════════════════════════════════╝

🚀 Starting LibreChat in development mode with Tilt

Frontend (Vite):  http://localhost:3090  (with HMR)
Backend API:      http://localhost:3080  (with nodemon)

📝 Live reload enabled:
   • Frontend: Vite HMR (instant)
   • Backend: Nodemon auto-restart
   • Packages: Rebuild on change

🔧 Dependencies:
   • MongoDB:      mongodb:27017
   • Meilisearch:  http://meilisearch:7700
   • RAG API:      http://rag_api:8000

💡 Tips:
   • Edit files in ./api, ./client, or ./packages
   • Changes sync automatically to the container
   • Check Tilt UI for logs and status
   • Press 'space' to open Tilt web UI

""")
