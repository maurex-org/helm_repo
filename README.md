# helm_repo
Helm repository

helm create homeassistant

Creating homeassistant


helm install --debug --dry-run=client homeassistant ./homeassistant

level=DEBUG msg="Original chart version" version=""
level=DEBUG msg="Chart path" path=/home/maurex/repos/workspace-vscode/helm_repo/homeassistant
level=DEBUG msg="number of dependencies in the chart" dependencies=0
NAME: homeassistant
LAST DEPLOYED: Sat Dec 27 08:59:49 2025
NAMESPACE: default
STATUS: pending-install
REVISION: 1
DESCRIPTION: Dry run complete
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
image:
  pullPolicy: IfNotPresent
  repository: homeassistant/home-assistant
  tag: "2025.10"
ingress:
  annotations:
    kubernetes.io/ingress.class: nginx
  className: nginx
  enabled: true
  hosts:
  - host: homeassistant.local
    paths:
    - path: /
      pathType: ImplementationSpecific
persistence:
  hostPath: /home/maurex/homeassistant
  size: 5Gi
replicaCount: 1
service:
  port: 8123
  type: ClusterIP
usbDevice:
  hostPath: /dev/ttyUSB0

HOOKS:
---
# Source: homeassistant/templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "homeassistant-test-connection"
  labels:
    helm.sh/chart: homeassistant-0.1.0
    app.kubernetes.io/name: homeassistant
    app.kubernetes.io/instance: homeassistant
    app.kubernetes.io/version: "2025.10"
    app.kubernetes.io/managed-by: Helm
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['homeassistant:8123']
  restartPolicy: Never
MANIFEST:
---
# Source: homeassistant/templates/pvc.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: homeassistant-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /home/maurex/homeassistant
---
# Source: homeassistant/templates/pvc.yamlrepo
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: homeassistant-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  volumeName: homeassistant-pv
---
# Source: homeassistant/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: homeassistant
spec:
  type: ClusterIP
  ports:
    - port: 8123
      targetPort: 8123
      protocol: TCP
      name: http
  selector:
    app: homeassistant
---
# Source: homeassistant/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: homeassistant
spec:
  replicas: 1
  selector:
    matchLabels:
      app: homeassistant
  template:
    metadata:
      labels:
        app: homeassistant
    spec:
      hostNetwork: true  # Equivale a network_mode: host
      containers:
        - name: homeassistant
          image: "homeassistant/home-assistant:2025.10"
          securityContext:
            privileged: true # Equivale a privileged: true
          volumeMounts:
            - name: config
              mountPath: /config
            - name: localtime
              mountPath: /etc/localtime
              readOnly: true
            - name: usb
              mountPath: /dev/ttyUSB0
          # Mapeo de dispositivos
          resources: {}
      volumes:
        - name: config
          persistentVolumeClaim:
            claimName: homeassistant-pvc
        - name: localtime
          hostPath:
            path: /etc/localtime
            type: File
        - name: usb
          hostPath:
            path: /dev/ttyUSB0
---
# Source: homeassistant/templates/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: homeassistant
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  ingressClassName: nginx
  rules:
    - host: "homeassistant.local"
      http:
        paths:
          - path: /
            pathType: ImplementationSpecific
            backend:
              service:
                name: homeassistant
                port:
                  number: 8123


helm package homeassistant                                    
Successfully packaged chart and saved it to: /home/maurex/repos/workspace-vscode/helm_repo/homeassistant-0.1.0.tgz
 
mv homeassistant-0.1.0.tgz charts  

helm repo index . --url https://maurex-org.github.io/helm_repo

apiVersion: v1
entries:
  homeassistant:
  - apiVersion: v2
    appVersion: "2025.10"
    created: "2025-12-27T09:02:46.108918051-03:00"
    description: Un chart de Helm para desplegar Home Assistant con acceso a hardware
    digest: cc35d88b1d4223abb85b89797f8deedfc2259048763f2f3b27c354df686b77e6
    name: homeassistant
    type: application
    urls:
    - https://maurex-org.github.io/helm_repo/charts/homeassistant-0.1.0.tgz
    version: 0.1.0
  mosquitto:
  - apiVersion: v2
    appVersion: 2.0.22
    created: "2025-12-27T09:02:46.110986987-03:00"
    description: MQTT Broker Helm Chart
    digest: 5a763c15daf9448eba027b006031712895cced8522bd88a364a03eaebecc5ce0
    name: mosquitto
    type: application
    urls:
    - https://maurex-org.github.io/helm_repo/charts/mosquitto-0.1.0.tgz
    version: 0.1.0
  nginx:
  - apiVersion: v2
    appVersion: 1.29.4
    created: "2025-12-27T09:02:46.112861215-03:00"
    description: A Helm chart for Kubernetes
    digest: 9ed81d993a2c4b8adc0d5df49072cc3384632073616482c9a20b1189763292ba
    name: nginx
    type: application
    urls:
    - https://maurex-org.github.io/helm_repo/charts/nginx-0.1.0.tgz
    version: 0.1.0
generated: "2025-12-27T09:02:46.107463926-03:00"

git add . 
git commit -m "add homeassistant"
git push
helm repo update