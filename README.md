# All-is-Well
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
  annotations:
    description: "Production-grade deployment for NGINX"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
      annotations:
        sidecar: "true"
    spec:
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - nginx
              topologyKey: "kubernetes.io/hostname"
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - nginx
                topologyKey: "kubernetes.io/hostname"
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: kubernetes.io/e2e-az-name
                    operator: In
                    values:
                      - us-west-1a
      tolerations:
        - key: "key1"
          operator: "Equal"
          value: "value1"
          effect: "NoSchedule"
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
          resources:
            limits:
              cpu: "500m"
              memory: "256Mi"
            requests:
              cpu: "250m"
              memory: "128Mi"
          livenessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 3
            periodSeconds: 5
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 3
            periodSeconds: 5
          env:
            - name: ENVIRONMENT
              value: "production"
            - name: LOG_LEVEL
              value: "debug"
          volumeMounts:
            - name: nginx-config
              mountPath: /etc/nginx/nginx.conf
              subPath: nginx.conf
          securityContext:
            runAsUser: 1000
            runAsGroup: 3000
            readOnlyRootFilesystem: true
        - name: log-sidecar
          image: busybox
          args: ["sh", "-c", "tail -f /var/log/nginx/access.log"]
          volumeMounts:
            - name: nginx-logs
              mountPath: /var/log/nginx
      volumes:
        - name: nginx-config
          configMap:
            name: nginx-config
        - name: nginx-logs
          emptyDir: {}