#-----------------
# Building the app
#-----------------
    FROM node:17-alpine AS build

    WORKDIR /app/frontend
    
    COPY package*.json ./
    
    RUN npm install
    
    COPY . .
    
    ARG VITE_API_URL
    
    ENV VITE_API_URL=$VITE_API_URL
    
    RUN npm run build
    
    #----------------
    # Serving the app
    #----------------
    FROM nginx:alpine
    
    COPY --from=build /app/frontend/dist /usr/share/nginx/html
    
    COPY nginx.conf /etc/nginx/conf.d/default.conf
    
    EXPOSE 80
    
    CMD [ "nginx", "-g", "daemon off;" ]