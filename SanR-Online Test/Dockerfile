# Multi-stage build - lightweight nginx image
FROM nginx:alpine

LABEL maintainer="CodeAssess Team"
LABEL description="CodeAssess - Online Test Platform Container"

# Remove default nginx files
RUN rm -rf /usr/share/nginx/html/*

# Copy HTML files
COPY index.html /usr/share/nginx/html/
COPY admin.html /usr/share/nginx/html/
COPY 404.html /usr/share/nginx/html/

# Copy CSS files
COPY css/ /usr/share/nginx/html/css/

# Copy data files (MCQ questions, coding questions)
COPY data/ /usr/share/nginx/html/data/

# Copy dataconnect files
COPY dataconnect/ /usr/share/nginx/html/dataconnect/

# Copy Firebase configuration files
COPY firebase.json /usr/share/nginx/html/
COPY firestore.rules /usr/share/nginx/html/
COPY firestore.indexes.json /usr/share/nginx/html/

# Copy custom nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Expose ports
EXPOSE 80 443

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
